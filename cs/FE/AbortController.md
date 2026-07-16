# AbortController

`AbortController`는 진행 중인 비동기 작업에 취소 의도를 전달하기 위한 Web API이다. `fetch` 요청 취소에 자주 사용하지만, `AbortSignal`을 지원하도록 구현된 다른 작업에도 사용할 수 있다.

## AbortController와 AbortSignal

- `AbortController`: 취소 명령을 내리는 주체
- `AbortSignal`: 취소 여부와 이유를 작업에 전달하는 신호

```js
const controller = new AbortController();

fetch('/api/users', {
  signal: controller.signal,
});

controller.abort();
```

`abort()`를 호출하면 `controller.signal.aborted`가 `true`가 되고, 해당 signal을 사용하는 작업에 취소가 전달된다.

## Promise를 직접 취소하는 기능은 아니다

`AbortController`가 일반 Promise를 강제로 멈추는 것은 아니다. API나 함수가 `AbortSignal`을 전달받고 취소 신호에 반응하도록 구현되어 있어야 한다.

```js
function wait(ms, { signal }) {
  signal.throwIfAborted();

  return new Promise((resolve, reject) => {
    const onAbort = () => {
      clearTimeout(timerId);
      reject(signal.reason);
    };

    const timerId = setTimeout(() => {
      signal.removeEventListener('abort', onAbort);
      resolve();
    }, ms);

    signal.addEventListener('abort', onAbort, { once: true });
  });
}
```

직접 만든 비동기 함수가 signal을 확인하지 않으면 `abort()`를 호출해도 작업은 계속된다.

## Signal은 한 번만 사용할 수 있다

한 번 취소된 signal은 계속 `aborted: true` 상태를 유지한다. 이미 취소된 signal을 새로운 `fetch`에 전달하면 새로운 요청도 바로 취소된다.

```js
controller.abort();

// 이미 취소된 signal이므로 바로 취소된다.
fetch('/api/users', { signal: controller.signal });
```

새 요청에는 새로운 controller를 만들어야 한다.

## React Effect에서 사용하기

사용자 A를 조회한 직후 B를 조회했는데 A의 응답이 늦게 도착하면, A의 결과가 B의 최신 상태를 덮어쓰는 race condition이 발생할 수 있다. Effect cleanup에서 이전 요청을 취소하면 이를 줄일 수 있다.

```tsx
useEffect(() => {
  const controller = new AbortController();

  async function loadUser() {
    try {
      const response = await fetch(`/api/users/${userId}`, {
        signal: controller.signal,
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      const user = await response.json();
      setUser(user);
    } catch (error) {
      if (error instanceof DOMException && error.name === 'AbortError') {
        return;
      }

      setError(error);
    }
  }

  loadUser();

  return () => controller.abort();
}, [userId]);
```

의존성이 바뀌면 이전 cleanup에서 기존 요청을 취소한 뒤, 새로운 Effect에서 새로운 controller와 요청을 만든다. controller를 Effect 바깥에서 만들어 여러 렌더링에 재사용하면 한 번 취소된 signal 때문에 이후 요청도 즉시 취소될 수 있다.

취소로 발생한 `AbortError`만 구분해야 한다. 모든 에러를 무시하면 네트워크 장애, 응답 파싱 오류, 코드 오류까지 숨길 수 있다. 또한 `fetch`는 HTTP `404`나 `500`만으로 reject되지 않으므로 `response.ok`를 별도로 확인한다.

## 여러 작업과 취소 범위

같은 signal을 여러 작업에 전달하면 한 번의 `abort()`로 관련 작업을 함께 취소할 수 있다.

```js
const controller = new AbortController();

const userRequest = fetch('/api/user', { signal: controller.signal });
const orderRequest = fetch('/api/orders', { signal: controller.signal });

controller.abort();
```

두 작업의 생명주기가 같다면 편리하지만, 하나만 계속 실행해야 할 수 있다면 controller를 분리해야 한다. 취소 범위는 작업의 생명주기를 기준으로 결정한다.

## Timeout과 Signal 조합

`AbortSignal.timeout()`은 제한 시간이 지난 요청을 취소하는 상황을 표현한다.

```js
fetch('/api/report', {
  signal: AbortSignal.timeout(5000),
});
```

사용자 이동이나 최신 요청으로의 교체처럼 애플리케이션이 직접 취소해야 할 때는 `controller.abort()`를 사용한다. 직접 취소와 timeout을 함께 적용하려면 `AbortSignal.any()`로 여러 signal을 결합할 수 있다.

```js
const controller = new AbortController();
const signal = AbortSignal.any([
  controller.signal,
  AbortSignal.timeout(5000),
]);

fetch('/api/report', { signal });
```

## Abort와 Ignore 비교

| 방식 | 작업 처리 | 오래된 상태 반영 | 자원 사용 |
| --- | --- | --- | --- |
| Abort | 지원하는 작업에 중단을 요청 | 방지 | 네트워크와 브라우저 작업을 줄일 수 있음 |
| Ignore | 작업은 계속 실행 | 결과만 무시 | 요청, 응답 처리 등의 작업이 계속될 수 있음 |

일반적으로 abort가 네트워크와 브라우저 자원을 더 절약할 수 있다. 다만 signal을 지원하지 않는 비동기 작업은 cleanup 이후 결과를 무시하는 방식으로 오래된 상태 반영을 막아야 한다.

## 서버 작업 취소와의 차이

클라이언트에서 `fetch`를 abort했다고 서버의 작업까지 취소됐다고 볼 수는 없다. 요청이 이미 서버에 도착했다면 결제나 주문 처리가 완료될 수 있다.

따라서 중요한 변경 요청에는 AbortController만 의존하지 않고 다음과 같은 서버 측 설계가 함께 필요하다.

- 중복 처리를 막는 Idempotency-Key
- 데이터 일관성을 위한 transaction
- 처리 상태 조회 API
- 필요한 경우 별도의 취소 API

## 정리

> AbortController는 비동기 작업을 강제로 제거하는 기능이 아니라 AbortSignal을 통해 취소 의도를 전달하는 방식이다. signal은 한 번 취소되면 재사용할 수 없으며, React에서는 Effect마다 controller를 만들고 cleanup에서 abort해 오래된 요청이 최신 상태를 덮어쓰는 문제를 줄일 수 있다. 다만 클라이언트 요청 취소는 서버 작업의 rollback을 보장하지 않는다.

## 참고

- [WHATWG DOM Standard: Aborting ongoing activities](https://dom.spec.whatwg.org/#aborting-ongoing-activities)
- [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
- [MDN: AbortSignal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)
