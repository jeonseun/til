# CREATE TABLE로 테이블 만들기

DDL 중 테이블 생성을 위해 사용하는 `CREATE TABLE` 기본 개념 정리

:gear: 쿼리 실행 환경(타 RDBMS 호환성 보장 X)

- MySQL 8.0.34

## TOC

- [CREATE TABLE 개요](#create-table-개요)
- [속성 정의](#속성-정의)
  - [속성의 데이터 타입](#속성의-데이터-타입)
  - [속성의 null 허용 여부](#속성의-null-허용-여부)
  - [속성의 기본값 설정](#속성의-기본값-설정)
- [키 속성 정의](#키-속성-정의)
  - [기본키, PRIMARY KEY](#기본키-primary-key)
  - [대체키, UNIQUE](#대체키-unique)
  - [외래키, FOREIGN KEY](#외래키-foreign-key)
    - [참조 대상 변경에 따른 작업 설정](#참조-대상-변경에-따른-작업-설정)
- [제약조건 설정](#제약조건-설정)

<br>

## CREATE TABLE 개요

`CREATE TABLE` 은 신규 테이블을 생성하는 sql로 테이블을 구성하는 다음 내용을 포함한다.

- 테이블 이름
- 속성 정의
- 키 속성 정의
- 제약조건 설정

`CREATE TABLE` 의 기본 구조

```sql
CREATE TABLE table_name
(
    column_name data_type [NOT NULL] [DEFAULT 기본값],
    [CONSTRAINT constraint_name] [PRIMARY KEY (column_name ...)],
    [CONSTRAINT constraint_name] [UNIQUE (column_name ...)],
    [CONSTRAINT constraint_name] [FOREIGN KEY (column_name ...) REFERENCES table_name (column_name ...)] [ON DELETE 옵션] [ON UPDATE 옵션],
    [CONSTRAINT constraint_name] [CHECK (condition)]
);
```

`CREATE TABLE` 은 여러줄로 구성될 수 있으며 각 줄을 구분하기 위해 줄의 끝에는 `,` 를 붙인다. 맨 마지막 줄인 경우 생략한다.

<br>

## 속성 정의

테이블을 구성하는 속성 즉 컬럼(column)을 정의할 때는 컬럼의 데이터 타입과 null 허용여부, 기본값 등을 지정해야 한다.

```sql
CREATE TABLE table_name
(
    ...
    column_name data_type [NOT NULL] [DEFAULT 기본값],
    ...
)
```

### 속성의 데이터 타입

다음은 표준 sql에서 지원하는 데이터 타입이다. (DBMS 별로 차이있을 수 있음)

| 데이터 타입                        | 설명                                                                                |
| ---------------------------------- | ----------------------------------------------------------------------------------- |
| INT, INTEGER                       | 정수                                                                                |
| SMALLINT                           | INT 보다 작은 정수                                                                  |
| CHAR(n) \| CHARACTER(n)            | 길이 n의 고정 문자열                                                                |
| VARCHAR(n) \| CHARACTER VARYING(n) | 최대 길이 n의 가변 문자열                                                           |
| NUMERIC(p, s) \| DECIMAL(p, s)     | 고정 소수점 실수, p는 소수점을 제외한 전체 숫자의 길이, s는 소수점 이하 숫자의 길이 |
| FLOAT(n)                           | 길이 n의 부동 소수점 실수                                                           |
| REAL                               | 부동 소수점 실수                                                                    |
| DATE                               | 연, 월, 일로 표현하는 날짜                                                          |
| TIME                               | 시, 분, 초로 표현하는 시간                                                          |
| DATETIME                           | 연, 월, 일, 시, 분, 초로 표현하는 날짜와 시간                                       |

나이, 이름, 생일 정보를 저장하는 member 테이블 생성 예시

```sql
CREATE TABLE member
(
    age   INT,         -- 숫자 타입 속성
    name  VARCHAR(10), -- 문자열 타입 속성, ()내 문자열 길이 필수 지정
    birth DATE         -- 날짜(연, 월, 일) 타입 속성
);
```

### 속성의 null 허용 여부

속성이 null 값을 가질 수 있는지 지정하는 옵션이다.

- `NULL` → null 값을 허용함, 기본값
- `NOT NULL` → null 값을 가지지 못하도록 함

```sql
CREATE TABLE tbl1
(
    column1 VARCHAR(10),         -- null 허용 속성
    column2 VARCHAR(10) NULL,    -- null 허용 속성, 키워드 생략가능
    column3 VARCHAR(10) NOT NULL -- null 불가능 속성
);
```

### 속성의 기본값 설정

기본값은 속성에 값을 지정하지 않았을 때의 값 말한다.

- `DEFAULT 기본값` → 지정된 기본값이 입력되게 함
- 기본값으로 지정한 값과 속성의 데이터 타입이 서로 맞아야 함
- 기본값을 별도로 지정하지 않는 경우 기본값은 null로 처리됨

```sql
CREATE TABLE member
(
    name  VARCHAR(10) DEFAULT 'unknown', -- 속성의 기본값을 'unknown'으로 설정
    point INT         DEFAULT 0,         -- 속성의 기본값을 0으로 설정
    email VARCHAR(10)                    -- 속성의 기본값 null (DEFAULT null)
);
```

> 속성의 기본값을 설정해도 데이터를 입력할 때 직접 null을 넣는 경우 해당 속성의 값은 null이 된다.

<br>

## 키 속성 정의

`CREATE TABLE` 내에서 다음 키 속성을 정의할 수 있다.

- 기본키
- 대체키
- 외래키

### 기본키, PRIMARY KEY

기본키는 테이블 내에서 튜플 즉 행(row)를 식별하기 위한 키 속성 중 기본으로 사용할 대상을 말하며 다음 특징을 지닌다.

- `PRIMARY KEY` 로 기본키 지정
- 각 테이블 당 하나만 지정 가능
- 1 ~ n개의 속성으로 구성됨
- null, 중복된 값을 가질 수 없음 (개체 무결성 제약조건)
  - 기본키가 다수의 속성으로 구성되는 경우 속성 집합 자체가 유일성을 만족해야 함
  - 기본키가 다수의 속성으로 구성되는 경우에도 각 속성은 null 값을 가질 수 없음
- 기본키가 없는 테이블도 만들 수 있지만 기본키를 지정하는 것을 권장

기본키 지정 방법

```sql
-- 속성을 정의할 때 기본키로 지정, 단일 속성으로만 구성 가능
CREATE TABLE member
(
    id INT PRIMARY KEY
);

-- 기본키 제약조건 설정
CREATE TABLE member
(
    id   INT,
    name VARCHAR(20),
    PRIMARY KEY (id, name)
);

-- 기본키 제약조건 설정 (이름 지정)
CREATE TABLE member
(
    id INT,
    CONSTRAINT pk_member PRIMARY KEY (id) -- 기본키 제약조건 이름 'pk_member'로 지정
);
```

### 대체키, UNIQUE

대체키는 테이블 내에서 각 행(row)를 구분하기 위한 키 속성으로 몇 가지 사항을 제외하면 기본키와 비슷한 역할을 한다.

- `UNIQUE` 로 대체키 지정
- 테이블 내에서 여러개 지정 가능
- 1 ~ n개의 속성으로 구성됨
- 중복값은 가질 수 없으나 null 값은 가질 수 있음
  - 여러 행이 null 값을 가져도 무관
  - 대체키를 구성하는 속성에 null 값이 포함되면 다른 행과 구분하는 용도로 사용 불가능 (유일성 상실)
  - 기본키 처럼 유일성을 항상 만족하려면 대체키를 구성하는 모든 속성을 `NOT NULL` 로 지정 해야함

대체키 지정 방법

```sql
-- 속성을 정의할 때 대체키로 지정, 단일 속성으로만 구성 가능
CREATE TABLE member
(
    nickname VARCHAR(20) UNIQUE
);

-- 대체키 제약조건 설정
CREATE TABLE member
(
    id       INT,
    nickname VARCHAR(20),
    UNIQUE (id, nickname)
);

-- 대체키 제약조건 설정 (이름 지정)
CREATE TABLE member
(
    id INT,
    CONSTRAINT uk_member UNIQUE (id) -- 대체키 제약조건 이름 'uk_member'로 지정
);

-- 항상 유일성을 만족하는 대체키 설정
CREATE TABLE member
(
    -- 유일성을 만족하는 단일 속성 대체키 지정
    id       INT         NOT NULL UNIQUE,
    -- 유일성을 만족하는 다중 속성 대체키 지정
    name     VARCHAR(20) NOT NULL,
    nickname VARCHAR(20) NOT NULL,
    CONSTRAINT uk_member UNIQUE (name, nickname)
);
```

### 외래키, FOREIGN KEY

외래키는 특정 테이블에서 다른 테이블의 기본키(또는 대체키)를 참조하는 키 속성으로 테이블 간의 관계를 나타내기 위해 사용된다.

- `FOREIGN KEY (column, ...)` 로 외래키 지정, `REFERENCES table_name (column, ...)` 로 참조할 테이블과 속성 지정
- 외래키는 참조하는 테이블의 기본키(`PRIMARY KEY`) 또는 대체키(`UNIQUE`)만 참조할 수 있음 (참조 무결성 제약조건)
- 테이블 내에서 여러개 지정 가능
- 1 ~ n개의 속성으로 구성됨
- 외래키와 참조하는 테이블의 키 속성(기본키 또는 대체키)은 데이터 타입이 같아야 함(일부 DBMS의 경우 유사한 타입까지 허용)
- 참조하는 테이블의 기본키(또는 대체키)가 여러 속성으로 구성되는 경우 그 중 일부 속성만 참조하도록 지정 가능

> 키의 일부 속성만 참조하도록 외래키를 지정할 경우 참조 무결성과 유연성을 강화할 수 있는데 이유는 다음과 같다.
>
> 1. 참조하는 테이블의 키 속성 중 일부만 참조하기 때문에 외래키가 일부 속성에만 의존하게 됨
> 2. 참조하는 테이블의 키 속성 중 일부가 변경되어도 외래키의 참조가 유효할 수 있음
> 3. 참조하는 테이블의 키 속성 중 일부의 값이 변경되어도 외래키의 변경 없이 참조를 유지할 수 있음

외래키 지정 방법

![member team erd](../resources/images/sql/CREATE%20TABLE로%20테이블%20만들기/erd.png)

_회원과 팀의 관계를 나타내는 ERD [powered by DrawSQL](https://drawsql.app/diagrams)_

```sql
-- team, member 두 테이블의 관계를 나타내기 위한 외래키 지정, 편의상 일부 속성 생략

-- member 테이블의 외래키가 참조하게될 team 테이블
CREATE TABLE team
(
    id INT PRIMARY KEY -- 외래키가 참조할 수 있는 속성은 기본키 또는 대체키
);

-- 외래키 제약조건 설정
CREATE TABLE member
(
    id      INT PRIMARY KEY,
    team_id INT,                               -- 외래키로 쓰일 속성
    FOREIGN KEY (team_id) REFERENCES team (id) -- 외래키 지정
)

-- 외래키 제약조건 설정 (이름 지정)
CREATE TABLE member
(
    id      INT PRIMARY KEY,
    team_id INT,
    CONSTRAINT fk_member_team FOREIGN KEY (team_id) REFERENCES team (id) -- 외래키 제약조건 이름 'fk_member_team'로 지정
)
```

#### 참조 대상 변경에 따른 작업 설정

외래키를 통해 관계가 형성된 두 테이블에서 관련있는 행의 삭제, 수정에 따라 수행될 동작을 지정할 수 있다.

- 외래키가 참조하는 대상의 삭제, 수정에 따라 외래키가 지정된 테이블에 수행되는 동작을 지정하는 설정
- 참조하는 테이블의 데이터 삭제, 수정 시 수행될 동작을 지정할 수 있으며 세부적인 4가지 옵션을 지정 가능

```sql
CREATE TABLE team
(
    id INT PRIMARY KEY
);

CREATE TABLE member
(
    id      INT PRIMARY KEY,
    team_id INT,
    FOREIGN KEY (team_id) REFERENCES team (id)
        [ON DELETE] [옵션] -- team 테이블의 행 삭제시 동작 지정
        [ON UPDATE] [옵션] -- team 테이블의 행 수정시 동작 지정
);
```

- `ON DELETE` → 외래키가 참조하는 행의 삭제 시 수행될 동작 지정
- `ON UPDATE` → 외래키가 참조하는 행의 수정 시 수행될 동작 지정

옵션 목록

- `NO ACTION` → 외래키가 참조하는 행의 삭제, 수정을 금지, 기본값
- `CASCADE` → 외래키가 참조하는 행의 삭제, 수정 시 관련 행을 함께 삭제, 수정
- `SET NULL` → 외래키가 참조하는 행의 삭제, 수정 시 해당 외래키 값을 null로 변경
- `SET DEFAULT` → 외래키가 참조하는 행의 삭제, 수정 시 해당 외래키 값을 `DEFAULT` 에서 설정한 값으로 변경

<br>

## 제약조건 설정

기본키, 대체키, 외래키 같은 제약조건 외에도 특정 속성에 대한 제약조건을 설정할 수 있다.

- `CHECK (condition)` 로 제약조건 설정
- 제약조건을 설정할 경우 데이터의 삽입, 수정시 설정된 제약조건을 지켜야 함 (데이터 무결성 제약조건)
- 여러개의 제약조건을 설정할 수 있음

제약조건 설정 방법

```sql
-- 속성을 정의할 때 제약조건 설정
CREATE TABLE product
(
    name  VARCHAR(20),
    stock INT CHECK ( stock >= 0) -- stock (재고량)은 음수일 수 없음
);

-- 제약조건 설정
CREATE TABLE product
(
    name  VARCHAR(20),
    stock INT,
    CHECK ( stock >= 0 )
);

-- 제약조건 설정 (이름 지정)
CREATE TABLE product
(
    name  VARCHAR(20),
    stock INT,
    CONSTRAINT constraint_stock CHECK ( stock >= 0 )
);
```

<br>

## References

- [데이터베이스 개론 3판 #ch7 - 김연희, 한빛출판네트워크](https://www.hanbit.co.kr/store/books/look.php?p_code=B6505632990)
- [Real MySQL 8.0 2권 - 백은빈, 이성욱, 위키북스](https://wikibook.co.kr/realmysql802/)
