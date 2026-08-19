# Next Server와 배포 구조

Next.js는 정적 파일만 만드는 도구가 아니다. 사용하는 기능과 배포 방식에 따라 요청을 받아 Server Component, SSR, Server Action과 Route Handler를 실행하는 **Server Runtime**을 가진다.

별도의 Backend가 있다면 Next Server는 UI Rendering과 Backend for Frontend(BFF) 역할을 담당할 수 있다. 작은 서비스에서는 Route Handler와 Server Action을 사용해 Backend 일부를 직접 담당할 수도 있지만, Next Server와 Backend Server가 반드시 하나이거나 반드시 분리되어야 하는 것은 아니다.

## Next Server가 하는 일

개발 환경의 `next dev`와 운영 환경의 `next start`는 Next.js 요청을 처리하는 Server Process를 실행한다.

```bash
pnpm build
pnpm start
```

Next Server는 구성에 따라 다음 작업을 수행한다.

- 요청 URL에 맞는 Route 선택
- Server Component 실행
- SSR HTML과 RSC Payload 생성
- Client JavaScript와 정적 자산 제공
- Server Action과 Route Handler 실행
- Cookie와 Header 처리
- Cache와 재검증 처리
- Image Optimization

## 별도 Backend가 있는 구조

Spring이나 NestJS Backend가 있다면 Next Server에서 Backend API를 호출할 수 있다.

```text
Browser
-> Next Server
-> Spring / NestJS Backend
-> Database
```

첫 Page 요청의 흐름은 다음과 같다.

1. Browser가 Next Server에 Page를 요청한다.
2. Next Server에서 Server Component가 실행된다.
3. Server Component가 Backend API를 호출한다.
4. Backend가 Database를 조회한다.
5. Next Server가 결과로 HTML과 RSC Payload를 생성한다.
6. Browser가 초기 화면을 표시하고 Client Component를 hydration한다.

```tsx
export default async function ProductPage() {
  const response = await fetch(
    `${process.env.BACKEND_URL}/api/products/1`,
  );
  const product = await response.json();

  return <h1>{product.name}</h1>;
}
```

이 요청은 Browser가 아니라 Next Server에서 발생하므로 `BACKEND_URL`을 Server 전용 환경 변수로 관리할 수 있다. 같은 사설 Network 안에 배포했다면 공개 주소 대신 Backend의 내부 주소를 사용할 수도 있다.

## Browser가 Backend를 직접 호출하는 구조

Client Component가 Backend API를 직접 호출할 수도 있다.

```text
Browser -> Next Server: Page와 정적 자산 요청
Browser -> Backend Server: Client API 요청
```

이 구조는 Mobile App과 Web이 같은 공개 API를 사용하거나 Client에서 자주 갱신하는 데이터를 가져올 때 유용하다. 다만 서로 다른 Origin이라면 CORS, Cookie의 `SameSite`와 `Secure`, CSRF, 인증 정보와 Backend 공개 주소를 함께 고려해야 한다.

## BFF 구조

Browser가 Backend를 직접 호출하지 않고 Next Server를 거칠 수도 있다. 명시적인 HTTP API가 필요하다면 Route Handler를 사용하고, Next UI의 form 제출이나 mutation을 처리한다면 Server Action을 사용할 수 있다.

```text
Browser
-> Next Route Handler / Server Action
-> Backend Server
-> Database
```

Next Server가 현재 UI에 필요한 형태로 응답을 조합하고 내부 Backend 주소를 감추는 **Backend for Frontend** 역할을 맡는다.

### Route Handler의 주소 흐름

Client Component가 `fetch('/api/products')`를 호출하고 Route Handler가 Spring API를 대신 호출한다면 각 구간의 주소는 다음과 같다.

```text
Browser
-> https://shop.example.com/api/products
-> Next Route Handler
-> http://spring-service:8080/api/products
-> Spring Backend
```

Browser는 공개된 Next Application 주소를 사용하고, Next Server는 Server 환경 변수에 저장된 Spring의 내부 주소를 사용할 수 있다.

### 기능별 역할

| 기능 | 주요 목적 |
| --- | --- |
| Server Component | Server에서 UI와 조회 데이터를 준비 |
| Server Action | Form 제출 등 Next UI에서 발생한 mutation 처리 |
| Route Handler | 명시적인 HTTP API 또는 BFF endpoint 제공 |

Route Handler는 첫 Page를 생성하는 기능이 아니다. 첫 화면은 Server Component와 SSR 또는 정적 Rendering이 담당한다. Server Component에서 Spring 데이터를 조회할 때는 특별한 이유가 없다면 같은 Next Application의 Route Handler를 다시 호출하기보다 Spring 내부 API를 직접 호출하는 편이 불필요한 HTTP 단계를 줄인다.

| 직접 호출 | Next BFF 경유 |
| --- | --- |
| Browser가 Backend API 호출 | Browser가 Next Server 호출 |
| 공개 API와 CORS 설정 필요 | Next가 내부 Backend와 Server-to-Server 통신 가능 |
| Mobile App 등과 API 공유에 유리 | Web UI에 맞춘 응답 조합에 유리 |
| Browser가 Backend 인증을 직접 처리 | Next가 Cookie와 인증 흐름을 중계할 수 있음 |

BFF가 권한 검사를 자동으로 해결하는 것은 아니다. Next와 Backend 중 각 보안 경계에서 인증, 인가와 입력 검증이 필요하다.

Mobile App과 Web이 같은 핵심 API를 공유하고 Spring이 실제 Backend라면 두 Client가 Spring API를 직접 사용하는 구조가 더 명확할 수 있다. Route Handler는 Web UI에 특화된 응답 조합이나 중계가 필요할 때 선택한다.

## Next Server의 실제 위치

Server는 특정 제품명이 아니라 실행 중인 Process와 그 Process가 위치한 Computing 환경을 의미한다.

로컬에서는 다음처럼 개발자 Computer에 있다.

```text
Browser -> localhost:3000 Next Process
Next Process -> localhost:8080 Spring Process
```

운영 환경에서는 VM, Container 또는 Platform의 Function Runtime에 위치한다.

```text
Browser
-> DNS / CDN
-> Load Balancer 또는 Reverse Proxy
-> Next Server Container
-> Backend Container
-> Database
```

Vercel에서는 Platform이 Next Runtime과 CDN, Function, Cache 등을 배치한다. AWS, GCP 등에서는 개발자가 Next Process가 실행될 VM이나 Container를 선택하고 주변 Infra를 구성한다.

## Vercel 외의 배포 방법

### Node.js Server

EC2, Compute Engine 또는 일반 VM에 Node.js와 Application을 배치한다.

```bash
pnpm install --frozen-lockfile
pnpm build
pnpm start
```

운영 환경에서는 보통 Next Server 앞에 Nginx 같은 Reverse Proxy나 Cloud Load Balancer를 둔다.

```text
Internet :443
-> Reverse Proxy
-> Next Server :3000
```

직접 관리할 항목은 다음과 같다.

- HTTPS와 Domain
- 환경 변수와 Secret
- Process 재시작과 Health Check
- Log와 Monitoring
- 배포 자동화와 Rollback
- Scaling과 Load Balancing

### Docker Container

Next Application을 Docker Image로 만든 뒤 ECS, Cloud Run, Kubernetes나 일반 Container Server에서 실행할 수 있다.

```ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  output: 'standalone',
};

export default nextConfig;
```

`output: 'standalone'`은 운영 실행에 필요한 파일을 `.next/standalone`에 모아 더 작은 배포 Image를 만드는 데 도움을 준다. Container 안에서는 생성된 Node.js Server Process가 Next Server가 된다.

### Static Export

```ts
const nextConfig = {
  output: 'export',
};
```

Static Export는 Build 결과인 HTML, CSS와 JavaScript를 CDN이나 정적 Hosting에 올리는 방식이다. 배포 후 실행되는 Next Server가 없으므로 요청 시 Server Runtime이 필요한 다음 기능은 사용할 수 없다.

- SSR과 동적 Rendering
- Server Action
- 동적 Route Handler
- Cookie 기반 Server Rendering
- Runtime Image Optimization

Server 기능이 필요하다면 Node.js Server, Docker 또는 Next.js를 지원하는 Platform Runtime으로 배포해야 한다.

## 여러 Instance로 확장할 때

사용자가 많아지면 Load Balancer 뒤에 여러 Next Instance를 실행할 수 있다.

```text
Load Balancer
├─ Next Instance A
├─ Next Instance B
└─ Next Instance C
```

각 Instance의 Memory와 Local Cache는 기본적으로 별도이므로 다음을 고려해야 한다.

- 한 Instance의 Cache 재검증을 다른 Instance와 공유할 방법
- Redis 같은 외부 Cache 저장소
- 모든 Instance에서 같은 Build와 환경 변수 사용
- Server Action 암호화 Key의 일관성
- 배포 버전이 섞일 때 발생할 수 있는 요청 불일치
- Session을 Process Memory에만 저장하지 않는 구조

## 구조 선택 기준

| 상황 | 가능한 구조 |
| --- | --- |
| 기존 Spring Backend가 있음 | Next Server에서 Backend 호출 또는 BFF 구성 |
| Mobile과 Web이 같은 공개 API 사용 | Browser가 Backend API 직접 호출 |
| 작은 Web Application | Next Route Handler와 Server Action 활용 |
| SSR과 Server Action 필요 | Node.js, Container 또는 지원 Platform 배포 |
| 완전한 정적 Site | Static Export와 CDN 배포 |

## 면접에서 설명하기

> Next Server는 Server Component 실행, SSR HTML과 RSC Payload 생성, Server Action과 Route Handler 처리를 담당하는 Next.js Runtime입니다. 별도 Backend가 있다면 Server Component에서 Backend API를 호출하거나 Browser와 Backend 사이의 BFF로 사용할 수 있습니다. Vercel이 아닌 환경에서도 Node.js Process나 Docker Container로 실행할 수 있지만 HTTPS, 환경 변수, Monitoring, Scaling과 여러 Instance의 Cache 일관성 같은 운영 요소를 직접 구성해야 합니다.

## References

- [Next.js: Backend for Frontend](https://nextjs.org/docs/app/guides/backend-for-frontend)
- [Next.js: Self-Hosting](https://nextjs.org/docs/app/guides/self-hosting)
- [Next.js: Deploying](https://nextjs.org/docs/app/getting-started/deploying)
- [Next.js: Static Exports](https://nextjs.org/docs/app/guides/static-exports)
