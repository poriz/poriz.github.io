---
layout: post
title: Kafka - 실시간 데이터 개요
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---
# Kafka - 실시간 데이터 개요

## 데이터 처리의 발전 단계

- 배치 처리로 시작해서 서비스가 고도화 되면 점점 실시간 처리 요구가 생긴다.
- Throughput vs Latency
    - 대역폭(Bandwidth) = 처리량 x 지연시간
    - Throughput은 배치 시스템에서 중요하고, Latency는 실시간 시스템에서 더 중요하다.
- SLA (Service Level Agreement)
    - 서비스 제공 업체와 고객간의 계약 또는 합의
    - 사내 시스템들간에도 SLA를 정의하기도 한다.
        - 지연시간이나 업타임등이 SLA로 사용된다.
        - 데이터의 시의성도 중요한 포인트가 된다.

## 실시간 처리

: 연속적인 데이터 처리로, 지연시간이 중요하다.

- 초 단위의 연속적인 데이터 처리 ⇒ Event라고 부르며, 이것을 Event Stream이라 부르기도한다.
- 다양한 형태의 서비스들이 필요
    - 이벤트 데이터를 저장하기 위한 메세지큐: Kafka, Kinesis, Pub/Sub…
    - 이벤트 처리를 위한 처리 시스템: Spark Streaming, Flink, Samza…
- 처리 시스템 구조
    - Producer가 있어 데이터를 생성한다.
    - 생성된 데이터를 메세지 큐와 같은 시스템에 저장한다. (Kafka,Kinesis등)
    - Consumer는 큐로부터 데이터를 읽고 처리한다.
- 람다 아키텍처
    - 배치 레이어와 실시간 레이어 두개를 별도로 운영

### 실시간 데이터 종류와 사용 사례

- 데이터
    - Funnel Data
        - Product Impressions, Clicks, Purchase …
        - 회원 등록
    - Page Views and Performance Data
        - 페이지 별로 렌더링 시간을 기록, 디바이스 타입에 따라 기록
        - 에러 발생 시 에러 이벤트도 등록
    - 사용자 행동 데이터들의 데이터 모델 정의와 수집이 중요해졌다.
    - IoT 데이터
        - 센서 판독 값, 장치 상태 업데이트, 알람 이벤트 등등
- 유스케이스
    - Real-time Reporting
    - Real-time Alerting
    - Real-time Prediction ⇒ ML

### 실시간 데이터 처리 챌린지

- 실시간 데이터 처리 단계
    - 이벤트 데이터 모델 결정
    - 전송 & 저장
    - 데이터 처리
    - 데이터 관리, 모니터링
    
- 전송 & 저장
    - Point to Point: Many to Many 연결이 필요하다. → 데이터 유실의 위험성이 있다.
        - Latency가 더 중요한 시스템에서 사용하며, API들이 이런식으로 동작한다.
        - 다수의 소비자들이 존재하는 경우에는 데이터를 중복해서 보내야한다.
        - 데이터가 바로바로 처리되어야한다.
    - Messaging Queue: 중간에 데이터 저장소를 두고, 생산자와 소비자가 분리된 상태로 작업.
        - 다수의 소비자들이 공통의 데이터 소비가 가능하다.
        - 보통 micro-batch라는 형태로 아주 짧은 주기로 데이터를 모아서 처리한다.
        - Point-to-Point 보다는 운영에 용이하다.
    - Backpressure (배압)
    : 데이터의 소비가 데이터의 생성을 따라가지 못하는 경우에 발생하는 시스템 장애를 backpressure 이슈라고 한다. → 이를 줄이기 위한 방법 중 하나는 메세지 큐를 도입하는 것.


---
### 이전 포스트
- [Spark - file_format](https://poriz.github.io/dataengineering/camp/2024-01-19-dataengineering-camp-Spark_5/)

### 다음 포스트
- [Kafka - 소개 & 개념](https://poriz.github.io/dataengineering/camp/2024-01-24-dataengineering-camp-Kafka_2/)



