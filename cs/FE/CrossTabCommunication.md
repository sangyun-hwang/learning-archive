# 여러 탭과 창 사이의 통신

## 개념

각 탭의 JavaScript 실행 환경과 React state는 별도다. 한 탭에서 state를 바꿔도 다른 탭의 state가 자동으로 갱신되지 않는다. 메시지를 전달하고, 받은 쪽이 자신의 상태를 변경하도록 구현한다.

대표적으로 BroadcastChannel과 window.postMessage를 사용할 수 있다. BroadcastChannel에서 메시지를 보내는 메서드도 postMessage이지만 통신 대상과 사용 조건은 다르다.

| 구분 | BroadcastChannel | window.postMessage |
| --- | --- | --- |
| 대상 | 같은 이름의 채널에 연결한 탭·창 등 | 참조를 가진 특정 창이나 iframe |
| Origin | 같은 Origin과 저장소 파티션 내 | 다른 Origin에도 전달 가능 |
| 대상 창 참조 | 필요 없음 | 필요 |
| 활용 | 여러 탭의 로그아웃·테마 변경 알림 | 부모 창과 팝업·iframe 통신 |

Origin은 프로토콜, 호스트와 포트의 조합이다. localhost:3000과 localhost:3001은 다른 Origin이다. BroadcastChannel은 저장소 파티션 조건도 적용되므로 서로 다른 최상위 사이트에 포함된 iframe까지 같은 Origin이라는 이유만으로 모두 연결되는 것은 아니다. [MDN: Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API)

## BroadcastChannel 예제

같은 Origin의 두 탭에서 아래 코드를 실행한다고 가정한다. changeTheme은 사용자 동작에서 호출한다.

```js
const channel = new BroadcastChannel('app-events');

function applyTheme(theme) {
  document.documentElement.dataset.theme = theme;
}

channel.onmessage = (event) => {
  const message = event.data;
  if (
    message?.type === 'THEME_CHANGED' &&
    ['light', 'dark'].includes(message.theme)
  ) {
    applyTheme(message.theme);
  }
};

function changeTheme(theme) {
  if (!['light', 'dark'].includes(theme)) return;
  applyTheme(theme);
  channel.postMessage({ type: 'THEME_CHANGED', theme });
}

// 사용을 끝낼 때 channel.close()
```

메시지를 보낸 BroadcastChannel 객체 자신에게는 같은 메시지가 돌아오지 않으므로 보내는 탭의 상태도 직접 변경한다. 수신부에서는 값을 적용하기만 하고 무조건 재전송하지 않아 메시지가 반복되는 상황을 피한다.

메시지는 structured clone 방식으로 전달된다. 객체 등을 전달할 수 있지만 함수나 DOM 요소까지 임의로 공유하는 것은 아니다. 다른 탭의 메모리나 React state를 직접 공유하는 기능도 아니다.

## 메시지 전달과 상태 저장은 다르다

BroadcastChannel은 과거 메시지를 보관해 새 탭에 재생하는 저장소가 아니다.

```text
탭 A가 dark 변경 알림 전송
-> 이미 연결된 탭 B가 수신

그 후 탭 C 열기
-> 과거 변경 알림은 자동 수신하지 않음
```

새 탭의 초기 상태를 맞추려면 다음 중 하나를 추가한다.

- 변경값을 localStorage 등에 저장하고 새 탭이 시작할 때 읽는다.
- 새 탭이 현재 상태를 요청하는 메시지를 보내고 기존 탭이 응답하도록 구현한다.

첫 방식은 저장과 실시간 알림을 조합하는 것이다. 두 번째 방식은 새 탭이 연결된 뒤 새로운 요청과 응답을 주고받는 것이다. 응답할 기존 탭이 없는 경우의 기본값도 정해야 한다.

## window.postMessage

window.open의 반환값, iframe.contentWindow 등으로 대상 창의 참조를 얻어 메시지를 보낸다. 서로 다른 Origin의 창에서도 명시적인 메시지 통신을 할 수 있지만 상대 DOM에 자유롭게 접근할 수 있게 되는 것은 아니다.

아래는 app.example.com의 부모 페이지와 widget.example.com의 iframe이 통신하는 부분 예제다. iframe이 수신 리스너를 등록한 뒤 보내야 한다.

```js
// 부모 페이지: iframe은 해당 요소의 참조
iframe.contentWindow.postMessage(
  { type: 'SET_THEME', theme: 'dark' },
  'https://widget.example.com',
);
```

```js
// iframe 페이지
function handleMessage(event) {
  if (event.origin !== 'https://app.example.com') return;
  if (event.source !== window.parent) return;

  const message = event.data;
  if (
    message?.type === 'SET_THEME' &&
    ['light', 'dark'].includes(message.theme)
  ) {
    document.documentElement.dataset.theme = message.theme;
  }
}

window.addEventListener('message', handleMessage);
// 정리할 때 window.removeEventListener('message', handleMessage)
```

targetOrigin은 누구에게 전달할 수 있는지 제한하고, 수신 시 event.origin은 누가 보냈는지 확인한다. event.source로 예상한 창인지 확인하고 메시지 데이터의 형식과 허용 값도 검증한다. 목적지를 알면 '*' 대신 정확한 Origin을 지정한다. [MDN: window.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)

## React와 간단한 실습

React에서는 Effect에서 채널이나 message 리스너를 등록하고 cleanup에서 close 또는 removeEventListener로 정리한다. 수신 메시지로 setState를 호출하되 콜백이 오래된 state를 읽지 않도록 구성한다.

같은 로컬 서버의 페이지를 두 탭으로 열고 테마 변경을 전달해 볼 수 있다. 이어서 새 탭을 열어 과거 메시지가 재생되지 않는 것을 확인하고, localStorage 초기값 읽기를 추가한다. postMessage는 iframe에 별도 페이지를 넣어 송수신과 Origin 검증을 비교할 수 있다.

## 면접용 설명

탭마다 실행 환경과 상태가 별도이므로 메시지를 받아 각자의 상태를 갱신해야 합니다. BroadcastChannel은 같은 Origin과 저장소 파티션의 여러 탭에 알림을 전달하기 편하고, postMessage는 참조가 있는 특정 창이나 iframe으로 보내며 다른 Origin과도 통신할 수 있습니다. BroadcastChannel은 메시지를 저장하지 않으므로 새 탭 초기화는 별도로 구현하고, postMessage는 목적지와 발신 Origin, 메시지 형식을 검증합니다.

## 관련 학습

- [State Management](StateManagement.md)
- [Provider 패턴](ProviderPattern.md)

개념 설명용 예제이며 별도 다중 탭 실습은 진행하지 않았다.
