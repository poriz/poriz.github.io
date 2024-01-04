---
layout: post
title: Snowflake
sitemap: false

---
# snowflake 정리
: 프로젝트를 진행하며 snowflake를 이용해 데이터 웨어하우스를 구축하였고, 시행착오와 팁들을 정리한다.

## Colab을 활용하여 snowflake조작하기

: snowflake에 데이터를 colab으로 넣을 수 없을까 생각하다가 찾았던 라이브러리이다. 그러나 실사용에서는 snowflake의 웹을 통해서 넣는 것이 편해 잘 사용되지는 않았다.

1. `%pip install snowflake-connector-python` 
2. `import snowflake.connector`
3. 계정 정보를 입력해서 연결 수행

```python
ctx = snowflake.connector.connect(
    user='<user_name>',
    password='<password>',
    account='<account_name>'
    )
cs = ctx.cursor()
try:
    cs.execute("SELECT current_version()")
    one_row = cs.fetchone()
    print(one_row[0])
finally:
    cs.close()
ctx.close()
```

1. SQL 입력

```python
cs.cusor().execute(query)
```

추가 정보는 다음 블로그를 참고

[Snowflake Python에서 사용하는 법(Python Connector, Snowpark)](https://zzsza.github.io/data-engineering/2023/05/29/python-with-snowflake/)

## Superset과의 연결

: 결과부터 말하면 키를 몰라서 실패했던 것으로 추정된다. snowflake를 Docker를 사용한 Superset과 연결하기 위해 공부했던 내용이다.

1. 기존 컨테이너 삭제 `docker rm -f $(docker ps -qa)`
    1. 실제로는 딱히 삭제까지는 안하고 진행했던것같다.
2. superset/docker에 requirements-local.txt를 생성해야한다. `touch superset/docker/requirements-local.txt` 이렇게 생성하면 compose up 시에 버전에 맞추어 설치된다.
3. snowflake-sqlalchemy 설치 → requirements-local.txt에 입력한다.
4. 컨테이너 실행 `docker-compose -f docker-compose-non-dev.yml up`
5. 연결..

문제) 자꾸 valid error가 발생하였는데 개인키와 공개키를 등록해주면 된다고합니다.. 키 발급과 관련된 부분은 위에 colab 블로그에 같은 게시물을 참고하시면 됩니다…

## S3와 연결

: 연결과정의 경우 블로그의 글을 참고하며, 실제로 사용하였고, 동작한 코드위주로 정리글을 남긴다.

클라우드와 s3를 연결하기위해서는 3가지의 과정이 필요하다.

Integration → staging → COPY INTO

### IAM Policy 구성

: S3폴더에 액세스 하려면 몇가지 권한이 필요하다.

- `s3:GetBucketLocation`
- `s3:GetObject`
- `s3:GetObjectVersion`
- `s3:ListBucket`
- `s3:PutObject`
- `s3:DeleteObject`

JSON코드로 정책을 생성하면 다음과 같다.

```sql
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "s3:PutObject",
              "s3:GetObject",
              "s3:GetObjectVersion",
              "s3:DeleteObject",
              "s3:DeleteObjectVersion"
            ],
            "Resource": "arn:aws:s3:::<bucket>/<prefix>/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation"
            ],
            "Resource": "arn:aws:s3:::<bucket>",
            "Condition": {
                "StringLike": {
                    "s3:prefix": [
                        "<prefix>/*"
                    ]
                }
            }
        }
    ]
}
```

<bucket>에는 권한을 갖는 s3버킷을 입력하고, <prefix>에는 오브젝트의 경로를 입력한다.

### Integration

: 클라우드의 키나 액세스 토큰같은 자격 증명을 전달하지 않고도 연결을 수행하게 해주는 개체이다. 이를 이용하면 자격 증명 정보를 코드에 넣지 않고도 클라우드와의 연결이 가능해진다.

```sql
-- integration
CREATE STORAGE INTEGRATION s3_connect
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = '<aws_role_arn>'
  STORAGE_ALLOWED_LOCATIONS = ('<s3_bucket_location>');
```

### STAGING

: 파일을 snowflake로 로드하기 전에 임시로 저장하는 영역을 생성한다. 종류는 사용자, 테이블, 외부 stage가 있다.

```sql
-- staging
USE SCHEMA DATABASE_PROJECT.RAW_DATA;

CREATE STAGE my_s3_stage
  STORAGE_INTEGRATION = s3_connect
  URL = '<s3_bucket_location>';
```

### COPY INTO

- load

```sql
-- 로드
COPY INTO SCHEMA.TABLE_NAME
FROM <파일 경로>
FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1); --첫 줄의 헤더를 스킵하는 포맷
```

- unload

```sql
-- CSV 형식으로 언로드
COPY INTO @my_ext_unload_stage/TABLE_NAME
FROM TABLE_NAME
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"' FIELD_DELIMITER = ',' COMPRESSION = 'NONE');
-- SINGLE = TRUE; single로 하게되면 파일 하나로 언로드된다.
```

참고 및 출처) [Amazon S3에 액세스하도록 Snowflake 설정하기](https://velog.io/@ujeongoh/Amazon-S3에-액세스하도록-Snowflake-설정하기)