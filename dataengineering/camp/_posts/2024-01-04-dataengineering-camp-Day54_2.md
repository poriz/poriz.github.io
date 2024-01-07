---
layout: post
title: 데브코스 54일차(2) - DBT (Data Build Tool) - 1
comments: true
categorys:
  - dataengineering
  - camp
---

# 데브코스 54일차(2) - DBT (Data Build Tool) - 1
- 데이터 품질의 중요성 증대
    - 입출력 체크, 다양한 품질검사, 리니지 체크, 데이터 히스토리파악
    - 데이터 품질을 유지하는것은 비용,노력의 감소와 생산성 증대의 지름길이다.

## DB Normalization
: 데이터베이스를 좀 더 조직적이고, 일관된 방법으로 디자인하는 방법 (수정,적재,삭제의 용이)

- 1정규화
    - 한 셀에는 하나의 값만 있어야한다.
    - PK가 있어야한다.
    - 중복된 키나 레코드가 없다.
- 2정규화
    - 1 정규화가 성립해야한다.
    - pk를 중심으로 의존결과를 알 수 있어야한다.
- 3정규화
    - 2 정규화가 성립
    - 모든 key가 아닌 컬럼은 key에 완전히 종속되어야한다. (부분 종속성이 없어야함)

참고: [정규화 정리 블로그](https://mr-dan.tistory.com/10)

## Slowly Changing Dimensions
- DW나 DL에서는 모든 테이블들의 히스토리를 유지하는 것이 중요하다.
    - 생성시간과 수정시간 두개의 timestamp필드를 갖는 것이 좋다.
    - 칼럼의 성격에따라서 유지 방법에 차이가 난다. (SCD Type으로 관리)
- SCD Type
    - 0: 한번 쓰고나면 바꿀 이유가 없는 경우들. → 고정필드들
    - 1: 데이터가 새로 생기는 경우 덮어쓰면 되는 컬럼들
    - 2: 특정 entity에 대한 데이터가 새로운 레코드로 추가되어야 하는 경우
        - ex. 고객 테이블에서 고객의 등급이 변화하는 경우
    - 3: 특정 entity가 새로운 컬럼을 추가되는 경우
    - 4: 특정 entity에 대한 데이터를 새로운 Dimension 테이블에 저장된다.
        - 별도 테이블에 저장하고, 일반화도 가능하다.

## DBT 설치하기(로컬)
Redshift에서의 사용을 위해서 dbt-redshift를 설치한다.

- `pip3 install dbt-redshift`
- 프로젝트 생성: `dbt init learn_dbt`
    - Redshift의 연결정보를 입력해준다.
- `~/.dbt/profiles.yml` : 연결정보들이 여기에 입력되어있다.
    - outputs 아래에 다수의 개발환경을 정의하고, 필요한 것들을 target에서 선택한다.
- `/dbt_project.yml` : dbt 메인 환경 설정 파일

```yml
name: 'learn_dbt'
version: '1.0.0'
config-version: 2

# ~/.dbt/profiles.yml 안에 존재해야한다.
profile: 'learn_dbt'

# 폴더의 이름들과 일치해야한다.
model-paths: ["models"]
analysis-paths: ["analyses"]
test-paths: ["tests"]
seed-paths: ["seeds"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]

# 결과들이 저장되는 폴더
target-path: "target"

# directories to be removed by `dbt clean`
clean-targets:         
  - "target"
  - "dbt_packages"

models:
  learn_dbt:
```

## DBT Input

### Model : 테이블 생성에 있어 기본이 되는 빌딩 블록
- 테이블이나, 뷰, CTE(임시 테이블)의 형태로 진행된다.
  - View: Select 결과를 기반으로 만들어진 가상 테이블
  - CTE: WITH 구문을 사용한 임시 테이블 아래와 같은 형태
    ```sql
    WITH cte AS (
        SELECT * FROM table
    )
    SELECT * FROM cte
    ```

- models 폴더 안에 생성된다.
- 진행 순서: raw → staging → core 
- 구성요소
  - Input
    - raw: CTE로 정의되는 경우가 많음.
    - staging,src: View로 정의된다.
  - Output
    - core: 테이블로 정의된다.
- models 내의 sql파일로 존재하며, jinja template과 함께 사용 가능하다.

## DBT Output

### Materialization
- 4가지의 Materialization이 존재한다.
  - table: 테이블로 저장
  - view: 뷰로 저장
  - incremental: 테이블로 저장하며, 새로운 데이터만 추가
  - ephemeral: CTE로 저장
- 예제

```sql
{% raw %}
{{
	config(
		materialized='Incremental',
		on_schema_change='fail'
	)
}}
{% endraw %}
```
- incremental_strategy : incremental update를 하는 경우에 사용가능하다.
  - append
  - merge
  - insert_overwrite: 이 경우에는 unique_key와 merge_update_columns필드를 사용하기도 한다.
  - [DBT_Incremental_docs](https://iomete.com/resources/docs/guides/dbt/dbt-incremental-models-by-examples)

- incremental Table의 config
    - on_schema_change : 스키마가 바뀐 경우 대응방법을 지정가능하다.
        - append_new_columns
        - ignore
        - sync_all_columns
        - fail
- incremental Table 주의: 조건을 걸어서 src할때마다 전체 테이블을 불러오는것이아니라, 새로생긴 테이블만 가져오도록 해야한다. (datestamp등을 사용하면된다!)

```sql
{% raw %} {% if if_incremental() %} {% endraw %}
	{% raw %} AND datestamp > (SELECT max(datestamp) from {{ this }}) {% endraw %}
{% raw %} {% endif %} {% endraw %}
```

### dbt 명령어
- dbt compile : SQL 코드 까지만 생성하고 실행하지는 않는다.
- dbt run : 생성된 코드를 실제 실행한다.

---
### 이전 포스트
- [데브코스 54일차(2) - DBT (Data Build Tool)](https://poriz.github.io/dataengineering/camp/2024-01-04-dataengineering-camp-Day54_2/)
