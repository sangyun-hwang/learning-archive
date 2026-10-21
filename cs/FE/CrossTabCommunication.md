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

## 두 API를 말로 비교하기

BroadcastChannel은 **같은 사이트의 여러 탭이 공통 채널에 참여해서 소식을 주고받는 방식**이다. 각 탭이 같은 이름의 채널을 열면, 상대 탭을 직접 가리키는 참조 없이도 채널에 메시지를 보낼 수 있다. 예를 들어 쇼핑몰 탭 세 개를 열어 둔 상태에서 한 탭이 로그아웃 변경을 알리면 다른 탭들이 이를 받아 자신의 화면을 갱신할 수 있다. 실제 서버 세션 종료는 별도로 처리해야 한다.

window.postMessage는 **참조를 가진 특정 창이나 iframe을 대상으로 메시지를 보내는 방식**이다. 부모 페이지가 자신이 연 팝업이나 포함한 iframe에 데이터를 전달할 때 사용한다. 상대가 다른 사이트여도 메시지를 보낼 수 있지만, 모든 탭을 찾아 전송하는 기능은 아니다. 보내려는 창의 참조가 필요하고 상대 페이지에도 메시지를 받는 코드가 있어야 한다.

선택할 때는 '여러 탭에 공통으로 알릴 것인가, 특정 창에 보낼 것인가'를 먼저 생각한다. 같은 사이트를 독립적으로 연 탭들에 테마 변경을 알리려면 BroadcastChannel이 편하다. 부모 페이지와 외부 위젯 iframe처럼 상대가 정해진 통신에는 window.postMessage가 적합하다. 같은 사이트의 특정 창끼리도 postMessage를 사용할 수 있다.

두 방식 모두 메시지를 받은 쪽이 그 의미를 해석하고 자신의 상태를 갱신해야 한다. 메시지를 보냈다는 이유만으로 상대 화면이 자동으로 변경되거나 과거 메시지가 영구 저장되지는 않는다.

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

### 30초 답변

브라우저의 탭은 각각 별도의 실행 환경과 상태를 가지므로 메시지로 변경을 알려야 합니다. BroadcastChannel은 같은 사이트의 여러 탭이 같은 이름의 채널로 소식을 주고받는 방식입니다. 상대 창을 직접 알 필요가 없어 여러 탭의 로그아웃이나 테마 변경 알림에 편리합니다. window.postMessage는 참조가 있는 특정 팝업이나 iframe에 메시지를 보내는 방식이고, 다른 사이트와도 통신할 수 있습니다. 그래서 여러 탭의 공통 알림에는 BroadcastChannel을, 특정 창과의 통신에는 postMessage를 고려합니다.

### 추가 질문에 이어서 설명하기

BroadcastChannel의 '같은 사이트'를 정확히 말하면 같은 Origin과 저장소 파티션 조건을 만족해야 합니다. Origin은 프로토콜, 호스트와 포트의 조합이므로 포트가 다르면 같은 채널 이름을 사용해도 연결되지 않습니다.

postMessage는 다른 사이트와도 통신하므로 보낼 때 정확한 목적지 Origin을 지정하고, 받을 때 발신 Origin과 예상한 창인지 확인합니다. 허용한 출처에서 온 메시지라도 데이터 형식과 수행할 동작을 검증합니다.

또 BroadcastChannel은 실시간 알림 수단이므로 나중에 열린 탭이 과거 메시지를 자동으로 받지는 못합니다. 새 탭은 localStorage 같은 저장소에서 초기값을 읽거나, 기존 탭에 현재 상태를 요청하도록 구현합니다.

## 관련 학습

- [State Management](StateManagement.md)
- [Provider 패턴](ProviderPattern.md)

개념 설명용 예제이며 별도 다중 탭 실습은 진행하지 않았다.
