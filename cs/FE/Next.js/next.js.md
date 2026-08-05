# next.js

- [CSR, SSR, SSG](#csr-ssr-ssg)
- [Server Component와 Client Component](#server-component와-client-component)
- [Server Component와 SSR](#server-component와-ssr)
- [Streaming](#streaming)
- [next/link](#nextlink)
- [next/router](#nextrouter)
- [next/image](#nextimage)
- [Data fetching](#data-fetching)

<br>

## CSR, SSR, SSG

세 rendering 방식은 HTML을 언제, 어디서 생성하는지를 기준으로 구분합니다.

### CSR

Client-Side Rendering은 browser가 JavaScript를 실행해 HTML을 생성합니다.

```text
Browser가 HTML shell과 JavaScript 수신
-> JavaScript 다운로드 및 실행
-> API data 요청
-> 화면 rendering
```

초기 HTML에는 metadata, script와 application의 root element가 주로 포함됩니다.

```html
<div id="root"></div>
<script src="/app.js"></script>
```

초기 loading 이후 화면 전환과 interaction에 유리하지만, JavaScript와 data가 준비되기 전에는 실제 content가 부족할 수 있습니다. 관리자 dashboard나 사내 도구처럼 SEO보다 interaction이 중요한 화면에 적합합니다.

### SSR

Server-Side Rendering은 사용자 request마다 server가 HTML을 생성합니다.

```text
사용자 request
-> Server가 data 조회
-> Server가 HTML 생성
-> Browser에 content가 포함된 HTML 전달
-> JavaScript hydration
```

초기 HTML에 content가 포함되어 SEO에 유리하고 cookie처럼 request 시점에만 알 수 있는 정보와 사용자별 data를 반영할 수 있습니다. 대신 request마다 data 조회와 server rendering 비용이 발생합니다.

### SSG

Static Site Generation은 build 시점에 HTML을 미리 생성합니다.

```text
Build
-> Data 조회
-> HTML 미리 생성
-> CDN에 배포

사용자 request
-> 이미 생성된 HTML 전달
-> JavaScript hydration
```

Request 시 HTML을 다시 생성하지 않아 response가 빠르고 server 부하가 적습니다. 회사 소개, 문서, 변경이 적은 blog post처럼 여러 사용자에게 같은 content를 제공하는 page에 적합합니다.

Build 이후 server data가 바뀌어도 기존 HTML에는 바로 반영되지 않으므로 새로운 content를 반영하려면 다시 생성하는 과정이 필요합니다.

### 비교

| 구분 | CSR | SSR | SSG |
| --- | --- | --- | --- |
| HTML 생성 위치 | Browser | Server | Server |
| HTML 생성 시점 | JavaScript 실행 시점 | Request 시점 | Build 시점 |
| 초기 HTML | Content가 적은 shell | Content 포함 | Content 포함 |
| Data 최신성 | Client 요청으로 반영 | Request마다 반영 가능 | 다시 생성하기 전까지 고정 |
| Server rendering 비용 | 적음 | Request마다 발생 | Build 때 발생 |
| SEO | 상대적으로 불리 | 유리 | 유리 |
| 대표 사례 | 관리자 dashboard | 사용자별 page | 소개, 문서, blog |

### SSR과 SSG의 공통점

SSR과 SSG는 browser에 content가 포함된 HTML을 먼저 전달하는 pre-rendering 방식입니다.

```text
SSR
-> Request 시 pre-rendering

SSG
-> Build 시 pre-rendering
```

검색 crawler가 JavaScript 실행에 의존하지 않고 초기 HTML에서 content를 확인할 수 있어 CSR보다 SEO에 유리할 수 있습니다. CSR도 검색이 불가능한 것은 아니지만 JavaScript 실행과 API 요청이 추가로 필요합니다.

### Hydration

Hydration은 server에서 생성한 HTML을 browser에서 재사용하면서 React component tree, state와 event handler를 연결해 interactive한 application으로 만드는 과정입니다.

Server HTML을 삭제하고 client에서 DOM을 전부 다시 만드는 것이 아니라 기존 DOM에 React를 연결합니다.

```text
1. Server가 React component를 HTML로 rendering
2. Browser가 HTML을 받아 화면에 표시
3. React JavaScript bundle 다운로드
4. React가 기존 HTML과 component tree 연결
5. State와 event handler 활성화
```

#### HTML 표시와 Interaction

Server HTML을 받은 직후에는 content와 element가 보이지만 React interaction은 아직 준비되지 않을 수 있습니다.

```text
Server가 생성한 HTML
-> Content와 element 표시

JavaScript hydration 완료
-> React event handler 연결
-> Application interaction 활성화
```

HTML 자체 기능인 link 이동, form 제출과 focus는 JavaScript 없이도 동작할 수 있습니다. React의 `onClick`처럼 application에서 작성한 event handler는 hydration 이후에 동작합니다.

JavaScript bundle이 크거나 main thread 작업이 무거우면 화면은 보이지만 button 반응이 늦을 수 있습니다.

#### CSR Rendering과 차이

CSR은 비어 있는 root에 React가 DOM을 처음 생성합니다.

```tsx
createRoot(document.getElementById('root')).render(<App />);
```

SSR이나 SSG에서는 이미 server가 만든 HTML이 있으므로 React를 연결합니다.

```tsx
hydrateRoot(document.getElementById('root'), <App />);
```

```text
CSR
-> 기존 HTML이 없음
-> React가 DOM 생성

Hydration
-> Server HTML이 이미 있음
-> 기존 DOM을 재사용하며 React 연결
```

Next.js 같은 framework에서는 `hydrateRoot()` 호출을 내부에서 처리합니다.

#### Next.js App Router

Next.js의 최초 page load는 HTML, RSC Payload와 Client Component JavaScript를 사용합니다.

```text
HTML
-> 빠른 non-interactive 초기 화면 표시

RSC Payload
-> Server와 Client Component tree 조정

JavaScript
-> Client Component hydration
-> Interaction 활성화
```

모든 Component가 hydration되는 것은 아닙니다.

```text
Server Component
-> Server에서 실행
-> Client JavaScript가 전달되지 않음
-> Hydration 대상이 아님

Client Component
-> 최초 HTML pre-rendering에 참여할 수 있음
-> Browser에서 hydration
-> State와 event 활성화
```

`'use client'`가 붙었다고 최초 화면에서 server rendering에 전혀 참여하지 않는다는 뜻은 아닙니다. 최초 page load에서는 Client Component도 HTML preview 생성에 참여할 수 있고, browser에서 hydration되어 interaction이 활성화됩니다.

#### SSR과 SSG

HTML 생성 시점은 다르지만 React interaction이 있다면 둘 다 hydration이 필요합니다.

```text
SSR
-> Request 시 HTML 생성
-> Browser에서 hydration

SSG
-> Build 시 HTML 생성
-> Browser에서 hydration
```

#### Hydration Mismatch

Server가 만든 HTML과 client의 최초 rendering 결과가 다르면 hydration mismatch가 발생합니다.

대표적인 원인은 다음과 같습니다.

- `Date.now()`나 `Math.random()`처럼 실행마다 달라지는 값
- Rendering 중 `window`, `localStorage` 같은 browser API 사용
- Server와 client가 서로 다른 data 사용
- 잘못된 HTML tag 중첩
- Browser extension이 HTML 변경

`Date.now()`는 server rendering과 client hydration의 실행 시점이 다르므로 값이 달라질 수 있습니다. `localStorage`는 server에 존재하지 않고 client에만 저장된 값이므로 최초 UI에 바로 사용하면 server HTML과 다른 결과가 만들어질 수 있습니다.

Browser에서만 필요한 값은 hydration 이후 Effect에서 읽을 수 있습니다.

```tsx
'use client';

export function ClientValue() {
  const [value, setValue] = useState<string | null>(null);

  useEffect(() => {
    setValue(localStorage.getItem('value'));
  }, []);

  return <p>{value}</p>;
}
```

Server rendering 결과와 client의 최초 rendering 결과를 동일하게 유지하는 것이 핵심입니다.

#### Hydration 면접 답변

> Hydration은 server에서 rendering한 HTML을 browser에서 재사용하면서 React component tree와 state, event handler를 연결해 interactive한 application으로 만드는 과정입니다. Next.js App Router에서는 Client Component도 최초 HTML 생성에 참여할 수 있지만 browser에서 hydration되어야 interaction이 활성화됩니다. 반면 Server Component는 server에서만 실행되고 client JavaScript가 전달되지 않으므로 hydration되지 않습니다. Server HTML과 client 최초 rendering 결과가 다르면 hydration mismatch가 발생하므로 두 환경의 초기 결과를 동일하게 유지해야 합니다.

### Rendering 방식 선택

한 application이 하나의 방식만 선택할 필요는 없습니다.

```text
회사 소개
-> SSG

사용자 주문 내역
-> SSR

관리자 dashboard
-> CSR
```

실제 application에서는 page 요구사항에 따라 여러 방식을 조합합니다. 현대 Next.js는 route 또는 component 수준에서 static, cached, dynamic content를 함께 구성할 수 있습니다.

### 면접 답변

> CSR은 browser가 JavaScript를 실행해 HTML을 만드는 방식이고, SSR은 request마다 server가 HTML을 생성하는 방식이며, SSG는 build 시점에 HTML을 미리 생성하는 방식입니다. CSR은 interaction이 많은 application에 적합하지만 초기 content와 SEO에 상대적으로 불리할 수 있습니다. SSR은 최신 data와 사용자별 화면에 유리하지만 request마다 server 비용이 발생합니다. SSG는 빠르고 server 부하가 적지만 build 이후 변경된 data가 바로 반영되지 않습니다. 따라서 page 요구사항에 따라 rendering 방식을 조합합니다.

### 참고

- [Next.js: Rendering Strategies](https://nextjs.org/learn/seo/rendering-strategies)
- [Next.js: Static and Dynamic Rendering](https://nextjs.org/learn/dashboard-app/static-and-dynamic-rendering)
- [React: hydrateRoot](https://react.dev/reference/react-dom/client/hydrateRoot)
- [Next.js: Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- [Next.js: Hydration Error](https://nextjs.org/docs/messages/react-hydration-error)

## Server Component와 Client Component

Next.js App Router에서는 Component를 Server Component와 Client Component로 나누어 Server와 Browser가 맡을 일을 구분합니다. `page.tsx`와 `layout.tsx`를 포함한 Component는 기본적으로 Server Component입니다.

| 구분 | Server Component | Client Component |
| --- | --- | --- |
| 선언 | App Router의 기본값 | 파일 상단에 `'use client'` 선언 |
| 주 실행 환경 | Server | Browser |
| Client JavaScript | Component 코드가 bundle에 포함되지 않음 | Component 코드가 bundle에 포함됨 |
| Data 접근 | DB, filesystem, Server 전용 자원에 접근 가능 | API나 Server Function 등을 통해 접근 |
| State와 Effect | `useState`, `useEffect` 사용 불가 | 사용 가능 |
| Event handler | `onClick`, `onChange` 사용 불가 | 사용 가능 |
| Browser API | `window`, `localStorage` 사용 불가 | 사용 가능 |
| Hydration | 대상이 아님 | Browser에서 hydration됨 |

### Server Component

Server Component는 Server에서 실행되고 그 결과가 Client에 전달됩니다.

```tsx
export default async function ProductPage() {
  const product = await db.product.findUnique({
    where: { id: 1 },
  });

  return <h1>{product?.name}</h1>;
}
```

다음과 같은 작업에 적합합니다.

- DB나 내부 API에서 data 조회
- API key 등 Server 전용 정보 사용
- 정적인 content와 layout rendering
- Client에 전달할 JavaScript 감소

Server Component는 Browser에 살아 있는 Component가 아니므로 state, Effect와 event handler를 사용할 수 없습니다.

Server Component를 선언하기 위해 `'use server'`를 붙이지 않습니다. `'use server'`는 Client에서 호출할 수 있는 Server Function을 표시하는 directive입니다.

### Client Component

State, 사용자 interaction 또는 Browser API가 필요하다면 Client Component를 사용합니다.

```tsx
'use client';

import { useState } from 'react';

export function LikeButton() {
  const [liked, setLiked] = useState(false);

  return (
    <button onClick={() => setLiked((value) => !value)}>
      {liked ? '좋아요 취소' : '좋아요'}
    </button>
  );
}
```

다음과 같은 작업에 필요합니다.

- `useState`, `useEffect` 등 Client Hook 사용
- `onClick`, `onChange` 등 사용자 event 처리
- `window`, `document`, `localStorage` 사용
- Browser에서 동작하는 library 사용

### Client Component도 pre-rendering될 수 있음

`'use client'`는 Server Rendering을 끄는 표시가 아닙니다. 최초 page load에서 Client Component도 Server가 초기 HTML을 만드는 데 참여할 수 있습니다.

```text
Server에서 초기 HTML 생성
-> Browser에 HTML 표시
-> Client Component JavaScript 다운로드
-> 기존 HTML에 React state와 event handler 연결
```

초기 HTML을 Server에서 만들 수 있어도 Component 코드가 Client bundle에 포함되고 Browser에서 실행되므로 Client Component라고 부릅니다.

### `'use client'` 경계

`'use client'`는 파일의 가장 위에서 import보다 먼저 작성합니다. Component 함수 안이나 특정 함수 위에 붙이는 표시가 아닙니다.

```tsx
'use client';

import { useState } from 'react';
```

`'use client'`는 해당 파일만 표시하는 것이 아니라 Client module graph의 경계를 만듭니다. 그 파일에서 import한 module도 Client bundle에 포함될 수 있습니다.

```text
Server Component
└── 'use client' LikeButton
    ├── Icon
    └── client-utils
```

`Icon`과 `client-utils`는 Client 경계 안에서 import되므로 각 파일에 `'use client'`를 반복해서 작성할 필요가 없습니다.

최상위 `page.tsx`에 `'use client'`를 붙이면 하위 import까지 Client 영역이 넓어질 수 있습니다. 따라서 상호작용이 시작되는 작은 경계에 선언하는 것이 좋습니다.

### Server와 Client 조합

상품 상세 정보는 Server Component에서 조회하고, 상호작용이 필요한 버튼만 Client Component로 분리할 수 있습니다.

```tsx
// Server Component
export default async function ProductPage() {
  const product = await getProduct();

  return (
    <main>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <AddToCartButton productId={product.id} />
    </main>
  );
}
```

이렇게 구성하면 상품 조회는 Server에서 처리하고, Browser에는 장바구니 interaction에 필요한 JavaScript만 전달할 수 있습니다.

### Server Component를 children으로 전달

Client Component에서 Server Component를 직접 import해 실행하는 대신, Server Component에서 구성한 JSX를 `children`으로 전달할 수 있습니다.

```tsx
// Server Component
export default function Page() {
  return (
    <ClientModal>
      <ServerProductList />
    </ClientModal>
  );
}
```

```tsx
// ClientModal.tsx
'use client';

export function ClientModal({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}
```

`ClientModal`은 열고 닫는 interaction을 담당하고 `ServerProductList`는 Server에서 실행됩니다. 화면의 부모·자식 관계가 아니라 어떤 module이 누구를 import하는지가 Client 경계를 결정합니다.

### Props 전달

Server Component에서 Client Component로 전달하는 props는 React가 직렬화할 수 있어야 합니다. 문자열, 숫자와 일반적인 객체는 전달할 수 있지만 DB connection이나 일반 event handler 함수 같은 Server 객체는 그대로 전달할 수 없습니다. React가 지원하는 Server Function은 별도 규칙을 통해 전달할 수 있습니다.

또한 직렬화할 수 있다는 이유만으로 조회한 객체 전체를 전달하는 것은 피합니다.

```tsx
<AddToCartButton productId={product.id} />
```

필요한 값만 전달하면 다음과 같은 장점이 있습니다.

- Client에 전달되는 payload 감소
- 원가, 내부 메모 등 불필요한 Server data 노출 방지
- Client Component의 역할과 의존성 단순화

### 면접 답변

> `'use client'`는 Next.js App Router에서 Server와 Client module graph의 경계를 만드는 directive입니다. State, Effect, event handler나 Browser API가 필요한 Component에 사용하며, 해당 파일이 import하는 module도 Client bundle에 포함될 수 있습니다. Client Component도 최초 HTML 생성에는 참여할 수 있지만 Browser에서 hydration되어야 상호작용이 활성화됩니다. 따라서 page 전체보다 상호작용이 시작되는 작은 Component에 경계를 두는 것이 좋습니다.

### 참고

- [Next.js: Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- [React: Server Components](https://react.dev/reference/rsc/server-components)
- [React: `'use client'`](https://react.dev/reference/rsc/use-client)

## Server Component와 SSR

Server Component와 SSR은 이름에 Server가 들어가지만 서로 다른 질문에 답하는 개념입니다.

```text
Server Component
-> Component 코드를 어디에서 실행할 것인가?

SSR
-> HTML을 언제 어디에서 생성할 것인가?
```

Server Component는 Component 실행 환경과 Client bundle 구성에 관한 개념이고, SSR은 요청 시점에 Server에서 HTML을 생성하는 rendering 전략입니다.

| 구분 | Server Component | SSR |
| --- | --- | --- |
| 기준 | Component 실행 환경 | HTML 생성 시점과 위치 |
| 실행 시점 | Build 또는 Request | Request마다 |
| 주요 결과 | RSC Payload | HTML |
| Client JavaScript | Component 코드가 전달되지 않음 | Client Component 코드는 전달됨 |
| Hydration | Server Component는 대상이 아님 | Client Component는 hydration 필요 |

### 같은 개념이 아닌 이유

Server Component는 Build 시점과 Request 시점 모두에서 실행될 수 있습니다.

```text
Server Component + Build 시 실행
-> SSG 또는 prerendering

Server Component + Request 시 실행
-> SSR 또는 Dynamic Rendering
```

회사 소개를 Server Component로 Build 시 생성했다면 Server Component와 SSG의 조합입니다. 사용자별 주문 내역을 Server Component가 요청마다 조회한다면 Server Component와 SSR 또는 Dynamic Rendering의 조합입니다.

반대로 Client Component도 최초 접속에서는 Server의 HTML 생성에 참여할 수 있습니다.

```text
Server
-> Client Component의 초기 HTML preview 생성

Browser
-> HTML 표시
-> Client Component JavaScript 다운로드
-> Hydration으로 state와 event handler 연결
```

따라서 다음처럼 동일한 개념으로 보면 안 됩니다.

```text
Server Component = SSR  (X)
Client Component = CSR  (X)
```

### RSC Payload와 HTML

Server Component의 주요 결과는 HTML 자체가 아니라 RSC Payload입니다.

```text
RSC Payload
-> Server Component의 rendering 결과
-> Client Component가 들어갈 위치
-> Client Component JavaScript module 참조
-> Server에서 Client로 전달한 props
```

두 결과물의 역할은 다음과 같습니다.

```text
HTML
-> Browser가 직접 해석
-> DOM과 초기 화면 표시

RSC Payload
-> React와 Next.js가 해석
-> Server와 Client Component tree 구성 및 갱신
```

HTML만으로는 어떤 영역이 Server Component인지, 어떤 Client JavaScript가 필요한지, navigation에서 tree의 어느 부분을 갱신할지 알 수 없습니다. RSC Payload가 이 연결 정보를 제공합니다.

### App Router의 최초 접속

```text
1. Server Component 실행
2. RSC Payload 생성
3. RSC Payload와 Client Component로 초기 HTML 생성
4. Browser가 HTML로 초기 화면 표시
5. RSC Payload로 Server와 Client Component tree 조정
6. Client Component만 hydration
```

```text
Server Component
-> Component JavaScript 전달 안 됨
-> Hydration 안 함

Client Component
-> 초기 HTML 생성에 참여 가능
-> Component JavaScript 전달
-> Hydration함
```

### 이후 Client Navigation

최초 접속에서는 Browser가 문서를 표시하기 위해 HTML을 사용합니다. `<Link>`를 이용한 이후 navigation에서는 전체 HTML 문서를 다시 받는 대신 RSC Payload를 받아 기존 tree에 반영할 수 있습니다.

```text
최초 접속
-> HTML + RSC Payload + Client JavaScript

이후 navigation
-> RSC Payload 중심으로 전달
-> 필요한 route 영역만 갱신
-> 기존 Client state와 공유 layout 유지
```

RSC는 단순히 Server에서 HTML을 만드는 기능이 아니라 Server와 Client Component tree를 연결하고 부분적으로 갱신하기 위한 data format 역할도 합니다.

### 면접 답변

> Server Component와 SSR은 기준이 다른 개념입니다. Server Component는 Component를 Server에서만 실행하고 해당 JavaScript를 Client bundle에 포함하지 않는 구조이며 결과는 RSC Payload로 전달됩니다. SSR은 요청마다 Server에서 초기 HTML을 생성하는 rendering 전략입니다. Server Component는 요청 시 실행되어 SSR과 함께 사용될 수도 있고 Build 시 실행되어 SSG에 사용될 수도 있습니다. 반대로 Client Component도 최초 접속에서는 Server에서 HTML로 미리 rendering될 수 있지만 Browser에서 hydration이 필요합니다.

### 참고

- [Next.js: Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- [Next.js: Linking and Navigating](https://nextjs.org/docs/app/getting-started/linking-and-navigating)
- [React: Server Components](https://react.dev/reference/rsc/server-components)

## Streaming

Next.js Streaming은 Server에서 page 전체가 완성될 때까지 기다리지 않고 준비된 UI부터 여러 chunk로 나누어 Client에 전달하는 방식입니다.

```text
상품 정보: 2초
리뷰: 4초

Streaming 적용
-> 2초에 상품 영역 전달
-> 4초에 리뷰 영역 전달
```

Streaming이 느린 DB query 자체를 빠르게 만들지는 않습니다. 전체 작업 완료 시간보다 초기 화면과 사용자의 체감 대기 시간을 개선합니다.

### Suspense와의 관계

`Suspense`는 어떤 UI를 먼저 보내고 준비되지 않은 영역에 어떤 fallback을 보여줄지 정하는 경계입니다.

```tsx
import { Suspense } from 'react';

export default async function ProductPage() {
  const product = await getProduct();

  return (
    <main>
      <ProductInfo product={product} />

      <Suspense fallback={<ReviewSkeleton />}>
        <Reviews productId={product.id} />
      </Suspense>
    </main>
  );
}
```

```text
ProductInfo 준비
-> ProductInfo와 ReviewSkeleton 먼저 전달

Reviews 준비
-> Reviews HTML 추가 전달
-> ReviewSkeleton을 실제 Reviews로 교체
```

Streaming은 준비된 HTML을 나누어 전송하는 방식이고 Suspense는 Streaming을 나눌 UI 경계와 fallback을 지정합니다.

### `loading.tsx`와 Suspense

```text
loading.tsx
-> Route segment 단위
-> Next.js가 page를 Suspense 경계로 자동으로 감쌈

Suspense
-> Component 단위
-> 개발자가 세밀한 경계와 fallback 지정
```

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />;
}
```

`loading.tsx`가 있는 route로 이동해도 공유 `layout.tsx`는 유지됩니다. 변경되는 page 영역에 fallback이 표시되고, page가 준비되면 실제 content로 교체됩니다.

서로 독립적인 추천 상품과 리뷰를 별도 Suspense로 감싸면 먼저 준비된 영역부터 각각 표시할 수 있습니다. 하나의 경계로 함께 감싸면 경계 안의 UI가 모두 준비된 뒤 함께 표시됩니다.

### Data 요청 위치

느린 data를 Suspense 바깥의 상위 Component에서 먼저 기다리면 React가 경계에 도달하기 전에 rendering이 중단됩니다.

```tsx
async function Reviews() {
  const reviews = await getReviews();

  return <ReviewList reviews={reviews} />;
}
```

느린 요청을 Suspense 내부의 `Reviews`에서 기다리도록 분리해야 바깥 UI와 fallback을 먼저 보낼 수 있습니다.

### 병렬 요청과 차이

Streaming이 순차적으로 작성한 요청을 자동으로 병렬 요청으로 바꾸지는 않습니다.

```tsx
// 순차 실행: 약 2초 + 4초
const product = await getProduct();
const reviews = await getReviews();

// 병렬 실행: 약 4초
const [product, reviews] = await Promise.all([
  getProduct(),
  getReviews(),
]);
```

`Promise.all()`은 두 요청을 동시에 실행하지만 위 코드 다음의 UI는 두 요청이 모두 끝난 뒤 만들 수 있습니다. 먼저 끝난 영역부터 표시하려면 각 요청을 별도 Suspense 내부 Component에서 실행합니다.

```text
병렬 요청
-> 독립적인 작업을 동시에 실행해 전체 data 준비 시간 단축

Streaming
-> 준비된 UI부터 전달해 초기 화면과 체감 대기 시간 개선
```

상품 조회가 2초, 리뷰 조회가 4초라면 두 요청을 독립된 경계 안에서 시작해 상품을 2초에 먼저 보여주고 리뷰를 4초에 추가할 수 있습니다.

### Streaming과 Hydration

```text
Streaming
-> Server HTML을 준비되는 순서대로 Client에 전달

Hydration
-> 전달된 HTML에 Client Component의 state와 event 연결
```

Server Component의 HTML도 Streaming될 수 있지만 Server Component 자체는 hydration되지 않습니다. Client Component가 포함된 영역은 HTML이 먼저 표시되고 JavaScript가 준비된 뒤 hydration되어 상호작용할 수 있습니다.

### 주의점

- 사용자에게 의미 있는 UI 단위로 Suspense 경계 설정
- 실제 content와 크기가 비슷한 skeleton으로 layout 변화 감소
- 너무 많은 경계로 화면이 조각조각 나타나는 경험 방지
- Streaming과 별개로 순차적인 data 요청 waterfall 확인

### 면접 답변

> Next.js Streaming은 Server에서 page 전체가 완성될 때까지 기다리지 않고 준비된 UI부터 chunk 단위로 Client에 전달하는 방식입니다. `loading.tsx`는 route segment 단위의 Suspense 경계를 자동으로 만들고, 직접 사용하는 Suspense는 느린 Component 단위로 더 세밀한 fallback을 제공합니다. Streaming은 data 조회 자체를 빠르게 만들기보다 느린 영역이 전체 화면을 막지 않게 해 초기 화면과 체감 성능을 개선합니다.

### 참고

- [Next.js: Streaming](https://nextjs.org/learn/dashboard-app/streaming)
- [React: Suspense](https://react.dev/reference/react/Suspense)
- [React: Server Rendering APIs](https://react.dev/reference/react-dom/server)

## next/link

`<a>` 태그를 통해 링크를 이동하면, 대게 해당 페이지에 대한 정보를 미리 가져오지 않고 이동 시에 새로운 HTML을 받아와야 합니다. 반면, next/link 컴포넌트를 사용하면 링크가 생성된 페이지 정보를 사전에 가져와 JavaScript 파일로 미리 로드하게 됩니다. 이를 통해 페이지 이동 전에 필요한 데이터를 사전에 가져와 클라이언트 측에서 CSR과 유사한 방식으로 처리할 수 있습니다. 또한, next/link는 프리렌더링을 통해 서버에서 HTML을 생성하므로 SEO 측면에서도 문제가 발생하지 않습니다.

### Props

#### href(필수)

이동할 경로나 URL을 지정합니다.

```javascript
import Link from "next/link";

// <Link href="/링크경로">링크이름</Link>;
<Link href="/section1/getStaticProps">/getStaticProps</Link>;
```

![image-20230713231356303](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20230713231356303.png)

렌더링 후 element 상에서는 `<a>`태그로서 동작한다.

#### replace(default: false)

현재의 기록 상태(history state)를 대체하여 새로운 URL을 브라우저 기록 스택에 추가해줍니다.

- 활용 예시

게시글을 읽기 위해 리캡챠나 로그인과 같은 인증 절차가 필요한 사이트의 경우, 사용자가 게시글을 읽은 후에 게시판 목록으로 돌아가려는 경우가 종종 있습니다. 그러나 기존의 방식으로는 이미 완료한 인증 절차를 다시 거쳐야 하고, 다시 한 번 뒤로가기 버튼을 클릭해야 하는 번거로움이 생길 수 있습니다. 이러한 상황에서 replace 속성을 활용하면, 사용자가 게시글 페이지에서 뒤로가기 버튼을 클릭할 때, 이미 수행한 인증 페이지로 되돌아가지 않고 바로 게시판 목록 페이지로 이동할 수 있습니다. 이로써 사용자의 경험을 개선하고 의도한 작업을 보다 원활하게 수행할 수 있게 됩니다.

#### prefetch(default: true)

백그라운드에서 페이지(주소로 표시되는 페이지)를 프리로딩합니다.뷰포트 안에 있는 모든 <Link />는 프리로드되며 뷰포트 바깥에 있을 경우 스크롤을 통해 <Link />가 뷰포트 내에 들어온 순간 프리로드 됩니다. prefetch는 프로덕션 환경에서만 활성화 되고 개발환경에서는 동작하지 않습니다.

- 예시

```javascript
import Link from "next/link";

export default function Links() {
  return (
    <main>
      <h1>Links</h1>
      <div style={{ height: "200vh" }}></div>
      <Link href="/section1/getStaticProps">/getStaticProps</Link>
    </main>
  );
}
```

위 코드와 같이 `<Link>`태그를 페이지에 보이는 영역 바깥에 위치하게 될 경우 `<Link>`태그가 보이지 않을 때는 링크 페이지가 프리로드되지 않다가 해당 영역에 스크롤이 닿았을 때 프리로드 됩니다.

![image-20230713222227408](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20230713222227408.png)
![image-20230713222247138](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20230713222247138.png)

#### scroll(default: true)

<Link>가 새 경로로 이동할 때 페이지의 맨 위로 스크롤해줍니다. 뒤로/앞으로 탐색하는 경우에는 이전의 스크롤 위치를 유지해 줍니다. false로 할 경우 <Link>가 페이지 이동 후 페이지 맨 위로 스크롤하지 않습니다.

## next/router

> [next/link](#nextlink)의 내용 참고

`next/router`는 Next.js에서 제공하는 라우팅 관련 기능을 사용할 수 있는 모듈입니다. `next/link`와 유사한 역할을 하며, 클라이언트 사이드에서의 라우팅을 지원합니다. 그러나 `next/link`와 달리 프리로딩 기능을 제공하지는 않습니다.

- 사용법

```jsx
import { useRouter } from "next/router";

export default function Links() {
  const router = useRouter();

  return (
    <button
      onClick={() => {
        router.push("/section1/getStaticProps");
      }}
    >
      /getStaticProps
    </button>
  );
}
```

`const router = useRouter();`를 선언한 후 onClick에 콜백 함수로 페이지 링크를 push 해주면됩니다.

### useRouter 메서드

- router.push: 지정한 경로로 클라이언트 사이드 네비게이션을 수행하며, 히스토리 스택에 기록합니다.
- router.replace: 경로가 이동되며, 현재의 히스토리 스택을 이동한 경로로 교체합니다.
- router.refresh: 현재 경로를 새로고침합니다. 서버에 새로 요청을 보내 데이터를 리패치하고 서버 컴포넌트를 리렌더링합니다. 클라이언트는 업데이트 된 서버 컴포넌트 데이터를 받아와서 클라이언트의 리액트 상태(ex: useState)나 브라우저의 상태(ex: 스크롤 위치)를 초기화 하지 않고 업데이트합니다.
- router.back: 히스토리 스택에서 이전 경로로 이동합니다.
- router.forward: 히스토리 스택에서 다음 경로로 이동합니다.

## next/image

이미지는 일반적인 웹사이트의 크기의 상당히 많은 부분을 차지합니다.이는 웹사이트의 LCP 성능에 상당한 영향을 미칠수 있습니다.

Next.js에서는 `<Image/>` 컴포넌트를 사용하여 이미지를 자동으로 최적화하여 LCP 성능을 개선합니다.

#### 이미지 포멧 변경

jpeg, png등의 원본 파일을 Webp 나 AVIF 같은 최신의 이미지 형식으로 자동으로 변환하여 이미지 용량을 크게 줄여줍니다. `quality` 속성을 통해 이미지 최적화율을 조정할 수 있습니다.(default: 75)

#### 시각적인 안정

웹사이트를 이용하는 도중 메뉴를 클릭했는데 갑자기 광고나 이미지가 그자리에 로딩되어 의도하지 않은 페이지로 이동하는 경험이 있을 것이다.이러한 현상을 `Cumulative Layout Shift` 줄여서 `CLS`라고 부릅니다. Next는 이러한 현상을 방지하기 위해 기본적으로 `width`, `height`의 값을 받아 이미지가 로딩되는 동안 해당 영역을 비워 둡니다. 이를 통해 페이지 로딩이 완료되는 사이에 레이아웃이 변경되는 것을 방지합니다. `placeholder`속성을 사용하여 비어있는 영역을 blur이미지나 대체 이미지로 채워놓을 수도 있습니다.

#### lazy loading

이미지가 뷰포트 바깥에 있을 때는 로딩을 지연시키다가 뷰포트 내부로 들어왔을 경우 로드하는 기술입니다.눈에 보이는 부분만 먼저 로드하고 그외의 부분은 지연시킴으로서 페이지의 초기로딩을 개선시킬수 있습니다. `<img/>`태그에 `loading="lazy"`속성을 추가하는 것으로도 적용할 수 있으나 next/image는 이를 자동으로 적용시켜줍니다.

### 캐싱 동작

이미지는 요청 시 동적으로 최적화되어 `<distDir>/cache/images` 디렉터리에 저장됩니다. 이 최적화된 이미지 파일은 만료되기 전까지 후속 요청에 대해 제공됩니다. 이미 캐시되었지만 만료된 파일과 일치하는 요청이 발생하면, 이전 버전의 이미지가 해당 요청에 사용됩니다(이를 "stale"이라고도 합니다). 이후 이미지는 백그라운드에서 다시 최적화되며, 새로운 만료 날짜와 함께 캐시에 저장됩니다.

응답 헤더의 x-nextjs-cache 값을 읽음으로써 Next 이미지의 캐시 상태를 확인할 수 있습니다.

- MISS: 해당 경로의 이미지가 캐시에 없습니다. (첫 방문 시에 한 번만 발생)
- STALE: 해당 경로의 이미지는 캐시에 있지만, 만료 시간을 초과하여 백그라운드로 업데이트됩니다.
- HIT: 해당 경로의 이미지가 캐시에 있으며, 만료 시간동안 유효합니다.

만료시간은 `minimumCacheTTL`과 `Cache-Control`를 통해 조절할 수 있습니다.

### 사용법

```jsx
import Image from "next/image";
```

#### 로컬 이미지

로컬 이미지 는 .`jpg`, `.png`, `.webp`등의 파일을 import 구문으로 가져와서 사용합니다. Next가 사전에 이미지의 크기를 알 수 있기 때문에 `width`, `height`의 값을 생략할 수 있습니다. 이 사전에 측정된 크기를 통해 `CLS`를 방지합니다. 빌드시 분석되기 때문에 `다이나믹 import`나 `require문`은 지원하지 않습니다.

```jsx
import Image from "next/image";
import profilePic from "./me.png";

export default function Page() {
  return <Image src={profilePic} alt="Picture of the author" />;
}
```

#### 원격 이미지

`next/image`의 src속성을 문자열이나 URL로 사용하는 경우입니다. Next.js는 빌트 타입에 원격 파일에 액세스할 수 없기 때문에 `width`, `height`값을 제공해줘야 합니다. 이미지의 사이즈를 모를 경우 `fill`속성을 통해 부모의 사이즈로 대체할 수 있습니다.

```jsx
import Image from "next/image";

export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  );
}
```

Next.js는 기본적으로 외부사이트를 허용하지 않기 때문에 `next.config.js`파일에 예외로 할 URL 패턴 목록을 정의해야됩니다.

```jsx
// next.config.js

module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "s3.amazonaws.com",
        port: "",
        pathname: "/my-bucket/**",
      },
    ],
  },
};
```

만일 해당 도메인에서 제고오디는 모든 컨텐츠를 소유한 경우에는 `domains`을 사용해도 됩니다. 그외에는 앱을 보호하기 위해 `remotePatterns`으로 엄격한 구성을 권장합니다.

```jsx
// next.config.js

module.exports = {
  images: {
    domains: ["assets.acme.com"],
  },
};
```

## Data fetching

### Page router (Next 13 이전)

#### getStaticProps

SSG 방식에서 사용됩니다. SSG는 빌드 시에 페이지를 생성하며, 이후 변경이 없으면 한 번만 호출됩니다. 빌드 타임에 데이터를 가져오므로 초기 페이지 로딩 속도가 매우 빠릅니다. 데이터 변경이 필요한 경우 페이지를 재빌드해야 합니다.

#### getStaticPaths

페이지가 동적 라우팅을 사용하면서 `getStaticProps`도 사용할 경우, `getStaticPaths`를 추가하여 정적으로 렌더링할 경로를 설정합니다. 이렇게 하지 않으면 해당 경로로 접근 시 404 페이지가 표시됩니다. 따라서 동적 라우팅 시 모든 가능한 경우의 경로를 정의해야 합니다.

#### getServerSideProps

SSR 방식에서 사용됩니다. 각 페이지 요청마다 서버에서 데이터를 가져와 페이지 컴포넌트의 props로 전달합니다. 서버 사이드에서 데이터를 가져오기 때문에 항상 최신 데이터를 보여줄 수 있습니다. 실시간 업데이트가 필요한 경우에 적합합니다.

### App router

데이터를 가져오기 위해 `fetch()`를 사용합니다. 서버 컴포넌트의 경우 `async/await`를 사용하여 비동기적으로 데이터를 가져옵니다. 기존 브라우저의 `Fetch API`를 확장한 API를 Next에서 제공해 줍니다. 별도 설정을 하지않아도 Next가 자동으로 데이터를 캐시해줍니다.
