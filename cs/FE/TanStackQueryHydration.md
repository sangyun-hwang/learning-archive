# TanStack Query Hydration API

TanStack Query의 Hydration API는 Server의 QueryClient에서 미리 조회한 query cache를 직렬화해 Browser의 별도 QueryClient가 이어받도록 합니다.

React DOM hydration이 Server HTML에 state와 event handler를 연결하는 과정이라면, TanStack Query hydration은 Server에서 만든 query cache snapshot을 Browser cache에 복원하는 과정입니다.

```text
Server
1. 요청별 QueryClient 생성
2. prefetchQuery로 data 조회
3. dehydrate로 cache snapshot 생성

Server -> Browser
4. dehydratedState 전달

Browser
5. HydrationBoundary가 Browser QueryClient에 snapshot 복원
6. useQuery가 같은 queryKey의 cache 사용
```

Server와 Browser가 같은 QueryClient instance를 공유하는 것은 아닙니다. 서로 다른 instance를 만들고 초기 cache 상태만 한 번 전달합니다.

## QueryClient의 실행 환경

### Server

Server에서는 요청마다 새로운 QueryClient를 생성합니다.

```text
사용자 A 요청 -> QueryClient A
사용자 B 요청 -> QueryClient B
```

Server module 최상위의 QueryClient 하나를 모든 요청에서 공유하면 사용자별 cache가 섞이고 인증 data가 다른 사용자에게 전달될 수 있습니다. 요청이 반복되면서 memory에 cache가 누적될 가능성도 있습니다.

### Browser

Browser에서는 같은 QueryClient를 재사용합니다.

```text
최초 rendering -> Browser QueryClient 생성
재렌더링       -> 기존 QueryClient 재사용
Client 이동    -> 기존 cache 유지
```

렌더링마다 QueryClient를 만들면 기존 cache와 hydration된 data를 잃고 같은 API를 다시 요청할 수 있습니다. Mutation과 invalidation이 서로 다른 QueryClient를 대상으로 실행되는 문제도 생길 수 있습니다.

```tsx
// app/get-query-client.ts
import {
  environmentManager,
  QueryClient,
} from '@tanstack/react-query';

function makeQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000,
      },
    },
  });
}

let browserQueryClient: QueryClient | undefined;

export function getQueryClient() {
  if (environmentManager.isServer()) {
    return makeQueryClient();
  }

  if (!browserQueryClient) {
    browserQueryClient = makeQueryClient();
  }

  return browserQueryClient;
}
```

## Provider 설정

`QueryClientProvider`는 React Context를 사용하므로 Client Component에 둡니다. 이를 위해 root layout 전체를 Client Component로 바꿀 필요는 없습니다.

```tsx
// app/providers.tsx
'use client';

import { QueryClientProvider } from '@tanstack/react-query';
import { getQueryClient } from './get-query-client';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={getQueryClient()}>
      {children}
    </QueryClientProvider>
  );
}
```

```tsx
// app/layout.tsx - Server Component
import { Providers } from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

## Prefetch와 Dehydrate

Server Component에서 QueryClient를 만들고 필요한 query를 미리 가져옵니다.

```tsx
// app/posts/page.tsx - Server Component
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query';

export default async function PostsPage() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery({
    queryKey: ['posts'],
    queryFn: getPosts,
  });

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <Posts />
    </HydrationBoundary>
  );
}
```

각 API의 역할은 다음과 같습니다.

```text
prefetchQuery
-> Server QueryClient의 cache 채우기

dehydrate
-> QueryClient 자체가 아닌 전송 가능한 cache snapshot 생성

HydrationBoundary
-> snapshot을 Browser QueryClient의 cache에 복원
```

기본적으로 `prefetchQuery()`는 error를 throw하지 않고, `dehydrate()`는 성공한 query를 snapshot에 포함합니다. 중요한 query의 실패를 직접 처리해야 한다면 error를 throw하는 `fetchQuery()`를 고려할 수 있습니다.

## Client에서 Cache 사용

```tsx
// app/posts/Posts.tsx - Client Component
'use client';

import { useQuery } from '@tanstack/react-query';

export function Posts() {
  const { data } = useQuery({
    queryKey: ['posts'],
    queryFn: getPosts,
  });

  return data?.map((post) => <p key={post.id}>{post.title}</p>);
}
```

Server의 `prefetchQuery()`와 Client의 `useQuery()`가 같은 `queryKey`를 사용해야 같은 query로 인식합니다.

```text
Server: ['posts']
Client: ['posts']
-> hydration된 cache 사용

Server: ['posts']
Client: ['post-list']
-> 별개의 query로 판단하고 Client에서 새 요청 가능
```

## staleTime

TanStack Query의 기본 `staleTime`은 `0`이므로 hydration된 data도 즉시 stale로 판단될 수 있습니다. SSR에서는 `staleTime`을 `0`보다 크게 설정해 Browser가 hydration 직후 같은 API를 다시 요청하는 것을 줄일 수 있습니다.

```text
Hydration
-> 초기 cache 복원

staleTime
-> 복원한 cache를 언제까지 fresh하게 볼지 결정
```

Freshness는 Server에서 query를 가져온 시점을 기준으로 계산합니다. Hydration이 이후의 refetch를 영구적으로 막아주는 것은 아닙니다.

## 일회성 Snapshot 전달

Hydration은 Server와 Browser QueryClient를 지속적으로 동기화하지 않습니다.

```text
Server QueryClient
-> prefetch
-> cache snapshot 전달
-> 해당 요청에서 역할 종료

Browser QueryClient
-> snapshot으로 시작
-> 이후 refetch, mutation과 invalidation을 독립적으로 관리
```

Browser cache가 refetch로 변경되어도 과거 Server 요청에서 사용한 QueryClient가 함께 변경되지는 않습니다.

## 면접 답변

> TanStack Query의 Hydration API는 Server QueryClient에서 prefetch한 query cache를 `dehydrate()`로 직렬화하고, Browser의 별도 QueryClient가 `HydrationBoundary`를 통해 복원하는 방식입니다. Server에서는 사용자 cache가 섞이지 않도록 요청마다 QueryClient를 생성하고 Browser에서는 cache 유지를 위해 하나의 QueryClient를 재사용합니다. Client의 `useQuery`가 같은 queryKey를 사용하면 hydration된 cache를 읽을 수 있으며, 적절한 staleTime을 설정해 hydration 직후 불필요한 refetch를 줄일 수 있습니다. 두 QueryClient가 계속 동기화되는 것은 아니고 초기 cache snapshot을 전달하는 과정입니다.

## 참고

- [TanStack Query: Server Rendering & Hydration](https://tanstack.com/query/v5/docs/framework/react/guides/ssr#using-the-hydration-apis)
- [TanStack Query: Advanced Server Rendering](https://tanstack.com/query/latest/docs/framework/react/guides/advanced-ssr)
