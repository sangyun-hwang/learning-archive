# Ky

## 개념

Ky는 Fetch API를 기반으로 반복적인 HTTP request 처리를 편리하게 만든 작은 client library다. 새로운 network protocol이 아니라 Fetch API를 감싼 wrapper다.

```ts
import ky from 'ky';

const user = await ky.get('/api/users/1').json<User>();
```

## Fetch와 비교

Fetch API에서는 JSON request, HTTP status 검사와 response parsing을 직접 작성하는 경우가 많다.

```ts
const response = await fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Kim' }),
});

if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}

const data = await response.json();
```

Ky는 method shortcut과 `json` option, response parsing을 제공한다.

```ts
const data = await ky
  .post('/api/users', {
    json: { name: 'Kim' },
  })
  .json<User>();
```

주요 기능은 다음과 같다.

- `get`, `post`, `put`, `delete` 등 method shortcut
- JSON request와 response parsing
- Non-2xx response의 `HTTPError` 처리
- Timeout과 retry
- Request와 response hook
- 공통 설정을 가진 instance
- TypeScript generic과 schema validation 연계
- `AbortSignal` 지원

## HTTP Error와 Network Error

Fetch의 Promise는 HTTP `404`나 `500`에서도 서버의 response를 받았다면 resolve된다. 개발자가 `response.ok`를 확인해 HTTP 오류를 분류해야 한다.

```text
fetch 404
-> Response로 resolve
-> response.ok === false

Ky 404
-> 기본적으로 HTTPError throw
```

Fetch가 reject될 수 있는 대표적인 경우는 DNS 실패, 연결 단절, CORS로 인한 network failure 또는 `AbortSignal` 취소처럼 유효한 response를 얻지 못한 경우다. HTTP 오류 응답과 network-level 실패를 구분해야 한다.

## 공통 Instance와 Hook

공통 API URL, Header와 request 전후 처리를 instance에 모을 수 있다.

```ts
const api = ky.create({
  baseUrl: 'https://example.com/api/',
  timeout: 10_000,
  hooks: {
    beforeRequest: [
      ({ request }) => {
        request.headers.set('Authorization', `Bearer ${token}`);
      },
    ],
  },
});

const users = await api.get('users').json<User[]>();
```

인증 token 갱신처럼 여러 request에 공통으로 필요한 처리를 hook으로 구성할 수 있다. 다만 hook이 지나치게 많은 책임을 가지면 request 흐름을 추적하기 어려워질 수 있다.

## Retry와 멱등성

Ky는 retry를 지원하지만 HTTP client의 재시도가 Server 작업을 취소하거나 멱등성을 보장하지는 않는다. Ky의 기본 retry method에는 `POST`가 포함되지 않지만 설정이나 application code로 POST를 재시도할 때는 주의해야 한다.

```text
결제 POST 전송
-> Server에서 결제 완료
-> Client가 response를 받기 전에 연결 문제
-> Client가 실패로 판단하고 재시도
-> 중복 결제 가능
```

결제와 주문에는 `Idempotency-Key`와 Server의 처리 기록을 사용해 같은 요청이 여러 번 도착해도 중복 처리되지 않게 해야 한다. Client의 timeout이나 abort도 이미 시작된 Server 작업을 자동으로 되돌리지 않는다.

## 면접 답변

> Ky는 Fetch API를 기반으로 method shortcut, JSON 처리, non-2xx 오류 처리, timeout, retry와 hook을 제공하는 HTTP client wrapper입니다. Fetch는 404 응답에서도 resolve되므로 `response.ok`를 직접 검사해야 하지만 Ky는 기본적으로 non-2xx를 HTTPError로 처리합니다. Retry는 편리하지만 결제나 주문의 중복 처리를 막아주지는 않으므로 Server에서 Idempotency-Key 같은 멱등성 보장이 필요합니다.

## 참고 자료

- [Ky](https://github.com/sindresorhus/ky)
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
