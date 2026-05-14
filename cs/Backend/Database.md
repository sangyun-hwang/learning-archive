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
