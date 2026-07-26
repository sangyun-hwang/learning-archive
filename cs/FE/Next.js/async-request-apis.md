# Async Request APIs

Next.js의 Async Request APIs는 요청이 들어와야 결정되는 정보를 Promise로 제공하는 방식입니다. Next.js 15에서 도입되었고, Next.js 16부터 동기 접근이 제거되어 반드시 비동기로 읽어야 합니다.

## 대상 API

### Request 관련 함수

```tsx
import { cookies, draftMode, headers } from 'next/headers';

const cookieStore = await cookies();
const headerStore = await headers();
const { isEnabled } = await draftMode();
```

- `cookies()`: 요청 cookie
- `headers()`: 요청 header
- `draftMode()`: Draft Mode 활성화 상태

### Route 관련 Props

```tsx
export default async function Page({
  params,
  searchParams,
}: {
  params: Promise<{ id: string }>;
  searchParams: Promise<{ sort?: string }>;
}) {
  const { id } = await params;
  const { sort } = await searchParams;

  return <div>{`${id}: ${sort ?? 'default'}`}</div>;
}
```

- `params`: `/posts/[id]` 같은 dynamic segment 값
- `searchParams`: `?sort=latest` 같은 query string

`params`는 `page.tsx`, `layout.tsx`, `route.ts`, `default.tsx`와 metadata 관련 함수 등에 전달됩니다. `searchParams` prop은 `page.tsx`에 제공됩니다.

Layout은 client navigation마다 반드시 다시 렌더링되지 않습니다. 따라서 Layout에 `searchParams`가 전달되면 이전 query 값을 유지할 수 있으므로, page에서 필요한 값을 전달하거나 Client Component에서 `useSearchParams()`를 사용합니다.

## 비동기로 변경된 이유

기존의 동기 API는 코드를 실행하는 시점에 요청 정보가 이미 준비되었다고 가정했습니다.

```text
페이지 렌더링 시작
-> 요청 정보 확인
-> 나머지 렌더링
```

비동기 API를 사용하면 요청 정보가 필요하지 않은 작업과 요청에 의존하는 작업을 구분할 수 있습니다.

```text
요청과 무관한 UI 준비 --------┐
                             ├─ 최종 렌더링
요청 도착 -> 요청 정보 확인 --┘
```

Next.js는 요청과 무관한 부분을 먼저 준비하고 `params`, cookie, header처럼 요청에 의존하는 부분만 기다릴 수 있습니다. 이는 정적 shell과 동적 영역을 나누는 Cache Components 및 PPR 렌더링 모델과도 연결됩니다.

## await params는 외부 요청이 아니다

```tsx
const { id } = await params;
```

`await params`는 외부 API 요청을 추가로 보내는 코드가 아닙니다. Next.js가 전달한 Promise에서 route 정보를 꺼냅니다. 값이 이미 준비되었다면 바로 다음 코드를 실행할 수 있습니다.

핵심은 `await`의 실행 비용이 아니라 코드가 요청 정보에 의존하기 시작하는 지점을 명시하는 것입니다.

## Server Component

Server Component는 component function을 `async`로 선언하고 Promise를 `await`합니다.

```tsx
export default async function ProductPage({
  params,
}: PageProps<'/products/[id]'>) {
  const { id } = await params;
  const product = await getProduct(id);

  return <ProductDetail product={product} />;
}
```

`PageProps<'/products/[id]'>`는 route 구조를 기반으로 생성되는 global type helper입니다.

## Client Component

Client Component는 component function을 `async`로 선언할 수 없으므로 필요한 경우 React의 `use()`로 Promise를 읽습니다.

> React `use()`의 동작은 [React use API](../ReactUse.md)에서 자세히 정리합니다.

```tsx
'use client';

import { use } from 'react';

export default function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = use(params);

  return <div>{id}</div>;
}
```

일반적으로는 Server Component에서 먼저 Promise를 해결하고 Client Component에는 필요한 값만 전달하는 편이 단순합니다.

```tsx
export default async function Page({
  params,
}: PageProps<'/posts/[slug]'>) {
  const { slug } = await params;

  return <PostClient slug={slug} />;
}
```

이렇게 분리하면 Client Component가 Promise와 Suspense를 직접 다룰 필요가 없고, server와 client의 책임 및 전달 데이터가 명확해집니다. Client Component의 재사용과 테스트도 쉬워집니다.

## Route Handler

Route Handler에는 `PageProps`가 아니라 `RouteContext`를 사용합니다.

```tsx
export async function GET(
  request: Request,
  { params }: RouteContext<'/api/posts/[id]'>,
) {
  const { id } = await params;

  return Response.json({ id });
}
```

```text
PageProps
-> Page

LayoutProps
-> Layout

RouteContext
-> Route Handler
```

## Metadata

`generateMetadata()`에서도 `params`를 비동기로 읽습니다.

```tsx
export async function generateMetadata({
  params,
}: PageProps<'/posts/[slug]'>) {
  const { slug } = await params;

  return {
    title: `게시글: ${slug}`,
  };
}
```

Migration 시 `page.tsx`뿐 아니라 Layout, Route Handler, metadata 및 image 생성 함수도 확인해야 합니다.

## 정적 경로와 동적 경로

`params`가 Promise라는 사실만으로 route가 항상 동적 렌더링되는 것은 아닙니다. `generateStaticParams()`를 사용하면 경로를 build 시점에 미리 만들 수 있습니다.

```text
정적 경로
generateStaticParams()
-> build 또는 prerender 과정에서 params Promise 해결

동적 경로
사용자 요청
-> request 과정에서 params Promise 해결
```

해결되는 시점은 다르지만 component는 정적 및 동적 route에서 모두 동일한 Promise 형태로 `params`를 전달받습니다. Route의 생성 방식이 바뀌어도 component API를 동일하게 유지할 수 있습니다.

`cookies()`와 `headers()`는 사용자 요청마다 값이 달라질 수 있으므로 build 시점에 모든 사용자를 위한 하나의 결과로 확정할 수 없습니다.

## 독립적인 비동기 작업

여러 데이터 요청이 같은 `id`를 필요로 하지만 서로 의존하지 않는다면 `params`를 먼저 해결한 뒤 동시에 실행합니다.

```tsx
const { id } = await params;

const [product, reviews] = await Promise.all([
  getProduct(id),
  getReviews(id),
]);
```

상품 조회가 끝나야 리뷰 조회를 시작할 이유가 없으므로 불필요한 request waterfall을 줄일 수 있습니다.

## Migration 확인 범위

Next.js 14에서 16으로 migration할 때 다음 위치를 확인합니다.

- `page.tsx`
- `layout.tsx`
- `default.tsx`
- `route.ts`
- `generateMetadata()`
- `generateViewport()`
- `opengraph-image.tsx`
- `twitter-image.tsx`
- `icon.tsx`, `apple-icon.tsx`
- 여러 sitemap을 생성하는 함수의 `id`
- `cookies()`, `headers()`, `draftMode()`를 사용하는 Server Component, Server Action, Route Handler

Route type은 다음 명령으로 생성할 수 있습니다.

```bash
next typegen
```

공식 codemod는 기존의 동기 접근을 `await` 또는 React `use()` 방식으로 변환합니다.

```bash
npx @next/codemod@canary next-async-request-api .
```

자동 변환할 수 없는 코드는 `@next-codemod-error` 주석이 남을 수 있습니다. 이 경우 호출 위치를 수동으로 검토하고 비동기 접근으로 변경해야 합니다.

## 정리

> Async Request APIs는 단순히 `await` 문법을 추가한 변경이 아닙니다. Next.js가 요청과 무관한 작업을 먼저 준비하고, route, query, cookie, header처럼 요청에 의존하는 지점을 구분하기 위한 렌더링 구조의 변화입니다.

## 참고

- [Next.js 15 Upgrade Guide](https://nextjs.org/docs/app/guides/upgrading/version-15)
- [Next.js 16 Upgrade Guide](https://nextjs.org/docs/app/guides/upgrading/version-16)
- [Dynamic APIs are Asynchronous](https://nextjs.org/docs/messages/sync-dynamic-apis)
- [Next.js Codemods](https://nextjs.org/docs/app/guides/upgrading/codemods)
