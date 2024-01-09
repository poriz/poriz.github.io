---
layout: post
title: 데브코스 55일차(2) - 데이터 카탈로그
sitemap: false
comments: true
categorys:
  - dataengineering
  - camp
---

# 데브코스 55일차(2) - 데이터 카탈로그
## 데이터 카탈로그

: 데이터 카탈로그는 중요한 데이터 기술스택이다. 

### **데이터 카탈로그란 무엇인가?**

- 데이터 카탈로그는 데이터 자산 메타 정보 중앙 저장소이다. → 메타 정보만 수집
- 데이터 거버넌스의 첫 걸음이다.
- 데이터 카탈로그의 중요한 기능
    - 자동화된 메타 데이터 수집
    - 메타 데이터만을 수집!
- 데이터 자산의 효율적 관리를 위한 프레임워크
    - 데이터 용어와 지표, 태그를 명확하게 관리
    - 데이터 오너를 지정
    - 표준화된 문서 템플릿
- 데이터 리지니 지원

### 주요 데이터 플랫폼 지원

: 다양한 데이터 플랫폼들을 지원한다. 

- Data Warehouse (Redshift, Snowflake, BigQuery …)
- BI Tools (Looker, Tableau …)
- ELT & ETL (DBT, Spark, Airflow …)

이외에도 NoSQL이나 Azure AD등등 여러 플랫폼을 지원한다.

### 비즈니스 용어집 (Business Glossary)

- 계층 구조로 관리하여 유용하게 사용 가능하다.
- 권한을 설정해서 사용하게된다.
- 비즈니스 용어와 Entity 연결이 가능하다.
- 협업과 태그설정이 가능한데 아래와 같은 기능을 제공한다.
    - 태그 기능: 비즈니스 용어이외에도 태그를 지원
    - 문서화 표준 제공

### 데이터 거버넌스 관점에서 데이터 카탈로그의 중요성

- 데이터 자산에 대한 통합 뷰를 제공한다.
- 생산성의 증대: 설문이나 데이터 티켓의 감소로 확인 가능하다.
- 위험 감소: 잘못된 결정과 개인정보등의 민감한 정보 전파를 방지 가능
- 인프라 비용 감소: 불필요한 정보의 생성 방지와 방치되는 데이터셋 삭제
- 데이터 티켓 감소
- 데이터 변경으로 인한 이슈 감소

---
### 이전 포스트
- [데브코스 55일차(1) - DBT (Data Build Tool) - 2](https://poriz.github.io/dataengineering/camp/2024-01-05-dataengineering-camp-Day55_1/)

### 다음 포스트
- [데브코스 3차 프로젝트 (1)](https://poriz.github.io/dataengineering/camp/2024-01-05-dataengineering-camp-project3_1/)