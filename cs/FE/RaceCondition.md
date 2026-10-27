# Race Condition: 비동기 응답 순서와 최신 화면

## 개념

Race Condition은 여러 작업의 실행·완료 순서에 따라 결과가 달라지는 문제다. 프론트엔드에서는 이전 요청의 응답이 늦게 도착해 최신 화면을 덮어쓰는 사례로 이해할 수 있다.

```text
0초: react 검색 요청 A 시작
1초: next 검색 요청 B 시작
2초: B 응답 -> next 결과 표시
3초: A 응답 -> react 결과로 덮어씀
```

사용자가 마지막으로 선택한 검색어는 next이지만 화면에는 react 결과가 남는다. 네트워크와 서버 처리 시간 때문에 요청 시작 순서와 응답 완료 순서가 다를 수 있다.

## 싱글 스레드에서도 발생하는 이유

메인 스레드의 JavaScript 코드는 한 번에 하나씩 실행되지만, 여러 비동기 요청의 진행 시간은 겹칠 수 있다. 각 응답을 처리하는 콜백이 하나씩 실행된다는 사실이 요청 시작 순서대로 완료된다는 뜻은 아니다.

이 사례는 두 코드가 같은 순간 CPU에서 실행되어야만 생기는 문제가 아니다. 뒤늦게 실행된 이전 요청의 콜백이 최신 상태를 덮어쓸 수 있다는 것이 핵심이다.

## 이전 응답 무시하기

React Effect에서는 cleanup으로 이전 실행의 결과를 더 이상 반영하지 않도록 할 수 있다. 아래는 컴포넌트 안에서 query와 상태 변경 함수를 사용하는 부분 예제다.

```jsx
useEffect(() => {
  let ignore = false;

  async function load() {
    try {
      const response = await fetch(
        `/api/search?q=${encodeURIComponent(query)}`,
      );
      if (!response.ok) throw new Error('검색 실패');
      const data = await response.json();

      if (!ignore) setResults(data);
    } catch (error) {
      if (!ignore) setError(error);
    }
  }

  load();
  return () => {
    ignore = true;
  };
}, [query]);
```

query가 바뀌어 새 Effect가 실행되기 전에 이전 Effect의 cleanup이 실행된다. 각 Effect 실행은 별도의 ignore 변수를 가지므로, 이전 요청의 결과는 무시하고 현재 실행의 결과는 반영할 수 있다. 컴포넌트가 제거될 때도 cleanup이 실행된다. [React 공식 예제](https://react.dev/reference/react/useEffect#fetching-data-with-effects)

ignore는 네트워크 요청을 취소하지 않는다. 요청과 응답 처리는 계속될 수 있지만 오래된 실행이 상태를 변경하지 않도록 막는다. 실제 화면에서는 오류 초기화와 로딩 표시도 구성하며, 이전 요청의 오류나 finally에서 최신 요청의 상태를 변경하지 않도록 같은 유효성 검사를 적용한다.

## 취소와 요청 ID 비교

| 방법 | 역할 |
| --- | --- |
| Ignore | 오래된 작업의 결과를 상태에 반영하지 않음 |
| AbortController | signal을 지원하는 작업에 취소 신호 전달 |
| 요청 ID 비교 | 현재 요청 ID와 일치하는 응답만 반영 |

AbortController로 불필요한 fetch를 취소하면 자원 사용을 줄일 수 있다. 지원하지 않는 후속 작업까지 강제로 중단시키는 것은 아니므로 필요하면 결과 반영 시 유효성 검사도 함께 사용한다. 클라이언트에서 요청을 취소해도 이미 실행된 서버 작업이 되돌아가는 것은 아니다. [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)

요청 ID 비교는 요청을 시작할 때 번호를 증가시키고 응답 시 최신 번호인지 확인하는 방식이다. React 이벤트 핸들러에서는 ref 등에 최신 번호를 유지할 수 있다.

```js
// 이벤트 핸들러 내부의 흐름 예시
const requestId = ++latestRequestId.current;
const data = await search(query);
if (requestId === latestRequestId.current) {
  setResults(data);
}
```

위 코드는 아이디어를 설명하는 부분 예제이며 오류 처리와 컴포넌트 제거 시 처리는 별도로 구성한다. '최신 요청만 반영'은 검색 같은 작업의 정책이지, 모든 작업에 적용하는 규칙은 아니다. 여러 상품을 추가하는 작업이라면 각 작업의 결과가 모두 중요할 수 있다.

## Debounce만으로 해결되지 않는 이유

Debounce는 입력이 이어지는 동안 실행을 미루어 요청 횟수를 줄인다. 이미 전송된 요청의 완료 순서를 보장하거나 자동으로 취소하지는 않는다. 따라서 요청 횟수 제어와 오래된 응답 반영 방지를 구분한다.

## 간단한 실습 방향

검색어 A의 응답은 2초, 검색어 B는 0.3초 뒤에 도착하도록 가짜 API를 구성한다. A를 요청한 뒤 B를 요청해 오래된 결과가 화면을 덮는지 확인하고, ignore나 요청 ID 비교를 추가해 결과를 비교할 수 있다. 실제 실습과 브라우저 검증은 아직 진행하지 않았다.

## 말로 설명하기

프론트엔드의 Race Condition은 비동기 작업의 완료 순서에 따라 화면 결과가 달라지는 문제입니다. 검색 요청을 연속으로 보냈을 때 먼저 보낸 응답이 늦게 도착해 최신 결과를 덮어쓸 수 있습니다. JavaScript는 싱글 스레드지만 네트워크 요청의 완료 순서까지 보장하지는 않습니다. 이전 결과를 무시하거나 요청 ID로 최신 응답인지 확인하고, 취소 가능한 작업에는 AbortController를 사용할 수 있습니다. Debounce는 요청 횟수를 줄이는 방법이라 응답 순서 문제는 별도로 처리해야 합니다.

## 관련 학습

- [AbortController](AbortController.md)
- [React Effect](React.md)
- [Event Loop](EventLoop.md)
