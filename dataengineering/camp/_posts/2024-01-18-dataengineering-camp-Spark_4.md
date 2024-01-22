---
layout: post
title: Spark - SparkSQL, Hive-metaStore
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---

# Spark - SparkSQL, Hive-metaStore

## Spark SQL

: 구조화된 데이터 처리를 위한 Spark 모듈. 데이터 프레임 작업을 SQL로 처리 가능하다.

### Spark SQL vs DataFrame

- Familiarity, Readability: SQL이 가독성이 더 좋고 많은 사람들이 사용 가능하다.
- Optimization: Spark SQL 엔진이 최적화하기에 더 좋다.
- Interoperability, Data Management: SQL이 포팅이 더 쉽고, 접근 권한 체크도 간단하다.

### SQL 사용법

- 데이터 프레임을 기반으로 테이블 뷰 생성
    - createOrReplaceTempView: spark session이 살아있는 동안 존재한다.
    - createOrReplaceGlobalTempView: spark 드라이버가 살아있는 동안 존재.
- Spark session의 read함수를 호출해 외부 데이터베이스 연결
    - 결과는 데이터 프레임으로 리턴된다.

### Aggregation, Join, UDF

### Aggregation

- Group By
- Window
- Rank

### JOIN

- SQL JOIN은 두개 혹은 그 이상의 테이블들을 공통 필드를 가지고 머지.
- INNER JOIN: 매치가 되는 레코드들만 리턴한다. 모두 채워진 상태로 리턴

```sql
SELECT * FROM Vital v
JOIN Alert a ON v.vitalID = a.vitalID;
```

- LEFT JOIN: 왼쪽 테이블이 Base, 오른쪽 테이블의 필드는 왼쪽과 매칭되는 경우에만 리턴

```sql
SELECT * FROM raw_data.Vital v
LEFT JOIN raw_data.Alert a ON v.vitalID = a.vitalID;
```

- FULL JOIN: 모든 테이블들의 레코드를 리턴한다. 매칭되는 경우에만 양쪽 테이블들의 모든 필드들이 채워진 상태로 리턴된다.

```sql
SELECT * FROM raw_data.Vital v
FULL JOIN raw_data.Alert a ON v.vitalID = a.vitalID;
```

- CROSS JOIN: 왼쪽 테이블과 오른쪽 테이블의 모든 레코드들의 조합을 리턴함

```sql
SELECT * FROM raw_data.Vital v CROSS JOIN raw_data.Alert a;
```

- SELF JOIN: 동일한 테이블을 alias를 달리해서 자기 자신과 조인한다.

```sql
SELECT * FROM raw_data.Vital v1
JOIN raw_data.Vital v2 ON v1.vitalID = v2.vitalID;
```

- 최적화 관점에서 본 조인의 종류들
    - shuffle join: 일반 조인 방식이다.
        - Bucket join: 조인 키를 바탕으로 새로 파티션을 새로 만들고 조인하는 방식.
    - Broadcast join: 큰 데이터와 작은 데이터 간의 조인.
        - 작은 데이터 프레임을 다른 데이터 프레임이있는 서버로 뿌리는 방식.
        - 데이터 프레임 하나가 충분히 작아야한다.

### UDF(User Defined Function)

- DataFrame이나 SQL에서 적용가능한 사용자 정의 함수이다.
- 데이터프레임의 경우 .withColumn 함수와 같이 사용하는 것이 일반적이다.
- Scalar 함수와 Aggregation 함수가 존재한다.
    - Scalar: UPPER, LOWER…
    - Aggregation: SUM, MIN, MAX ⇒ UDAF(User Defined Aggregation Function)
- 성능적인 면에서 Scala나 Java로 구현하는것이 제일 좋다.
    - 파이썬 사용시 Pandas UDF로 구현한다.
- UDF - DataFrame 사용해보기 (lambda와 SQL 두 버전) - Upper

```python
# lambda 사용해서 적용
import pyspark.sql.functions as F
from pyspark.sql.types import *
# 주어진 값을 대문자로 변경하는 UDF 생성
# 이거 Apply와 비슷하다.
upperUDF = F.udf(lambda z:z.upper())
df.withColumn("Curated Name", upperUDF("Name"))

# spark SQL
def upper(s):
	return s.upper()

# pandas UDF
import pandas as pd
@pandas_udf(StringType())
def upper_udf2(s: pd.Series) -> pd.Series:
	return s.str.upper()

# test code
upperUDF = spark.udf.register("upper",upper)
spark.sql("SELECT upper('abcd')").show() # test 적용

# DataFrame기반 SQL에 적용하기
df.createOrReplaceTempView("test")
spark.sql("""SELECT name, upper(name) 'Curated Name' FROM test""").show
```

- UDF - DataFrame 사용해보기 (lambda와 SQL 두 버전) - add

```python
data = [
{"a": 1, "b": 2},
{"a": 5, "b": 5}
]
df = spark.createDataFrame(data)
# "c"는 새로 만들어진 컬럼의 이름이다.
df.withColumn("c",F.udf(lambda x,y: x+y)("a","b"))

# spark SQL
def plus(x,y):
	return x+y

# pandas UDF
import pandas as pd
@pandas_udf(FloatType())
def average(v: pd.Series) -> float:
	return v.mean()

plusUDF = spark.udf.register("plus",plus)
spark.sql("SELECT plus(1,2)").show()

df.createOrReplaceTempView("test")
spark.sql("SELECT a,b, plus(a,b) c FROM test").show()
```

- 생성한 테이블과 함수를 보려 하는 경우
    - 테이블: `spark.catalog.listTables()`
    - 함수: `spark.catalog.listFunctions()`
    

## Hive-메타스토어 사용하기

### Spark DB와 테이블

- 카탈로그: 테이블과 뷰에 관한 메타 데이터 관리
    - 메모리 기반의 카탈로그를 기본으로 제공한다.
    - Hive와 호환되는 카탈로그를 제공하는데 이는 spark 세션이 끝나도 유지된다.
- 데이터베이스라 부르는 폴더와 같은 구조로 관리한다.
- 스토리지 기반 테이블
    - HDFS와 Parquet 포맷을 사용한다.
    - Managed Table: Spark이 실제 데이터와 메타 데이터 모두 관리
    - Unmanaged Table: Spark은 메타 데이터만 관리한다. → 실제 데이터는 외부에 있음.

### 사용 방법

- SparkSession 생성시 enableHiveSupport()를 호출하여 사용한다.
- Managed Table 사용 방법
    - `dataframe.saveAsTable(”table_name”)`
    - SparkSQL에서 create table 등을 사용
    - spark.sql.warehouse.dir이 가리키는 위치에 데이터가 저장된다.
- External Table 사용방법
    - Location ‘hdfs_path’ 이렇게 위치를 지정해줘야한다.



---
### 이전 포스트
- [Spark - 실습 정리](https://poriz.github.io/dataengineering/camp/2024-01-17-dataengineering-camp-Spark_3/)

### 다음 포스트
- [Spark - file_format](https://poriz.github.io/dataengineering/camp/2024-01-19-dataengineering-camp-Spark_5/)
