---
layout: post
title: Spark
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---

# Spark

## Spark 특징

### Spark vs MapReduce

- Spark는 메모리 기반, MapReduce는 디스크 기반이다.
- MapReduce는 하둡위에서만 동작하나, Spark는 K8s등의 다른 분산 컴퓨팅 환경을 지원한다.
- MapReduce는 키,밸류 기반의 데이터 구조만 지원, Spakr는 판다스 데이터프레임과 개념적으로 동일한 데이터 구조를 지원한다.
- Spark는 다양한 방식의 컴퓨팅을 지원한다.
    - 배치 데이터 처리, 스트림 처리, SQL, 머신러닝, 그래프 분석등

### Spark 프로그래밍 API

- RDD
    - low level 프로그래밍 API로 세밀한 제어가 가능하나, 코딩의 복잡도가 증가한다.
- DataFrame & Dataset
    - 판다스의 데이터프레임과 흡사하다.
    - 구조화 데이터 조작인 경우 보통 Spark SQL을 사용한다.
    - SQL만으로 할 수 없는 일의 경우에 사용된다.
- Spark SQL
    - 구조화된 데이터를 SQL로 처리한다.
    - 데이터 프레임을 SQL로 처리 가능하다.
    - Hive 쿼리보다 최대 100배까지 빠른 성능을 보장한다.
        - 최근에는 메모리와 디스크 모두 사용하는 추세이다.
- Spark ML
    - 머신러닝 관련 다양한 알고리즘과 유틸리티로 구성된 라이브러리
    - 딥러닝 지원은 아직 미약하다.
    - spark.mllib와 spark.ml
        - mllib의 경우 RDD위에서 동작하며 더이상 업데이트가 안된다.
        - spark.ml: 데이터 프레임 기반
    - 장점
        - 원스톱 프레임워크이다: 전처리, 모델 빌딩, 모델 빌딩 자동화, 관리 및 서빙
        - 대용량 데이터도 처리가 가능하다.
- Spark데이터 시스템 사용 예시
    - 대용량 데이터의 배치, 스트림 처리 및 모델 빌딩
        - 대용량의 비구조화된 데이터 처리
        - ML 모델에 사용되는는 대용량 피쳐 처리
        - Spark ML을 사용한 대용량 훈련 데이터 모델 학습

### Spark 실행 환경 및 프로그램 구조

- 실행 환경
    - 개발/테스트/학습 환경
        - 주피터나 spark shell 사용
    - 프로덕션 환경
        - spark-submit(가장 많이 사용된다.), 데이터브릭스 노트북, REST API
- 프로그램 구조
    - Driver: 실행되는 코드의 마스터 역할을 수행한다. (YARN의 AM)
        - 사용자 코드를 실행하며, 실행 모드에 따라서 실행되는 곳이 달라진다.(client, cluster)
        - 코드를 실행하는데 필요한 리소스들을 지정한다.
        - SparkContext를 만들어서 Spark 클러스터와 통신을 수용한다.
            - Cluster Manager(RM)
            - Executor(Container)
        - 사용자 코드를 실제 Spark task로 변경해서 spark cluster에서 실행한다.
    - Executor:
        - 실제 태스크를 실행해주는 역할을 수행한다.(YARN의 컨테이너)
- Spark 클러스터 매니저 옵션
    - local[n]
        - 개발, 테스트용으로 사용된다.
        - n을 통해 Executor의 스레드 수를 조절 가능하다.
    - YARN
        - Client, Cluster의 두 실행모드가 존재한다.
        - Client: Driver가 Spark cluster 밖에서 동작한다. → Spark cluster를 바탕으로 개발,테스트용으로 사용한다.
        - Cluster: Driver가 Spark cluster 안에서 동작한다. → 실제 프로덕션에서 사용
    - Kubernetes, Mesos, Standalone


## Spark 프로그래밍

### Spark 데이터 처리

- 병렬 처리
    - 우선 데이터가 분산되어야 한다.
    - 데이터 블록 & 파티션
        - 하둡의 데이터 블록은 128MB이고 Spark 에서는 이를 파티션이라 부른다.
    - 나누어진 데이터를 각각 따로 동시에 처리한다.
        - Spark에서는 파티션 단위로 메모리로 로드되어 Executor가 배정된다.
- 전체 과정
    1. 데이터를 나누고
    2. 파티션으로 만들기 (파티셔닝)
        1. 적절한 파티션의 수는 Executor의 수 x Executor의 CPU 수가 된다.
    3. 병렬 처리
- Spark 데이터 처리 흐름
    - 데이터 프레임은 작은 파티션들로 구성된다. 또한 데이터 프레임은 수정 불가!
    - pandas 데이터프레임과 비슷하나, size가 더 크다.
    - 입력 데이터프레임을 원하는 결과 도출까지 다른 데이터 프레임으로 계속 변환한다.
- Shuffling: 파티션간에 데이터의 이동이 필요한 경우에 발생한다.
    - 발생하는 경우: 명시적 파티션을 새롭게 하는경우, 시스템에 이루어지는 셔플링(group,sort)
    - 셔플링 발생 시 네트워크를 타고 데이터가 이동한다.
        - 오퍼레이션에 따라 파티션 수가 결정되나, 최대 200개이다.
    - Data Skew가 발생하기 때문에 셔플링을 최소화하는 것이 중요하며, 파티션 최적화가 필요하다.

### Spark 데이터 구조: RDD, DataFrame, Dataset

- RDD(Resilient Distributed Dataset)
    - 클러스터내의 서버에 분산된 데이터를 지칭한다. 다수의 파티션으로 구성된다.
    - 레코드별로 존재하나, 스키마가 존재하지 않는다.
    - 일반 파이썬 데이터를 parallelize 함수로 RDD로 변환한다. (반대는 collect)
- DataFrame & Dataset
    - 필드 정보를 갖고 있다.(테이블), RDB 테이블 처럼 컬럼으로 나눠 저장된다.
    - Dataset은 타입 정보가 존재하며, 컴파일 언어에서 사용 가능하다(scala,java)
    - PySpark에서는 DataFrame을 사용한다.
- 두 데이터 구조 모두 변경이 불가능한 분산 저장된 데이터이다.

---

### Spark 프로그램 구조

- Spark Session 생성: spark 프로그램의 시작은 sparksession을 만드는 것이다.
    - 프로그램마다 하나를 만들어 spark cluster와 통신한다.
    - spark session을 통해서 spark가 제공하는 다양한 기능을 사용한다.
    - [Spark Session docs](https://spark.apache.org/docs/3.1.1/api/python/reference/api/pyspark.sql.SparkSession.html)

> ✅ 싱글턴 패턴이란?
소프트웨어의 디자인 패턴중 하나이다. 생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고, 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴하는 유형. 공통된 객체를 여러개 생성해서 사용하는 상황에서 많이 사용된다.
- 참고로 파이썬의 모듈은 그 자체로 싱글턴이다.
- [싱글턴_위키](https://ko.wikipedia.org/wiki/%EC%8B%B1%EA%B8%80%ED%84%B4_%ED%8C%A8%ED%84%B4)
> 

```python
# SparkSession 예시
from pyspark.sql import SparkSession

# SparkSession은 싱글턴이다.
# pyspark.sql을 받은 것 처럼 spark sql engine이 중심으로 동작한다.

spark = SparkSession.builder\
		.master("local[*]")\
		.appName('Hello')\
		.getOrCreate()
...
# 이 함수를 여러번 호출하더라도, sparksession이 여러개 생성되는 것이 아니다.
spark.stop()

```

- Spark Session 환경 변수
    - session 생성 시 다양한 환경 설정이 가능하다: [docs_link](https://spark.apache.org/docs/latest/configuration.html#application-properties)
    - 환경 설정 방법 (위에서부터 우선순위)
        - sparksession을 만들때 지정하기 - SparkConf,
        - spark-submit 명령의 커맨드라인 파라미터
        - spark_defaults.conf 사용
        - 환경 변수
    - spark session 환경 설정
        - `.config("option")` 이런 식으로 적용
        - SparkConf 생성하여 일괄 설정
        
        ```python
        from pyspark import SparkConf
        conf = SparkConf()
        conf.set("spark.app.name","TestName")
        conf.set(...)
        ```
        
- 전체 플로우: Spark 세션 만들기 > 입력 데이터 로딩 > 데이터 조작 > 결과 저장
- SparkSession이 지원하는 데이터 소스
    - `spark.read(DataFrameReader)`를 사용해서 데이터 프레임으로 로드
    - `DataFrame.write(DataFrameWriter)`를 사용해서 데이터 프레임을 저장
    - HDFS파일, Hive 테이블, JDBC 관계형 데이터베이스, 클라우드 기반 데이터 시스템, 스트리밍 시스템들이 많이 사용되는 데이터 소스들이다.

---
### 이전 포스트
- [빅데이터 처리 시스템, Hadoop](https://poriz.github.io/dataengineering/camp/2024-01-17-dataengineering-camp-Spark_1/)



