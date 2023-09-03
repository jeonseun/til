# sql 스크립트로 DB 초기화하기

spring boot(3.1.3 사용)에서는 sql 스크립트를 통한 DB 초기화 기능을 제공한다.

### TOC

- [실행될 sql 스크립트 지정](#실행될-sql-스크립트-지정)
- [초기화 모드 설정하기](#초기화-모드-설정하기)

<br>

## 실행될 sql 스크립트 지정

spring boot는 classpath 경로내에 sql 스크립트가 있는 경우 해당 스크립트 파일을 애플리케이션 실행 시 실행해준다. 실행하는 파일은 다음과 같다.

- `schema.sql`
- `data.sql`

내부적으로 처리될 때는 같은 메서드를 사용해 처리되므로 각 스크립트 파일에 들어가야 하는 sql의 종류가 엄격하게 정해진 것은 아니다. 단 구분을 위해 `schema.sql`에는 DDL을 `data.sql`에는 DML을 작성하는 것이 보편적이다.

### sql 스크립트 경로 지정

초기화를 위해 실행하는 sql 스크립트의 기본 경로는 다음과 같다.

- `optional:classpath*:schema-all.sql`
- `optional:classpath*:schema.sql`
- `optional:classpath*:data-all.sql`
- `optional:classpath*:data.sql`

<br>

## 초기화 모드 설정하기

sql 스크립트를 통한 초기화는 h2같은 내장 DB를 사용할 때만 동작하는데 이는 `sql.init.mode` 옵션의 기본값이 `embedded`로 설정되어있기 때문이다. 모드는 총 3개가 있으며 다음과 같다.

- `always` 항상 sql 스크립트를 실행하는 설정
- `embedded` 내장 DB에서만 sql 스크립트를 실행하는 설정(기본값)
- `never` 항상 sql 스크립트를 실행하지 않는 설정

따라서 외장 DB 사용 시에도 sql 스크립트를 통한 초기화 작업을 수행하려면 다음과 같이 `application.yml`을 설정해야 한다.

```yml
# application.yml
spring:
  sql:
    init:
      mode: always
```
