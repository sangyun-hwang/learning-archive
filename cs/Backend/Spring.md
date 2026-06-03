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

## Stage 07 Spring MVC와 JSP
날짜: 2026-05-23
분류: Spring / MVC / JSP
상태: 이해 중

### 질문

REST API처럼 JSON을 응답하는 방식과, Spring MVC + JSP처럼 서버에서 HTML 화면을 만들어 응답하는 방식은 어떻게 다를까?

### RestController와 Controller

`@RestController`는 메서드 반환값을 HTTP Response Body로 응답한다.

객체를 반환하면 JSON으로 변환된다.

`@Controller`는 MVC 화면 응답에 사용한다. 문자열을 반환하면 그 문자열을 view 이름으로 해석한다.

```java
@Controller
public class StudyLogPageController {
    @GetMapping("/mvc/study-logs")
    public String studyLogListPage(Model model) {
        return "study-log/list";
    }
}
```

### View 경로 설정

JSP를 찾기 위해 `application.properties`에 view prefix와 suffix를 설정했다.

```properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

Controller가 아래 문자열을 반환하면:

```java
return "study-log/list";
```

Spring MVC는 실제 JSP 파일을 아래 경로로 찾는다.

```text
/WEB-INF/views/study-log/list.jsp
```

프로젝트 파일 위치는 아래와 같다.

```text
src/main/webapp/WEB-INF/views/study-log/list.jsp
```

### Model

`Model`은 Controller가 JSP에 데이터를 전달하기 위한 상자 역할을 한다.

Controller는 `model.addAttribute(...)`로 데이터를 담고, JSP는 EL 문법으로 값을 꺼내 쓴다.

```java
model.addAttribute("message", "Model data is working.");
model.addAttribute("logs", studyLogMapper.search(title, category));
```

```jsp
${message}
${logs}
```

### JSP 목록 출력

Controller에서 `logs`라는 이름으로 학습 기록 목록을 Model에 담았다.

JSP에서는 JSTL의 `<c:forEach>`로 목록을 반복 출력했다.

```jsp
<c:forEach var="log" items="${logs}">
    <tr>
        <td>${log.id}</td>
        <td>${log.title}</td>
        <td>${log.category}</td>
        <td>${log.minutes}</td>
        <td>${log.memo}</td>
    </tr>
</c:forEach>
```

### JSP 검색 폼

검색 폼은 GET 방식으로 `/mvc/study-logs`에 요청을 보낸다.

```jsp
<form method="get" action="/mvc/study-logs">
```

`method="get"`은 form 값을 URL query parameter로 붙여 GET 요청을 보내겠다는 뜻이다.

`action="/mvc/study-logs"`는 form submit 시 요청을 보낼 주소이다.

input과 select의 `name`은 query parameter 이름이 된다.

```jsp
<input type="text" name="title" value="${title}">
<select name="category">
```

검색 버튼을 누르면 브라우저는 form 안의 `name`과 값을 모아서 아래와 같은 URL을 만든다.

```text
/mvc/study-logs?title=xml&category=SPRING
```

Controller는 `@RequestParam`으로 query parameter 값을 받는다.

```java
@GetMapping("/mvc/study-logs")
public String studyLogListPage(
        @RequestParam(required = false) String title,
        @RequestParam(required = false) StudyCategory category,
        Model model
) {
    model.addAttribute("logs", studyLogMapper.search(title, category));
    model.addAttribute("title", title);
    model.addAttribute("category", category);
    return "study-log/list";
}
```

### 검색 조건 유지

검색 후에도 사용자가 입력한 조건을 화면에 다시 보여주기 위해 `title`과 `category`를 Model에 다시 담았다.

```java
model.addAttribute("title", title);
model.addAttribute("category", category);
```

JSP에서는 title input의 value에 `${title}`을 넣어 검색어를 유지했다.

category select는 현재 category와 option 값을 비교해서 `selected`를 출력했다.

```jsp
<option value="SPRING" ${category == 'SPRING' ? 'selected' : ''}>SPRING</option>
```

### 빈 결과 처리

검색 결과가 없을 때는 JSTL의 `<c:if>`와 `${empty logs}`를 사용했다.

```jsp
<c:if test="${empty logs}">
    <tr>
        <td colspan="5">No study logs found.</td>
    </tr>
</c:if>
```

`${empty logs}`는 `logs`가 null이거나 비어 있는 컬렉션일 때 true가 된다.

### REST API + React와 MVC + JSP 비교

REST API + React 방식은 백엔드가 JSON을 응답하고, React가 브라우저에서 그 데이터를 받아 화면을 만든다.

Spring MVC + JSP 방식은 Controller가 Model에 데이터를 담고, JSP가 서버에서 HTML을 만든 뒤 브라우저에 응답한다.

가장 큰 차이는 HTML을 어디서 만드느냐이다.

```text
REST API + React
-> 브라우저/React가 화면 생성

Spring MVC + JSP
-> 서버/JSP가 화면 생성
```

### 다시 볼 포인트

- `@RestController`는 Response Body를 응답하고, `@Controller`는 view 이름을 반환할 수 있다.
- Controller의 view 이름은 prefix와 suffix를 통해 실제 JSP 경로로 해석된다.
- `Model`은 Controller가 JSP에 데이터를 전달하는 객체이다.
- JSP의 `${...}`는 Model에 담긴 값을 꺼내는 EL 문법이다.
- `<c:forEach>`는 JSP에서 목록을 반복 출력할 때 사용한다.
- GET form submit은 input/select의 `name`과 값을 query parameter로 만든다.
- Spring MVC + JSP는 서버에서 HTML을 만들어 응답하는 SSR 방식이다.

## Stage 07 JSP 생성 폼과 redirect
날짜: 2026-05-24
분류: Spring / MVC / JSP
상태: 이해 중

### 질문

JSP form에서 입력한 데이터를 Spring MVC Controller로 보내고 DB에 저장하려면 어떤 흐름을 거칠까?

### GET 화면 요청과 POST 처리 요청

`GET /mvc/study-logs/new`는 새 학습 기록을 작성할 JSP form 화면을 보여준다.

`POST /mvc/study-logs`는 form에서 제출한 데이터를 받아 새 학습 기록을 생성한다.

```text
GET /mvc/study-logs/new
-> 작성 화면 조회

POST /mvc/study-logs
-> 작성 데이터 처리
-> DB 저장
```

`POST /mvc/study-logs`는 REST API라기보다는 JSP form 제출을 처리하는 MVC Controller 요청이다.

### JSP 작성 폼

작성 폼은 POST 방식으로 `/mvc/study-logs`에 데이터를 보낸다.

```jsp
<form method="post" action="/mvc/study-logs">
    <input type="text" name="title">
    <select name="category">
        <option value="JAVA">JAVA</option>
        <option value="SPRING">SPRING</option>
        <option value="DATABASE">DATABASE</option>
    </select>
    <input type="number" name="minutes">
    <textarea name="memo"></textarea>
    <button type="submit">Create</button>
</form>
```

`method="post"`는 form submit 시 POST 요청을 보내겠다는 뜻이다.

`action="/mvc/study-logs"`는 form submit 시 요청을 보낼 URL이다.

### ModelAttribute

`@ModelAttribute`는 form에서 전달된 request parameter들을 Java 객체로 묶어준다.

form input/select/textarea의 `name`과 DTO의 필드 이름을 기준으로 값을 바인딩한다.

```java
@PostMapping("/mvc/study-logs")
public String createStudyLog(@ModelAttribute CreateStudyLogRequest request) {
    StudyLog studyLog = new StudyLog(
            studyLogMapper.getNextId(),
            request.getTitle(),
            request.getCategory(),
            request.getMinutes(),
            request.getMemo()
    );

    studyLogMapper.save(studyLog);

    return "redirect:/mvc/study-logs";
}
```

예를 들어 form의 `name="title"` 값은 request parameter `title`로 전달되고, Spring MVC가 `CreateStudyLogRequest`의 `title` 필드에 바인딩한다.

### setter가 필요했던 이유

`@ModelAttribute`가 form 데이터를 DTO 객체에 넣을 때 setter를 통해 값을 주입하려고 한다.

setter가 없으면 `title`, `category`, `minutes`, `memo` 값이 DTO에 제대로 들어가지 않아 null 또는 기본값으로 남을 수 있다.

그래서 `CreateStudyLogRequest`에 setter를 추가했다.

```java
public void setTitle(String title) {
    this.title = title;
}
```

### redirect

```java
return "redirect:/mvc/study-logs";
```

위 코드는 POST 요청 처리 후 브라우저에게 `/mvc/study-logs`로 다시 요청하라고 응답한다.

즉 저장 후 목록 화면으로 이동하게 만든다.

POST 처리 후 redirect하면 마지막 브라우저 요청이 `GET /mvc/study-logs`가 된다.

그래서 사용자가 목록 화면에서 새로고침해도 POST 요청이 다시 실행되지 않고, GET 목록 조회만 다시 실행된다.

이 방식은 Post/Redirect/Get 패턴이다.

### 다시 볼 포인트

- GET `/new`는 작성 화면을 보여준다.
- POST `/mvc/study-logs`는 form 데이터를 받아 저장한다.
- JSP form의 `name`은 request parameter 이름이 된다.
- `@ModelAttribute`는 form parameter를 DTO 객체로 묶어준다.
- form 바인딩을 위해 DTO에 setter가 필요할 수 있다.
- 저장 후에는 `redirect:/mvc/study-logs`로 목록 화면을 다시 요청하게 만든다.
- redirect를 사용하면 새로고침으로 인한 중복 POST 제출을 줄일 수 있다.

## Stage 07 JSP 수정/삭제 흐름
날짜: 2026-05-26
분류: Spring / MVC / JSP
상태: 이해 중

### 질문

HTML form 기반 JSP MVC에서는 학습 기록 수정과 삭제를 어떻게 처리할 수 있을까?

### 수정 화면 요청

`GET /mvc/study-logs/{id}/edit`는 수정할 데이터를 id로 조회해서 수정 폼 화면에 보여주는 요청이다.

이 요청은 데이터를 수정하는 요청이 아니라, 수정 화면을 열기 위한 GET 요청이다.

```java
@GetMapping("/mvc/study-logs/{id}/edit")
public String editStudyLogPage(@PathVariable Long id, Model model) {
    StudyLog studyLog = studyLogMapper.findById(id);

    if (studyLog == null) {
        throw new StudyLogNotFoundException();
    }

    model.addAttribute("log", studyLog);

    return "study-log/edit";
}
```

JSP는 Model에 담긴 `log`를 사용해 기존 값을 폼에 채운다.

```jsp
<input type="text" name="title" value="${log.title}">
<input type="number" name="minutes" value="${log.minutes}">
```

### 수정 처리 요청

`POST /mvc/study-logs/{id}/edit`는 수정 폼에서 제출한 데이터를 받아 해당 id의 학습 기록을 수정하는 요청이다.

```java
@PostMapping("/mvc/study-logs/{id}/edit")
public String updateStudyLog(
        @PathVariable Long id,
        @ModelAttribute UpdateStudyLogRequest request
) {
    StudyLog studyLog = studyLogMapper.findById(id);

    if (studyLog == null) {
        throw new StudyLogNotFoundException();
    }

    studyLogMapper.updatePartial(id, request);

    return "redirect:/mvc/study-logs";
}
```

수정 처리 후에는 목록 화면으로 redirect한다.

### 삭제 버튼

삭제는 데이터를 변경하는 요청이므로 `<a>` 링크보다 POST form으로 처리하는 것이 더 적절하다.

`<a>` 링크는 기본적으로 GET 요청을 만든다. GET은 조회 성격의 요청으로 보는 것이 자연스럽고, 링크 클릭이나 크롤러 접근만으로 데이터가 삭제되면 위험할 수 있다.

그래서 목록 화면에서는 삭제 버튼을 POST form으로 만들었다.

```jsp
<form method="post" action="/mvc/study-logs/${log.id}/delete" style="display:inline;">
    <button type="submit">Delete</button>
</form>
```

### 삭제 처리 요청

`POST /mvc/study-logs/{id}/delete`는 삭제 요청을 받아 DB에서 해당 학습 기록을 삭제한다.

```java
@PostMapping("/mvc/study-logs/{id}/delete")
public String deleteStudyLog(@PathVariable Long id) {
    int deleteRows = studyLogMapper.delete(id);

    if (deleteRows == 0) {
        throw new StudyLogNotFoundException();
    }

    return "redirect:/mvc/study-logs";
}
```

삭제 후에도 목록 화면으로 redirect한다.

### REST API와 JSP MVC form 처리 비교

REST API 방식은 HTTP Method 자체로 행위 의미를 표현한다.

```text
PATCH  /study-logs/{id}
DELETE /study-logs/{id}
```

JSP MVC 방식은 HTML form이 기본적으로 GET과 POST만 지원하기 때문에, 수정/삭제도 POST로 처리하고 URL 경로로 의미를 구분할 수 있다.

```text
POST /mvc/study-logs/{id}/edit
POST /mvc/study-logs/{id}/delete
```

즉 REST API는 Method 중심이고, JSP MVC form 처리는 POST + action URL 중심으로 이해하면 된다.

### redirect를 사용하는 이유

수정/삭제 같은 POST 처리 후 redirect를 사용하면 마지막 요청이 GET 목록 조회가 된다.

그래서 사용자가 목록 화면에서 새로고침해도 수정/삭제 POST가 다시 실행되는 문제를 줄일 수 있다.

### 다시 볼 포인트

- `GET /edit`는 수정 화면을 열기 위한 요청이다.
- `POST /edit`는 수정 폼 데이터를 받아 수정 처리하는 요청이다.
- 수정 폼의 `${log.title}` 같은 값은 Controller가 Model에 담은 `log`에서 온다.
- HTML form은 PATCH/DELETE를 직접 지원하지 않으므로 JSP MVC에서는 POST + 처리 URL로 수정/삭제를 표현할 수 있다.
- 삭제는 GET 링크가 아니라 POST form으로 처리하는 것이 안전하다.
- 수정/삭제 후에는 redirect로 목록 화면을 다시 요청하게 만든다.

## Stage 07 JSP form 검증과 에러 처리
날짜: 2026-05-27
분류: Spring / MVC / JSP / Validation
상태: 이해 중

### 질문

JSP form에서 잘못된 입력이 들어왔을 때 서버 에러가 아니라 다시 form 화면에 에러 메시지를 보여주려면 어떻게 해야 할까?

### Valid

`@Valid`는 Controller에 들어온 DTO의 검증 어노테이션을 실행한다.

예를 들어 `@NotBlank`, `@NotNull`, `@Min(1)` 같은 조건을 확인한다.

```java
@PostMapping("/mvc/study-logs")
public String createStudyLog(
        @Valid @ModelAttribute CreateStudyLogRequest request,
        BindingResult bindingResult,
        Model model
) {
    ...
}
```

### BindingResult

`BindingResult`는 `@Valid` 검증 결과를 담는 객체이다.

검증 실패가 있는지 확인할 수 있고, 어떤 필드에서 어떤 에러가 났는지도 꺼낼 수 있다.

```java
bindingResult.hasErrors()
bindingResult.getFieldErrors()
```

`BindingResult`는 바로 앞의 `@Valid` 대상에 대한 검증 결과를 담는다.

그래서 어떤 객체의 검증 결과인지 Spring MVC가 연결할 수 있도록 `@Valid`가 붙은 파라미터 바로 뒤에 와야 한다.

```java
@Valid @ModelAttribute CreateStudyLogRequest request,
BindingResult bindingResult
```

### 생성 폼 검증 실패 처리

생성 폼에서 검증 실패 시 DB에 저장하지 않고 다시 `new.jsp`를 보여줬다.

이때 사용자가 입력했던 값을 `request`로 다시 Model에 담고, 검증 에러 목록도 Model에 담았다.

```java
if (bindingResult.hasErrors()) {
    model.addAttribute("request", request);
    model.addAttribute("errors", bindingResult.getFieldErrors());
    return "study-log/new";
}
```

JSP에서는 `request`를 사용해 기존 입력값을 유지했다.

```jsp
<input type="text" name="title" value="${request.title}">
<input type="number" name="minutes" value="${request.minutes}">
<textarea name="memo">${request.memo}</textarea>
```

### 에러 메시지 출력

Controller에서 `bindingResult.getFieldErrors()`를 Model에 `errors`라는 이름으로 담았다.

```java
model.addAttribute("errors", bindingResult.getFieldErrors());
```

JSP에서는 JSTL의 `<c:forEach>`로 에러 목록을 반복 출력했다.

```jsp
<c:if test="${not empty errors}">
    <ul>
        <c:forEach var="error" items="${errors}">
            <li>${error.field}: ${error.defaultMessage}</li>
        </c:forEach>
    </ul>
</c:if>
```

### 수정 폼 검증 실패 처리

수정 폼에서도 `@Valid`와 `BindingResult`를 사용했다.

검증 실패 시 DB를 수정하지 않고 다시 `edit.jsp`를 보여줬다.

```java
if (bindingResult.hasErrors()) {
    model.addAttribute("errors", bindingResult.getFieldErrors());
    model.addAttribute("id", id);
    model.addAttribute("request", request);

    return "study-log/edit";
}
```

### 도메인 객체와 request DTO 구분

수정 폼 검증 실패 시 잘못된 입력값으로 `StudyLog` 객체를 새로 만들면 문제가 됐다.

`StudyLog` 도메인 객체는 생성자에서 이미 정상 데이터인지 검증한다.

```java
if (title == null || title.isBlank()) {
    throw new IllegalArgumentException("title must not be blank");
}
```

따라서 title이 비어 있거나 minutes가 0인 검증 실패 값을 `StudyLog`에 넣으면, JSP에 에러 메시지를 보여주기 전에 도메인 객체 생성 과정에서 예외가 발생한다.

도메인 객체는 정상 상태의 데이터를 표현해야 한다.

검증 실패한 form 입력값은 아직 정상 데이터가 아니므로 도메인 객체로 만들기보다, 사용자가 입력한 값을 담는 request DTO 상태로 다시 JSP에 넘기는 것이 자연스럽다.

### REST API와 JSP MVC form 검증 차이

REST API에서는 검증 실패 정보를 JSON 응답으로 내려주는 방식이 자연스럽다.

JSP MVC form에서는 다시 form 화면을 보여주고, 그 화면 안에 에러 메시지와 기존 입력값을 함께 표시하는 방식이 자연스럽다.

```text
REST API
-> JSON 에러 응답

JSP MVC form
-> form JSP 다시 렌더링
-> 입력값 유지
-> 에러 메시지 표시
```

### 다시 볼 포인트

- `@Valid`는 DTO의 검증 어노테이션을 실행한다.
- `BindingResult`는 바로 앞의 `@Valid` 대상에 대한 검증 결과를 담는다.
- 검증 실패 시 저장/수정하지 않고 form JSP로 돌아간다.
- 에러 메시지는 `bindingResult.getFieldErrors()`로 꺼내 JSP에서 반복 출력할 수 있다.
- 검증 실패한 입력값은 도메인 객체가 아니라 request DTO로 다시 JSP에 넘기는 것이 자연스럽다.
- 도메인 객체는 정상 상태의 데이터를 표현해야 한다.
## Stage 08 세션 로그인과 Interceptor
날짜: 2026-05-28
분류: Spring / MVC / Session / Interceptor
상태: 이해 중

### 질문

Spring MVC + JSP 방식에서 로그인 상태를 유지하고, 로그인하지 않은 사용자의 화면 접근을 막으려면 어떻게 해야 할까?

### HTTP와 세션

HTTP는 기본적으로 stateless이다. 즉 이전 요청에서 로그인했다는 사실을 다음 요청이 자동으로 기억하지 않는다.

그래서 로그인 기능을 만들려면 서버가 별도의 저장 공간에 로그인 상태를 기억해야 한다. Spring MVC에서는 `HttpSession`을 사용해 이 흐름을 직접 다뤄볼 수 있다.

세션 로그인에서는 서버와 브라우저가 서로 다른 것을 저장한다.

```text
서버
-> 세션 저장소에 loginUser 같은 로그인 상태 저장

브라우저
-> 서버 세션을 찾기 위한 세션 ID를 쿠키로 저장
```

브라우저가 다음 요청을 보낼 때 세션 ID 쿠키를 같이 보내면, 서버는 그 ID로 기존 세션을 찾아 로그인 상태를 확인한다.

### 로그인 처리

로그인 성공 시 서버 세션에 사용자 정보를 저장했다.

```java
session.setAttribute("loginUser", username);
```

이 코드는 현재 세션에 `loginUser`라는 이름으로 로그인한 사용자 이름을 저장한다는 뜻이다.

```text
loginUser -> student
```

이후 같은 브라우저에서 요청이 들어오면 서버는 세션에서 `loginUser`를 꺼내 로그인 상태인지 확인할 수 있다.

### 로그아웃 처리

로그아웃은 세션 상태를 변경하는 요청이다. 그래서 GET 링크보다 POST form으로 처리하는 것이 자연스럽다.

```java
@PostMapping("/mvc/logout")
public String logout(HttpSession session) {
    session.invalidate();
    return "redirect:/mvc/login";
}
```

`session.invalidate()`는 현재 세션을 종료한다. 세션 안에 저장된 `loginUser` 같은 값도 더 이상 사용할 수 없게 된다.

로그아웃 후에는 `redirect:/mvc/login`으로 보냈다. 이렇게 하면 POST 요청 후 브라우저 주소도 로그인 페이지로 바뀌고, 새로고침으로 로그아웃 POST가 반복되는 문제도 줄일 수 있다.

### Controller 안에서 직접 로그인 체크하기

처음에는 Controller 메서드마다 세션을 확인했다.

```java
Object loginUser = session.getAttribute("loginUser");

if (loginUser == null) {
    return "redirect:/mvc/login";
}
```

이 방식은 동작하지만 여러 메서드에 같은 코드가 반복된다. 접근 조건이 바뀌면 모든 메서드를 찾아 수정해야 하고, Controller가 실제 화면 기능보다 인증 체크 코드로 지저분해진다.

### Interceptor

Interceptor는 Spring MVC 요청 흐름 중 Controller 실행 직전에 끼어드는 객체이다.

```text
브라우저 요청
-> DispatcherServlet
-> HandlerMapping이 Controller 찾음
-> Interceptor preHandle()
-> Controller 메서드
-> JSP 렌더링
```

로그인 체크처럼 여러 Controller에서 반복되는 공통 검사를 Interceptor로 분리할 수 있다.

### preHandle()

`preHandle()`은 Controller 실행 전에 호출된다.

```java
@Override
public boolean preHandle(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler
) throws Exception {
    HttpSession session = request.getSession(false);

    if (session == null || session.getAttribute("loginUser") == null) {
        response.sendRedirect("/mvc/login");
        return false;
    }

    return true;
}
```

`return true`는 원래 요청을 계속 진행하라는 뜻이다.

`return false`는 Controller를 실행하지 않고 여기서 요청 흐름을 멈추겠다는 뜻이다.

`request.getSession(false)`에서 `false`를 넣은 이유는 기존 세션만 확인하기 위해서이다. 로그인 체크 중에 새 빈 세션을 만들 필요가 없기 때문이다.

### WebConfig와 Interceptor 등록

Interceptor 클래스를 만들기만 해서는 실제 요청에 적용되지 않는다. Spring MVC 설정에서 어떤 경로에 적용할지 등록해야 한다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginCheckInterceptor())
                .addPathPatterns("/mvc/study-logs/**");
    }
}
```

`addPathPatterns("/mvc/study-logs/**")`는 `/mvc/study-logs` 아래의 요청들에 Interceptor를 적용한다는 뜻이다.

예를 들면 다음 경로들이 포함된다.

```text
/mvc/study-logs
/mvc/study-logs/new
/mvc/study-logs/1/edit
/mvc/study-logs/1/delete
```

### Interceptor 적용 후 Controller 변화

Interceptor를 적용한 뒤에는 Controller 메서드마다 반복하던 로그인 체크를 제거할 수 있었다.

Controller는 목록 조회, 생성, 수정, 삭제 같은 실제 화면 기능에 집중하고, 로그인 여부 확인은 Interceptor가 담당한다.

목록 화면에서 `student님 로그인 중` 같은 표시가 필요할 때만 Controller에서 세션 값을 꺼내 Model에 담았다.

```java
Object loginUser = session.getAttribute("loginUser");
model.addAttribute("loginUser", loginUser);
```

이 코드는 접근 제어가 아니라 JSP 화면에 표시할 데이터를 전달하는 역할이다.

### 다시 볼 포인트

- HTTP는 stateless라서 로그인 상태를 자동으로 기억하지 않는다.
- 세션 로그인에서 서버는 로그인 상태를 저장하고, 브라우저는 세션 ID를 쿠키로 들고 다닌다.
- `session.setAttribute("loginUser", username)`은 세션에 로그인 사용자 정보를 저장한다.
- `session.invalidate()`는 현재 세션을 종료한다.
- 로그아웃은 서버 상태를 변경하므로 POST로 처리하는 것이 자연스럽다.
- Interceptor는 Controller 실행 전에 공통 검사를 수행할 수 있다.
- `preHandle()`에서 `true`는 계속 진행, `false`는 Controller 실행 중단을 의미한다.
- `request.getSession(false)`는 기존 세션만 확인하고 새 세션은 만들지 않는다.
- `WebConfig`의 `addPathPatterns()`로 Interceptor 적용 경로를 지정한다.
- Interceptor를 적용하면 Controller의 반복 로그인 체크 코드를 줄일 수 있다.
## Stage 09 검색 조건 유지와 페이징
날짜: 2026-05-29
분류: Spring / MVC / JSP / MyBatis / Pagination
상태: 이해 중

### 질문

업무형 목록 화면에서 데이터가 많아졌을 때 검색 조건을 유지하면서 일부 데이터만 조회하려면 어떻게 해야 할까?

### 페이징이 필요한 이유

목록 데이터가 많을 때 한 번에 전부 조회하면 DB가 많은 row를 읽어야 하고, 서버 메모리에 많은 객체가 올라오며, 응답 HTML도 커지고 브라우저 렌더링도 무거워진다.

페이징은 필요한 만큼만 조회하고 보여주는 방식이다.

```text
전체 데이터 10만 개
-> 한 번에 모두 조회하지 않음
-> 현재 페이지에 필요한 10개 또는 20개만 조회
```

그래서 DB 처리량, 서버 메모리, 네트워크 응답 크기, 화면 렌더링 부담을 줄일 수 있다.

### page, size, offset

`page`는 사용자가 보고 싶은 페이지 번호이다. 보통 사용자가 보는 페이지 번호는 1부터 시작한다.

`size`는 한 페이지에 보여줄 데이터 개수이다.

`offset`은 DB에서 앞에서 몇 개의 row를 건너뛸지 나타낸다. DB 기준 건너뛰기는 0부터 시작한다.

계산식은 다음과 같다.

```text
offset = (page - 1) * size
```

예를 들어 `size = 10`이면 다음과 같다.

```text
page = 1 -> offset = 0
page = 2 -> offset = 10
page = 3 -> offset = 20
```

### LIMIT과 OFFSET

MySQL 기준으로 `LIMIT`은 가져올 데이터 개수를 의미하고, `OFFSET`은 앞에서 건너뛸 데이터 개수를 의미한다.

```sql
ORDER BY id
LIMIT #{size}
OFFSET #{offset}
```

이 SQL은 `id` 기준으로 정렬한 뒤, `offset`만큼 건너뛰고, 그 다음 `size`개를 가져온다.

예를 들어 다음 SQL은 앞의 20개를 건너뛰고 그 다음 10개를 가져온다.

```sql
LIMIT 10 OFFSET 20
```

한 페이지 크기가 10이라면 3페이지 데이터에 해당한다.

### MyBatis 페이징 조회

Mapper 인터페이스에는 검색 조건과 페이징 조건을 함께 받는 메서드를 추가했다.

```java
List<StudyLog> searchPage(
        @Param("title") String title,
        @Param("category") StudyCategory category,
        @Param("size") int size,
        @Param("offset") int offset
);
```

XML Mapper에서는 기존 검색 조건에 `LIMIT`, `OFFSET`을 붙였다.

```xml
<select id="searchPage" resultMap="studyLogResultMap">
    SELECT id, title, category, minutes, memo
    FROM study_logs
    <where>
        <if test="title != null and title != ''">
            AND title LIKE CONCAT('%', #{title}, '%')
        </if>
        <if test="category != null">
            AND category = #{category}
        </if>
    </where>
    ORDER BY id
    LIMIT #{size}
    OFFSET #{offset}
</select>
```

### 전체 개수 조회가 필요한 이유

현재 페이지의 목록만 조회하면 마지막 페이지가 몇 번인지 알 수 없다.

그래서 목록 조회 쿼리와 별도로 검색 조건에 맞는 전체 개수 조회 쿼리가 필요하다.

```java
int countSearch(
        @Param("title") String title,
        @Param("category") StudyCategory category
);
```

```xml
<select id="countSearch" resultType="int">
    SELECT COUNT(*)
    FROM study_logs
    <where>
        <if test="title != null and title != ''">
            AND title LIKE CONCAT('%', #{title}, '%')
        </if>
        <if test="category != null">
            AND category = #{category}
        </if>
    </where>
</select>
```

목록 조회는 현재 페이지에 보여줄 row를 가져오는 역할이고, 전체 개수 조회는 페이지 버튼과 마지막 페이지를 계산하기 위한 역할이다.

### totalPages 계산

전체 페이지 수는 전체 개수를 페이지 크기로 나누어 계산한다. 마지막 페이지에는 `size`보다 적은 데이터가 들어갈 수 있으므로 올림 계산이 필요하다.

정수 계산으로는 다음 식을 사용할 수 있다.

```java
int totalPages = (totalCount + size - 1) / size;
```

예를 들어 `totalCount = 31`, `size = 10`이면 다음과 같다.

```text
(31 + 10 - 1) / 10
= 40 / 10
= 4
```

### page와 size 보정

사용자가 URL을 직접 수정하면 `page=0`, `page=-1`, `size=0` 같은 값이 들어올 수 있다.

이런 값은 음수 offset이나 0으로 나누기 오류를 만들 수 있으므로 Controller에서 최소값을 보정했다.

```java
if (page < 1) {
    page = 1;
}

if (size < 1) {
    size = 10;
}
```

이 보정은 `totalPages`나 `offset`을 계산하기 전에 해야 한다.

### 검색 조건 유지

검색 조건이 있는 상태에서 페이지를 이동할 때 `title`, `category`를 링크에 같이 붙여야 한다.

```jsp
<a href="/mvc/study-logs?title=${title}&category=${category}&page=${pageNumber}&size=${size}">
    ${pageNumber}
</a>
```

검색 결과 화면에서 페이지를 넘기는 것은 전체 목록의 다음 페이지가 아니라, 현재 검색 결과 안에서 다음 페이지를 보는 행위이다.

따라서 페이지 링크에 검색 조건이 빠지면 사용자가 검색 결과를 보다가 갑자기 전체 목록으로 돌아가는 문제가 생긴다.

### JSP 페이지 번호 출력

`totalPages`를 사용해 JSP에서 페이지 번호 링크를 만들었다.

```jsp
<c:forEach var="pageNumber" begin="1" end="${totalPages}">
    <c:choose>
        <c:when test="${pageNumber == page}">
            <strong>${pageNumber}</strong>
        </c:when>
        <c:otherwise>
            <a href="/mvc/study-logs?title=${title}&category=${category}&page=${pageNumber}&size=${size}">
                ${pageNumber}
            </a>
        </c:otherwise>
    </c:choose>
</c:forEach>
```

현재 페이지는 링크가 아니라 굵게 표시하고, 다른 페이지는 링크로 이동할 수 있게 했다.

이전/다음 링크도 `page > 1`, `page < totalPages` 조건으로 필요한 경우에만 보여줬다.

### 아직 개선할 수 있는 점

현재 방식은 전체 페이지 수가 많아지면 페이지 번호가 너무 많이 출력된다.

```text
1 2 3 4 5 6 7 8 9 10 ... 100
```

실무에서는 보통 현재 페이지 주변의 일부 번호만 보여준다.

```text
현재 page = 7
-> 5 6 7 8 9
```

초반과 마지막 구간에서는 표시할 번호가 부족할 수 있으므로 시작 번호와 끝 번호를 따로 계산해야 한다.

또한 지금은 `page`, `size`, `title`, `category`를 Controller 파라미터로 직접 받고 있는데, 조건이 더 많아지면 검색 조건 DTO나 페이지 요청 객체로 묶는 것도 고려할 수 있다.

### 다시 볼 포인트

- 페이징은 DB 조회량, 서버 메모리, 응답 크기, 브라우저 렌더링 부담을 줄이기 위해 필요하다.
- `page`는 사용자 기준 페이지 번호이고, `offset`은 DB 기준으로 건너뛸 row 수이다.
- `offset = (page - 1) * size`로 계산한다.
- `LIMIT`은 가져올 개수, `OFFSET`은 건너뛸 개수이다.
- 현재 페이지 목록 조회와 전체 개수 조회는 역할이 다르므로 둘 다 필요하다.
- `totalPages`는 마지막 페이지와 페이지 번호 링크를 만들기 위해 필요하다.
- 검색 조건이 있는 페이지 링크에는 `title`, `category`를 같이 붙여야 한다.
- `page`, `size`는 잘못된 query parameter에 대비해 최소값 보정이 필요하다.
- JSP의 `<c:forEach>`로 `1`부터 `totalPages`까지 페이지 링크를 만들 수 있다.
- 페이지 수가 많아지면 현재 페이지 주변 일부만 보여주는 방식으로 개선할 수 있다.

## Stage 10 Spring Security 로그인과 권한 처리
날짜: 2026-06-01
분류: Backend / Spring Security
상태: 이해 중

### 질문

직접 만든 세션 로그인과 Interceptor 기반 접근 제한을 Spring Security 방식으로 바꾸면 요청 흐름과 책임이 어떻게 달라질까?

### 요청 흐름 변화

Spring Security를 추가하면 기존 MVC 요청 흐름 앞에 Security Filter가 먼저 동작한다.

```text
브라우저 요청
-> Spring Security Filter
-> Spring MVC Interceptor
-> Controller
```

기존에는 Interceptor에서 직접 로그인 여부를 확인했지만, Security를 적용한 뒤에는 인증과 인가 처리를 Security FilterChain이 먼저 맡는다.

### SecurityFilterChain

`SecurityFilterChain`은 HTTP 요청에 적용할 보안 규칙을 정의한다.

예를 들어 어떤 URL은 모두 접근 가능하게 하고, 어떤 URL은 로그인 사용자만 접근 가능하게 하며, 어떤 URL은 ADMIN 권한만 접근 가능하게 설정한다.

```java
.authorizeHttpRequests(auth -> auth
        .requestMatchers("/mvc/login").permitAll()
        .requestMatchers("/mvc/study-logs/*/delete").hasRole("ADMIN")
        .requestMatchers("/mvc/study-logs/**").authenticated()
        .anyRequest().permitAll()
)
```

규칙은 위에서 아래로 평가되므로 더 구체적인 URL 규칙을 먼저 작성해야 한다.

### UserDetailsService

`UserDetailsService`는 Spring Security가 로그인할 사용자 정보를 어디서 가져올지 알려주는 역할을 한다.

이번 학습에서는 DB가 아니라 메모리에 임시 사용자를 등록했다.

```java
UserDetails student = User.withUsername("student")
        .password(passwordEncoder.encode("1234"))
        .roles("USER")
        .build();

UserDetails admin = User.withUsername("admin")
        .password(passwordEncoder.encode("1234"))
        .roles("ADMIN")
        .build();

return new InMemoryUserDetailsManager(student, admin);
```

실무에서는 보통 `UserDetailsService`가 DB에서 사용자 정보를 조회하도록 만든다.

### PasswordEncoder와 BCrypt

`PasswordEncoder`는 비밀번호를 어떤 방식으로 해시하고 검증할지 정한다.

이번 학습에서는 `BCryptPasswordEncoder`를 사용했다.

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

BCrypt는 비밀번호를 복구 가능한 암호문으로 저장하지 않고, 비교 가능한 해시값으로 저장한다.

로그인할 때는 사용자가 입력한 비밀번호와 저장된 해시를 비교한다.

### loginPage와 loginProcessingUrl

`loginPage("/mvc/login")`은 로그인 화면을 보여줄 GET 페이지 주소이다.

`loginProcessingUrl("/mvc/login")`은 로그인 form이 POST 요청을 보내는 처리 주소이다.

```java
.formLogin(form -> form
        .loginPage("/mvc/login")
        .loginProcessingUrl("/mvc/login")
        .defaultSuccessUrl("/mvc/study-logs", true)
        .permitAll()
)
```

같은 `/mvc/login` 주소를 써도 GET은 화면 표시, POST는 Security 로그인 처리로 역할이 다르다.

### CSRF 토큰

Spring Security는 기본적으로 form POST 요청에 CSRF 토큰을 요구한다.

CSRF 토큰은 사용자가 의도해서 현재 사이트의 form을 제출한 것인지 확인하는 방어 장치이다.

따라서 로그인, 로그아웃, 삭제 같은 POST form에는 hidden input으로 토큰을 함께 보내야 한다.

```jsp
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
```

권한이 맞아도 CSRF 토큰이 없으면 POST 요청이 먼저 막힐 수 있다.

### authenticated와 hasRole

`authenticated()`는 로그인한 사용자라면 통과시킨다.

`hasRole("ADMIN")`은 로그인했고 ADMIN 역할을 가진 사용자만 통과시킨다.

`.roles("ADMIN")`으로 등록한 역할은 내부적으로 `ROLE_ADMIN` 권한명으로 저장된다.

설정에서는 보통 `hasRole("ADMIN")`처럼 `ROLE_` 접두사를 빼고 쓴다.

### JSP sec:authorize

JSP에서 Spring Security 권한에 따라 화면 요소를 보여주려면 Security taglib를 사용할 수 있다.

```jsp
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>

<sec:authorize access="hasRole('ADMIN')">
    <button type="submit">Delete</button>
</sec:authorize>
```

이를 사용하려면 `spring-security-taglibs` 의존성이 필요하다.

```gradle
implementation 'org.springframework.security:spring-security-taglibs'
```

`sec:authorize`는 화면 표시 제어이고, 실제 보안은 서버의 `SecurityFilterChain`에서 막아야 한다.

버튼을 숨기는 것은 UX이고, 서버에서 URL 접근을 막는 것이 보안이다.

### 직접 세션 로그인과 Spring Security 방식의 차이

기존 방식은 개발자가 직접 세션에 로그인 정보를 넣고 확인했다.

```java
session.setAttribute("loginUser", username);
```

그리고 Interceptor에서 `loginUser`가 있는지 직접 확인했다.

Spring Security 방식에서는 인증 성공 후 Security가 인증 정보를 관리한다.

따라서 Controller에서 `loginUser`를 직접 세션에 넣거나, Interceptor에서 직접 확인하는 코드가 필요 없어졌다.

### 아직 부족한 점

이번 단계에서는 학습을 위해 사용자를 메모리에 등록했다.

실무에 가까워지려면 다음 단계가 필요하다.

- 사용자 정보를 DB에 저장한다.
- 회원가입을 구현한다.
- DB에서 사용자 정보를 조회하는 `UserDetailsService`를 만든다.
- 권한 정보를 DB와 연결한다.
- 비밀번호 변경, 계정 잠금, 로그인 실패 처리 등을 다룬다.

### 다시 볼 포인트

- Spring Security를 추가하면 요청 앞단에 Security Filter가 동작한다.
- `SecurityFilterChain`은 URL별 인증/인가 규칙을 정의한다.
- `UserDetailsService`는 사용자 정보를 어디서 가져올지 담당한다.
- `PasswordEncoder`는 비밀번호 해시와 검증 방식을 담당한다.
- `loginPage`는 로그인 화면 주소이고, `loginProcessingUrl`은 로그인 처리 주소이다.
- form POST 요청에는 CSRF 토큰이 필요하다.
- `authenticated()`는 로그인 여부, `hasRole("ADMIN")`은 역할까지 확인한다.
- `.roles("ADMIN")`은 내부적으로 `ROLE_ADMIN`으로 저장된다.
- JSP의 `<sec:authorize>`는 화면 표시 제어이며, 서버 권한 체크를 대체할 수 없다.
- 직접 세션을 다루던 로그인 코드는 Security가 인증 상태를 관리하도록 정리할 수 있다.

## Stage 11 DB 기반 로그인과 회원가입
날짜: 2026-06-03
분류: Backend / Spring Security / MyBatis / Signup
상태: 이해 중

### 질문

InMemory 사용자 등록에서 DB 기반 사용자 조회로 바꾸고, 회원가입까지 연결하면 로그인 흐름과 책임 분리는 어떻게 달라질까?

### InMemory 로그인과 DB 기반 로그인

Stage 10에서는 `InMemoryUserDetailsManager`에 사용자를 코드로 직접 등록했다.

```java
student / 1234 / USER
admin / 1234 / ADMIN
```

이 방식은 학습과 테스트에는 간단하지만, 사용자를 코드에 고정해야 하므로 회원가입, 탈퇴, 권한 변경 같은 기능을 유연하게 처리하기 어렵다.

Stage 11에서는 사용자 정보를 `users` 테이블에 저장하고, 로그인 시 DB에서 사용자 정보를 조회하도록 변경했다.

### users 테이블

`users` 테이블은 로그인에 필요한 사용자 정보를 저장한다.

```sql
CREATE TABLE IF NOT EXISTS users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,
    enabled BOOLEAN NOT NULL
);
```

각 컬럼의 역할은 다음과 같다.

- `id`: 내부 식별자
- `username`: 로그인 ID
- `password`: BCrypt로 해시된 비밀번호
- `role`: `ROLE_USER`, `ROLE_ADMIN` 같은 권한
- `enabled`: 계정 활성화 여부

### 로그인 조회 흐름

로그인 요청이 들어오면 Spring Security는 `UserDetailsService.loadUserByUsername(username)`을 호출한다.

우리가 만든 `DbUserDetailsService`는 `UserMapper.findByUsername(username)`으로 DB에서 사용자를 조회한다.

```java
AppUser appUser = userMapper.findByUsername(username);
```

사용자가 없으면 `UsernameNotFoundException`을 던지고, 사용자가 있으면 Spring Security가 이해할 수 있는 `UserDetails`로 변환한다.

```java
return User.withUsername(appUser.getUsername())
        .password(appUser.getPassword())
        .authorities(appUser.getRole())
        .disabled(!appUser.isEnabled())
        .build();
```

DB에 `ROLE_USER`, `ROLE_ADMIN`처럼 이미 `ROLE_`이 붙은 값이 저장되어 있으므로 `.roles()`가 아니라 `.authorities()`를 사용했다.

### PasswordEncoder의 역할

회원가입에서는 사용자가 입력한 평문 비밀번호를 BCrypt 해시로 바꿔 저장한다.

```java
String encodedPassword = passwordEncoder.encode(request.getPassword());
```

로그인에서는 입력한 비밀번호와 DB에 저장된 BCrypt 해시가 맞는지 검증한다.

단순히 입력값을 다시 해시해서 문자열 비교하는 것이 아니라, Spring Security가 `PasswordEncoder.matches(rawPassword, encodedPassword)` 방식으로 비교한다.

BCrypt는 같은 비밀번호라도 매번 다른 해시가 나올 수 있으므로 이 차이를 이해해야 한다.

### SignupRequest와 ModelAttribute

JSP form 데이터를 `@ModelAttribute SignupRequest`로 받기 위해 DTO에는 기본 생성자와 setter가 필요하다.

Spring은 form 요청을 받을 때 대략 다음 흐름으로 값을 채운다.

```java
SignupRequest request = new SignupRequest();
request.setUsername(username);
request.setPassword(password);
```

그래서 `SignupRequest`에는 기본 생성자, getter, setter를 두었다.

입력값 검증은 `@NotBlank`, `@Size`로 처리했다.

```java
@NotBlank(message = "username is required")
@Size(min = 3, max = 20, message = "username must be between 3 and 20 characters")
private String username;

@NotBlank(message = "password is required")
@Size(min = 4, max = 30, message = "password must be between 4 and 30 characters")
private String password;
```

### 회원가입 흐름

회원가입은 로그인과 달리 DB에 새 사용자 row를 만드는 기능이다.

```text
GET /mvc/signup
-> 회원가입 form 표시

POST /mvc/signup
-> SignupRequest 검증
-> username 중복 확인
-> password BCrypt 해시
-> role은 ROLE_USER로 지정
-> enabled는 true로 지정
-> users 테이블에 INSERT
-> /mvc/login?signupSuccess로 redirect
```

사용자가 role과 enabled를 직접 보내지 않게 한 이유는, 권한과 계정 상태는 사용자가 결정할 값이 아니라 서버 정책으로 정해야 하는 값이기 때문이다.

### 중복 username 처리

`username`은 로그인 식별자이므로 중복되면 안 된다.

회원가입 전에 Service에서 먼저 중복을 확인했다.

```java
public boolean isUsernameDuplicated(String username) {
    AppUser existingUser = userMapper.findByUsername(username);
    return existingUser != null;
}
```

사전 중복 체크는 사용자에게 자연스럽게 오류 메시지를 보여주기 위한 UX 방어이다.

DB의 `UNIQUE` 제약은 동시에 같은 username 가입 요청이 들어오는 경우까지 막는 최종 데이터 무결성 방어선이다.

즉, 검증은 여러 단계로 둔다.

```text
프론트 검증: 빠른 사용자 피드백
백엔드 검증: 신뢰할 수 없는 요청 차단
DB 제약: 최종 데이터 무결성 보장
```

### Controller, Service, Mapper 역할 분리

회원가입 로직은 Controller에서 Service로 분리했다.

```text
SignupPageController
-> 요청/응답 흐름, 검증 실패 시 화면 처리

SignupService
-> 중복 username 확인, 비밀번호 해시, 기본 role/enabled 지정

UserMapper
-> users 테이블 조회와 저장
```

`@Controller`, `@Service`, `@Mapper`, `@Bean`으로 등록된 객체들은 Spring Bean으로 관리된다.

생성자 파라미터에 필요한 Bean 타입을 적으면 Spring이 자동으로 찾아서 주입한다.

이것이 의존성 주입, DI이다.

### 실패 시 입력값 유지

회원가입 실패 시 사용자가 입력한 username은 다시 보여주고, password는 다시 보여주지 않았다.

```jsp
<input type="text" name="username" value="${signupRequest.username}">
<input type="password" name="password">
```

username은 다시 입력하는 번거로움을 줄이기 위해 유지한다.

password는 민감한 값이므로 실패 후 다시 화면에 출력하지 않는 것이 좋다.

### 아직 부족한 점

현재 회원가입/로그인은 기본 흐름을 학습하기 위한 형태이다.

실무에 가까워지려면 다음 요소들이 더 필요하다.

- 비밀번호 확인 입력
- 더 강한 비밀번호 정책
- 이메일 또는 휴대폰 인증
- OAuth 로그인
- 비밀번호 변경과 재설정
- 계정 잠금, 비활성화, 탈퇴 처리
- role을 별도 테이블로 분리
- 중복 가입 경쟁 상황에 대한 예외 처리
- 로그인 실패 횟수 제한

### 다시 볼 포인트

- InMemory 로그인은 코드에 사용자를 고정하고, DB 기반 로그인은 users 테이블에서 사용자를 조회한다.
- `UserMapper.findByUsername()`은 DB에서 username으로 사용자 row를 조회한다.
- `DbUserDetailsService`는 DB 사용자 정보를 Spring Security의 `UserDetails`로 변환한다.
- DB 비밀번호는 평문이 아니라 BCrypt 해시로 저장해야 한다.
- 회원가입에서는 `PasswordEncoder.encode()`로 비밀번호를 해시한다.
- 로그인에서는 `PasswordEncoder.matches()` 흐름으로 입력 비밀번호와 저장 해시를 검증한다.
- `@ModelAttribute` form DTO에는 기본 생성자와 setter가 필요하다.
- role과 enabled는 사용자가 아니라 서버에서 정해야 한다.
- 사전 중복 체크는 UX이고, DB UNIQUE 제약은 최종 방어선이다.
- Controller는 요청/응답, Service는 비즈니스 로직, Mapper는 DB 접근을 담당한다.
- 실패 화면에서 username은 유지하고 password는 유지하지 않는 것이 좋다.
