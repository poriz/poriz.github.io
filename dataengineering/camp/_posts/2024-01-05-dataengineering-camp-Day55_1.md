---
layout: post
title: 데브코스 55일차(1) - DBT (Data Build Tool) - 2
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---

# 데브코스 55일차(1) - DBT (Data Build Tool) - 2
## dbt Seeds

: 작은 dimension테이블들을 csv파일로 생성 후 데이터 웨어하우스로 로드하는 방법

### 실습

1. seeds 폴더를 생성해서 그 아래에 .csv 파일을 저장/생성한다.
    - csv 파일에 헤더가 있어야한다.
2. `dbt seed` 명령을 실행한다.
3. 자신의 스키마 아래에 csv파일명으로 테이블이 생성됨을 볼 수 있다.

---

## dbt Sources

: Staging 테이블 생성 시 입력 테이블들이 자주 변경되는 경우 models아래의 .sql파일들을 일일이 찾아 바꿔주어야한다. → 이것을 해결하기 위해 사용한 것이 Sources

- 기본적으로 처음 입력이 되는 ETL 테이블을 대상으로 한다.
- 테이블 이름들에 별명을 준다. (alias)
    - 별명은 source 이름과 새 테이블 이름 두가지로 구성된다.
- Source 테이블들에 새 레코드가 있는지 체크해주는 기능도 제공한다.

### 실습

1. `models/sources.yml` 파일 생성하기
2. models 아래의 `.sql` 파일들을 적절하게 수정해야한다.
    - `FROM Schema.table` ⇒ {% raw %}`FROM {{ source ("schema","table") }}` {% endraw %}
    - 보통 src에서 다음 형태를 많이 사용하기 때문에 src위주로 변경한다.
- Sources 최신성 체크하기. (Freshness)
    - `dbt source freshness` 명령으로 수행한다.
- `sources.yml` 코드



```yaml
version: 2

sources:
	- name: user_id
		schema: schema_name
		tables:
			- name: alias
				identifier: table_name
				# 아래는 freshness를 결정해주는 필드들
				loaded_at_field: datestamp
				freshness:
					warn_after: { count: 1,period: hour }
					error_after: { count: 24,period: hour }
```

---

## dbt Snapshots

: Dimension테이블은 변경이 자주 생길 수 있기 떄문에 테이블의 변화를 기록함으로써 관리한다.

### 처리 방법

- snapshots 폴더에 환경설정하기
- snapshot을 하려면 pk가 존재해야하고, 레코드의 변경시간을 나타내는 타임스탬프가 필요하다.
- 변경 감지기준 설정: pk를 기준으로 변경 시간이 현재 DW에 있는 시간보다 미래인 경우
- snapshot 테이블에는 4개의 타임스탬프가 존재한다.
    - dbt_scd_id, dbt_updated_at, valid_from, valid_to

### 적용

- snapshots 폴더에 적용할 파일을 편집한다.

```yaml
{% raw %}{% snapshot scd_user_metadata %}{% endraw %}
{% raw %}{{
config(
	target_schema='source_name',
	unique_key='user_id',
	strategy='timestamp',
	updated_at='updated_at',
	invalidate_hard_deletes=True
)
}}{% endraw %}
SELECT * FROM {% raw %} {{ source('source_name', 'metadata') }} {% endraw %}
{% raw %}{% endsnapshot %}{% endraw %}
```


- `dbt snapshot` 명령으로 실행

### 결과

- 기존 테이블

| EMPLOYEE_ID | JOB_CODE |
| --- | --- |
| E001 | J01 |
| E002 | J02 |

- 변경 → J02가 변경됨.

| EMPLOYEE_ID | JOB_CODE | DBT_VALID_FROM | DBT_VALID_TO |
| --- | --- | --- | --- |
| E001 | J01 | 2020-01-01 | 2024-01-01 |
| E002 | J02 | 2023-01-02 | NULL |

: 이 처럼 변경값들이 추가되며, dbt_scd_id, dbt_updated_at등의 필드도 추가된다.

- 새로 생긴 컬럼에는 DBT_VALID_FROM에 값이 들어간다.

---

## dbt TEST

: 데이터의 품질을 테스트하기 위해 사용한다.

- Generic : 내장 일반 테스트
    - unique, not_null, accepted_values, relationships 등의 테스트를 지원한다.
    - models 폴더 내에 존재한다.
- Singular : 커스텀 테스트
    - 기본적으로 SELECT로 간단하게 작성한다.
    - 결과가 리턴되는 경우 “실패”로 간주한다.
    - tests 폴더 내에 존재한다.

### Generic Test 구현

- `models/schema.yml` 파일 생성

```yaml
version: 2
models:
- name: dim_user_metadata
columns:
- name: user_id
tests:
- unique
- not_null
```

### Singular Test 구현

- `tests/test_table_name.sql` 생성
- pk 테스트 : 테이블의 pk가 unique한지 확인해보는 코드

```yaml
SELECT
*
FROM (
SELECT
user_id, COUNT(1) cnt
FROM
	{% raw %} {{ ref("table_name") }} {% endraw %}
	GROUP BY 1
	ORDER BY 2 DESC
	LIMIT 1
)
WHERE cnt > 1
```


### Test 실행

- `dbt test` 명령으로 실행하기
- 특정 테이블 대상의 테스트만 진행하고 싶은 경우
    - `dbt test --select table_name` 의 형태로 진행한다.
- 디버깅 : `dbt --debug test --select table_name`

---

## dbt Documentation

- 문서와 소스 코드를 최대한 가깝게 배치하는 것이 기본 철학
- 문서화에는 두 가지 방법이있다.
    - 기존 `.yml` 파일에 문서화를 추가하는 방법
    - 독립적인 markdown파일을 생성하는 방법
- `dbt docs serve` 를 이용해서 경량 웹서버로 띄울 수 있다.

### models 문서화하기

- description 키를 추가한다. `models/schema.yml, models/sources.yml`

```yaml
version: 2
models:
- name: dim_user_metadata
description: A dimension table with user metadata
columns:
- name: user_id
description: The primary key of the table
tests:
- unique
- not_null
```

- `dbt docs generate` 를 수행한다.
    - 결과 파일은 `target/catalog.json` 으로 저장된다.
- `dbt docs serve` 로 웹서버 실행
    - Lineage Graph 확인도 가능하다.

---

## dbt Expectations

: Greate Expectations에서 영감을 받아 dbt용으로 만든 확장판이다.

### 설치

- git 경로: [https://github.com/calogica/dbt-expectations](https://github.com/calogica/dbt-expectations)
- `packages.yml`에 등록하기
    
    ```yaml
    packages:
    	-package: calogica/dbt_expectations
    	 version: [">=0.7.0", "<0.8.0"]
    ```
    

### Docs

: dbt-expectations 관련 함수들이 많아 정리가 잘되어있는 docs 링크를 첨부한다.

- [dbt-expectations](https://hub.getdbt.com/calogica/dbt_expectations/latest/)


---
### 이전 포스트
- [데브코스 54일차(2) - DBT (Data Build Tool)](https://poriz.github.io/dataengineering/camp/2024-01-04-dataengineering-camp-Day54_2/)
### 다음 포스트
- [데브코스 55일차(2) - 데이터 카탈로그](https://poriz.github.io/dataengineering/camp/2024-01-05-dataengineering-camp-Day55_2/)
