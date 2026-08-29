# Chapter 8. Static and Dynamic Rendering

## 학습 목표

Static Rendering과 Dynamic Rendering을 HTML 생성 시점으로 구분하고, 느린 Data 조회가 Build와 Request에 미치는 영향을 이해한다. 또한 Streaming이 조회 시간을 단축하는 기능이 아니라 준비된 UI를 먼저 보여주는 방식이라는 점을 확인한다.

## Static Rendering

Static Rendering은 Page의 결과를 Build 또는 Revalidation 시점에 미리 생성한다.

```text
Build 또는 Revalidation
-> Data 조회
-> HTML과 Rendering 결과 생성
-> Cache 또는 정적 자산으로 보관

User Request
-> 이미 생성된 결과 응답
```

사용자마다 결과가 달라지지 않는 회사 소개나 공통 상품 설명처럼 미리 확정할 수 있는 Content에 적합하다. 요청마다 Page를 다시 만들 필요가 없으므로 Server 부하가 작고, Cache와 CDN을 활용해 빠르게 응답할 수 있다.

Static Page에서 Data 조회에 3초가 걸린다면 그 비용은 Build 또는 Revalidation 과정에서 발생한다. 배포 후 사용자가 Page를 열 때마다 같은 3초를 다시 기다리는 것은 아니다. 다만 원본 Data가 변경되어도 다시 Build하거나 Revalidate하기 전까지 이전 결과가 보일 수 있다.

## Dynamic Rendering

Dynamic Rendering은 사용자 Request가 들어온 시점에 필요한 Data를 조회하고 결과를 생성한다.

```text
User Request
-> Cookie, Header와 Request 정보 확인
-> Data 조회
-> 사용자에게 맞는 결과 생성
-> Response
```

로그인한 사용자별 주문 내역처럼 Cookie와 인증 정보에 따라 결과가 달라지거나, 요청할 때마다 최신 Data가 필요한 Page에 적합하다. Build 시점에는 어떤 사용자가 요청할지 알 수 없으므로 하나의 공통 결과로 미리 확정할 수 없기 때문이다.

Dynamic Rendering에서는 Request마다 Server 작업이 발생한다. Page가 느린 조회를 먼저 모두 `await`한 뒤 결과를 만든다면 가장 느린 작업이 끝날 때까지 초기 화면도 기다릴 수 있다.

## 인위적인 지연으로 확인한 동작

`fetchRevenue()`에 3초 지연을 추가해 느린 Data 조회를 재현했다.

```ts
console.log('Fetching revenue data...');

await new Promise((resolve) => setTimeout(resolve, 3000));

const data = await sql<Revenue[]>`SELECT * FROM revenue`;

console.log('Data fetch completed after 3 seconds.');
```

개발 환경에서는 Dashboard를 열 때 지연과 Terminal Log를 관찰할 수 있다. 하지만 개발 환경에서 Request마다 코드가 실행되는 모습만으로 Production Page가 Dynamic이라고 판단하면 안 된다. Production Build 결과에서 Route가 Static인지 Dynamic인지 별도로 확인해야 한다.

Chapter 7의 Production Build에서는 `/dashboard`가 Static Route로 표시됐다. 이 경우 3초 지연은 Production의 사용자 요청마다 발생하는 지연이 아니라 Build 시점에 발생하는 지연이다.

## Rendering 방식과 느린 조회 구분하기

Static과 Dynamic은 **결과를 언제 생성하는가**에 관한 구분이다. 느린 조회를 어떻게 화면에 나누어 보여줄지는 별도의 문제다.

```text
Static Rendering
-> Build 또는 Revalidation 시점에 결과 생성

Dynamic Rendering
-> Request 시점에 결과 생성

Streaming
-> 준비된 HTML부터 나누어 전송

Suspense
-> 느린 UI 영역의 경계와 Fallback 지정
```

Streaming은 매출 조회의 3초 실행 시간을 줄이지 않는다. 매출 차트가 준비되는 동안 다른 Dashboard 영역을 먼저 보여줘 사용자가 느끼는 대기 시간을 줄인다. 느린 영역을 `Suspense`로 분리하면 해당 영역에는 Fallback을 표시하고 준비된 다른 영역은 먼저 응답할 수 있다.

## 구현에서 확인한 내용

- `fetchRevenue()`에 3초의 인위적인 지연을 추가했다.
- Terminal Log로 조회 시작과 완료 시점을 확인할 수 있게 했다.
- Static Page의 지연 비용은 Build 또는 Revalidation 시점에 발생한다는 점을 구분했다.
- 사용자별 Request 정보가 필요한 Page에는 Dynamic Rendering이 적합한 이유를 확인했다.
- Streaming은 Data 조회 시간을 단축하지 않고 준비된 UI를 먼저 보여주는 방식임을 정리했다.
- `ch08: explore static and dynamic rendering` Commit으로 실습 변경을 구분했다.

## 핵심 정리

> Static Rendering은 Build 또는 Revalidation 시점에 결과를 미리 생성해 요청 시 빠르게 제공하고, Dynamic Rendering은 Request 시점의 사용자 정보와 최신 Data를 반영해 결과를 생성한다. Static Page의 느린 조회는 Build 비용이 되고 Dynamic Page의 느린 조회는 사용자 응답을 지연시킬 수 있다. Streaming과 Suspense는 작업 자체를 빠르게 만들지는 않지만 준비된 UI를 먼저 보여줘 체감 대기를 줄인다.

## 참고 자료

- [Next.js Learn: Static and Dynamic Rendering](https://nextjs.org/learn/dashboard-app/static-and-dynamic-rendering)
- [Next.js: Rendering](https://nextjs.org/docs/app/getting-started/partial-prerendering)
- [React: Suspense](https://react.dev/reference/react/Suspense)
