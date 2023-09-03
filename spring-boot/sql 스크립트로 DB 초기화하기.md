# sql 스크립트로 DB 초기화하기

spring boot가 제공하는 sql 스크립트를 통한 DB 초기화 기능 사용방법 정리

:gear: versions

- spring boot 3.1.3

### TOC

- [초기화 스크립트 설정](#초기화-스크립트-설정)
  - [적용 모드 설정](#적용-모드-설정)
  - [sql 스크립트 기본 경로](#sql-스크립트-기본-경로)
  - [sql 스크립트 경로 설정](#sql-스크립트-경로-설정)
  - [DB 플랫폼 별 스크립트 지정](#db-플랫폼-별-스크립트-지정)
  - [스크립트 에러 시 처리 설정](#스크립트-에러-시-처리-설정)
  - [JPA와 함께 쓸 경우 주의사항](#JPA와-함께-쓸-경우-주의사항)
- [동작원리](#동작원리)

<br>

## 초기화 스크립트 설정

초기화 스크립트의 적용 방식과 대상 스크립트 경로 및 기타 오류 처리 등은 `application.yml`의 설정을 통해서 변경할 수 있다.

### 적용 모드 설정

sql 스크립트는 기본적으로 임베디드 DB(h2 등의 인 메모리 모드)에서만 실행된다. 따라서 MySQL 같은 외부 DB와 연결할 경우 실행되지 않는다. 이는 `spring.sql.init.mode` 설정 값을 변경함으로써 동작하도록 할 수 있는데 설정할 수 있는 옵션은 다음과 같다.

- `embedded` (기본값) → 임베디드 DB에서만 스크립트 실행
- `never` → 어떤 경우에도 스크립트를 실행하지 않음
- `always` → 항상 스크립트를 실행

```yml
# application.yml
spring:
  sql:
    init:
      mode: embedded
      # mode: never
      # mode: always
```

### sql 스크립트 기본 경로

기본적으로 resources 디렉토리에 있는 schema.sql, data.sql 파일을 읽어서 실행하며 schema.sql에는 주로 DDL, data.sql에는 DML이 들어가는게 일반적이다. 기본값으로 설정된 경로는 다음과 같다.

- `optional:classpath*:/schema.sql`
- `optional:classpath*:/data.sql`

경로의 각 부분이 의미하는 것은 다음과 같다.

- `optional:` → 해당 sql 스크립트가 선택사항임을 나타내며 없거나 읽기 불가능한 상태 또는 디렉토리일 경우 에러 없이 무시하고 넘어감
- `classpath*:` → classpath 경로 부터 읽는 것을 의미하며 \*는 일치하는 리소스가 여러개일 경우 모두 읽는것을 뜻함
- `/schema.sql`, `/data.sql` → 실행할 스크립트 파일 경로

### sql 스크립트 경로 설정

sql 스크립트의 기본 경로를 변경 하려면 `spring.sql.init.schema-locations`, `spring.sql.init.data-locations` 설정값을 변경하면 된다.

```yml
# application yml
spring:
  sql:
    init:
      schema-locations: classpath*:/db/ddl.sql
      data-locations: classpath*:/db/dml.sql
```

경로를 직접 설정할 때 읽기 불가능한 상태를 무시하려면 optional: 접두사를 붙이면 된다. 또한 일반적으로 resources 디렉토리에 위치하는 것을 고려해서 classpath 경로로 시작하는 것이 바람직하다.

### DB 플랫폼 별 스크립트 지정

sql 스크립트 파일의 이름을 다음 형식으로 작성할 경우 특정 DB 플랫폼일 때 실행될 스크립트를 결정할 수 있다.

- `schema-${platform}.sql`
- `data-${platform}.sql`

이 때 `${paltform}`의 값은 `spring.sql.init.platform`에서 지정한 값으로 치환된다. 예를 들어 sql 스크립트가 `schema-mysql.sql`이고 `spring.sql.init.platform`의 설정값이 mysql이면 해당 스크립트가 실행된다. 반면에 플랫폼이 h2로 설정된 경우 해당 스크립트는 무시된다.

```yml
# application.yml
spring:
  sql:
    init:
      platform: mysql # schema-mysql.sql, data-mysql.sql이 실행됨
      # platform: h2 # schema-h2.sql, data-h2.sql이 실행됨
```

플랫폼 별 sql 스크립트 실행 시 다음 사항을 주의해야 한다.

- `spring.sql.init.platform`의 기본값은 all이므로 별도의 플랫폼을 지정해둔 스크립트 파일은 무시된다.
- `spring.sql.init.schema-locations`, `spring.sql.init.data-locations` 설정을 변경해서 sql 스크립트 경로를 변경한 경우 이 기능은 동작하지 않는다.

### 스크립트 에러 시 처리 설정

`spring.sql.init.continue-on-error` 설정값에 따라 sql 스크립트 실행 시 발생한 에러에 대한 처리방식이 달라진다.

- `true` → sql 스크립트 실행 시 에러가 발생해도 무시하고 애플리케이션을 실행
- `false` → sql 스크립트 실행 시 에러가 발생하면 애플리케이션을 종료

### JPA와 함께 쓸 경우 주의사항

`EntityManagerFactory` 빈 생성보다 sql 스크립트를 통한 초기화가 먼저 수행된다. 만약 JPA의 ddl-auto 옵션등을 통해 DB를 초기화한 후 추가작업을 sql 스크립트로 처리하고 싶은 경우 `spring.jpa.defer-datasource-initialization` 설정값을 `true`로 변경하면 된다. 이 경우 sql 스크립트에 의한 초기화작업을 `EntityManagerFactory` 빈 생성과 초기화 이후로 미룰 수 있다.

단 sql 스크립트를 통한 초기화와 다른 방식의 초기화(JPA, Flyway, Liquibase)를 혼용하는 것은 권장되지 않는다.

<br>

## 동작원리

sql 스크립트를 통한 초기화는 `SqlDataSourceScriptDatabaseInitializer` 빈에 의해 수행된다. 해당 빈은 `InitializationBean` 인터페이스의 하위 타입으로 빈 초기화 메서드를 구현하고 있으며 초기화 메서드 내에서 sql 스크립트를 실행하는 구조로 구현되어 있다.

`SqlDataSourceScriptDatabaseInitializer`는 spring boot 자동 설정에 의해 등록되는 빈이며 `DataSource` 빈이 등록된 경우에만 빈으로 등록된다.

<br>

## References

- [Spring Boot Reference Documentation #18.9.3](https://docs.spring.io/spring-boot/docs/3.1.3/reference/htmlsingle/#howto.data-initialization.using-basic-sql-scripts)
