# Chapter 7. Fetching Data

## 학습 목표

Server Component에서 PostgreSQL을 직접 조회해 Dashboard UI에 필요한 Data를 표시하고, SQL에서 필요한 Data만 선택하는 이유와 Request Waterfall 및 병렬 실행의 차이를 이해한다.

## Data 조회 방식

Application은 상황에 따라 API, ORM 또는 SQL을 사용해 Data를 조회할 수 있다.

```text
Client Component
-> API, Route Handler 또는 Server Function
-> 인증, 인가와 입력 검증
-> Database

Server Component
-> Server에서 직접 SQL 실행 가능
-> Database
```

Client Component가 Database에 직접 연결하면 접속 정보가 Browser에 노출되고 사용자가 요청을 임의로 조작할 수 있다. Server 경계를 두어 인증, 인가, 입력 검증과 Rate Limiting을 적용해야 한다.

Server Component는 Server에서 실행되므로 Database 접속 문자열과 Query Code를 Client Bundle에 포함하지 않고 직접 Data를 조회할 수 있다. `async` Component에서 `await`을 사용하므로 조회를 위해 `useEffect`와 별도 Client State를 만들 필요도 없다.

다만 Server에서 조회했다는 사실만으로 조회 결과까지 비공개가 되는 것은 아니다. Password나 내부 원가를 JSX 또는 Client Component Props에 포함하면 RSC Payload와 Network Response를 통해 Browser에 전달될 수 있다. Client에는 표시와 상호작용에 필요한 최소한의 값만 전달한다.

## SQL에서 필요한 Data만 조회하기

최근 Invoice 5개가 필요하다면 모든 Invoice를 Application으로 가져와 JavaScript로 정렬하지 않고 Database에서 필요한 행만 선택한다.

```sql
SELECT invoices.amount,
       customers.name,
       customers.image_url,
       customers.email
FROM invoices
JOIN customers ON invoices.customer_id = customers.id
ORDER BY invoices.date DESC
LIMIT 5;
```

이 방식은 JavaScript와 SQL 언어 자체의 단순한 속도 비교가 아니다. 불필요한 Data를 Server로 전송하지 않으므로 Network 전송량, Server Memory와 JavaScript 정렬 작업을 줄일 수 있다. 적절한 Index가 있다면 Database가 필요한 행을 찾는 과정도 효율화할 수 있다.

## Request Waterfall

서로 독립적인 요청을 순차적으로 호출하고 기다리면 앞의 요청이 끝나야 다음 요청이 시작된다.

```ts
const revenue = await fetchRevenue();
const invoices = await fetchLatestInvoices();
const cards = await fetchCardData();
```

각 요청이 2초, 4초, 1초 걸린다면 전체 시간은 대략 7초가 된다.

```text
Revenue 2초
-> Invoices 4초
-> Cards 1초
-> 약 7초
```

Waterfall은 항상 잘못된 것은 아니다. 먼저 User ID를 조회한 후 그 ID로 친구 목록을 조회하는 것처럼 다음 요청이 이전 결과에 의존한다면 순차 실행이 필요하다.

## 요청을 함께 시작하기

독립적인 작업은 Promise를 먼저 생성해 함께 시작할 수 있다.

```ts
const revenuePromise = fetchRevenue();
const invoicesPromise = fetchLatestInvoices();
const cardsPromise = fetchCardData();

const [revenue, invoices, cards] = await Promise.all([
  revenuePromise,
  invoicesPromise,
  cardsPromise,
]);
```

`Promise.all()`이 비동기 작업을 직접 병렬로 만드는 것은 아니다. 비동기 함수를 호출할 때 작업이 시작되고, `Promise.all()`은 이미 시작된 Promise가 모두 끝날 때까지 함께 기다리며 결과를 모은다.

두 Promise를 먼저 만든 뒤 각각 `await`해도 두 작업은 이미 진행 중이다.

```ts
const revenuePromise = fetchRevenue(); // 요청 시작
const invoicesPromise = fetchLatestInvoices(); // 요청 시작

const revenue = await revenuePromise;
const invoices = await invoicesPromise;
```

두 요청이 2초와 4초라면 전체 준비 시간은 합계인 6초가 아니라 오래 걸리는 요청을 기준으로 약 4초다. Waterfall 여부는 `await`의 개수가 아니라 **각 작업을 언제 시작했는가**로 판단한다.

## Static Rendering과 Data 갱신

Chapter 7의 Production Build에서는 `/dashboard`가 Static Route로 Prerendering됐다. Build 시 생성한 결과는 Database가 변경되어도 자동으로 새 HTML을 만들지 않으므로 화면에 최신 Data가 즉시 반영되지 않을 수 있다.

이 문제는 다음 Chapter에서 Static Rendering과 Dynamic Rendering의 차이로 이어진다.

## 구현에서 확인한 내용

- `fetchRevenue()`로 최근 12개월 Revenue를 조회했다.
- `fetchLatestInvoices()`로 최근 Invoice 5개를 조회했다.
- `fetchCardData()`로 Invoice와 Customer 집계 Data를 조회했다.
- Server Component인 Dashboard Page에서 조회 결과를 각 UI Component에 전달했다.
- `fetchCardData()` 내부의 독립적인 Query를 `Promise.all()`로 함께 기다렸다.
- Production Build와 TypeScript 검사를 통과했다.
- `/dashboard`가 Static Route로 Prerendering되는 것을 확인했다.
- `ch07: fetch dashboard data from postgres` Commit으로 Data 조회 작업을 구분했다.
- `ch07: remove temporary database routes` Commit으로 임시 Endpoint를 정리했다.

## 핵심 정리

> Server Component는 Database 접속 정보와 Query Code를 Browser에 전달하지 않고 Server에서 직접 Data를 조회할 수 있다. SQL에서 필요한 행과 집계 결과만 가져오면 Network 전송량과 Application의 작업을 줄일 수 있다. 서로 독립적인 비동기 작업은 첫 `await` 전에 시작해 Waterfall을 줄이며, `Promise.all()`은 이미 시작된 Promise들을 함께 기다리고 결과를 모은다. 반대로 다음 요청이 이전 결과에 의존한다면 의도적인 순차 실행이 필요하다.

## 참고 자료

- [Next.js Learn: Fetching Data](https://nextjs.org/learn/dashboard-app/fetching-data)
- [React: Server Components](https://react.dev/reference/rsc/server-components)
- [MDN: Promise.all](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
- [postgres.js](https://github.com/porsager/postgres)
