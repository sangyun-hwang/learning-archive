# Software Supply Chain Attack

## 개념

Software Supply Chain Attack은 애플리케이션 코드를 직접 공격하는 대신, 애플리케이션이 의존하는 외부 구성 요소나 개발, 빌드, 배포 경로를 공격하는 방식이다.

소프트웨어 공급망에는 다음 요소가 포함된다.

- 오픈소스 패키지
- 패키지 저장소
- 패키지 maintainer 계정
- CI/CD pipeline
- 빌드 스크립트
- Docker image
- GitHub Actions
- 배포 artifact
- IDE extension

즉 내 코드 자체에 취약점이 없어도, 내가 신뢰하고 가져다 쓰는 패키지나 빌드 경로가 오염되면 위험해질 수 있다.

## 내 코드 취약점과의 차이

내 코드 취약점은 내가 작성한 코드의 버그나 보안 실수 때문에 공격당하는 경우다.

공급망 공격은 내가 직접 작성하지 않은 외부 의존성이나 배포 과정이 침해되어 공격이 전파되는 경우다.

비유하면 다음과 같다.

```txt
내 코드 취약점: 성벽 자체에 약한 부분이 있어 뚫림
공급망 공격: 신뢰하고 들여온 물건 안에 악성 코드가 숨어 있음
```

공급망 공격이 위험한 이유는 개발자가 신뢰하는 경로를 이용하기 때문이다.

## 공격 방식

### Typosquatting

유명 패키지와 비슷한 이름의 가짜 패키지를 배포하는 방식이다.

```txt
react-query
react-qurey
```

개발자가 오타로 잘못 설치하면 악성 패키지가 실행될 수 있다.

### Dependency Confusion

사내 private package와 같은 이름의 package를 public registry에 등록해 잘못 설치되게 만드는 공격이다.

패키지 매니저 설정이 잘못되어 public registry의 패키지를 우선 설치하면 내부 패키지 대신 공격자의 패키지가 설치될 수 있다.

### Maintainer 계정 탈취

정상 패키지 maintainer의 npm, GitHub 계정을 탈취해 정상 패키지에 악성 버전을 게시한다.

사용자는 평소처럼 신뢰하던 패키지를 업데이트했을 뿐인데 악성 코드가 포함된 버전을 설치할 수 있다.

### Transitive Dependency 공격

내가 직접 설치한 패키지가 아니라, 그 패키지가 의존하는 하위 패키지를 공격하는 방식이다.

```txt
my-app
-> package-a
   -> package-b
      -> compromised-package
```

직접 설치 목록에는 보이지 않아도 dependency tree 안에서 악성 코드가 들어올 수 있다.

### Install Script 악용

npm 패키지는 설치 과정에서 lifecycle script를 실행할 수 있다.

대표적으로 `postinstall`이 있다.

```json
{
  "scripts": {
    "postinstall": "node steal-token.js"
  }
}
```

개발자나 CI가 `npm install`을 실행하는 순간 설치 스크립트가 실행될 수 있다. 이 시점에는 개발자 로컬이나 CI 환경에 GitHub token, npm token, cloud key, SSH key, `.env` 파일 같은 민감 정보가 있을 수 있다.

그래서 install script는 공급망 공격에서 특히 위험한 지점이다.

## TanStack npm 공급망 침해

2026년 5월 11일 UTC, TanStack 일부 npm 패키지가 공급망 공격을 받았다.

TanStack 공식 포스트모템에 따르면 42개 `@tanstack/*` 패키지에서 84개 악성 버전이 게시되었다. 악성 패키지는 개발자 로컬 환경이나 CI/CD 환경의 credential을 탈취하려는 성격이었다.

정확히 표현하면 다음과 같다.

```txt
TanStack 생태계 일부 npm 패키지가 공급망 공격을 받았다.
```

다만 공식 포스트모템 기준으로 `@tanstack/query*` 패키지군은 confirmed-clean으로 분류되었다. 따라서 “TanStack Query 자체가 감염됐다”라고 말하면 부정확하다.

## 왜 개발자와 CI가 위험한가?

개발자 로컬과 CI/CD 환경에는 보통 중요한 권한이 많다.

- GitHub token
- npm token
- cloud credentials
- SSH key
- `.env` 파일
- CI secrets
- 배포 권한

공격자가 패키지 설치 시점에 악성 코드를 실행하면 이런 값을 탈취하려고 시도할 수 있다. 탈취된 credential은 다른 저장소, 패키지, cloud resource로 공격을 확산시키는 데 사용될 수 있다.

## lockfile의 역할과 한계

lockfile은 설치할 dependency의 버전과 integrity 정보를 고정한다.

```txt
package-lock.json
pnpm-lock.yaml
yarn.lock
```

lockfile을 사용하면 의도하지 않은 최신 버전이 자동으로 설치되는 일을 줄일 수 있다.

하지만 lockfile이 공급망 공격을 완전히 막지는 못한다.

- lockfile이 이미 악성 버전을 가리키면 그대로 설치된다.
- lockfile을 업데이트하는 순간 악성 버전이 들어올 수 있다.
- 하위 의존성까지 모두 안전하다는 보장은 아니다.
- 설치 스크립트 실행 자체를 막아주지는 않는다.
- registry나 배포 계정이 침해되면 추가 검증이 필요하다.

따라서 lockfile은 중요한 방어 수단이지만, 단독으로 충분하지 않다.

## 방어 방법

### lockfile 기반 설치

CI에서는 `npm install`보다 lockfile을 기준으로 설치하는 `npm ci` 같은 방식을 사용하는 것이 좋다.

### 자동 업데이트 주의

dependency update PR을 바로 자동 병합하지 않는다. 보안 이슈, 릴리즈 노트, maintainer 변경, 갑작스러운 버전 증가를 확인한다.

### 설치 스크립트 제한

필요하다면 설치 스크립트 실행을 제한한다.

```bash
npm install --ignore-scripts
```

다만 일부 패키지는 정상 동작에 설치 스크립트가 필요할 수 있으므로 영향 범위를 확인해야 한다.

### 권한 최소화

CI token, npm token, GitHub token, cloud credential의 권한을 필요한 범위로 제한한다.

### Secret rotation

공급망 공격이 의심되면 먼저 token과 secret을 교체한다. 악성 패키지가 이미 실행되었다면 credential이 노출되었을 가능성을 고려해야 한다.

### 2FA, Trusted Publishing, Provenance

패키지 배포 계정에는 2FA를 적용하고, 가능하면 trusted publishing과 provenance를 활용한다. 다만 이런 기능도 완전한 방어는 아니므로 다른 검증과 함께 사용해야 한다.

### Dependency scanning과 SBOM

의존성 스캔과 SBOM을 통해 어떤 패키지를 사용하고 있는지 추적한다. 직접 의존성뿐 아니라 transitive dependency도 확인해야 한다.

## 정리

공급망 공격은 내 코드가 아니라 내가 신뢰하는 외부 패키지, maintainer 계정, 빌드 및 배포 경로를 공격하는 방식이다.

내 코드가 안전해도 공급망이 오염되면 개발자 로컬, CI/CD, 배포 환경의 credential이 노출될 수 있다.

따라서 lockfile, dependency scanning, 권한 최소화, secret rotation, 설치 스크립트 제한, 배포 계정 보안 같은 방어를 함께 적용해야 한다.
