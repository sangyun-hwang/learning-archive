# Spring

Java Spring 백엔드 학습을 시작하면서 필요한 Spring Boot 기초 개념을 정리합니다.

## Gradle과 build.gradle

날짜: 2026-05-12
분류: Spring / Build Tool
상태: 이해 중

### 질문

Gradle과 `build.gradle`은 각각 어떤 역할인가?

### 짧은 답

Gradle은 Java 프로젝트의 빌드와 의존성 관리를 도와주는 도구이고, `build.gradle`은 이 프로젝트가 어떤 설정과 의존성을 사용하는지 적어두는 명세 파일입니다.

### 내가 이해한 내용

프론트엔드의 npm, yarn, pnpm처럼 Gradle도 프로젝트에서 필요한 의존성을 관리한다.

`build.gradle`은 `package.json`처럼 해당 프로젝트에서 사용하는 의존성을 명시해두는 파일이다. 협업할 때도 이 파일을 보면 프로젝트가 어떤 라이브러리를 필요로 하는지 알 수 있고, Gradle은 이 파일을 기준으로 필요한 의존성을 가져온다.

다만 Gradle은 의존성 관리만 하는 도구는 아니다. Java 프로젝트를 컴파일하고, 테스트를 실행하고, 애플리케이션을 실행하거나 빌드 파일을 만드는 일까지 담당할 수 있는 빌드 자동화 도구다.

### 프론트엔드와 비교

```text
package.json
→ 프로젝트 의존성, 스크립트, 설정을 적는 파일

npm / yarn / pnpm
→ package.json을 보고 의존성을 설치하고 스크립트를 실행하는 도구
```

```text
build.gradle
→ Java/Spring 프로젝트의 의존성, 플러그인, 빌드 설정을 적는 파일

Gradle
→ build.gradle을 보고 의존성을 가져오고, 컴파일/테스트/실행/빌드를 수행하는 도구
```

### 예시

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
}
```

위 설정은 이 프로젝트가 Spring Web과 Validation 기능을 사용한다는 뜻이다.

### 다시 볼 포인트

- Gradle은 의존성 관리 도구이면서 빌드 자동화 도구다.
- `build.gradle`은 Gradle이 읽는 프로젝트 설정 파일이다.
- 프론트엔드의 `package.json`과 비슷하게 의존성 명세 역할을 한다.
- Spring Boot 프로젝트에서 외부 라이브러리를 직접 다운로드하지 않고 Gradle을 통해 관리한다.

## Spring Web Dependency

날짜: 2026-05-12
분류: Spring / Web
상태: 이해 중

### 질문

Spring Web dependency는 왜 필요한가?

### 짧은 답

Spring Web은 Spring Boot 애플리케이션이 HTTP 요청을 받고, Controller 메서드로 연결하고, HTTP 응답을 보낼 수 있게 해주는 웹 기능 묶음입니다.

### 내가 이해한 내용

Spring Web dependency는 HTTP API 서버를 만들기 위한 기능들이 들어 있는 통합 라이브러리처럼 이해할 수 있다.

프론트엔드에서 React가 UI를 만들기 위한 여러 기능을 제공하듯이, Spring Web은 백엔드에서 HTTP API를 만들기 위한 기능을 제공한다. 다만 React는 화면 UI를 만들기 위한 라이브러리이고, Spring Web은 서버에서 HTTP 요청과 응답을 처리하기 위한 웹 기능 묶음이라는 차이가 있다.

Spring Web을 추가하면 아래 같은 어노테이션과 기능을 사용할 수 있다.

```java
@RestController
@GetMapping
@PostMapping
@RequestBody
```

예를 들어 아래 코드는 `GET /health` 요청을 Java 메서드와 연결한다.

```java
@RestController
public class HealthController {

    @GetMapping("/health")
    public String health() {
        return "OK";
    }
}
```

### Tomcat과 Controller의 관계

Spring Web을 사용하면 보통 내장 Tomcat 서버가 함께 사용된다.

처음 이해는 아래처럼 하면 된다.

```text
Tomcat
→ HTTP 요청을 실제로 받아주는 내장 서버

Spring Controller
→ Tomcat이 받은 요청을 Java 메서드로 처리하는 코드
```

흐름으로 보면:

```text
브라우저 또는 API 클라이언트
→ HTTP 요청
→ 내장 Tomcat
→ Spring Web
→ Controller 메서드
→ HTTP 응답
```

Stage 01 콘솔 프로그램과 비교하면:

```text
콘솔 프로그램
→ Scanner가 사용자 입력을 받음
→ Main이 입력을 처리함
```

```text
Spring Boot 웹 프로그램
→ Tomcat/Spring Web이 HTTP 요청을 받음
→ Controller가 요청을 처리함
```

### 다시 볼 포인트

- Spring Web은 HTTP API 서버 기능을 제공한다.
- `@RestController`, `@GetMapping`, `@PostMapping` 같은 기능은 Spring Web 덕분에 사용할 수 있다.
- Tomcat은 요청을 받아주는 서버 역할을 하고, Controller는 요청을 처리하는 Java 코드 역할을 한다.
- Stage 01의 `Scanner`가 콘솔 입력을 받았다면, Stage 03에서는 Spring Web/Tomcat이 HTTP 요청을 받는다.

## Validation Dependency

날짜: 2026-05-12
분류: Spring / Validation
상태: 이해 중

### 질문

Validation dependency는 왜 필요한가?

### 짧은 답

Validation dependency는 클라이언트가 보낸 요청 값이 유효한지 검사하기 위해 사용합니다.

### 내가 이해한 내용

프론트엔드에서 `zod` 같은 라이브러리로 입력값을 검증하듯이, Spring에서도 요청 데이터가 올바른지 검증하는 기능이 필요하다.

다만 Validation은 Spring 자체에 완전히 내장된 기능이라기보다는, Bean Validation/Jakarta Validation 기반의 검증 기능을 Spring이 연결해서 편하게 사용할 수 있게 해주는 방식으로 이해하면 좋다.

예를 들어 Stage 02에서 설계한 `POST /study-logs` 요청에는 아래 규칙이 있었다.

```text
title은 비어 있으면 안 된다.
minutes는 0보다 커야 한다.
category는 정해진 값이어야 한다.
memo는 없어도 된다.
```

Stage 01에서는 이런 검증을 생성자에서 직접 작성했다.

```java
if (title == null || title.isBlank()) {
    throw new IllegalArgumentException("title must not be blank");
}

if (minutes <= 0) {
    throw new IllegalArgumentException("minutes must be greater than 0");
}
```

Spring에서는 나중에 요청 DTO에 어노테이션으로 검증 규칙을 표현할 수 있다.

```java
public class CreateStudyLogRequest {

    @NotBlank
    private String title;

    @Min(1)
    private int minutes;
}
```

### Stage 01과 연결

```text
Stage 01
→ 생성자 안에서 if문으로 직접 검증

Spring
→ Request DTO에 @NotBlank, @Min 같은 어노테이션으로 검증
```

즉 Validation dependency는 Stage 01에서 직접 작성했던 입력값 검증을 Spring API 요청 단계에서 더 선언적으로 표현하게 해준다.

### 다시 볼 포인트

- Validation은 요청 데이터가 올바른지 검사하는 기능이다.
- 프론트엔드의 `zod`처럼 입력값의 조건을 명시하는 역할로 이해할 수 있다.
- Spring에서는 주로 Request DTO에 `@NotBlank`, `@Min`, `@Valid` 같은 어노테이션을 사용한다.
- 생성자 검증은 객체가 잘못된 상태로 만들어지는 것을 막고, Request DTO 검증은 HTTP 요청 값이 잘못 들어오는 것을 먼저 막는다.

## Java 어노테이션

날짜: 2026-05-12
분류: Java / Spring
상태: 이해 중

### 질문

Java 어노테이션은 무엇이고, Spring에서는 어떻게 사용되는가?

### 짧은 답

어노테이션은 클래스, 메서드, 필드, 파라미터에 붙이는 메타데이터입니다. Spring은 실행 과정에서 어노테이션을 읽고, 그 의미에 따라 객체 등록, 요청 매핑, 값 변환, 검증 같은 동작을 수행합니다.

### 내가 이해한 내용

어노테이션은 `@`로 시작하는 표식이다.

```java
@RestController
public class HealthController {
}
```

```java
@GetMapping("/health")
public String health() {
    return "OK";
}
```

Spring은 애플리케이션이 시작될 때 클래스와 메서드를 살펴보고, 어노테이션이 붙은 코드를 특별한 의미로 처리한다.

예를 들어:

```text
@RestController
→ 이 클래스가 REST API 요청을 처리하는 컨트롤러임을 나타낸다.

@GetMapping("/health")
→ GET /health 요청을 이 메서드와 연결한다.
```

즉 어노테이션은 직접 비즈니스 로직을 실행하는 코드라기보다, Spring에게 이 코드의 역할을 알려주는 표시다.

### 어노테이션이 혼자 동작하지 않는다는 뜻

어노테이션은 붙어 있다고 해서 항상 자동으로 동작하는 것이 아니다. 그 어노테이션을 읽고 처리하는 주체가 있어야 한다.

Spring에서는 Spring 프레임워크가 어노테이션을 읽고 처리한다.

예를 들어:

```java
@NotBlank
private String title;
```

이렇게 필드에 검증 어노테이션을 붙였다고 해서 모든 상황에서 Java가 자동으로 검사하는 것은 아니다.

Controller 파라미터에 `@Valid`가 붙고, Validation 기능이 연결되어 있을 때 Spring이 요청 객체 안의 `@NotBlank`, `@Min` 같은 검증 어노테이션을 읽고 검사한다.

```java
public void create(@Valid @RequestBody CreateStudyLogRequest request) {
}
```

이 경우 흐름은 아래처럼 이해할 수 있다.

```text
@RequestBody
→ HTTP 요청 body를 Java 객체로 변환한다.

@Valid
→ 변환된 객체 안의 검증 어노테이션을 실행한다.

@NotBlank, @Min
→ 각 필드의 검증 규칙을 나타낸다.
```

따라서 `@Valid`와 `@NotBlank`는 상위/하위 관계라기보다는, `@Valid`가 요청 객체 검증을 실행하게 만들고 그 과정에서 객체 내부의 검증 어노테이션들이 사용되는 관계로 이해하는 것이 좋다.

### 다시 볼 포인트

- 어노테이션은 클래스, 메서드, 필드, 파라미터에 붙이는 메타데이터다.
- Spring은 어노테이션을 읽고 객체 등록, 요청 매핑, 값 변환, 검증 등을 수행한다.
- 어노테이션 자체가 혼자 동작하는 것이 아니라, Spring 같은 프레임워크가 읽고 처리해야 의미가 있다.
- `@RestController`는 REST API 컨트롤러를 나타낸다.
- `@GetMapping("/health")`는 `GET /health` 요청을 메서드와 연결한다.

## @SpringBootApplication

날짜: 2026-05-12
분류: Spring Boot
상태: 이해 중

### 질문

`@SpringBootApplication`은 어떤 역할을 하는가?

### 짧은 답

`@SpringBootApplication`은 해당 클래스가 Spring Boot 애플리케이션의 시작점임을 나타내는 어노테이션입니다.

### 내가 이해한 내용

React에서 render 함수가 HTML의 특정 지점을 시작점으로 삼아 화면을 구성하듯이, Spring Boot에서는 `@SpringBootApplication`이 붙은 클래스를 애플리케이션 시작점으로 이해할 수 있다.

다만 React가 DOM을 구성하는 것과 달리, Spring Boot는 시작 클래스를 기준으로 패키지를 스캔하고 필요한 객체를 Spring의 관리 공간에 등록한다. 이 관리 공간을 Spring에서는 ApplicationContext라고 부른다.

예시:

```java
@SpringBootApplication
public class Stage03SpringCrudApplication {

    public static void main(String[] args) {
        SpringApplication.run(Stage03SpringCrudApplication.class, args);
    }
}
```

### SpringApplication.run(...)

`SpringApplication.run(...)`이 실행되면 대략 아래 과정이 진행된다.

```text
1. Spring Boot 애플리케이션 시작
2. 초기 설정 로딩
3. Component Scan 실행
4. 어노테이션이 붙은 클래스 탐색
5. 필요한 객체 생성 및 ApplicationContext에 등록
6. 내장 Tomcat 서버 실행
7. localhost:8080에서 HTTP 요청 대기
```

처음에는 이 과정을 아래처럼 이해할 수 있다.

```text
Spring Boot 실행
→ 설정 읽기
→ 어노테이션 붙은 클래스 찾기
→ 필요한 객체 등록
→ 서버 실행
→ 요청 대기
```

### Component Scan과 패키지 위치

Spring Boot는 `@SpringBootApplication`이 붙은 클래스가 있는 패키지를 기준으로 하위 패키지를 스캔한다.

예를 들어 시작 클래스가 아래 위치에 있으면:

```text
com.study.stage03.Stage03SpringCrudApplication
```

Spring은 기본적으로 아래 패키지와 그 하위 패키지를 살펴본다.

```text
com.study.stage03
```

좋은 구조:

```text
com.study.stage03
├── Stage03SpringCrudApplication
├── HealthController
├── StudyLogController
└── StudyLogService
```

이렇게 두면 Controller나 Service가 정상적으로 스캔될 가능성이 높다.

반대로 시작 클래스 기준의 상위 패키지나 완전히 다른 패키지에 클래스를 두면 Spring이 스캔하지 못할 수 있다.

### 다시 볼 포인트

- `@SpringBootApplication`은 Spring Boot 애플리케이션의 시작점을 나타낸다.
- `SpringApplication.run(...)`이 실제 애플리케이션 실행을 시작한다.
- 실행 과정에서 Component Scan을 통해 어노테이션이 붙은 클래스를 찾는다.
- 찾은 클래스들은 Spring의 ApplicationContext에 등록되어 관리된다.
- Controller, Service 같은 클래스는 시작 클래스와 같은 패키지 또는 하위 패키지에 두는 것이 좋다.

## Controller, @RestController, @GetMapping

날짜: 2026-05-12
분류: Spring / Web
상태: 이해 중

### 질문

Spring에서 Controller는 어떤 역할을 하고, `@RestController`와 `@GetMapping`은 무엇을 의미하는가?

### 짧은 답

Controller는 HTTP 요청을 받아 적절한 Java 메서드로 처리하고 응답을 반환하는 계층입니다. `@RestController`는 REST API 컨트롤러임을 나타내고, `@GetMapping`은 GET 요청 경로와 메서드를 연결합니다.

### 내가 이해한 내용

Controller는 HTTP API 요청을 처리하는 클래스다.

요청의 HTTP method와 URL에 따라 어떤 메서드가 실행될지 결정되는데, 이 매핑은 Spring이 어노테이션을 읽어 처리한다. Controller는 그렇게 매핑된 요청을 실제 Java 메서드로 처리하는 역할을 한다.

예시:

```java
@RestController
public class HealthController {

    @GetMapping("/health")
    public String health() {
        return "OK";
    }
}
```

이 코드는 아래 의미를 가진다.

```text
@RestController
→ 이 클래스가 REST API 요청을 처리하는 컨트롤러임을 나타낸다.
→ 메서드의 반환값을 HTTP 응답 body로 보낸다.

@GetMapping("/health")
→ GET /health 요청을 health() 메서드와 연결한다.
```

요청 흐름:

```text
GET /health 요청
→ Spring이 @GetMapping("/health")를 보고 health() 메서드와 연결
→ health() 실행
→ "OK" 반환
→ HTTP 응답 body에 OK 전달
```

### Controller와 Service의 역할 분리

Stage 01에서는 `Main`이 사용자 입력과 출력 흐름을 담당했고, `StudyLogManager`가 공부 기록 관리 로직을 담당했다.

Spring에서는 이 역할이 아래처럼 이어진다.

```text
Controller
→ HTTP 요청과 응답 처리

Service
→ 비즈니스 로직 처리
```

즉 Controller는 HTTP 요청을 받고 응답을 만드는 입구 역할을 하고, 실제 공부 기록 생성/조회/계산 같은 로직은 Service로 분리하는 것이 좋다.

### 다시 볼 포인트

- Controller는 HTTP 요청을 처리하는 계층이다.
- 실제 요청 매핑은 Spring이 어노테이션을 읽어 처리한다.
- `@RestController`는 REST API 컨트롤러를 나타낸다.
- `@RestController`가 있으면 메서드 반환값이 HTTP 응답 body로 전달된다.
- `@GetMapping("/health")`는 GET `/health` 요청과 메서드를 연결한다.
- Controller는 HTTP 요청/응답을 담당하고, Service는 비즈니스 로직을 담당한다.

## Spring Boot DevTools와 서버 재시작

날짜: 2026-05-13
분류: Spring Boot / Development
상태: 이해 중

### 질문

Spring Boot는 프론트엔드 개발 서버처럼 코드 수정이 바로 반영되는가?

### 짧은 답

기본 Spring Boot 실행은 Java 코드 수정이 실행 중인 서버에 바로 반영되지 않는다. 보통 서버를 재시작해야 변경 내용이 적용된다.

### 내가 이해한 내용

프론트엔드의 Vite나 Next.js 개발 서버는 파일 변경을 감지하고 브라우저에 빠르게 반영해준다.

```text
React / Vite
→ 파일 저장
→ HMR 또는 자동 새로고침
```

반면 기본 Spring Boot 서버는 Java 애플리케이션이 이미 실행 중인 상태이므로, 코드를 수정해도 실행 중인 서버에는 바로 반영되지 않는다.

```text
Spring Boot 기본 실행
→ Java 코드 수정
→ 실행 중인 서버에는 바로 반영되지 않음
→ 서버 재시작 필요
```

자동 재시작이 필요하면 `Spring Boot DevTools`를 사용할 수 있다.

```gradle
developmentOnly 'org.springframework.boot:spring-boot-devtools'
```

다만 DevTools는 프론트엔드 HMR처럼 화면 일부를 즉시 교체하는 느낌이라기보다, 변경을 감지해서 Spring Boot 애플리케이션을 빠르게 재시작해주는 기능에 가깝다.

### 다시 볼 포인트

- 기본 Spring Boot 실행은 코드 수정 후 서버 재시작이 필요하다.
- `405 Method Not Allowed` 같은 결과가 이전 코드 기준으로 나올 때는 서버가 예전 코드로 떠 있는지 확인한다.
- 자동 재시작이 필요하면 Spring Boot DevTools를 사용할 수 있다.
- DevTools는 프론트엔드 HMR보다는 자동 서버 재시작에 가깝다.
## Validation 실패와 전역 예외 처리
날짜: 2026-05-13
분류: Spring / Exception Handling
상태: 이해 중

### 질문

`@Valid` 검증에 실패했을 때 기본 에러 응답 대신 직접 만든 JSON 응답으로 바꾸려면 어떻게 해야 하는가?

### 지금의 답

Spring에서는 컨트롤러에서 발생한 예외를 `@RestControllerAdvice`가 붙은 클래스에서 한곳에 모아 처리할 수 있다. `@Valid` 검증 실패는 `MethodArgumentNotValidException`으로 전달되고, 이 예외를 `@ExceptionHandler`로 잡아서 원하는 응답 DTO로 바꿔 반환할 수 있다.

### 내가 이해한 내용

요청 DTO에 `@NotBlank`, `@NotNull`, `@Min` 같은 검증 어노테이션을 붙이고, 컨트롤러의 `@RequestBody` 앞에 `@Valid`를 붙이면 Spring이 요청 값을 검사한다.

```java
public StudyLog createStudyLog(@Valid @RequestBody CreateStudyLogRequest request) {
}
```

검증에 실패하면 컨트롤러 메서드 본문이 정상 실행되는 것이 아니라, Spring 내부에서 `MethodArgumentNotValidException`이 발생한다. 이 예외를 전역 예외 처리 클래스에서 잡으면 기본 에러 응답 대신 내가 만든 응답 형태를 내려줄 수 있다.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(
            MethodArgumentNotValidException exception
    ) {
        // 검증 실패 정보를 꺼내 ErrorResponse로 변환한다.
    }
}
```

### 주요 어노테이션

`@RestControllerAdvice`는 여러 컨트롤러에서 발생하는 예외를 한곳에서 처리하기 위한 클래스임을 나타낸다.

`@ExceptionHandler(MethodArgumentNotValidException.class)`는 `MethodArgumentNotValidException` 예외가 발생했을 때 해당 메서드가 실행되게 연결한다.

`@Valid`는 요청 DTO 안에 붙어 있는 검증 어노테이션을 실제로 검사하게 만든다.

### ResponseEntity

`ResponseEntity<T>`는 응답 body뿐 아니라 HTTP 상태 코드까지 직접 정할 수 있는 Spring의 응답 객체이다.

```java
return ResponseEntity.badRequest().body(errorResponse);
```

위 코드는 HTTP 상태 코드를 `400 Bad Request`로 설정하고, 응답 body에는 `errorResponse`를 담아 보내겠다는 뜻이다.

### 검증 실패 흐름

```text
POST /study-logs 요청
-> @RequestBody가 JSON을 CreateStudyLogRequest 객체로 변환
-> @Valid가 DTO의 검증 어노테이션 확인
-> 검증 실패 시 MethodArgumentNotValidException 발생
-> @ExceptionHandler가 예외를 잡음
-> ErrorResponse DTO로 변환
-> 400 Bad Request JSON 응답
```

### 다시 볼 포인트

- 검증 어노테이션은 DTO 필드에 요청 값의 조건을 표현한다.
- `@Valid`가 있어야 DTO 안의 검증 조건이 실제 요청 처리 중 실행된다.
- 검증 실패는 일반 반환값이 아니라 예외 흐름으로 넘어간다.
- `@RestControllerAdvice`는 여러 컨트롤러의 예외 처리를 한곳에 모으는 역할을 한다.
- `@ExceptionHandler`는 특정 예외와 처리 메서드를 연결한다.
- `ResponseEntity`를 쓰면 HTTP 상태 코드와 응답 body를 함께 직접 정할 수 있다.
## Controller, Service, Repository 책임 분리
날짜: 2026-05-13
분류: Spring / Architecture
상태: 이해 중

### 질문

Spring에서 Controller, Service, Repository는 각각 어떤 역할을 담당하는가?

### 지금의 답

Controller는 HTTP 요청과 응답을 담당하고, Service는 애플리케이션의 주요 로직을 처리하며, Repository는 데이터 저장소와 직접 대화하는 역할을 한다.

### 내가 이해한 내용

처음에는 Controller에 요청 처리, 데이터 생성, 목록 관리, 필터링, 합계 계산이 모두 들어가 있었다. 기능이 작을 때는 동작하지만, 코드가 커지면 한 클래스가 너무 많은 책임을 가지게 된다.

Spring에서는 보통 역할을 다음처럼 나눈다.

```text
Controller
-> HTTP 세계 담당
-> URL, HTTP Method, QueryParam, RequestBody, Valid, 응답 반환

Service
-> 애플리케이션 로직 담당
-> 요청 DTO를 도메인 객체로 바꾸기
-> id 계산하기
-> 필터링하기
-> 총합 계산하기
-> 어떤 Repository 메서드를 쓸지 결정하기

Repository
-> 데이터 저장소 담당
-> 저장하기
-> 전체 조회하기
-> 나중에는 DB에서 조회하기
```

짧게 정리하면 다음과 같다.

```text
Controller는 요청을 받는다.
Service는 일을 판단하고 처리한다.
Repository는 데이터를 넣고 꺼낸다.
```

### Repository에 대한 이해

Repository는 단순히 파일을 수정하는 계층이라기보다, 데이터가 저장되는 곳과 직접 대화하는 계층이다.

지금 Stage 03에서는 실제 DB가 없기 때문에 Repository가 메모리 `List<StudyLog>`를 관리한다.

```text
현재
Repository -> 메모리 List
```

나중에 DB를 배우면 Repository가 DB와 대화하는 역할로 바뀐다.

```text
나중
Repository -> Database
```

이렇게 분리해두면 Service는 “무슨 일을 할지”에 집중하고, Repository는 “데이터를 어디에서 가져오고 어디에 저장할지”에 집중할 수 있다.

### 다시 볼 포인트

- Controller는 HTTP 요청과 응답을 다루는 입구이다.
- Service는 기능의 흐름과 비즈니스 로직을 담당한다.
- Repository는 데이터 저장소와 직접 대화한다.
- DTO를 도메인 객체로 바꾸거나 id를 계산하는 로직은 Service에 두는 것이 자연스럽다.
- Repository는 `CreateStudyLogRequest` 같은 요청 DTO를 몰라도 된다.
- Repository의 `save()`는 저장할 대상인 `StudyLog`를 받는 편이 책임 분리에 맞다.
## Stage 03 CRUD 마무리
날짜: 2026-05-13
분류: Spring / CRUD
상태: 이해 중

### 질문

Spring Boot로 메모리 기반 CRUD API를 만들 때 전체 흐름은 어떻게 나뉘는가?

### 지금의 답

Stage 03에서는 `StudyLog`를 대상으로 생성, 조회, 수정, 삭제 API를 만들면서 Controller, Service, Repository, DTO, Exception Handler의 역할을 나누는 연습을 했다.

### 만든 API

```text
POST   /study-logs          학습 기록 생성
GET    /study-logs          학습 기록 목록 조회
GET    /study-logs/{id}     학습 기록 단건 조회
PATCH  /study-logs/{id}     학습 기록 부분 수정
DELETE /study-logs/{id}     학습 기록 삭제
GET    /study-logs/summary  카테고리별 학습 시간 합계
```

### 계층별 흐름

```text
Controller
-> HTTP 요청을 받는다.
-> URL, Method, PathVariable, RequestParam, RequestBody, Valid를 다룬다.
-> Service를 호출하고 응답을 반환한다.

Service
-> 실제 기능 흐름을 처리한다.
-> DTO를 StudyLog 도메인 객체로 바꾼다.
-> 없는 데이터인지 판단하고 예외를 던진다.
-> 필터링, 합계 계산, 부분 수정 값을 결정한다.

Repository
-> 데이터를 저장하고 꺼낸다.
-> 지금은 메모리 List를 사용한다.
-> 나중에는 DB와 직접 대화하는 계층이 된다.
```

### 예외 처리 흐름

단건 조회, 수정, 삭제에서 없는 id를 요청하면 Service의 `findById(id)`가 `StudyLogNotFoundException`을 던진다.

```text
Repository.findById(id)
-> 없으면 null
-> Service.findById(id)가 null을 예외로 변환
-> GlobalExceptionHandler가 404 Not Found 응답으로 변환
```

즉, Repository는 단순 조회를 하고, Service가 그 결과의 의미를 해석한다.

### 부분 수정 PATCH

처음 만든 PATCH는 모든 필드를 다 받아서 새 `StudyLog`로 교체하는 방식이었다. 이후에는 일부 필드만 보내도 기존 값이 유지되도록 수정했다.

```json
{
  "minutes": 90
}
```

위 요청에서는 `minutes`만 바뀌고, `title`, `category`, `memo`는 기존 값을 유지한다.

부분 수정에서는 "값을 안 보낸 것"과 "값을 보냈는데 값이 비어 있거나 잘못된 것"을 구분해야 한다. `int`는 값이 없으면 `0`이 되어버리기 때문에, 요청 DTO에서는 `Integer`를 사용해 `null`을 표현할 수 있게 했다.

```text
int
-> null 불가
-> 값이 없으면 0처럼 기본값으로 처리될 수 있음

Integer
-> null 가능
-> 요청에 값이 없는 상태를 표현할 수 있음
```

### 다시 볼 포인트

- `@PostMapping`, `@GetMapping`, `@PatchMapping`, `@DeleteMapping`은 HTTP Method와 Java 메서드를 연결한다.
- `@PathVariable`은 URL 경로의 값을 Java 파라미터로 받는다.
- `@RequestParam`은 query parameter를 받는다.
- `@RequestBody`는 JSON body를 Java DTO로 변환한다.
- `@Valid`는 DTO의 검증 어노테이션을 실행한다.
- `ResponseEntity`를 쓰면 상태 코드와 응답 body를 직접 정할 수 있다.
- 같은 이름의 메서드라도 `studyLogRepository.findById(id)`와 `findById(id)`는 서로 다른 클래스의 메서드일 수 있다.
- Service 안에서 `findById(id)`처럼 쓰면 `this.findById(id)`와 같은 의미이다.
