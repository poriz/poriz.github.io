---
layout: post
title: 데브코스 54일차(1) - Jinja, Dag Dependencies
sitemap: false
categorys:
  - dataengineering
  - camp
---

# 데브코스 54일차(1) - Airflow 운영과 대안

## 프로덕션 사용을 위한 Airflow 환경 설정

### airflow.cfg
- [core]의 dags_folder 경로 확인하기
- dag_dir_list_interval: DAG 파일의 스캔 주기 체크하기

### Airflow database upgrade
- 초기 Sqlite DB를 사용하고 있다면, Postgres나 MySQL로 변경해야 한다.
- sql_alchemy_conn를 이용해서 DB를 변경해주어야한다.
  - 주의: 상대경로를 입력하면 안된다.
  - Sqlite가 아닌 경우에 SequentialExecutor를 사용하면 안된다.
  - [sql_alchemy_conn을 이용해서 MySQL로 변경하기](https://spidyweb.tistory.com/349)

### 보안 및 백업, 서버 용량 관리
- Authentication사용 및 VPN을 통해 접근 제어하기.
- Scale Up을 먼저 진행 후 Scale Out을 진행하기. Cloud Service를 고려하자!
- Airflow metadata DB를 백업하기 → Variable, connections, 암호화키
- health-check monitoring하기 → DataDog, Grafana등을 사용

### log파일 삭제하기
: airflow.cfg를 확인하면 두 군데에 별도의 로그가 기록됨을 알 수 있음.
- base_log_folder = /var/lib/airflow/logs , 변경 될 수 있음.
- child_process_log_directory= /var/lib/airflow/logs/scheduler
- docker compose로 실행된 경우에 logs폴더가 host volume 형태로 유지된다.

### Airflow 대안
- Prefect
    - 오픈소스, Airflow와 흡사하며 경량화된 버전, 파이프라인을 동적으로 생성 가능하다.
- Dagster
    - 데이터 파이프라인과 데이터를 동시에 관리한다, 오픈소스
- Airbyte
- SaaS 형태의 데이터 통합툴들
    - FiveTran
    - Stitch Data
    - Segment