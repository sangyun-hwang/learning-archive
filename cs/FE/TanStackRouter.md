# TanStack Router

## 개념

TanStack Router는 React와 Solid Application을 위한 Router다. URL 이동뿐 아니라 Route, Path Params, Search Params, Loader Data와 Context를 하나의 Route Tree에서 TypeScript로 연결하는 데 강점이 있다.

```text
URL 변경
-> 일치하는 Route 탐색
-> Params와 Search Params 검증
-> beforeLoad와 Loader 실행
-> Component Rendering
```

File-based Routing과 Code-based Routing, 중첩 Layout, Prefetch, Pending UI, Error Boundary와 SSR 등을 지원한다.

## Type-safe Routing

TanStack Router는 전체 Route Tree의 타입을 알고 있어 Link와 Navigation을 코드 작성 시점에 검사할 수 있다.

```tsx
<Link to="/posts/$postId" params={{ postId: '123' }}>
  Post
</Link>
```

다음과 같은 실수를 TypeScript로 확인할 수 있다.

- 존재하지 않는 Route로 이동
- 필수 Path Params 누락
- Params와 Search Params의 잘못된 타입
- 해당 Route에 없는 Loader Data나 Context 사용

Route 경로를 변경하면 관련 Link와 Navigation에서 타입 오류가 발생하므로 Refactoring할 때 영향을 받는 코드를 찾기 쉽다.

## File-based Routing

```text
src/routes/
├─ __root.tsx
├─ index.tsx
├─ posts.tsx
└─ posts.$postId.tsx
```

```text
index.tsx
-> /

posts.tsx
-> /posts

posts.$postId.tsx
-> /posts/:postId
```

Vite Plugin이나 Router CLI가 Route 파일 구조를 읽어 `routeTree.gen.ts`를 생성한다. Router는 이 파일을 통해 전체 Route 타입을 추론한다. 생성 도구를 다시 실행하면 내용이 덮어써질 수 있으므로 일반적으로 직접 수정하지 않는다.

## Path Params와 Search Params

Path Params는 어떤 Resource인지 식별하는 값에 적합하다. Search Params는 같은 Resource나 목록을 검색, 정렬, 필터링 또는 Pagination하는 상태에 적합하다.

```text
/products/123?sort=price&page=2

Path Param
-> productId: 123

Search Params
-> sort: price
-> page: 2
```

Search Params를 URL에서 관리하면 새로고침, Bookmark, URL 공유와 Browser History 이동 후에도 화면 상태를 복원할 수 있다.

TanStack Router는 Search Params를 타입이 있는 URL 상태로 다룬다.

```tsx
export const Route = createFileRoute('/products')({
  validateSearch: (search) => ({
    page: Number(search.page) || 1,
    sort: search.sort === 'price' ? 'price' : 'latest',
  }),
});
```

URL은 사용자가 직접 수정할 수 있는 외부 입력이다. TypeScript 타입은 Runtime 입력을 검증하지 않으므로 `validateSearch` 또는 Zod 같은 Schema를 사용해 변환하고 검증해야 한다.

## Route Loader

Loader는 Route를 표시하는 데 필요한 Data를 준비한다.

```tsx
export const Route = createFileRoute('/posts/$postId')({
  loader: ({ params }) => fetchPost(params.postId),
  component: PostPage,
});

function PostPage() {
  const post = Route.useLoaderData();
  return <h1>{post.title}</h1>;
}
```

Component가 Mount된 뒤 `useEffect`에서 조회하는 것과 달리 Router는 Navigation 전에 Route의 Data 요구사항을 알 수 있다. 이를 바탕으로 Route 진입 전 조회, Link Prefetch, Pending UI, Error 처리와 Params별 Cache 구분을 구성할 수 있다.

Loader를 사용했다고 모든 요청이 자동으로 병렬 실행되거나 Backend API가 안전해지는 것은 아니다. 서로 의존하지 않는 작업은 Data 흐름에 맞게 함께 시작하고, Backend에서는 인증과 입력 검증을 별도로 수행해야 한다.

## `beforeLoad`와 인증

`beforeLoad`는 Loader와 Component Rendering 전에 실행되어 인증 상태 확인, Redirect와 Route Context 구성 등에 사용할 수 있다.

```text
Navigation
-> beforeLoad에서 인증 상태 확인
-> Loader에서 Data 준비
-> Component Rendering
```

Client Router에서 화면 접근을 막아도 사용자가 Backend API를 직접 호출할 수 있다. 실제 Data 접근 권한은 Backend에서 다시 인증하고 인가해야 한다.

## TanStack Query와의 차이

두 Library는 이름이 비슷하지만 담당 범위가 다르다.

```text
TanStack Router
-> URL과 Route Matching
-> Navigation
-> Route별 Data 준비 시점

TanStack Query
-> Server State Cache
-> Refetch와 Mutation
-> Invalidation, staleTime과 Garbage Collection
```

간단한 Route Data는 Router Loader와 내장 Cache로 관리할 수 있다. 여러 화면에서 공유하거나 Mutation과 세밀한 Cache 갱신이 필요한 Server State는 TanStack Query와 결합할 수 있다.

```tsx
loader: ({ context, params }) =>
  context.queryClient.ensureQueryData(postQueryOptions(params.postId))
```

Loader가 Query Cache를 미리 준비하고 Component가 같은 Query Key를 사용하면 준비된 Data를 재사용할 수 있다.

## 다른 Router와의 범위

```text
React Router
-> React의 범용 Routing과 Data Router

TanStack Router
-> Route와 URL 상태의 강한 TypeScript 추론에 집중

Next.js App Router
-> Routing, Server Component, Rendering, Server Action과 Build 포함
```

TanStack Router 자체는 Next.js와 동일한 범위의 Full-stack Framework가 아니다. TanStack 생태계에서 Full-stack 구성이 필요하면 Router를 기반으로 하는 TanStack Start를 선택할 수 있다.

## 핵심 정리

> TanStack Router는 Route Tree를 기반으로 Path, Params, Search Params, Loader Data와 Navigation의 타입을 연결한다. Resource 식별에는 Path Params를, 검색과 정렬처럼 공유할 화면 상태에는 Search Params를 사용한다. Route 진입에 필요한 Data는 Loader에서 준비하고, 복잡한 Server State Cache는 TanStack Query와 결합할 수 있다. Client Route Guard와 별개로 Backend의 인증과 인가는 반드시 유지해야 한다.

## 참고 자료

- [TanStack Router: Overview](https://tanstack.com/router/latest/docs/overview)
- [TanStack Router: Type Safety](https://tanstack.com/router/latest/docs/guide/type-safety)
- [TanStack Router: Search Params](https://tanstack.com/router/latest/docs/guide/search-params)
