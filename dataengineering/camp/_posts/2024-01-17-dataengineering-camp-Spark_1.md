---
layout: post
title: 빅데이터 처리 시스템, Hadoop
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---

# 빅데이터 처리 시스템, Hadoop

## 빅데이터의 정의와 예

### 빅데이터란.

: 서버 한대로 처리할 수 없는 규모의 데이터, 기존의 소프트웨어로는 처리가 불가능한 규모의 데이터.

- 4V
    - Volume: 크기가 대용량
    - Velocity: 처리 속도의 중요성
    - Variety: 구조화/비구조화 데이터 모두
    - Veracity: 데이터의 품질

### 빅데이터의 특징

- 빅 데이터의 문제들
    - 빅데이터를 손실없이 보관할 방법이 필요 → 분산 파일 시스템의 필요성
    - 처리 시간이 매우 오래걸림 → 분산 컴퓨팅 시스템이 필요
    - 비구조화된 데이터일 가능성이 높음. → 비구조화 데이터의 처리
- 해결 방안: 다수의 컴퓨터로된 프레임워크가 필요하다.
- 대용량 분산시스템
    - 분산 환경 기반: 다수의 서버로 구성이 되어야한다.
    - Fault Tolerance: 소수의 서버가 고장나도 동작해야한다.
    - 확장이 용이해야한다.(Scale Out)


---
## 하둡

### 하둡의 등장

- Doug Cutting이 구글랩 발표 논문들에 기반해 만든 오픈소스 프로젝트
- 첫 시작은 Nutch라는 오픈소스 검색엔진의 하부 프로젝트였다.

### 하둡이란?

- 다수 노드로 구성된 클러스터 시스템
    - 하나의 거대한 컴퓨터처럼 동작하나 다수의 컴퓨터들이 복잡한 소프트웨어로 통제되는 것.

### 하둡의 발전

- 1.0: HDFS위에 MapReduce라는 분산컴퓨팅 시스템이 돌아가는 구조.
- 2.0: YARN이라는 이름의 분산처리 시스템 위에서 동작하는 애플리케이션으로 변경됨.
- 3.0
    - YARN 프로그램들의 논리적인 그룹으로 나누어 자원 관리가 가능 → 다용도 사용 시 리소스의 적절한 배분이 가능
    - 타임라인 서버에서 HBase를 기본 스토리지로 사용
    - 파일 시스템: 네임노드의 경우 다수의 스탠바이 네임노드를 지원한다.

### HDFS - 분산 파일 시스템

- 데이터를 블록 단위로 나누어 저장한다. (default: 128MB)
- 블록 복제 방식
    - 각 블록은 3군데에 중복되어 저장된다. → Fault tolerance를 보장가능한 방식
- 네임노드 이중화 지원
    - Active & Standby
        - 둘 사이에 share efit log가 존재한다.

### MapReduce: 분산 컴퓨팅 시스템

- 하나의 잡 트래커와 다수의 태스크 트래커로 구성된다.
    - 잡 트래커가 일을 분배하나, 태스크 트래커에서 병렬로 처리한다.
- mapreduce만 지원한다.

## YARN의 동작 방식

### YARN의 구성

- 리소스 매니저
    - Job Scheduler, Application Manager
- 노드 매니저
- 컨테이너
    - 앱 마스터
    - 태스크

### YARN의 동작 방식

1. 실행 코드와 환경 정보를 Resource Manager에게 제출
    1. 실행에 필요한 파일들은 application ID에 해당하는 HDFS 폴더에 미리 복사된다.
2. Resource Manager는 Node Manager를 통해서이를 실행하기위한 Master를 생성한다. (Application Master)
    1. Application Master는 프로그램마다 하나씩 할당되는 프로그램 마스터이다.
3. Application Master는 Resource Manager를 통해 코드 실행에 필요한 리소스를 받아온다.
    1. Resource Manager는 적절하게 리소스ㅌ를 할당해줌
4. Application Master는 Node Manager를 통해서 컨테이너들을 받아 코드를 실행한다. (Task)
    1. 실행에 필요한 파일들은 HDFS에서 Container가 있는 서버로 먼저 복사된다.
5. task들은 주기적으로 자신의 상황을 Application Master에게 업데이트한다.(heartbeat)
- YARN Application마다 하나의 Application Master가 생긴다.

## 맵리듀스

### 맵리듀스 프로그래밍의 특징

- 데이터 셋은 key, value의 집합이며 변경이 불가능하다.
- 데이터 조작은 map과 reduce 오퍼레이션으로만 가능하다.
    - 두 오퍼레이션은 항상 하나의 쌍으로 연속으로 실행된다.
- Map의 결과를 Reduce로 모아준다. 이 때 네트워크를 이용한 데이터 이동이 생긴다. (셔플링)

### 맵과 리듀스

- Map: (k,v) → [(k’,v’)*]
    - 입력은 시스템에 의해서 주어지며, 입력으로 지정된 HDFS 파일에서 넘어온다.
    - key, value 페어를 새로운 key, value 페어 리스트로 변환한다.
    - 출력은 입력값일 수 있으나, 값이 없어도 된다.
    - 예시) input: (100,”I Am Groot”) → output: [(”I”,1),(”AM”,1)(”Groot”,1)]
- Reduce: (k’, [v1,v2,v3,…]) → (k’’,v’’)
    - 입력은 시스템에 의해서 주어진다.
        - 맵의 출력 중 같은 키를 갖는 key/value를 시스템이 묶어 입력으로 넣어준다.
        - SQL의 Group by와 비슷하다.
        - 출력은 HDFS에 저장된다.
    - 하나의 리듀스에 너무 많은 데이터가 몰리는 경향이 존재한다.(주의)
    - 예시) input: (”Groot”: [1,1,1]) → output: (”Grout”: 3)
- 문제점
    - 데이터 모델과 오퍼레이션에 제약이 많아 생산성이 떨어진다.
    - 모든 입출력이 디스크를 통해서 이루어진다. → 배치 프로세싱에 적합한다.
    - Shuffling 이후에 Data Skew가 발생하기 쉽다.
        - Reduce 태스크 수를 개발자가 정해야하는 문제가 있다.

### Shuffling & Sorting

- Shuffling
    - mapper의 출력을 reducer로 보내주는 프로세스를 말한다.
    - 전송되는 데이터의 크기가 크면 네트웤 병목을 초래한다.
- Sorting
    - 모든 Mapper의 출력을 Reducer가 받으면 이를 키별로 정렬한다.

### Data Skew

: 각 태스크가 처리하는 데이터의 크기에 불균형이 존재하는 경우에 병렬 처리가 의미가 없어지는 현상

- 가장 느린 태스크가 전체 처리 속도를 결정한다.


---
### 이전 포스트
- [데브코스 3차 프로젝트 (2)](https://poriz.github.io/dataengineering/camp/2024-01-17-dataengineering-camp-project3_2/)

### 다음 포스트
- [Spark](https://poriz.github.io/dataengineering/camp/2024-01-17-dataengineering-camp-Spark_2/)


