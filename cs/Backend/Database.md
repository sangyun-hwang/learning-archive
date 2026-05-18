# Database

DB와 SQL을 학습하면서 백엔드에서 데이터를 어떻게 저장하고 조회하는지 정리합니다.

## Stage 04 SQL 기초와 H2 Console
날짜: 2026-05-15
분류: Database / SQL
상태: 이해 중

### 질문

메모리 `List<StudyLog>`로 관리하던 데이터를 DB에서는 어떻게 저장하고 조회하는가?

### 지금의 답

Java에서는 여러 객체를 `List<StudyLog>`로 관리했지만, DB에서는 `study_logs` 테이블에 여러 row로 저장한다. Java 객체의 필드는 DB 테이블의 column과 대응되고, 객체 하나는 row 하나와 대응된다.

```text
StudyLog 클래스
-> study_logs 테이블 구조

StudyLog 객체 1개
-> study_logs 테이블의 row 1개

StudyLog 필드
-> study_logs 테이블의 column
```

### DB가 필요한 이유

- 서버가 종료되어도 데이터를 유지하기 위해
- 조건 검색과 정렬을 쉽게 하기 위해
- 수정, 삭제, 집계를 SQL로 처리하기 위해
- 여러 데이터 사이의 관계를 관리하기 위해

### 작성한 SQL

테이블 생성:

```sql
CREATE TABLE study_logs (
    id BIGINT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    category VARCHAR(50) NOT NULL,
    minutes INT NOT NULL,
    memo VARCHAR(500)
);
```

샘플 데이터 추가:

```sql
INSERT INTO study_logs (id, title, category, minutes, memo)
VALUES (1, 'Java class practice', 'JAVA', 60, 'field and constructor');
```

전체 조회:

```sql
SELECT *
FROM study_logs;
```

조건 조회:

```sql
SELECT *
FROM study_logs
WHERE category = 'JAVA';
```

정렬:

```sql
SELECT *
FROM study_logs
ORDER BY minutes DESC;
```

수정:

```sql
UPDATE study_logs
SET minutes = 90
WHERE id = 1;
```

삭제:

```sql
DELETE FROM study_logs
WHERE id = 3;
```

집계:

```sql
SELECT category, SUM(minutes)
FROM study_logs
GROUP BY category;
```

### 기억할 SQL 개념

`CREATE TABLE`은 데이터를 저장할 테이블 구조를 만든다.

`INSERT INTO`는 테이블에 row를 추가한다.

`SELECT`는 테이블에서 데이터를 조회한다.

`WHERE`는 조회, 수정, 삭제 대상을 제한한다.

`ORDER BY`는 결과를 정렬한다.

`UPDATE`는 기존 row를 수정한다.

`DELETE`는 기존 row를 삭제한다.

`COUNT(*)`는 row 개수를 센다.

`SUM(column)`은 특정 컬럼 값을 합산한다.

`GROUP BY`는 같은 값을 가진 row끼리 묶어서 집계한다.

### 주의할 점

SQL에서 문자열 값은 작은따옴표를 사용한다.

```sql
'JAVA'
```

큰따옴표는 DB에 따라 테이블명이나 컬럼명 같은 식별자에 사용될 수 있으므로, 문자열 값에는 작은따옴표를 쓰는 습관을 들인다.

`UPDATE`나 `DELETE`에서 `WHERE`를 빼면 모든 row가 영향을 받을 수 있다.

```sql
DELETE FROM study_logs;
```

위 쿼리는 테이블의 모든 학습 기록을 삭제할 수 있으므로 매우 조심해야 한다.

### H2 Console

IntelliJ Community 버전은 Database 실행 기능이 제한되어 있어서, Spring Boot의 H2 Console을 사용해 SQL을 실행했다.

설정:

```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

spring.datasource.url=jdbc:h2:mem:stage04;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=
```

접속 주소:

```text
http://localhost:8080/h2-console
```

로그인 정보:

```text
JDBC URL: jdbc:h2:mem:stage04;DB_CLOSE_DELAY=-1
User Name: sa
Password: 비워두기
```

H2 메모리 DB는 Spring Boot 서버를 재시작하면 데이터가 사라진다. 다시 사용하려면 `schema.sql`과 `seed.sql`을 순서대로 다시 실행한다.

### 다시 볼 포인트

- 메모리 `List`는 서버가 꺼지면 데이터가 사라진다.
- DB 테이블은 데이터를 row와 column 형태로 저장한다.
- `PRIMARY KEY`는 각 row를 구분하는 고유 값이다.
- `NOT NULL`은 값이 반드시 있어야 한다는 뜻이다.
- Java enum은 DB에서는 보통 문자열로 저장한다.
- SQL 파일 실행 순서는 `schema.sql` -> `seed.sql` -> 조회/수정/집계 SQL 순서가 자연스럽다.

## Stage 05 JDBC와 MySQL CRUD
날짜: 2026-05-19
분류: Database / JDBC
상태: 이해 중

### 질문

Java/Spring 애플리케이션에서 MySQL DB에 직접 연결해 CRUD를 수행하려면 어떤 흐름을 거치는가?

### 지금의 답

Spring Boot의 `application.properties`에 datasource 설정을 작성하면 Spring Boot가 `DataSource` 객체를 생성한다. Repository는 이 `DataSource`를 주입받고, `dataSource.getConnection()`으로 DB 연결을 얻어 SQL을 실행한다.

```text
application.properties
-> Spring Boot가 DataSource 생성
-> Repository가 DataSource 주입받음
-> Connection 획득
-> PreparedStatement로 SQL 준비
-> SQL 실행
-> ResultSet 또는 영향 받은 row 수 확인
```

### MySQL datasource 설정

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/spring_backend_study
spring.datasource.username=root
spring.datasource.password=${MYSQL_PASSWORD}
```

`localhost`는 내 컴퓨터의 MySQL 서버를 의미한다. 실제 배포 환경에서는 DB 서버 주소, 계정, 비밀번호를 운영 환경에 맞게 바꿔야 한다.

비밀번호는 GitHub에 올라가지 않도록 `${MYSQL_PASSWORD}` 환경변수로 분리했다.

### JDBC 핵심 객체

`Connection`은 DB와 연결된 통로이다.

`PreparedStatement`는 SQL을 담고, `?` placeholder에 값을 넣어 실행하는 객체이다.

`ResultSet`은 `SELECT` 실행 결과 row들을 읽는 객체이다.

### executeQuery와 executeUpdate

`SELECT`는 결과 row를 읽어야 하므로 `executeQuery()`를 사용한다.

```java
ResultSet resultSet = statement.executeQuery();
```

`INSERT`, `UPDATE`, `DELETE`는 영향을 받은 row 수를 확인하므로 `executeUpdate()`를 사용한다.

```java
int updatedRows = statement.executeUpdate();
```

### JDBC CRUD 흐름

전체 조회:

```text
SELECT 실행
-> ResultSet을 while문으로 순회
-> 각 row를 StudyLog 객체로 변환
-> List<StudyLog> 반환
```

단건 조회:

```text
SELECT ... WHERE id = ?
-> ResultSet에서 if (resultSet.next()) 확인
-> 있으면 StudyLog 반환
-> 없으면 null 반환
```

생성:

```text
다음 id 조회
-> StudyLog 객체 생성
-> INSERT 실행
-> 저장한 StudyLog 반환
```

수정:

```text
기존 StudyLog 조회
-> 요청에 없는 값은 기존 값 유지
-> UPDATE 실행
-> updatedRows가 0이면 없는 id로 판단
```

삭제:

```text
DELETE ... WHERE id = ? 실행
-> updatedRows가 0이면 삭제 대상 없음
-> 0보다 크면 삭제 성공
```

### 이번에 느낀 점

JDBC는 DB와 Java가 실제로 어떻게 연결되는지 이해하기 좋다. 하지만 코드가 반복된다.

반복되는 부분:

- Connection 얻기
- PreparedStatement 만들기
- `?`에 값 넣기
- SQLException 처리
- ResultSet에서 컬럼 하나씩 꺼내기
- row를 객체로 바꾸기
- try-with-resources로 자원 닫기

이 반복 때문에 다음 단계에서 MyBatis나 JdbcTemplate 같은 도구가 왜 필요한지 이해할 수 있다.

### 다시 볼 포인트

- JDBC는 ORM이 아니다.
- JDBC는 Java가 SQL을 직접 실행하기 위한 기본 통로이다.
- MyBatis와 JPA도 내부적으로는 JDBC를 통해 DB와 통신한다.
- `executeUpdate()`는 성공/실패 boolean이 아니라 영향 받은 row 수를 반환한다.
- 없는 id 수정/삭제는 SQL 에러가 아니라 `updatedRows == 0`으로 판단할 수 있다.
- Repository 코드는 DB 연결 대상이 H2든 MySQL이든 `DataSource`를 통해 연결한다.

## Stage 05 JDBC 반복 코드 정리
날짜: 2026-05-19
분류: Database / JDBC
상태: 이해 중

### mapRow()

SQL 조회 결과인 `ResultSet`의 한 행(row)을 자바 객체인 `StudyLog` 형식으로 바꾸는 과정이다.

`findAll`, `findByCategory`, `findById`에서 같은 변환 코드가 반복되어서 `mapRow()` 메서드로 분리했다.

### try-with-resources

예외가 발생하더라도 DB 연결(`Connection`), SQL 실행 객체(`PreparedStatement`), 조회 결과 객체(`ResultSet`)를 반드시 닫기 위한 문법이다.

서버는 오래 실행되기 때문에 DB 자원이 닫히지 않으면 연결이 쌓여 문제가 생길 수 있다.

### JDBC에서 아직 반복되는 코드

```java
Connection connection = dataSource.getConnection();
PreparedStatement statement = connection.prepareStatement(sql);
ResultSet resultSet = statement.executeQuery();
statement.setString(...);
statement.setLong(...);
```

그리고 예외 처리 코드도 여러 메서드에서 반복된다.

```java
catch (SQLException e) {
    throw new RuntimeException(e);
}
```

### 다음에 MyBatis가 필요한 이유

JDBC를 직접 쓰면 연결, SQL 실행, 파라미터 바인딩, 결과 매핑, 예외 처리 코드가 반복된다.

MyBatis는 이런 반복을 줄이고 SQL과 자바 객체 매핑을 더 깔끔하게 관리할 수 있게 도와준다.
