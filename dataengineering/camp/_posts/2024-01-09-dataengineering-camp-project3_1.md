---
layout: post
title: 데브코스 3차 프로젝트 (1)
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---

# 데브코스 3차 프로젝트 (1)
프로젝트 기간 : 01/08 ~ 01/12 (5일)

## 프로젝트 목표 & 주제
- 목표: 데이터 엔지니어링을 통한 데이터 수집, 저장, 가공, 분석, 시각화, 서비스 구현
- 주제: 기상에 따른 서울시 자전거 대여 현황파악

## 프로젝트 Github
- [Github](https://github.com/K-bike-DE)

---
<b>프로젝트에 관한 정리는 Git의 Readme를 활용하며, 블로그에는 프로젝트를 진행하며 배운 내용을 정리한다.</b>

## Airflow CI/CD 구축
: 기존에 Airflow와 git actions를 이용해서 구축했던 내용을 토대로 프로젝트에 맞게 수정하여 진행하였다.
- [Airflow CI/CD 구축](https://velog.io/@pori/series/airflow) (해당 포스팅은 GIT으로 다시 가져와서 수정할 계획이다.)
- github Actions의 path 설정</br>
`docker-image.yml`을 이용해서 변경된 부분을 감지하는 코드를 살짝 수정하였다. 특정 파일만 수정 시에 Action이 동작하도록 하였는데 `path`를 이용하는 방법이다.
```yml
on:
  push:
    paths:
      - "Dockerfile"
      - "docker-compose.yml"
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
```
## Airflow-snowflake 연결하기
Airflow에서 snowflake를 연결하기 위해 여러 자료들을 찾아보고 정리하였다.
- [Airflow-snowflake 연결하기](https://poriz.github.io/dataengineering/2024-01-09-dataengineering-airflow_snowflake/)
- 자료들이 영어로 되어있고, 버전이 업데이트되면서 달라진 부분이 있어 애를 먹었다. 그래도 Docs와 snowflake-community를 참고하여 해결하였다.
### `.env` 설정
DAG의 테스트를 위해 로컬에서 Airflow를 돌리다보니 환경설정을 통일해야하는 문제가 있었다.<br>
`.env`를 이용해서 환경변수를 관리하기로 하였고, 다음 블로그를 참고해서 진행했다.
- [Airflow - Connections, Variables 관리하기](https://wooiljeong.github.io/airflow/airflow-manage-env/) 블로그에 의하면 해줘야할 내용은 다음과 같다.
    1. `.env` 파일 생성
    2. `.env` 파일에 환경변수 설정, 예제는 아래와 같다.
        ```
        AIRFLOW_CONN_EXAMPLE='{
        "conn_type": "type",
        "values": "value"
        }'
        AIRFLOW_VAR_EXAMPLE="VALUE"
        ```
    3. docker-compose.yml에 env 설정하기
        ```yml
        env_file:
            - .env
        ```

---
### 이전 포스트
- [데브코스 55일차(2) - 데이터 카탈로그](https://poriz.github.io/dataengineering/camp/2024-01-05-dataengineering-camp-Day55_2/)

### 다음 포스트
- [데브코스 3차 프로젝트 (2)](https://poriz.github.io/dataengineering/camp/2024-01-09-dataengineering-camp-project3_2/)