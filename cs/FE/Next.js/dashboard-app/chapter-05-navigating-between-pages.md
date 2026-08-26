# Chapter 5. Navigating Between Pages

## 학습 목표

Next.js의 `<Link>`를 사용해 전체 Page를 새로 Loading하지 않고 Dashboard route 사이를 이동하고, `usePathname()`으로 현재 경로에 해당하는 Navigation Link를 표시한다.

## `<a>`와 `<Link>`의 차이

일반 `<a>` Element로 다른 Page에 이동하면 Browser가 새로운 HTML document를 요청하고 현재 document를 교체한다.

```text
<a> Navigation
-> Server에 새 HTML document 요청
-> 기존 React tree와 Client state 제거
-> 전체 Page Loading
```

Next.js의 `<Link>`는 App Router 안에서 Client-side Navigation을 제공한다.

```text
<Link> Navigation
-> 목적지 route 요청
-> 필요한 route data와 RSC Payload 수신
-> 공유 Layout과 Client state 유지
-> 변경되는 Page 영역 갱신
```

`<Link>`의 핵심은 Prefetch만이 아니다. 전체 document를 교체하지 않고 Next.js Router가 이동을 처리한다는 점이 먼저이고, Prefetch는 이 Navigation을 더 빠르게 만드는 최적화다.

## Route Code Splitting

Next.js는 application의 모든 code를 최초 접속에서 한 번에 내려보내지 않고 route segment를 기준으로 나눌 수 있다.

```text
Route Code Splitting
-> 현재 route에 필요한 code 중심으로 Loading
-> 초기 JavaScript 크기와 parsing 비용 감소
-> 다른 route의 오류와 code를 일정 부분 분리
```

Code Splitting은 **무엇을 나누어 Loading할 것인가**에 관한 기능이다.

## Prefetch

Production 환경에서 `<Link>`가 Viewport에 나타나면 Next.js가 목적지 route에 필요한 resource를 미리 준비할 수 있다. 사용자가 Link를 클릭하기 전에 일부 작업을 수행하므로 실제 Navigation의 대기 시간을 줄일 수 있다.

```text
Prefetch
-> 사용 가능성이 있는 목적지 resource를 미리 Loading
-> 클릭 이후 체감 대기 시간 감소
```

Prefetch는 **언제 미리 Loading할 것인가**에 관한 최적화다. Development 환경의 동작과 Production build의 Prefetch 동작은 다를 수 있으므로 실제 최적화는 Production 환경에서도 확인해야 한다.

## `usePathname()`과 Client Component

`usePathname()`은 현재 URL의 pathname을 읽는 Client Hook이다. Client Navigation으로 URL이 변경되면 새로운 pathname을 반영해 Component를 다시 Rendering할 수 있다.

```tsx
'use client';

import { usePathname } from 'next/navigation';

export default function NavLinks() {
  const pathname = usePathname();
  // ...
}
```

단순히 이름이 `use`로 시작해서 Client Component가 필요한 것은 아니다. `usePathname()`이 Browser의 현재 Navigation 상태를 구독하는 Client Hook이기 때문에 `'use client'` 경계가 필요하다.

Dashboard Layout 전체가 아니라 `NavLinks`에만 Client 경계를 두면 Side Navigation의 나머지 정적 UI와 Page를 불필요하게 Client module graph에 포함하지 않을 수 있다.

## Active Link 표시

현재 pathname과 Link의 `href`를 비교하고 `clsx`로 조건부 class를 적용한다.

```tsx
<Link
  href={link.href}
  className={clsx('기본 class', {
    'bg-sky-100 text-blue-600': pathname === link.href,
  })}
>
  {link.name}
</Link>
```

정확한 문자열 비교는 Chapter에서 사용하는 세 route에는 적합하지만 하위 route에서는 일치하지 않을 수 있다.

```text
/dashboard/invoices/123 === /dashboard/invoices
-> false
```

하위 route까지 같은 메뉴로 표시하려면 정확한 일치와 prefix 비교를 조합할 수 있다.

```ts
const isActive =
  pathname === link.href || pathname.startsWith(`${link.href}/`);
```

다만 `/dashboard`에 단순한 `startsWith()`를 적용하면 모든 Dashboard 하위 route에서 Home까지 활성화될 수 있다. Route 계층과 메뉴 정책에 맞게 Root 항목을 별도로 처리해야 한다.

## 구현에서 확인한 내용

- Side Navigation의 `<a>`를 Next.js `<Link>`로 변경했다.
- `usePathname()`으로 현재 Dashboard route를 확인했다.
- `clsx`를 사용해 활성 Link의 배경색과 글자색을 변경했다.
- `NavLinks`에만 `'use client'`를 선언해 Client 영역을 제한했다.
- `ch05: add client-side dashboard navigation` commit으로 Chapter 변경을 구분했다.
- Next.js 16 Production build와 TypeScript 검사를 통과했다.

## 핵심 정리

> Next.js의 `<Link>`는 전체 HTML document를 다시 Loading하는 대신 Client-side Navigation으로 필요한 route 영역을 갱신한다. Route Code Splitting은 최초에 필요한 code만 나누어 Loading해 초기 비용을 줄이고, Prefetch는 Link의 목적지 resource를 미리 준비해 클릭 후 대기 시간을 줄인다. `usePathname()`은 현재 Navigation 상태에 반응하는 Client Hook이므로 이를 사용하는 작은 Component에만 `'use client'` 경계를 두는 것이 좋다.

## 참고 자료

- [Next.js Learn: Navigating Between Pages](https://nextjs.org/learn/dashboard-app/navigating-between-pages)
- [Next.js: Linking and Navigating](https://nextjs.org/docs/app/getting-started/linking-and-navigating)
- [Next.js: usePathname](https://nextjs.org/docs/app/api-reference/functions/use-pathname)
