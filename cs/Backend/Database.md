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

## Stage 06 MyBatis와 JDBC 비교
날짜: 2026-05-19
분류: Database / MyBatis
상태: 이해 중

### 질문

JDBC로 직접 작성하던 DB 접근 코드는 MyBatis를 쓰면 어떻게 줄어드는가?

### 지금의 답

JDBC에서는 `Connection`을 얻고, `PreparedStatement`를 만들고, SQL 파라미터를 바인딩하고, `ResultSet`을 읽고, 자원을 닫는 코드를 직접 작성해야 했다.

MyBatis는 이런 반복적인 JDBC 실행 절차를 줄여준다. 다만 SQL 자체는 개발자가 직접 작성한다.

### Mapper 인터페이스

Mapper 인터페이스는 Java 메서드와 SQL을 연결하는 역할을 한다.

개발자가 인터페이스에 메서드를 선언하고 `@Select`, `@Insert`, `@Update`, `@Delete` 같은 어노테이션으로 SQL을 붙이면, MyBatis가 실행 시점에 구현체를 만들어 SQL을 실행한다.

```java
@Mapper
public interface StudyLogMapper {
    @Select("SELECT id, title, category, minutes, memo FROM study_logs")
    List<StudyLog> findAll();
}
```

### 파라미터 바인딩

MyBatis의 `#{id}`, `#{title}` 같은 문법은 JDBC의 `?` placeholder와 `statement.setLong(...)`, `statement.setString(...)` 같은 파라미터 바인딩 코드에 대응된다.

```sql
WHERE id = #{id}
```

위 코드는 JDBC에서 아래 코드와 비슷한 역할을 한다.

```java
WHERE id = ?
statement.setLong(1, id);
```

### SQL 어노테이션

`@Select`는 조회 SQL을 실행한다.

`@Insert`는 추가 SQL을 실행한다.

`@Update`는 수정 SQL을 실행한다.

`@Delete`는 삭제 SQL을 실행한다.

각 어노테이션은 Mapper 메서드와 실행할 SQL을 연결한다.

### MyBatis를 써도 직접 작성해야 하는 것

MyBatis를 써도 개발자는 어떤 SQL을 실행할지 직접 작성해야 한다.

또한 어떤 API가 필요한지, 어떤 Mapper 메서드가 필요한지, 파라미터를 어디에 바인딩할지, 조회 결과를 어떤 객체로 받을지, 없는 데이터나 예외 상황을 어떻게 처리할지도 직접 설계해야 한다.

### UPDATE / DELETE의 int 반환

`UPDATE`와 `DELETE`는 영향을 받은 row 수를 반환할 수 있다.

수정 또는 삭제된 row가 있으면 `1`, 대상이 없으면 `0`이 나올 수 있다.

따라서 반환 타입을 `int`로 두면 없는 id를 요청했는지 확인하고 예외 처리를 할 수 있다.

### 다시 볼 포인트

- MyBatis는 SQL을 없애는 도구가 아니다.
- MyBatis는 JDBC의 반복적인 실행 절차를 줄여주는 도구이다.
- Mapper 인터페이스는 Java 메서드와 SQL을 연결하는 입구이다.
- `#{}` 문법은 SQL 파라미터 바인딩에 사용된다.
- 조회 결과 row 수와 Mapper 메서드 반환 타입을 맞춰야 한다.
- 여러 row를 조회하면 `List<StudyLog>`, 한 row를 조회하면 `StudyLog`가 자연스럽다.

## Stage 06-2 MyBatis XML Mapper
날짜: 2026-05-21
분류: Database / MyBatis
상태: 이해 중

### 질문

MyBatis 어노테이션 방식에서 XML Mapper 방식으로 바꾸면 무엇이 달라지는가?

### 지금의 답

어노테이션 방식에서는 Mapper 인터페이스의 메서드 위에 `@Select`, `@Insert`, `@Update`, `@Delete`를 붙이고 괄호 안 문자열에 SQL을 작성했다.

XML Mapper 방식에서는 SQL을 Java 파일 밖의 XML 파일로 분리한다.

Java Mapper 인터페이스에는 반환 타입, 메서드 이름, 파라미터 타입, 파라미터 이름 같은 메서드 선언만 남고, 실제 SQL은 XML Mapper 파일에 작성한다.

### XML Mapper 파일 위치

이번 학습에서는 XML 파일을 아래 위치에 만들었다.

```text
src/main/resources/mapper/StudyLogMapper.xml
```

MyBatis가 이 XML 파일을 찾을 수 있도록 `application.properties`에 위치 설정을 추가했다.

```properties
mybatis.mapper-locations=classpath:mapper/*.xml
```

`classpath:mapper/*.xml`은 `src/main/resources/mapper` 폴더 아래의 XML 파일들을 Mapper XML로 읽으라는 의미이다.

### namespace와 id

XML의 `namespace`는 Java Mapper 인터페이스의 전체 경로와 연결된다.

```xml
<mapper namespace="com.study.stage03.mapper.StudyLogMapper">
```

위 설정은 이 XML 파일이 `com.study.stage03.mapper.StudyLogMapper` 인터페이스와 연결된다는 뜻이다.

XML 내부의 `id`는 Mapper 인터페이스의 메서드 이름과 연결된다.

```xml
<select id="findAll" resultType="com.study.stage03.domain.StudyLog">
    SELECT id, title, category, minutes, memo
    FROM study_logs
</select>
```

위 SQL은 Java Mapper의 아래 메서드와 연결된다.

```java
List<StudyLog> findAll();
```

즉 MyBatis는 `namespace`와 `id`를 조합해서 어떤 Java 메서드가 어떤 SQL을 실행해야 하는지 찾는다.

### XML 태그와 Mapper 메서드

`<select>`는 조회 SQL을 작성할 때 사용한다.

`<insert>`는 추가 SQL을 작성할 때 사용한다.

`<update>`는 수정 SQL을 작성할 때 사용한다.

`<delete>`는 삭제 SQL을 작성할 때 사용한다.

조회 결과를 객체로 받을 때는 `resultType`에 반환할 객체 타입을 작성할 수 있다.

```xml
<select id="findById" resultType="com.study.stage03.domain.StudyLog">
    SELECT id, title, category, minutes, memo
    FROM study_logs
    WHERE id = #{id}
</select>
```

### XML Mapper 방식의 장점

SQL을 Java 코드에서 분리해서 XML 파일에서 관리할 수 있다.

SQL이 길어지거나 조건이 복잡해질 때 어노테이션 방식보다 가독성이 좋고, SQL만 따로 확인하고 관리하기 쉽다.

특히 검색 조건, JOIN, 동적 SQL, 페이징처럼 복잡한 쿼리를 다루는 프로젝트에서 유리하다.

SI나 전자정부 계열 프로젝트에서는 길고 복잡한 업무 SQL을 다루는 경우가 많기 때문에 XML Mapper 방식을 자주 볼 수 있다.

### 다시 볼 포인트

- Java Mapper는 메서드 선언을 담당한다.
- XML Mapper는 실제 SQL을 담당한다.
- MyBatis는 `namespace`와 `id`로 Java 메서드와 XML SQL을 연결한다.
- `mybatis.mapper-locations` 설정은 XML Mapper 파일 위치를 MyBatis에게 알려준다.
- XML Mapper는 SQL이 길어질수록 어노테이션 방식보다 관리하기 쉽다.

## Stage 06-3 MyBatis 동적 SQL
날짜: 2026-05-22
분류: Database / MyBatis
상태: 이해 중

### 질문

검색 조건이 있을 때와 없을 때 SQL을 어떻게 다르게 만들 수 있을까?

### 지금의 답

MyBatis 동적 SQL은 요청 파라미터나 조건 값에 따라 SQL 문장이 달라져야 할 때 사용한다.

예를 들어 `category`가 있으면 category 조건을 붙이고, `title`이 있으면 title 검색 조건을 붙이며, 둘 다 없으면 전체 조회를 할 수 있다.

### Controller와 XML Mapper의 역할 분리

기존에는 Controller에서 category 유무를 직접 분기했다.

```java
if (category == null) {
    return studyLogMapper.findAll();
}

return studyLogMapper.findByCategory(category);
```

동적 SQL을 적용한 뒤에는 Controller가 요청 파라미터를 Mapper에 전달하기만 한다.

```java
return studyLogMapper.search(title, category);
```

검색 조건에 따라 SQL을 조립하는 책임은 XML Mapper가 담당한다.

### if 태그

`<if>` 태그는 `test` 조건이 참일 때만 내부 SQL 조각을 추가한다.

```xml
<if test="title != null and title != ''">
    AND title LIKE CONCAT('%', #{title}, '%')
</if>
```

위 코드는 `title` 값이 있을 때만 title 검색 조건을 SQL에 추가한다.

### where 태그

`<where>` 태그는 내부에 조건이 하나라도 있을 때만 `WHERE`를 자동으로 붙여준다.

또한 조건 맨 앞의 불필요한 `AND`나 `OR`를 자동으로 제거해준다.

```xml
<where>
    <if test="category != null">
        AND category = #{category}
    </if>
    <if test="title != null and title != ''">
        AND title LIKE CONCAT('%', #{title}, '%')
    </if>
</where>
```

조건이 있으면 올바른 `WHERE` 절을 만들고, 조건이 없으면 `WHERE` 자체를 만들지 않는다.

예를 들어 category만 있으면 아래처럼 정리된다.

```sql
WHERE category = ?
```

category와 title이 모두 있으면 아래처럼 정리된다.

```sql
WHERE category = ?
AND title LIKE ?
```

### Param 어노테이션

Mapper 메서드에 파라미터가 두 개 이상 있을 때는 `@Param`으로 이름을 명확하게 지정한다.

```java
List<StudyLog> search(
        @Param("title") String title,
        @Param("category") StudyCategory category
);
```

이렇게 작성하면 XML에서 `#{title}`, `#{category}`로 안정적으로 참조할 수 있다.

### LIKE 검색과 인덱스

`LIKE '%keyword%'`는 일반적인 문자열 인덱스를 효율적으로 쓰기 어려울 수 있다.

문자열 인덱스는 보통 앞부분부터 정렬된 구조를 활용한다.

`LIKE 'keyword%'`처럼 시작 부분이 고정되어 있으면 인덱스를 활용하기 쉽지만, `LIKE '%keyword%'`는 앞에 어떤 문자열이 올지 알 수 없어서 인덱스의 시작 지점을 잡기 어렵다.

따라서 많은 row를 확인해야 할 수 있다.

### 다시 볼 포인트

- 동적 SQL은 조건에 따라 SQL 문장이 달라져야 할 때 사용한다.
- `<if>`는 조건이 참일 때만 SQL 조각을 추가한다.
- `<where>`는 필요한 경우에만 `WHERE`를 붙이고, 앞쪽 `AND`나 `OR`를 정리한다.
- 파라미터가 여러 개면 `@Param`으로 XML에서 사용할 이름을 명시한다.
- Controller는 요청 파라미터를 전달하고, SQL 조건 조립은 XML Mapper가 담당하게 만들 수 있다.
- `LIKE '%keyword%'` 검색은 편하지만 데이터가 많아지면 성능 이슈가 생길 수 있다.
