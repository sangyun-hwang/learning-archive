# App Router

- [App Router](#app-router-1)
- [Pages](#pages)
- [Layouts](#layouts)
- [루트 Layouts](#루트-layout-필수)
- [Templates](#templates)

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
