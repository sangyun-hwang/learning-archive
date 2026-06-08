# Spring SI / 전자정부 구조 감각

날짜: 2026-06-08
분류: Backend / Spring MVC / JSP / MyBatis / SI
상태: 이해 중

## 전자정부프레임워크 이해

전자정부프레임워크는 완전히 새로운 기술이라기보다 Spring 기반 위에 공공 프로젝트에서 자주 쓰는 표준 구조, 공통 기능, 개발 가이드를 얹은 프레임워크에 가깝다.

SI/전자정부 계열에서는 빠르게 바뀌는 최신 기술보다 오래 검증되고 유지보수 인력이 많은 기술 조합을 선호하는 경우가 많다.

자주 보이는 조합은 다음과 같다.

```text
Java
Spring MVC
JSP / JSTL
MyBatis XML Mapper
Oracle / MySQL
Tomcat
전자정부프레임워크
```

## JSP + MyBatis 조합이 자주 보이는 이유

JSP와 MyBatis는 오래 검증된 조합이고, 복잡한 업무 SQL을 직접 관리하기 좋다.

MyBatis XML Mapper는 SQL을 Java 코드 밖으로 분리해서 관리할 수 있기 때문에 긴 조회 조건, 검색, 페이징, 정렬, 통계성 쿼리 같은 업무 쿼리에 어울린다.

JSP는 서버가 Model 데이터를 이용해 HTML을 만들어 응답하는 방식이므로, 별도 프론트엔드 앱과 REST API 계약을 나누지 않아도 화면을 구성할 수 있다.

## Spring MVC + JSP 요청 흐름

Spring MVC + JSP 흐름은 다음처럼 설명할 수 있다.

```text
브라우저 요청
-> Spring Security
-> Controller
-> Service
-> Mapper
-> Mapper XML
-> DB
-> Model
-> JSP
-> HTML 응답
```

예를 들어 다음 요청이 들어온다.

```text
GET /mvc/study-logs?title=java&page=1&size=10
```

흐름은 다음과 같다.

1. 브라우저가 title, page, size를 query parameter로 담아 요청한다.
2. Spring Security가 `/mvc/study-logs/**` 경로에 인증이 필요한지 확인한다.
3. Controller가 query parameter와 Authentication을 받는다.
4. Controller가 현재 로그인 사용자 정보를 UserMapper로 조회한다.
5. StudyLogMapper가 userId, 검색어, page, size를 기준으로 Mapper XML의 SQL을 실행한다.
6. DB가 현재 사용자의 검색 결과와 전체 개수를 반환한다.
7. Controller가 logs, page, totalPages, startPage, endPage 등을 Model에 담는다.
8. JSP가 Model 데이터를 출력하고 `<c:forEach>`로 목록을 반복 렌더링한다.
9. 서버가 완성된 HTML을 브라우저에 응답한다.

## 파일별 역할

`StudyLogPageController.java`는 StudyLog JSP 화면 요청을 처리하는 Controller다. 목록, 생성, 수정, 삭제 요청을 받고 Model에 데이터를 담아 JSP view 이름을 반환한다.

`SignupService.java`는 회원가입 비즈니스 로직을 담당한다. 중복 username 확인, 비밀번호 해싱, 기본 role/enabled 설정 같은 규칙을 처리한다.

`StudyLogMapper.java`는 Java 쪽 DB 접근 인터페이스다. Controller나 Service는 SQL을 직접 호출하지 않고 Mapper 메서드를 호출한다.

`StudyLogMapper.xml`은 실제 SQL을 관리한다. Mapper 인터페이스의 메서드 이름과 XML의 `id`가 연결된다.

`list.jsp`는 목록 화면 View다. Model에 담긴 `logs`, `page`, `totalPages`, `title`, `category` 등을 출력한다.

`SecurityConfig.java`는 Spring Security 보안 규칙 설정 파일이다. 로그인 없이 접근 가능한 URL, 로그인해야 접근 가능한 URL, 로그인/로그아웃 URL을 설정한다.

`DbUserDetailsService.java`는 Spring Security가 로그인할 때 DB에서 사용자 정보를 조회하도록 연결하는 클래스다. `UserMapper`로 사용자를 찾고, 그 결과를 Security가 이해하는 `UserDetails`로 바꾼다.

## 실무식 용어

| 쉬운 표현 | 실무식 표현 |
| --- | --- |
| 화면을 보여주는 파일 | JSP View, View Template, 화면 JSP |
| 사용자가 누르면 실행되는 주소 | URL, Endpoint, Controller Mapping |
| DB에서 데이터를 가져오는 함수 | Mapper Method, DAO Method, Repository Method |
| SQL을 모아둔 파일 | MyBatis Mapper XML, SQL Mapper XML |
| 로그인했는지 확인하는 설정 | Spring Security 설정, 인증/인가 설정, SecurityFilterChain 설정 |
| 페이지에 데이터를 넘겨주는 객체 | Model, Model 객체 |
| 사용자가 입력한 form 값을 담는 객체 | DTO, Request DTO, Form DTO, Command Object |
| 비즈니스 규칙을 처리하는 클래스 | Service, Service Class, Business Logic Layer |

## React + REST API 방식과 JSP + Spring MVC 방식의 차이

React + REST API 방식은 프론트엔드와 백엔드가 분리되어 JSON으로 데이터를 주고받는다. 프론트엔드는 받은 JSON을 상태로 관리하고 화면을 렌더링한다.

JSP + Spring MVC 방식은 서버가 데이터를 조회하고 JSP로 HTML까지 만들어서 응답한다. 화면과 서버 흐름이 더 붙어 있고, 서버 사이드 렌더링 방식으로 이해할 수 있다.

## 지원서 표현 초안

Java/Spring 기반 백엔드 학습 프로젝트로 Study Tracker를 구현하며 Spring MVC, JSP, MyBatis, MySQL, Spring Security를 사용했습니다. 학습 기록 CRUD를 시작으로 검색/페이징/정렬, 회원가입/로그인, 사용자별 데이터 소유권을 단계적으로 구현했고, React + REST API 방식과 JSP + Spring MVC 방식의 차이를 비교하며 서버 사이드 렌더링 기반 업무형 웹 애플리케이션 구조를 학습했습니다.

## 다시 볼 포인트

- 전자정부프레임워크는 Spring 기반 표준 구조와 공통 기능에 가깝다.
- JSP + MyBatis 조합은 오래 검증되었고 업무 SQL 관리에 강하다.
- SI에서는 표준 구조가 유지보수와 인수인계에 중요하다.
- Controller, Service, Mapper, Mapper XML, JSP의 책임을 구분해서 설명할 수 있어야 한다.
- 지원서에서는 단순히 기술명을 나열하기보다 구현한 기능과 배운 구조를 함께 설명한다.
