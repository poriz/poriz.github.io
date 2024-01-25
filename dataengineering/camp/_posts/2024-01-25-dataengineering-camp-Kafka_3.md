---
layout: post
title: Kafka - 기본 프로그래밍
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---
# 카프카 기본 프로그래밍

## Client Tool 사용

### kafka-topics

- 카프카 브로커를 설정하고 topic 리스트를 확인하기: `kafka-topics --bootstrap-server kafka1:9092 --list`
- topic 삭제하기: `kafka-topics --bootstrap-server kafka1:9092 --delete --topic topic_test`

### kafka-console-producer & consumer

- Command line을 통해서 Topic을 만들고 Message를 생성,소비한다.
- 생성: `kafka-console-producer --bootstrap-server kafka1:9092 --topic test_console`
- 소비: `kafka-console-consumer --bootstrap-server kafka1:9092 --topic test_console --from-beginning`
    - earliest >`--from-beginning` 옵션 넣기, 옵션이 없으면 latest로 동작.

## Topic 파라미터 설정

### Producer 옵션들

- Topic 생성시에 Partition이나 Replica를 여러개 만들기
    - KafkaAdminClient 오브젝트를 생성하고 create_topics함수로 Topic을 추가한다.
    - create_topics의 인자로 NewTopic 클래스의 오브젝트를 지정한다.

```python
# 예시
client = KafkaAdminClient(bootstrap_servers=bootstrap_severs)
topic = NewTopic(
	name=name,
	num_partitions=partitions, # 하나의 브로커가 다수의 partition을 처리 가능하다.
	replication_factor=replica) # replication_factor가 1이면 원본만 존재
client.creat_topics([topic])
```

- KafkaProducer 파라미터

| 파라미터 | 의미 | 기본 값 |
| --- | --- | --- |
| bootstrap_servers | 메세지를 보낼 때 사용할 브로커 리스트 | localhost:9092 |
| client_id | kafka producer의 이름 | kafka-python-{version} |
| key_serializer,value_serializer | 메세지의 key,value의 serialize 방법 지정(함수) |  |
| enable_idempotence | 중복 메세지 전송을 막을 것인가? | False |
| acks: 0, 1, ‘all’ | consistency level, 0: 바로 리턴, 1: 리더에 쓰일 때 까지 대기, all: 모든 partition leader/follower에 적용까지 대기 | 0 |
| retries,delivery.timeout.ms | 메세지 실패시 재시도 횟수, 메세지 전송 최대 시간 | 2147483647 120000ms > 2m |
| linger_ms, batch_size | 다수의 메세지를 동시에 보내기 위함.,메세지 송신전 대기시간, 메세지 송신전 데이터 크기 | 0, 16384 |
| max_in_flight_request_per_connection | Broker에게 응답을 기다리지 않고 보낼 수 있는 메세지의 최대 수 | 5 |

- 전체 코드: https://github.com/keeyong/kafka/blob/main/fake_person_producer.py

### Consumer 옵션들

- KafkaConsumer 파라미터

| 파라미터 | 의미 | 기본 값 |
| --- | --- | --- |
| bootstrap_servers | 메세지 전송 시 사용할 브로커 리스트 | localhost:9092 |
| client_id | Kafka Consumer의 이름 | kafka-python-{ve
rsion} |
| group_id | Kafka Consumer Group의 이름 → 같은 Group을 가지면 partition을 나눠줌 |  |
| key_deserializer, value_deserializer | 메세지의 키와 값의 deserialize 방법 지정 (함수) |  |
| auto_offset_reset | earliest, latest | latest |
| enable_auto_commit | True: 소비자의 오프셋이 백그라운드에서 주기적으로 커밋 False: 명시적으로 커밋을 해줘야한다. 오프셋은 별도 리셋이 가능, WebUI에서도 가능하다. | True |

- Consumer가 다수의 Partition들로부터 읽는 방법
    - Consumer가 하나이고 다수의 Partition들로 구성된 Topic으로부터 읽는경우: 기본적으로 라운드 로빈 형태로 읽게된다.
    - 이 경우에 Backpressure가 심해질 수 있는데 이것을 해결하기위해 Consumer Group을 사용한다.
    - 하나의 프로세스에서 다수의 Topic을 읽는 것을 가능하게 해준다.
- Consumer Group > 병렬성을 높이기 위해 사용
    - Consumer Group Rebalancing: 기존 consumer가 사라지거나 새로운 consumer가 group에 참여하는 경우에 partition들이 다시 지정되는 것.
- Message Processing Guarantee 방식 ⇒ 실시간 처리 및 전송의 과점에서 시스템의 보장방식
    - Exactly Once: 가장 보장하기 힘든 케이스. 딱 한 번만 전달되는 것을 보장한다. Producer에서는 enable_idempotence를 True로 설정하고 메세지 송수신시 Trancsaction API를 사용한다.
    - At Least Once: 모든 메세지가 consumer에게 적어도 한번 이상 전달되도록 보장하나, 중복 가능성이 존재한다. → 중복 제거 매커니즘을 구현해야한다.
    - At Most Once: 메세지가 손실이 있을 수 있으나, 중복이 없음을 의미한다. → 가장 흔한 메세지 전송 보장 방식이다.

### Consumer/Producer 패턴

- 많은 경우에 Consumer는 한 Topic의 메세지를 소비해서 새로운 Topic을 만들기도한다.
→ Consumer이면서 Producer로 동작하는 것이 흔한 패턴이다.
- Data Transformation, Filtering 과 같은 예시가 있다.

## ksqlDB 사용하기

### ksqlDB

- REST API나 ksql 클라이언트 툴을 사용해서 Topic을 테이블처럼 SQL로 조작
- ksql 데모
    - confluentinc/cp-ksqldb-server의 Container ID를 복사
    - `docker exec -it ContainerID sh`
    - `SELECT * FROM my_stream;`


---
### 이전 포스트
- [Kafka - 소개 & 개념](https://poriz.github.io/dataengineering/camp/2024-01-24-dataengineering-camp-Kafka_2/)


