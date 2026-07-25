# Next.js 14.2 이후 주요 변경사항

2024년 6월 Next.js 14.2를 학습한 이후 추가되거나 크게 달라진 내용을 정리한 개요입니다. 기능별 세부 사용법보다 Next.js 15와 16에서 프레임워크의 방향이 어떻게 변했는지 파악하는 것을 목표로 합니다.

> 정리 기준: 2026년 7월 25일, 최신 Active LTS 보안 버전 Next.js 16.2.11

## 변화의 큰 방향

- React 19와 React Server Components를 기반으로 한 서버 중심 기능 강화
- 자동 캐싱보다 필요한 경계를 개발자가 명시하는 캐싱 모델로 전환
- 요청 정보에 의존하는 API를 비동기로 변경해 렌더링 작업을 세분화
- Turbopack을 개발과 production build의 기본 번들러로 전환
- routing, type safety, debugging, AI agent 연동 및 배포 도구 개선

## Next.js 15

### React 19 지원

- App Router가 React 19를 기반으로 동작하기 시작했습니다.
- React 19의 Server Actions, `useActionState`, 개선된 `useFormStatus` 등을 사용할 수 있습니다.
- Next.js 15.1부터 Pages Router와 App Router 모두 React 19 stable을 공식 지원합니다.

### Async Request APIs

요청 시점에 결정되는 다음 API와 props가 비동기로 변경되었습니다.

- `cookies()`
- `headers()`
- `draftMode()`
- `params`
- `searchParams`

```tsx
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  return <h1>{id}</h1>;
}
```

Next.js가 요청 정보와 관계없는 부분을 먼저 준비하고, 실제 요청이 필요한 부분만 기다릴 수 있도록 렌더링 단계를 분리하기 위한 변화입니다. Next.js 15에서는 임시로 동기 접근을 지원했지만 Next.js 16에서는 비동기 접근이 필수가 되었습니다.

### 기본 캐싱 정책 변경

Next.js 14의 App Router는 여러 항목을 기본적으로 캐시했습니다. Next.js 15부터는 다음 항목이 기본적으로 캐시되지 않습니다.

- 별도 옵션이 없는 `fetch`
- Route Handler의 `GET`
- client navigation의 page segment

필요한 데이터만 `cache: 'force-cache'` 같은 옵션으로 명시적으로 캐시하는 방향으로 변경되었습니다. 공유 layout과 loading state 등 일부 router cache 동작은 계속 유지됩니다.

### 개발 도구와 API

- `@next/codemod` upgrade CLI 추가
- 응답 전송 이후 작업을 실행하는 `after()` 안정화
- `instrumentation-client.ts`를 통한 초기 monitoring 및 analytics 설정
- `<Link onNavigate>`와 `useLinkStatus` 추가
- Turbopack production build가 alpha와 beta 단계를 거쳐 안정화
- Node.js runtime 기반 Middleware 지원 안정화
- `typedRoutes`와 `PageProps`, `LayoutProps`, `RouteContext` type helper 안정화
- `next typegen` 명령 추가
- `next lint` 명령 deprecated

## Next.js 16

### Cache Components

페이지 전체를 정적 또는 동적으로 구분하는 대신, page, component, function 단위로 캐시 경계를 선택하는 모델입니다.

```text
상품 페이지
├─ 상품 설명: 캐시
└─ 사용자 장바구니: 요청마다 동적 렌더링
```

`"use cache"` directive와 Suspense를 조합해 캐시된 shell을 먼저 제공하고 동적인 부분은 이후에 렌더링할 수 있습니다. 기존의 실험적 PPR 설정은 Cache Components 모델로 통합되었습니다.

### 캐시 갱신 API

- `revalidateTag(tag, profile)`: stale data를 제공하면서 background revalidation
- `updateTag(tag)`: Server Action 이후 변경 결과를 즉시 반영
- `refresh()`: Server Action 안에서 client router 새로고침
- `cacheLife`, `cacheTag`: stable API로 전환

### Turbopack 기본 적용

- `next dev`와 `next build` 모두 Turbopack을 기본으로 사용합니다.
- 기존 custom Webpack 설정은 Turbopack 호환성을 확인해야 합니다.
- Webpack을 계속 사용하려면 `--webpack` option을 명시할 수 있습니다.
- Next.js 16.1부터 개발용 filesystem cache가 stable 및 기본 활성화되었습니다.

### Middleware에서 Proxy로

`middleware.ts`라는 이름은 deprecated 되고 `proxy.ts`로 변경되었습니다. 이 기능이 모든 요청에 적용하는 일반 middleware라기보다 렌더링 이전의 network boundary에서 요청을 가로채는 역할임을 명확히 하기 위한 변경입니다.

`proxy.ts`는 Node.js runtime을 사용합니다. Edge runtime이 필요한 기존 Middleware 사용 사례는 migration 조건을 별도로 확인해야 합니다.

### Routing과 Navigation

- 공통 layout을 한 번만 가져오는 layout deduplication
- 캐시에 없는 segment만 가져오는 incremental prefetching
- invalidation된 link의 re-prefetch
- Parallel Route의 모든 named slot에 명시적인 `default.tsx` 요구

### React와 Compiler

- React 19.2 기반 기능 지원
- View Transitions
- `useEffectEvent`
- `<Activity>`
- React Compiler integration stable

React Compiler는 component와 value의 memoization을 자동화하는 방향이지만 기본으로 활성화되지는 않습니다.

### 개발 및 배포 도구

- Next.js DevTools MCP를 통한 AI coding agent의 route, log, error 접근
- `create-next-app`의 `AGENTS.md`와 browser log forwarding
- Turbopack Bundle Analyzer 추가
- Server Function logging과 hydration mismatch 표시 개선
- Adapter API stable을 통한 deployment platform integration 표준화
- `next dev --inspect`, `next start --inspect` 지원

### 주요 호환성 변화

- Node.js 최소 버전: 20.9
- TypeScript 최소 버전: 5.1
- synchronous `params`, `searchParams`, `cookies()`, `headers()` 접근 제거
- `next lint` 제거, ESLint 또는 Biome CLI 직접 사용
- AMP 지원 제거
- `next/legacy/image`와 `images.domains` deprecated
- `next/image`의 cache 및 security 기본값 강화

## 학습 우선순위

1. Async Request APIs
2. Next.js 15의 caching 기본값 변경
3. Cache Components와 `"use cache"`
4. PPR과 Suspense
5. React 19와 Server Actions
6. `revalidateTag`, `updateTag`, `refresh`
7. `middleware.ts`에서 `proxy.ts`로의 변화
8. Turbopack
9. Typed Routes와 Navigation API
10. React Compiler
11. `after()`와 instrumentation
12. DevTools MCP와 Adapter API

## 참고

- [Next.js 15](https://nextjs.org/blog/next-15)
- [Next.js 15 Upgrade Guide](https://nextjs.org/docs/app/guides/upgrading/version-15)
- [Next.js 15.3](https://nextjs.org/blog/next-15-3)
- [Next.js 15.5](https://nextjs.org/blog/next-15-5)
- [Next.js 16](https://nextjs.org/blog/next-16)
- [Next.js 16 Upgrade Guide](https://nextjs.org/docs/app/guides/upgrading/version-16)
- [Next.js 16.1](https://nextjs.org/blog/next-16-1)
- [Next.js 16.2](https://nextjs.org/blog/next-16-2)
