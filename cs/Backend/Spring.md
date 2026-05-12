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
