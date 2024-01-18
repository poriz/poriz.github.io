---
layout: post
title: Spark - 실습 정리
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---

# Spark - 실습 정리
실습의 내용을 모두 정리하지는 않고 필요한 부분만 골라 정리하였습니다.

### Session 및 config 설정하기.

- SparkSession 구조

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder\
				.config(conf=conf)\
				.getOrCreate()
```

- config 정의하기

```python
from pyspark import SparkConf

conf = SparkConf()
conf.set("spark.app.name","PySpark DataFrame")
conf.set("spark.master",'local[*]')
```

### 파일 읽고 쓰기.

- 읽기

```python
# spark.read를 통해서 csv 파일을 읽어오기. 두 방법이 있다.
# df = spark.read.csv("1800.csv")
df = spark.read.format("csv").load("1800.csv")
```

- 쓰기

```python
# csv 저장
df.write.csv("df.csv")
# json 저장
df.write.format("json").save("extracted.json")
```

### 데이터 프레임 조작하기

- Schema 구조체 생성하기

```python
from pyspark.sql.types import StringType, IntegerType, FloatType
from pyspark.sql.types import StructType, StructField

schema = StructType([\
                     StructField("stationID", StringType(), True), \
                     StructField("date", IntegerType(), True), \
                     StructField("measure_type", StringType(), True), \
                     StructField("temperature", FloatType(), True)])

# read 시에 구조화된 schema를 불러와 적용 가능하다.
df = spark.read.schema(schema).csv("1800.csv")
```

- 필터 적용

```python
# 두가지 방법이 있다. 
# 1. column expression으로 필터링 적용
minTemps = df.filter(df.measure_type=="TMIN")
# 2. SQL expression으로 필터링 적용 -> 따옴표 사용에 주의하자!
# minTemps = df.where("df.measure_type='TMIN'")
```

- 집계

```python
# groupBy
minTempsByStation = minTemps.groupBy("stationID").min("temperature")
minTempsByStation.show()

# select
# 1. pandas 방식: stationTemps = minTemps[["stationID","temperature"]]
# 2. sql 방식
stationTemps = minTemps.select("stationID","temperature")
stationTemps.show(5)
```

- column_name 변경하기

```python
# 1. withColumnRenamed 사용
df_ca = df.groupBy(...).withColumnRenamed("sum(amount_spent)","sum")
# 2. alias 사용
import pyspark.sql.functions as f
df_ca = df.groupBy(...)\
        .agg(f.sum("amount_spent").alias("sum"))
# 2-a. alias를 이용해서 여러 컬럼에 한번에 적용하기.
df_groups = df.groupBy("cust_id")\
  .agg(
      f.sum('amount_spent').alias('sum'),
      f.max('amount_spent').alias('max'),
      f.avg('amount_spent').alias('avg')
  )
```

### Redshift와의 연결 구성하기

- redshift-driver 다운로드하기.
: pyspark의 경로로 이동해서 aws의 driver를 다운로드 받는 코드이다. 버전이 변경 될 수 있으니 파이썬과 드라이버의 버전에 주의해야한다.

참고) https://github.com/aws/amazon-redshift-jdbc-driver

```python
!cd /usr/local/lib/python3.10/dist-packages/pyspark/jars && wget https://redshift-downloads.s3.amazonaws.com/drivers/jdbc/2.1.0.24/redshift-jdbc42-2.1.0.24.jar
```

- 연결 구성

```python
# option을 통해서 driver를 설정하고 url에 연결, 테이블에 접근한다.
df_user_session_channel = spark.read \
    .format("jdbc") \
    .option("driver", "com.amazon.redshift.jdbc42.Driver") \
    .option("url", "redshift-connections") \
    .option("dbtable", "raw_data.user_session_channel") \
    .load()
```

### PySpark 함수 정리
뒤에서 pyspark의 함수들을 정리할 기회가 있을지 몰라서 참고 링크를 남겨두려한다.
- [PySpark 함수 정리 블로그](https://assaeunji.github.io/python/2022-03-26-pyspark/)

- [PySpark 다양한 데이터 타입 다루기](https://wook-lab.tistory.com/19)

---
### 이전 포스트
- [spark](https://poriz.github.io/dataengineering/camp/2024-01-17-dataengineering-camp-Spark_2/)

### 다음 포스트
- [Spark - SparkSQL, Hive-metaStore](https://poriz.github.io/dataengineering/camp/2024-01-18-dataengineering-camp-Spark_4/)


