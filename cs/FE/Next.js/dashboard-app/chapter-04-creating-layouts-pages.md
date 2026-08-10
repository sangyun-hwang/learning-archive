# Chapter 4. Creating Layouts and Pages

## 학습 목표

App Router의 file-system routing으로 중첩 route를 만들고, Root Layout과 Dashboard Layout을 조합해 route 범위별 공통 UI를 구성한다.

## File-system Routing

Next.js App Router에서는 `app` 아래의 folder가 URL segment를 표현한다. `app` 자체는 routing의 기준 folder이므로 URL에 포함되지 않는다.

```text
app/dashboard/customers/page.tsx

app
-> App Router 기준 folder
-> URL에 포함되지 않음

dashboard
-> /dashboard segment

customers
-> /customers segment

page.tsx
-> 해당 경로를 접근 가능한 Page로 만듦

결과
-> /dashboard/customers
```

Folder만 만든다고 외부에서 접근 가능한 Page가 생기지는 않는다. 해당 경로에 `page.tsx`가 있어야 UI route가 공개된다. `route.ts`는 같은 방식으로 HTTP endpoint를 정의하는 특별한 file이다.

## Colocation

`app` folder 안에는 Page와 함께 UI Component, utility, type과 test file을 가까이 둘 수 있다. Next.js가 특별한 file convention으로 인식하지 않는 일반 file은 자동으로 공개 route가 되지 않는다.

```text
app/dashboard/customers/
├─ page.tsx
├─ table.tsx
└─ helpers.ts
```

위 구조에서 `/dashboard/customers`는 공개 Page가 되지만 `table.tsx`와 `helpers.ts`에 대응하는 URL은 생기지 않는다. Route와 관련 code를 가까이 배치하는 방식을 colocation이라고 한다.

## Root Layout과 Nested Layout

Root Layout은 application의 모든 route에 적용되고 App Router에서 필수다. `<html>`과 `<body>`, global CSS와 기본 font처럼 application 전체에서 공유할 내용을 둔다.

Dashboard Layout은 `/dashboard`와 그 하위 route에만 적용된다. SideNav처럼 Dashboard에서만 필요한 공통 UI를 Root Layout에 넣지 않고 해당 범위의 nested layout에 둔다.

| 구분 | Root Layout | Dashboard Layout |
| --- | --- | --- |
| 위치 | `app/layout.tsx` | `app/dashboard/layout.tsx` |
| 범위 | 모든 route | `/dashboard` 하위 route |
| 역할 | `html`, `body`, global style, 기본 font | SideNav와 Dashboard 공통 UI |
| 필수 여부 | App Router에서 필수 | 해당 route에 필요할 때 추가 |

## children과 Layout 조합

Layout의 `children`에는 현재 경로와 일치하는 하위 Page 또는 더 안쪽 Layout이 들어온다.

```text
/dashboard
-> dashboard/page.tsx

/dashboard/customers
-> dashboard/customers/page.tsx

/dashboard/invoices
-> dashboard/invoices/page.tsx
```

`/dashboard/invoices`에 접근하면 다음 순서로 중첩된다.

```text
Root Layout
└─ Dashboard Layout
   ├─ SideNav
   └─ children
      └─ Invoices Page
```

## Partial Rendering

Dashboard 내부에서 Client Navigation이 발생하면 공유 Layout을 유지하고 변경된 Page 영역을 중심으로 갱신할 수 있다.

```text
/dashboard/invoices
-> /dashboard/customers

유지
-> Root Layout
-> Dashboard Layout
-> SideNav와 그 안의 Client state

교체
-> Invoices Page에서 Customers Page로 변경
```

Layout이 유지되면 내부 Client Component instance도 유지될 수 있어 navigation 사이에서 state를 보존할 수 있다. 주소창 직접 입력이나 새로고침은 새 document request이므로 Client Navigation에서의 Layout 재사용과 구분해야 한다.

## 구현에서 확인한 내용

- `/dashboard`, `/dashboard/customers`, `/dashboard/invoices` Page를 만들었다.
- Dashboard Layout에서 SideNav와 Page가 들어갈 `children` 영역을 분리했다.
- Dashboard 하위 Page에서만 SideNav가 공통으로 표시되는 것을 확인했다.
- Root Layout에는 application 전체의 Inter font와 `antialiased`를 유지했다.

## 핵심 정리

> App Router에서는 `app` 아래 folder가 URL segment를 표현하고 `page.tsx`가 해당 경로를 접근 가능한 Page로 만든다. 일반 Component와 utility는 route와 함께 배치해도 URL이 생기지 않으므로 colocation이 가능하다. Root Layout은 모든 route에 적용되고 nested layout은 특정 route 범위의 공통 UI를 담당한다. Client Navigation에서는 공유 Layout을 유지한 채 Page 영역을 중심으로 갱신할 수 있어 중복 rendering을 줄이고 Layout 내부 Client state를 보존할 수 있다.

## 참고 자료

- [Next.js Learn: Creating Layouts and Pages](https://nextjs.org/learn/dashboard-app/creating-layouts-and-pages)
- [Next.js: Layouts and Pages](https://nextjs.org/docs/app/getting-started/layouts-and-pages)
