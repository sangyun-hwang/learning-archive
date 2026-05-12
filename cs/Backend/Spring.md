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
