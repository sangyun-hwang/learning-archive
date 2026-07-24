# App Router

- [App Router](#app-router-1)
- [Pages](#pages)
- [Layouts](#layouts)
- [루트 Layouts](#루트-layout-필수)
- [Templates](#templates)
- [Parallel Routes](#parallel-routes)
- [Intercepting Routes](#intercepting-routes)
- [URL 기반 Modal](#parallel-routes와-intercepting-routes로-url-기반-modal-만들기)

<br>

## App Router

Next.js 13 버전에서는 React Server Components를 기반으로 한 새로운 App Router가 도입되었습니다. 이 App Router는 공유 레이아웃, 중첩 라우팅, 로딩 상태 처리, 에러 처리 등 다양한 기능을 지원합니다.

App Router는 "app"이라는 새 디렉토리에서만 작동합니다. 이 새 디렉토리는 기존의 "pages" 디렉토리와 함께 사용되며, 점진적으로 App Router로 바뀔 수 있도록 복수 지원합니다. 이를 통해 기존의 "pages" 디렉토리를 사용하던 기능과 함께 새로운 App Router를 사용하는 기능을 혼용할 수 있습니다. 다만, 두 가지 라우팅 방식이 같은 경로를 가지고 있다면 충돌을 방지하기 위해 빌드 타임에 App Router이 Pages Router보다 우선적용됩니다.

<br>

### 파일 기반 라우트

Next.js는 파일 시스템 기반의 라우터를 사용합니다. 이때 폴더는 라우트를 정의하는 데 사용됩니다.

![route](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Froute-segments-to-path-segments.png&w=1920&q=75&dpl=dpl_3guogY6YECQnnD8P1bp8UJe7CDCH)

각 폴더는 URL segment 에 매핑되는 라우트 segment 를 나타냅니다.

<br>

### 파일 컨벤션

Next.js는 중첩된 라우트에서 특정 동작을 가진 UI를 만들기 위해 다음과 같은 파일 컨벤션을 제공합니다.

- layout: segment 와 그 자식들을 위한 공유 UI를 생성합니다.
- page: 라우트의 고유한 UI를 생성하며 라우트를 공개적으로 접근 가능하게 합니다.
- loading: segment 와 그 자식들을 위한 로딩 UI를 생성합니다.
- not-found: segment 와 그 자식들을 위한 찾을 수 없음 UI를 생성합니다.
- error: segment 와 그 자식들을 위한 에러 UI를 생성합니다.
- global-error: 전역 에러 UI를 생성합니다.
- route: 서버 측 API 엔드포인트를 생성합니다.
- template: 특수화된 다시 렌더링되는 레이아웃 UI를 생성합니다.
- default: 병렬 라우트를 위한 대체 UI를 생성합니다.

특수 파일 이외에도, 자신이 생성한 파일(예: 컴포넌트, 스타일, 테스트 등)들을 app 디렉토리의 폴더 안에 함께 배치할 수 있습니다.

<br>

### 컴포넌트 계층구조

파일 컨벤션에서 정의된 React 컴포넌트들은 다음과 같은 특정한 계층 구조로 렌더링됩니다.

![컴포넌트 계층 구조](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Ffile-conventions-component-hierarchy.png&w=1920&q=75&dpl=dpl_Ev1SSnkTzSfmJGJRmYbn4JZhjkvm)

1. layout.js
2. template.js
3. error.js (React 에러 바운더리)
4. loading.js (React 서스펜스 바운더리)
5. not-found.js (React 에러 바운더리)
6. page.js 또는 중첩된 layout.js

중첩 라우트의 경우, 자식 segment의 컴포넌트들은 부모 segment의 컴포넌트들 안에 중첩됩니다.

![중첩 컴포넌트 계층](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Fnested-file-conventions-component-hierarchy.png&w=1920&q=75&dpl=dpl_Ev1SSnkTzSfmJGJRmYbn4JZhjkvm)

<br>

## Pages

페이지는 특정 URL 경로에 대한 화면을 담당하는 컴포넌트입니다. 각각의 라우트는 자신만의 페이지를 가지며, 이를 통해 각 라우트 폴더에 page.js 파일로 고유한 UI를 정의할 수 있습니다. 이렇게 정의된 페이지는 해당 라우트에 접속했을 때 화면에 표시됩니다. 이를 통해 다양한 페이지를 만들고 라우트별로 다른 UI를 제공할 수 있게 됩니다.

![pages](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Fpage-special-file.png&w=1920&q=75&dpl=dpl_BfrsMtEkFNtWCS4n2Nhqya4WuovP)

### ex)

```tsx
// `app/page.tsx` is the UI for the `/` URL
export default function Page() {
  return <h1>Hello, Home page!</h1>;
}
```

```tsx
// `app/dashboard/page.tsx` is the UI for the `/dashboard` URL
export default function Page() {
  return <h1>Hello, Dashboard Page!</h1>;
}
```

> #### 참고 사항
>
> - page 는 항상 라우트 서브트리의 맨 마지막에 위치합니다.
> - `.js`, `.jsx` 또는 `.tsx` 파일 확장자를 사용하여 페이지를 정의할 수 있습니다.
> - 라우트 segment를 공개적으로 접근 가능하게 하려면 page.js 파일이 필요합니다.
> - 기본적으로 페이지는 서버 컴포넌트이지만, 클라이언트 컴포넌트로도 설정할 수도 있습니다.

<br>

## Layouts

레이아웃은 sidebar, navbar와같이 여러 페이지에서 공유되는 UI입니다. 라우팅 중에 레이아웃은 상태를 보존하고 상호작용이 가능하며, 다시 렌더링되지 않습니다. 레이아웃은 중첩될 수도 있습니다.

![Layouts](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Flayout-special-file.png&w=1920&q=75&dpl=dpl_BfrsMtEkFNtWCS4n2Nhqya4WuovP)
![중첩 컴포넌트 계층](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Fnested-file-conventions-component-hierarchy.png&w=1920&q=75&dpl=dpl_Ev1SSnkTzSfmJGJRmYbn4JZhjkvm)

레이아웃은 layout.js 파일에서 React 컴포넌트를 내보내는 방식으로 정의할 수 있습니다. 이 컴포넌트는 children prop을 받아서 렌더링 중에 자식 layout(있을 경우) 또는 자식 page로 채워집니다.

![중첩](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Fnested-layouts-ui.png&w=1920&q=75&dpl=dpl_7rjDJs5gNWrZ6yx12qkY2XTnnxuc)

### ex)

```tsx
export default function DashboardLayout({
  children, // will be a page or nested layout
}: {
  children: React.ReactNode;
}) {
  return (
    <section>
      {/* Include shared UI here e.g. a header or sidebar */}
      <nav></nav>

      {children}
    </section>
  );
}
```

> #### 참고 사항
>
> - 최상위 layout은 루트 layout이라고 하며, 애플리케이션의 모든 페이지에서 공유됩니다. 루트 layout은 반드시 html과 body 태그를 포함해야 합니다.
> - 어떤 라우트 segment든지 자체적으로 layout을 정의할 수 있습니다. 이러한 layout은 해당 segment의 모든 페이지에서 공유됩니다.
> - 라우트 안의 layout은 기본적으로 중첩됩니다. 각 부모 layout은 React children prop을 사용하여 자식 layout을 감쌀 수 있습니다
> - Route Groups를 사용하여 특정 라우트 segment를 공유 layout에 포함하거나 제외시킬 수 있습니다.
> - 기본적으로 layout은 서버 컴포넌트이지만, 클라이언트 컴포넌트로도 설정할 수도 있습니다.
> - layout.js와 page.js 파일은 동일한 폴더에 정의할 수 있습니다. 이 경우 layout이 page를 감쌉니다.

<br>

## 루트 Layout (필수)

루트 layout은 앱 디렉토리의 최상위 수준에서 정의되며 애플리케이션의 모든 페이지에서 공유됩니다. 루트 layout은 반드시 html과 body 태그를 포함해야 합니다.

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

> #### 참고 사항
>
> - 앱 디렉토리에는 루트 layout이 포함되어야 합니다.
> - Next.js에서 <html> 및 <body> 태그가 자동으로 생성되지 않기 때문에 루트 layout에 정의해야 합니다.
> - 내장된 SEO 지원을 사용하여 <head> HTML 요소를 관리할 수 있습니다. 예를 들어, <title> 요소를 관리할 수 있습니다.

<br>

## Templates

템플릿은 레이아웃과 비슷한 역할을 하며, 자식 레이아웃이나 페이지를 감싸는 역할을 합니다. 하지만 템플릿으로 래핑된 페이지는 레이아웃과 달리 매번 리렌더링됩니다.

![template](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Ftemplate-special-file.png&w=1920&q=75&dpl=dpl_7rjDJs5gNWrZ6yx12qkY2XTnnxuc)

일반적인 레이아웃은 라우트 간에 지속되며 상태를 보존하고 상호작용을 유지합니다. 이는 라우트를 전환할 때마다 레이아웃이 다시 렌더링되지 않는다는 것을 의미합니다.

반면에 템플릿은 라우트를 전환할 때마다 새로운 인스턴스로 마운트되고, 이전 상태나 상호작용이 유지되지 않습니다. 이는 템플릿이 "정적"이지 않고 매번 새로운 상태를 갖게 된다는 것을 의미합니다.

따라서 템플릿은 특정 상황에서만 사용하는 것이 좋습니다. 예를 들어, CSS나 애니메이션 라이브러리를 사용한 진입/이탈 애니메이션, useEffect를 사용한 기능(예: 페이지 조회 기록 기록)이나 useState를 사용한 특정 페이지 피드백 폼 등에 유용합니다. 또한 기본 프레임워크 동작을 변경하고 싶을 때도 템플릿을 사용할 수 있습니다. 하지만 일반적으로는 레이아웃을 사용하여 라우트 간에 상태를 유지하는 것이 좋습니다.

## Parallel Routes

Parallel Routes는 하나의 layout에서 여러 page를 동시에 또는 조건부로 렌더링하는 기능입니다. Dashboard의 여러 panel, 독립적인 tab, 사용자 역할별 화면, modal과 drawer처럼 복잡한 UI에 사용할 수 있습니다.

### Named Slots

Parallel Route는 `@folder` 규칙으로 named slot을 만듭니다.

```text
app/
  dashboard/
    layout.tsx
    page.tsx
    @team/
      page.tsx
    @analytics/
      page.tsx
```

`@team`과 `@analytics`는 부모 layout의 props로 전달됩니다.

```tsx
export default function DashboardLayout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode;
  team: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return (
    <main>
      {children}

      <section>
        {team}
        {analytics}
      </section>
    </main>
  );
}
```

`@folder`는 URL segment가 아니라 layout에 전달할 UI slot입니다.

```text
파일 구조
app/dashboard/@analytics/views/page.tsx

URL
/dashboard/views
```

`children`도 별도 폴더 표기가 생략된 implicit slot으로 볼 수 있습니다. `app/page.tsx`는 개념적으로 `app/@children/page.tsx`와 같은 역할을 합니다.

### Soft Navigation

`Link`나 router를 이용한 client-side navigation에서는 Next.js가 각 slot의 활성 subpage를 추적합니다. 특정 slot이 변경돼도 현재 URL과 직접 일치하지 않는 다른 slot의 활성 상태를 유지할 수 있습니다.

```text
Soft navigation
-> 변경된 slot을 부분적으로 렌더링
-> 다른 slot의 활성 상태 유지
```

이 동작은 dashboard panel이나 tab을 독립적으로 이동할 때 유용합니다.

### Hard Navigation과 default.tsx

새로고침, 주소창 직접 접근, 외부 link 접근처럼 전체 page를 새로 요청하면 Next.js는 URL과 일치하지 않는 slot의 이전 활성 상태를 알 수 없습니다.

이때 `default.tsx`가 복구할 수 없는 slot의 fallback을 제공합니다.

```tsx
// app/dashboard/@analytics/default.tsx
export default function Default() {
  return null;
}
```

Next.js 16에서는 Parallel Route의 named slot마다 명시적인 `default.tsx`가 필요합니다. 존재하지 않으면 build가 실패할 수 있습니다. `children` implicit slot도 현재 URL과 일치하는 page를 찾을 수 없는 구조라면 parent level의 `default.tsx`가 fallback 역할을 할 수 있습니다.

`default.tsx`와 `loading.tsx`의 역할은 다릅니다.

```text
Hard navigation에서 어떤 slot 상태를 렌더링할지 복구할 수 없음
-> default.tsx

렌더링할 route는 정해졌지만 데이터나 component가 준비 중
-> loading.tsx
```

`loading.tsx`는 route segment의 Suspense fallback이고, `default.tsx`는 일치하지 않는 Parallel Route slot의 fallback입니다.

### 독립적인 Loading과 Error UI

각 slot은 자체 `loading.tsx`와 `error.tsx`를 가질 수 있어 독립적으로 streaming하고 오류를 처리할 수 있습니다.

```text
app/dashboard/
  @team/
    loading.tsx
    error.tsx
    page.tsx
  @analytics/
    loading.tsx
    error.tsx
    page.tsx
```

한 panel의 데이터 loading이나 오류가 다른 panel 전체를 막지 않도록 경계를 나눌 수 있습니다.

## Intercepting Routes

Intercepting Routes는 client-side navigation에서 다른 route를 현재 layout의 맥락 안으로 가로채 렌더링합니다.

사진 gallery를 예로 들면 다음과 같은 동작을 만들 수 있습니다.

```text
/gallery에서 /photo/123 Link 클릭
-> URL은 /photo/123
-> gallery 위에 photo modal 표시

/photo/123 주소창 직접 접근 또는 새로고침
-> photo 독립 page 표시
```

Soft navigation에서는 현재 화면의 맥락을 보존한 표현을 사용하고, hard navigation에서는 URL에 해당하는 일반 page를 렌더링합니다.

### Intercepting Convention

```text
(.)       같은 route segment level
(..)      한 segment 위
(..)(..)  두 segment 위
(...)     app root부터
```

Matcher는 실제 파일 시스템의 폴더 개수가 아니라 route segment를 기준으로 계산합니다.

다음 폴더들은 URL segment가 아니므로 matcher level 계산에 포함되지 않습니다.

- Named slot: `@modal`
- Route Group: `(shop)`

```text
app/
  photo/
    [id]/
      page.tsx
  @modal/
    (.)photo/
      [id]/
        page.tsx
```

`@modal`은 slot이므로 `(.)photo`는 파일 시스템상 다른 깊이에 있어도 같은 route segment level의 `/photo`를 intercept합니다.

## Parallel Routes와 Intercepting Routes로 URL 기반 Modal 만들기

두 기능을 함께 사용하면 다음 요구사항을 만족하는 modal을 만들 수 있습니다.

- Modal을 열어도 기존 gallery 화면과 scroll 맥락을 유지합니다.
- Modal이 열리면 의미 있는 URL로 변경됩니다.
- URL을 공유하거나 직접 접근할 수 있습니다.
- 뒤로 가기로 modal을 닫고 앞으로 가기로 다시 열 수 있습니다.
- 직접 접근하거나 새로고침하면 독립된 전체 page를 보여줍니다.

### 파일 구조

```text
app/
  layout.tsx
  page.tsx

  photo/
    [id]/
      page.tsx

  @modal/
    default.tsx
    page.tsx
    [...catchAll]/
      page.tsx
    (.)photo/
      [id]/
        page.tsx
```

### 독립된 Photo Page

일반 `/photo/[id]` route는 주소창 직접 접근과 새로고침에서 전체 page를 렌더링합니다.

```tsx
// app/photo/[id]/page.tsx
export default async function PhotoPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  return <PhotoDetail id={id} />;
}
```

### Intercept된 Modal Page

`@modal` slot의 Intercepting Route는 `/photo/[id]`로 가는 soft navigation을 가로채 modal 표현으로 렌더링합니다.

```tsx
// app/@modal/(.)photo/[id]/page.tsx
export default async function PhotoModalPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  return (
    <Modal>
      <PhotoDetail id={id} />
    </Modal>
  );
}
```

`PhotoDetail`을 공통 component로 분리하면 독립 page와 modal이 같은 콘텐츠를 재사용할 수 있습니다.

### Layout에서 Modal Slot 렌더링

Parallel Route가 기존 page와 modal을 동시에 배치할 named slot을 제공합니다.

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html lang="ko">
      <body>
        {children}
        {modal}
      </body>
    </html>
  );
}
```

### Modal의 기본 상태

Modal route가 활성화되지 않았거나 hard navigation에서 slot의 이전 상태를 복구할 수 없으면 `null`을 렌더링합니다.

```tsx
// app/@modal/default.tsx
export default function Default() {
  return null;
}
```

### Modal 열기와 닫기

```tsx
<Link href="/photo/123">사진 보기</Link>
```

Gallery에서 이 link로 이동하면 URL은 `/photo/123`으로 바뀌고 Intercepting Route가 modal을 렌더링합니다.

뒤로 가기를 이용해 modal을 열기 전의 gallery 상태로 돌아갈 수 있습니다.

```tsx
'use client';

import { useRouter } from 'next/navigation';

export function Modal({
  children,
}: {
  children: React.ReactNode;
}) {
  const router = useRouter();

  return (
    <div role="dialog" aria-modal="true">
      <button type="button" onClick={() => router.back()}>
        닫기
      </button>
      {children}
    </div>
  );
}
```

Routing은 modal의 접근성을 자동으로 해결하지 않습니다. 실제 modal에는 focus 이동과 복원, Escape 처리, focus trap, 배경 interaction 차단, 접근 가능한 이름을 별도로 구현해야 합니다.

### Catch-all로 Modal Slot 비우기

Parallel Route는 soft navigation에서 slot의 활성 상태를 유지합니다. 따라서 modal과 관련 없는 route로 이동해도 이전 modal이 남을 수 있습니다.

Root route로 이동할 때는 slot의 `page.tsx`가 `null`을 렌더링하도록 만듭니다.

```tsx
// app/@modal/page.tsx
export default function Page() {
  return null;
}
```

하위의 다른 route로 이동할 때는 catch-all을 사용합니다.

```tsx
// app/@modal/[...catchAll]/page.tsx
export default function CatchAll() {
  return null;
}
```

`page.tsx`와 catch-all route가 이동한 경로를 match해 `null`을 렌더링하면 modal slot을 명시적으로 비울 수 있습니다. 일반 catch-all인 `[...catchAll]`은 빈 root segment를 match하지 않으므로 root용 `page.tsx`를 별도로 둡니다.

### 새로고침 시 동작

Gallery 위에 photo modal을 연 상태에서는 URL이 이미 `/photo/123`입니다.

```text
Modal을 연 직후
URL: /photo/123
화면: gallery + photo modal

새로고침 후
URL: /photo/123
화면: photo 독립 page
```

새로고침하면 soft navigation의 gallery 맥락은 사라지고 현재 URL과 일치하는 일반 `/photo/[id]` route가 렌더링됩니다. Interception은 route를 삭제하거나 URL을 가짜로 바꾸는 것이 아니라 현재 navigation 맥락에서 다른 표현을 선택하는 기능입니다.

### 역할 구분

| 구성 | 역할 |
| --- | --- |
| Parallel Routes | 기존 page와 modal을 동시에 배치할 `@modal` named slot 제공 |
| Intercepting Routes | `/photo/123` soft navigation을 가로채 modal 표현 렌더링 |
| 일반 `/photo/[id]` | 직접 접근과 새로고침에 사용할 독립 page 제공 |
| `default.tsx` | Hard navigation에서 복구할 수 없는 slot fallback |
| Slot의 `page.tsx`, `[...catchAll]` | Root와 다른 route로 이동할 때 modal slot을 비움 |

공유와 검색 가능한 독립 화면을 제공하는 핵심은 실제 URL과 그 URL에 대응하는 일반 page route가 존재한다는 점입니다. Intercepting Route 자체가 SEO를 보장하는 것은 아닙니다.

### 언제 사용하는가?

다음과 같이 URL과 browser history가 의미 있는 overlay에 적합합니다.

- Social feed의 photo 상세
- 상품 목록의 상품 quick view
- Login page와 login modal
- 장바구니 drawer

단순 안내나 확인창처럼 URL 공유, 새로고침, 뒤로 가기와 연결할 필요가 없는 modal은 local state로 관리하는 편이 단순합니다.

### 정리

> Parallel Routes는 `@folder` named slot을 사용해 하나의 layout에서 여러 route를 동시에 렌더링합니다. Intercepting Routes는 soft navigation에서 다른 route를 현재 layout 안으로 가로채 렌더링합니다. 둘을 조합하면 URL은 `/photo/123`으로 변경하면서 gallery 위에 modal을 보여주고, 직접 접근하거나 새로고침하면 독립된 photo page를 제공할 수 있습니다.

### 참고

- [Next.js: Parallel Routes](https://nextjs.org/docs/app/api-reference/file-conventions/parallel-routes)
- [Next.js: Intercepting Routes](https://nextjs.org/docs/app/api-reference/file-conventions/intercepting-routes)
- [Next.js: default.js](https://nextjs.org/docs/app/api-reference/file-conventions/default)
