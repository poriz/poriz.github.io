---
layout: post
title: Spark - file_format
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---

# Spark - file_format
데이터는 디스크에 파일로 저장됨 → 최적화가 필요하다.

### Parquet

spark의 기본 파일 포맷이다.  컬럼기반의 저장포맷이다.

- 장점
    - 컬럼 단위의 구성으로 데이터가 균일하여 압축률이 좋다.
    - 데이터 읽기 시 일부 컬럼 선택이 가능해서 I/O가 줄어든다.
- 특징
    - 모든 컬럼이 자동 Null을 허용한다.
    - 스키마 정보가 포함되어있다.
- Parquet vs Avro
: Parquet은 열 기반의 저장, Avro는 행 기반 저장 데이터 포맷이다.

### 파일 읽고 저장해보기

- Spark에서 format을 이용해 저장해보기

```python
df.write\
	# "avro","parquet","json"등을 포맷에 지정하여 사용한다.
	.format(...)\
```

- 읽기

```python
df = spark.read. \
		parquet("file_path")
```

- 다양한 포맷을 한번에 읽어들이기

```python
df = spark.read. \
		option("mergeSchema",True). \
		parquet("*.parquet")
```

### Execution Plan

- lazy execution: Action이 발생하기 전까지는 처리되지 않는다.
    - 이것으로 Spark는 간단한 Operation들에 의한 성능적인 이슈를 고려하지 않아도 된다고 한다.
    - 많은 오퍼레이션들을 볼 수 있어서 최적화가 잘된다.
- Transformations
    - Narrow Dependencies: 독립적인 Partition level의 작업. Select, filter, map 등등 파티션 내에서만 작업이 수행된다.
    - Wide Dependencies: Shuffling이 필요한 작업. groupby, reduced by, partition by, repartition, coalesce 등등이 있다.
- Actions
    - Read, Write, Show, Collect: Job을 실행시키는 명령들이다.
- Job, Stages, Tasks
    - Job: **Spark Actions에 대한 응답으로 생성**되는 연산, 하나 이상의 Stage로 구성된다.
        - Action이 없는 경우에는 Job이 생성되지 않는다!
    - Stage: DAG의 형태로 구성된 Task들이 존재한다. (의존), Task들의 병렬 실행이 가능하다.
        - Stage는 Shuffling이 발생하는 경우에 생긴다고 생각!
    - Task: 가장 작은 실행 유닛으로 Executor에 의해 실행된다.
- Join 시 한쪽의 데이터가 작은 경우에 inner join을 하게되면 오버헤드가 발생한다.
    
    → Broadcast Join을 이용해서 작은 데이터프레임을 뿌려주는 형태로 조인해도 좋다.
    
    - `spark.sql.adaptive.autoBroadcastJoinThreshold`

### Bucketing과 Partitioning

: 입력 데이터를 최적화 한다면 처리시간을 단축하고 리소스를 절약 가능하다.

- 두 방법 모두 Hive 메타스토어의 사용이 필요하다. (saveAsTable)
- Bucketing: shuffling을 줄이는 것이 주 목적
    - Aggregation이나 Join이 많이 사용된다면 해당 컬럼들을 기준으로 테이블을 저장한다.
    - 이 때의 버킷의 수도 지정한다.
    - 데이터의 특성을 잘 알고 있는 경우에 사용 가능하다.
- File System Partitioning
    - 데이터들을 특정 컬럼(Partition Key)들을 기준으로 폴더 구조를 만들어 최적화
    - 예를 들어 로그 파일을 연도-월-일 단위로 폴더를 생성하여 저장 (읽기가 쉬워진다.)
    - 그러나 Partition key를 잘못 선택하면 너무 많은 파일들이 생성될 수 있다.


---
### 이전 포스트
- [Spark - SparkSQL, Hive-metaStore](https://poriz.github.io/dataengineering/camp/2024-01-18-dataengineering-camp-Spark_4/)

### 다음 포스트
- [kafka - 실시간 데이터 개요](https://poriz.github.io/dataengineering/camp/2024-01-22-dataengineering-camp-kafka_1/)



