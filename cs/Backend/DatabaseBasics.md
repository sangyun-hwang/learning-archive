# Database Basics

## 개념

관계형 데이터베이스를 이해할 때 자주 나오는 기본 단위는 다음과 같다.

```txt
Database
  Schema
    Table
      Row
      Column
```

다만 DBMS마다 `database`와 `schema`의 의미는 조금씩 다를 수 있다. PostgreSQL에서는 하나의 database 안에 여러 schema를 둘 수 있고, MySQL에서는 database와 schema를 거의 같은 의미로 사용하는 경우가 많다.

## Database

Database는 데이터를 저장하고 관리하는 가장 큰 단위다.

예시:

```txt
shopping_mall_db
```

하나의 database 안에는 여러 schema, table, view, function 같은 객체가 들어갈 수 있다.

## Schema

Schema는 database 안에서 객체를 묶는 논리적 namespace다.

예시:

```txt
public
admin
sales
```

PostgreSQL에서는 다음처럼 하나의 database 안에 여러 schema를 두고 테이블 이름을 구분할 수 있다.

```txt
shopping_mall_db
  public.users
  public.orders
  admin.admin_users
```

같은 이름의 테이블도 schema가 다르면 구분될 수 있다.

```txt
public.users
admin.users
```

## Table

Table은 같은 구조의 데이터를 저장하는 표다.

예시:

```txt
users
orders
products
```

각 table은 여러 column을 가지고, 실제 데이터는 row 단위로 저장된다.

## Column

Column은 table이 가지는 속성이다.

예시:

```txt
users.id
users.email
users.created_at
```

Column은 이름과 데이터 타입을 가진다.

```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY,
  email VARCHAR(255),
  created_at TIMESTAMP
);
```

여기서 `id`, `email`, `created_at`이 column이다.

## Row

Row는 table에 저장된 실제 데이터 한 건이다.

예시:

```txt
id: 1
email: user@example.com
created_at: 2026-06-16
```

Table을 표로 보면 row는 가로 한 줄이고, column은 세로 속성이다.

```txt
users table

| id | email            | created_at |
| -- | ---------------- | ---------- |
| 1  | user@example.com | 2026-06-16 |
```

## Database와 Schema의 차이

일반적인 개념으로 보면 database는 가장 큰 저장 단위이고, schema는 그 안에서 테이블과 객체를 묶는 namespace다.

```txt
Database: 저장 공간의 큰 컨테이너
Schema: 객체를 구분하고 묶는 논리적 namespace
```

하지만 DBMS마다 다르게 다룰 수 있으므로 사용하는 DB 기준으로 확인해야 한다.

## 정리

Database는 데이터를 저장하는 큰 컨테이너다.

Schema는 database 안에서 table 같은 객체를 묶는 논리적 namespace다.

Table은 데이터를 저장하는 표이고, column은 속성, row는 실제 데이터 한 건이다.

표로 보면 column은 세로 방향의 속성이고, row는 가로 방향의 데이터 한 줄이다.
