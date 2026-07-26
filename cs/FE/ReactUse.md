# React use API

`use()`는 React 19에서 정식으로 추가된 API로, rendering 중 Promise 또는 Context의 값을 읽습니다.

```tsx
const value = use(resource);
```

`use()`는 비동기 요청을 직접 생성하거나 Promise를 취소하는 API가 아닙니다. 이미 존재하는 resource의 상태를 React rendering과 연결합니다.

## Promise 상태에 따른 동작

Promise를 `use()`에 전달하면 현재 상태에 따라 다르게 동작합니다.

```text
fulfilled
-> 해결된 값 반환

pending
-> 현재 component rendering 중단
-> 가장 가까운 Suspense fallback 표시

rejected
-> 가장 가까운 Error Boundary로 error 전달
```

Pending Promise가 해결되면 React는 fallback을 실제 UI로 교체할 수 있도록 해당 component의 rendering을 다시 시도합니다.

## Promise 읽기

```tsx
'use client';

import { use } from 'react';

type User = {
  id: string;
  name: string;
};

export function UserProfile({
  userPromise,
}: {
  userPromise: Promise<User>;
}) {
  const user = use(userPromise);

  return <h1>{user.name}</h1>;
}
```

`use()`가 Promise를 읽는 component를 Suspense로 감싸면 pending 상태의 UI를 지정할 수 있습니다.

```tsx
<Suspense fallback={<UserSkeleton />}>
  <UserProfile userPromise={userPromise} />
</Suspense>
```

`use()`가 요청을 시작하는 것이 아니라 `userPromise`를 생성한 코드가 비동기 작업을 시작합니다. `use()`는 그 Promise의 현재 상태를 읽습니다.

## Server Component의 await과 Client Component의 use

Server Component는 component function을 `async`로 선언하고 Promise를 `await`할 수 있습니다.

```tsx
export default async function Page() {
  const user = await getUser();

  return <UserProfile user={user} />;
}
```

Client Component는 rendering function을 `async`로 만들 수 없으므로 전달받은 Promise를 `use()`로 읽을 수 있습니다.

```text
Server Component
-> await promise

Client Component
-> use(promise)
```

일반적으로는 Promise를 생성한 Server Component에서 바로 `await`하는 방식이 가장 단순합니다.

## Promise를 해결하지 않고 전달하는 경우

일부 UI만 data를 기다리게 하고 다른 UI를 먼저 보여주고 싶다면 Server Component에서 Promise를 바로 `await`하지 않고 하위 component에 전달할 수 있습니다.

```tsx
// Server Component
export default function Page() {
  const userPromise = getUser();

  return (
    <>
      <Header />

      <Suspense fallback={<UserSkeleton />}>
        <UserProfile userPromise={userPromise} />
      </Suspense>
    </>
  );
}
```

```text
Header
-> 즉시 rendering

UserProfile
-> Promise가 pending이면 UserSkeleton
-> Promise가 해결되면 실제 사용자 정보
```

Suspense boundary를 기준으로 기다리는 UI의 범위를 제한할 수 있습니다. Server Component에서 Client Component로 Promise를 전달할 때 해결된 값은 serialization 가능한 값이어야 합니다.

## 안정적인 Promise 재사용

`use()`에 전달하는 Promise는 re-render 사이에 같은 instance가 재사용되어야 합니다.

```tsx
function UserProfile() {
  // 잘못된 예: rendering할 때마다 새로운 Promise가 생성된다.
  const user = use(fetch('/api/user'));

  return <h1>{user.name}</h1>;
}
```

Rendering마다 새로운 Promise를 만들면 React는 매번 새로운 작업으로 판단해 component가 반복적으로 suspend될 수 있습니다.

다음과 같이 안정적인 Promise를 사용합니다.

- Server Component에서 생성해 prop으로 전달
- React framework가 제공하는 data fetching 및 cache 사용
- Suspense를 지원하는 library의 cache 사용

직접 Promise cache를 구현할 수도 있지만 일반 application에서는 framework나 library가 제공하는 caching 및 invalidation 방식을 우선 사용합니다.

## Suspense와 Error Boundary

Suspense는 pending 상태를 처리하고 Error Boundary는 rejected 상태를 처리합니다.

```tsx
<ErrorBoundary fallback={<ErrorMessage />}>
  <Suspense fallback={<Loading />}>
    <UserProfile userPromise={userPromise} />
  </Suspense>
</ErrorBoundary>
```

`use()`를 `try...catch` 안에서 호출하면 안 됩니다. React는 Promise 대기 상태를 Suspense와 연결하기 위해 rendering 제어 흐름을 사용하므로, `use()` 주변에서 이를 일반 error처럼 가로채면 Suspense 동작이 깨질 수 있습니다.

```text
Promise pending
-> Suspense

Promise rejected
-> Error Boundary
```

## Context 읽기

`use()`는 Promise뿐 아니라 Context도 읽을 수 있습니다.

```tsx
const theme = use(ThemeContext);
```

`use(Context)`는 가장 가까운 Provider의 값을 읽고 Provider 값이 바뀌면 component를 다시 rendering한다는 점에서 `useContext(Context)`와 같습니다.

차이는 호출 규칙입니다.

```text
useContext(Context)
-> Context만 읽음
-> Hook이므로 component 최상위에서 호출

use(resource)
-> Promise 또는 Context를 읽음
-> 조건문과 반복문에서도 호출 가능
```

```tsx
function Heading({ show }: { show: boolean }) {
  if (!show) return null;

  const theme = use(ThemeContext);

  return <h1 className={theme}>제목</h1>;
}
```

`use(Context)`는 Server Component에서 지원되지 않습니다.

## use는 Hook인가

`use()`는 이름과 호출 위치가 Hook과 비슷하지만 공식적으로 Hook은 아닙니다. 따라서 일반 Hook과 달리 조건문이나 반복문 안에서 호출할 수 있습니다.

하지만 React의 rendering 과정과 연결되어야 하므로 Component 또는 custom Hook 안에서만 호출해야 합니다.

React는 `use()`가 호출된 rendering context를 바탕으로 다음 작업을 수행합니다.

- 어떤 component가 Promise를 기다리는지 추적
- 가장 가까운 Suspense boundary 탐색
- Promise 해결 후 component rendering 재시도
- Context와 Provider 변경 연결

일반 JavaScript 함수에는 현재 rendering 중인 component context가 없으므로 `use()`를 호출할 수 없습니다.

## 다른 API와 비교

```text
useState
-> component 상태 관리

useEffect
-> rendering 이후 외부 시스템과 동기화

useContext
-> Context 값 읽기와 구독

use
-> rendering 중 Promise 또는 Context 값 읽기
```

## 정리

> React의 `use()`는 rendering 중 이미 존재하는 Promise 또는 Context를 읽는 API입니다. Promise가 pending이면 가장 가까운 Suspense fallback을 보여주고, fulfilled이면 값을 반환하며, rejected이면 Error Boundary로 전달합니다. Hook은 아니지만 React rendering context가 필요하므로 Component 또는 custom Hook 안에서만 호출할 수 있습니다.

## 참고

- [React: use](https://react.dev/reference/react/use)
- [React: Suspense](https://react.dev/reference/react/Suspense)
- [React 19](https://react.dev/blog/2024/12/05/react-19)
- [Next.js: Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
