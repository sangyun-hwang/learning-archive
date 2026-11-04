# NestJS, Fastify, Hono 비교

## 각 도구의 성격

세 도구 모두 웹 API를 만들 수 있지만 제공하는 구조와 실행 환경에 차이가 있다. NestJS는 백엔드 앱을 조직하는 구조를 제공하고, Fastify는 Node.js의 HTTP API 구성에 집중하며, Hono는 웹 표준 기반의 간결한 API를 여러 런타임에서 구성할 수 있게 한다.

### NestJS

NestJS는 TypeScript를 중심으로 모듈, 컨트롤러와 의존성 주입(DI)을 활용하는 백엔드 애플리케이션 프레임워크다. 관련 기능을 묶고 요청 처리와 업무 로직을 나누는 구조를 제공한다.

| 구성 | 역할 |
| --- | --- |
| Module | 관련 컨트롤러와 Provider 등을 묶는 단위 |
| Controller | 요청을 받아 처리 로직과 응답으로 연결 |
| Service | 상품 조회·주문 생성 같은 업무 로직을 담당하도록 구성 |
| DI | 필요한 객체를 프레임워크가 생성하고 연결하는 방식 |

Service는 Nest의 Provider로 등록할 수 있는 대표적인 형태다. 프레임워크가 모든 업무 로직의 배치를 강제하는 것은 아니지만, 공통 구조와 연결 장치를 제공하므로 팀의 작성 방식을 맞추기 좋다. 반면 모듈, 데코레이터와 DI를 익혀야 하는 학습 부담이 있다.

```text
ProductsModule
  -> ProductsController: 상품 HTTP 요청 처리
  -> ProductsService: 상품 조회와 등록 로직
```

Spring의 Controller·Service·DI와 연결해 이해할 수 있지만 세부 동작과 설정까지 동일한 것은 아니다. Next.js는 React 화면과 렌더링을 중심으로 서버 기능도 제공하는 프레임워크이고, NestJS는 백엔드 애플리케이션 구성에 초점을 둔다.

### Fastify

Fastify는 Node.js에서 HTTP API를 만드는 웹 프레임워크다. 라우팅, 요청·응답 처리, 플러그인, 스키마 기반 검증과 응답 직렬화 등을 제공한다.

스키마로 요청 파라미터와 본문의 조건을 정의하거나 응답 구조를 지정할 수 있다. 직렬화는 서버 데이터를 HTTP로 보낼 JSON 등의 형태로 변환하는 과정이다. [Fastify 검증·직렬화 문서](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/)

Fastify에서도 요청 처리와 Service를 나눌 수 있다. 다만 Nest의 모듈·DI 구조를 따르기보다 팀이 필요한 구조와 규칙을 정할 여지가 크다. 구조가 없는 도구라는 뜻은 아니며, 플러그인 등 자체 구성 방식도 제공한다.

### Hono

Hono는 웹 표준 Request·Response 기반의 간결한 웹 프레임워크다. 라우트와 미들웨어를 구성하며 Node.js, Cloudflare Workers, Bun, Deno 등 여러 실행 환경에서 사용할 수 있다. [Hono 공식 문서](https://hono.dev/docs)

작은 API, 웹훅 수신 서버나 프론트엔드용 BFF를 구성하는 사례로 이해하기 좋다. 작은 프로젝트에만 쓸 수 있다는 의미는 아니다. 실행 환경에 맞는 어댑터와 배포 구성을 사용한다.

## 같은 상품 API를 만든다면

GET /products/42로 상품을 조회하는 경우를 비교한다.

| 도구 | 요청 처리 흐름 |
| --- | --- |
| NestJS | Controller가 요청을 받아 주입된 Service의 조회 함수 호출 |
| Fastify | 등록된 라우트 핸들러에서 조회 함수 호출 후 결과 반환 |
| Hono | 등록된 라우트 핸들러에서 조회 함수 호출 후 c.json 등으로 응답 |

셋 모두 계층을 나누고 입력을 검증할 수 있다. 차이는 코드를 조직하는 기본 규칙, HTTP 처리 기능과 실행 환경에 있다. 프레임워크 이름만으로 실제 서비스의 성능 순위를 단정하지 않는다. DB 쿼리, 외부 API와 응답 크기 등도 성능에 영향을 준다.

## NestJS와 Fastify는 함께 사용할 수 있다

NestJS는 기본적으로 Express를 사용하며 Fastify 어댑터로 바꿀 수 있다.

```text
HTTP 요청
-> Fastify 어댑터
-> NestJS Controller
-> Service
-> 응답
```

이때 HTTP 처리 기반은 바뀌지만 Nest의 Module·Controller·DI 구조는 유지된다. Fastify 단독으로 API를 구성하는 것과 Nest의 구조 안에서 Fastify를 사용하는 것은 구분한다. Express 전용 미들웨어나 API를 사용한 코드는 호환성을 확인해야 한다. [NestJS HTTP 어댑터 문서](https://docs.nestjs.com/techniques/performance)

## 프레임워크 지원과 패키지 호환성

Hono가 Workers를 지원해도 앱의 모든 패키지가 자동으로 호환되지는 않는다.

```text
내 API
  -> Hono: 라우팅과 응답 처리
  -> DB 드라이버: 특정 API와 연결 방식 사용
  -> 다른 패키지: 파일 처리, 인증 등
```

DB 드라이버가 특정 Node.js API나 연결 방식을 요구한다면 Workers의 지원 범위와 필요한 호환성 설정을 확인한다. 로컬 Node.js에서 실행됐다는 사실만으로 Workers에서도 실행된다고 보장할 수 없다.

JavaScript 패키지라도 Node.js 파일 시스템에 의존하면 브라우저에서 그대로 사용할 수 없는 것과 같은 원리다. 런타임, 의존성, 외부 서비스 연결 방식을 함께 확인한다.

## 선택 기준과 실습 방향

| 고려할 필요 | 먼저 살펴볼 도구 |
| --- | --- |
| 일관된 앱 구조와 DI | NestJS |
| Node.js API와 플러그인·스키마 활용 | Fastify |
| 간결한 API와 여러 런타임 대응 | Hono |

이 표는 검토의 출발점이다. 프로젝트 크기만으로 선택하지 않고 팀의 경험, 필요한 생태계와 배포 환경을 함께 본다.

실습은 DB 없이 상품 두 개를 배열에 두고 각 도구로 GET /products/:id의 성공 응답과 404를 만들어 볼 수 있다. 라우팅, 업무 로직과 검증을 어디에 두는지 비교한다. 이 문서는 개념 학습이며 해당 실습을 실행한 기록은 아니다.

## 말로 설명하기

NestJS는 모듈과 DI 등 백엔드 앱을 조직하는 구조를 제공하고, Fastify는 Node.js의 HTTP API와 플러그인·스키마 기능을 제공합니다. Hono는 웹 표준 기반으로 여러 런타임에서 사용할 수 있는 간결한 프레임워크입니다. NestJS는 Fastify를 HTTP 어댑터로 사용할 수도 있어 셋이 항상 같은 층위의 대안은 아닙니다. 선택할 때는 팀에 필요한 구조와 배포 환경, 패키지 호환성을 함께 확인합니다.
