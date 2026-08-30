# Chapter 9. Streaming

## 학습 목표

느린 Data 조회가 Page 전체의 표시를 막지 않도록 Next.js의 Streaming과 React Suspense를 적용한다. Page 수준과 Component 수준의 Loading UI를 구분하고, Route Group으로 `loading.tsx`의 적용 범위를 조정하는 방법을 이해한다.

## Streaming

Streaming은 Server에서 Page 전체가 완성될 때까지 기다리지 않고, 준비된 HTML을 여러 Chunk로 나누어 Client에 먼저 전송하는 방식이다.

```text
기존 방식
-> 모든 Data 준비
-> 전체 UI 생성
-> Client에 응답

Streaming
-> 먼저 준비된 UI 전송
-> 느린 영역은 Fallback 표시
-> Data가 준비된 영역부터 추가 전송
```

Streaming은 느린 Query 자체의 실행 시간을 줄이지 않는다. `fetchRevenue()`가 3초 걸린다면 Revenue Chart는 여전히 3초를 기다린다. 대신 다른 Dashboard 영역을 먼저 보여줘 사용자가 느끼는 대기 시간을 줄인다.

## `loading.tsx`와 Suspense

Next.js의 `loading.tsx`는 해당 Route Segment의 Page에 Page 수준의 Suspense 경계를 자동으로 제공한다.

```tsx
import DashboardSkeleton from '@/app/ui/skeletons';

export default function Loading() {
  return <DashboardSkeleton />;
}
```

Dashboard Page가 준비되는 동안 공통 Layout과 SideNav는 유지되고 Page 영역에는 `DashboardSkeleton`이 표시된다. 사용자는 전체 Data가 끝나기 전에도 이미 준비된 Navigation을 사용할 수 있다.

직접 작성한 Suspense는 Component 단위로 더 좁은 경계를 만들 수 있다.

```tsx
<Suspense fallback={<RevenueChartSkeleton />}>
  <RevenueChart />
</Suspense>
```

Revenue 조회가 느려도 Revenue 영역만 Skeleton을 표시하고 다른 경계의 UI는 준비되는 대로 보여줄 수 있다.

## Data 조회를 Component로 내리기

Page가 `fetchRevenue()`를 먼저 `await`한 후 결과를 Props로 전달하면 Page 자체가 Data를 기다린다. 이 경우 Revenue Chart를 Suspense로 감싸도 Suspense 경계에 도달하기 전에 상위 Page가 중단되므로 해당 영역만 분리하기 어렵다.

```text
Page에서 조회
-> Page 전체가 Revenue를 기다림
-> RevenueChart에 결과 전달

RevenueChart에서 조회
-> RevenueChart의 Suspense 경계 안에서 대기
-> 다른 Page 영역은 먼저 Rendering 가능
```

실습에서는 각 조회를 실제 사용하는 Server Component로 이동했다.

- `fetchRevenue()`는 `RevenueChart`에서 실행한다.
- `fetchLatestInvoices()`는 `LatestInvoices`에서 실행한다.
- `fetchCardData()`는 `CardWrapper`에서 실행한다.

Suspense가 비동기 요청을 자동으로 병렬 실행시키는 것은 아니다. 독립적인 Component의 Rendering이 시작되며 각 Data 작업도 함께 시작될 수 있고, Suspense는 그 결과를 경계별로 기다리고 표시하는 역할을 한다.

## Suspense 경계의 크기

여러 Component를 하나의 Suspense로 묶으면 포함된 영역 중 하나가 아직 준비되지 않았을 때 전체 Fallback이 유지될 수 있다. 각각 분리하면 먼저 준비된 영역부터 개별적으로 보여줄 수 있다.

```text
큰 경계 하나
-> 화면 전환이 단순함
-> 느린 작업이 전체 영역 표시를 늦출 수 있음

작은 경계 여러 개
-> 준비된 영역부터 표시 가능
-> 너무 잘게 나누면 UI가 산발적으로 나타날 수 있음
```

경계는 무조건 작게 만드는 것이 아니라 사용자가 함께 보기를 기대하는 UI와 Data 의존 관계를 기준으로 정한다. Dashboard Card처럼 하나의 묶음으로 인식되는 항목은 Wrapper Component로 그룹화해 함께 표시할 수 있다.

## Route Group

`app/dashboard/loading.tsx`는 `/dashboard` 아래의 Invoices와 Customers Page에도 적용될 수 있다. Dashboard Overview에만 Loading UI를 적용하기 위해 `(overview)` Route Group을 사용했다.

```text
app/dashboard/
├─ (overview)/
│  ├─ page.tsx
│  └─ loading.tsx
├─ invoices/page.tsx
├─ customers/page.tsx
└─ layout.tsx
```

괄호로 감싼 Route Group은 파일을 논리적으로 묶지만 URL Segment가 되지 않는다. 따라서 `app/dashboard/(overview)/page.tsx`의 주소는 `/dashboard/(overview)`가 아니라 `/dashboard`다.

## Static Build와 Streaming

실습의 Production Build에서 `/dashboard`는 Static Route로 Prerendering됐다. `fetchRevenue()`의 3초 지연은 Build 시점에 발생하며 완성된 정적 결과가 배포되므로, 사용자가 요청할 때마다 3초 동안 Server Streaming이 발생하지 않는다.

개발 환경에서는 Request마다 Rendering되는 모습을 통해 Skeleton과 Streaming 경계를 관찰할 수 있다. Production에서 Request 시점 Streaming을 사용하려면 해당 Route가 Dynamic Rendering되는 조건도 필요하다.

## 구현에서 확인한 내용

- `(overview)` Route Group으로 Dashboard Loading UI의 적용 범위를 제한했다.
- `loading.tsx`에서 Dashboard Page 수준의 Skeleton을 제공했다.
- Revenue, Latest Invoices와 Cards 영역을 개별 Suspense 경계로 분리했다.
- Data 조회를 Page에서 실제 사용하는 Server Component로 이동했다.
- `fetchRevenue()`의 3초 지연으로 Revenue 영역의 Loading 상태를 확인했다.
- Production Build와 TypeScript 검사를 통과했다.
- Production Build에서 `/dashboard`가 Static Route로 표시되는 것을 확인했다.
- `ch09: add dashboard streaming` Commit으로 실습 변경을 구분했다.

## 핵심 정리

> Streaming은 느린 작업의 실행 시간을 줄이지 않고 준비된 UI부터 전송해 체감 대기를 줄인다. `loading.tsx`는 Route Segment의 Page 수준 Loading UI를 제공하고, 직접 작성한 Suspense는 Component 단위로 경계를 조절한다. 특정 영역만 Streaming하려면 Data 조회를 그 영역의 Server Component로 내려 Suspense 경계 안에서 대기하도록 구성한다.

## 참고 자료

- [Next.js Learn: Streaming](https://nextjs.org/learn/dashboard-app/streaming)
- [Next.js: Loading UI and Streaming](https://nextjs.org/docs/app/api-reference/file-conventions/loading)
- [React: Suspense](https://react.dev/reference/react/Suspense)
