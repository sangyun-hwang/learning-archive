# 브라우저 Gamepad와 Joystick 입력 처리

브라우저에서는 Gamepad API를 이용해 Gamepad와 일부 Joystick의 버튼, Trigger와 Analog Stick 상태를 읽을 수 있다.

키보드는 `keydown`, `keyup` Event를 중심으로 처리하지만 Gamepad는 `requestAnimationFrame()`마다 현재 입력 상태를 확인하는 **Polling 방식**을 주로 사용한다.

```text
Keyboard
-> keydown, keyup Event

Gamepad
-> 매 Frame 현재 buttons와 axes 상태 Polling
```

## 연결과 해제

Gamepad 연결과 해제는 Window Event로 감지한다.

```ts
window.addEventListener('gamepadconnected', (event) => {
  console.log('연결:', event.gamepad);
});

window.addEventListener('gamepaddisconnected', (event) => {
  console.log('해제:', event.gamepad.index);
});
```

연결된 장치의 현재 상태는 `navigator.getGamepads()`로 가져온다.

```ts
for (const gamepad of navigator.getGamepads()) {
  if (!gamepad) {
    continue;
  }

  console.log(gamepad.id);
}
```

Gamepad가 해제된 배열 위치에는 `null`이 남을 수 있다. Browser는 Fingerprinting을 줄이기 위해 사용자가 Page를 보고 Gamepad 버튼이나 축을 실제로 조작하기 전까지 장치를 노출하지 않을 수도 있다.

## Gamepad 객체

주요 속성은 다음과 같다.

```ts
gamepad.id
gamepad.index
gamepad.connected
gamepad.mapping
gamepad.buttons
gamepad.axes
gamepad.timestamp
```

- `id`: Controller 식별 문자열
- `index`: 현재 연결된 Gamepad 배열의 위치
- `connected`: 연결 상태
- `mapping`: Browser가 적용한 알려진 입력 배치
- `buttons`: 버튼과 Trigger 상태
- `axes`: Analog Stick과 Joystick 축 상태
- `timestamp`: 마지막 입력 정보 갱신 시점

## Button 입력

각 버튼은 `GamepadButton` 객체로 표현된다.

```ts
const button = gamepad.buttons[0];

button.pressed;
button.value;
```

```text
pressed
-> 현재 버튼이 눌렸는가

value
-> 버튼이 어느 정도 눌렸는가
-> 0.0 ~ 1.0
```

일반 Digital 버튼의 `value`는 보통 `0` 또는 `1`에 가깝다. Trigger처럼 압력을 표현하는 Analog 버튼은 `0.5` 같은 중간값을 가질 수 있다.

## Axis 입력

Analog Stick과 Joystick은 `axes` 배열로 표현된다.

```ts
const leftX = gamepad.axes[0] ?? 0;
const leftY = gamepad.axes[1] ?? 0;
```

일반적인 축 값의 범위는 `-1`부터 `1`이다.

```text
X축
왼쪽  -1 <---- 0 ----> 1  오른쪽

Y축
위쪽  -1 <---- 0 ----> 1  아래쪽
```

## Frame Polling

Gamepad API는 연결과 해제 Event를 제공하지만 모든 버튼과 축 변화에 대응하는 표준 개별 Event를 제공하지 않는다. Analog Stick 값은 사용자가 기울이는 동안 연속적으로 달라지고 게임은 현재 Frame의 입력을 바로 이동과 물리에 반영해야 한다.

```text
현재 Gamepad 상태 읽기
-> 현재 Frame의 이동량 계산
-> 화면 Rendering
-> 다음 Frame의 상태 읽기
```

```ts
function gameLoop() {
  const gamepad = navigator.getGamepads()[0];

  if (gamepad) {
    const x = gamepad.axes[0] ?? 0;
    const y = gamepad.axes[1] ?? 0;

    updatePlayer(x, y);
  }

  requestAnimationFrame(gameLoop);
}

requestAnimationFrame(gameLoop);
```

연결 Event에서 받은 객체를 오래 보관하기보다 알려진 `index`를 기준으로 매 Frame 최신 Gamepad 객체를 다시 읽는 편이 안전하다.

## 누르고 있는 상태와 누른 순간

Polling에서는 버튼을 누르고 있는 동안 매 Frame `pressed`가 `true`다.

```text
60FPS에서 버튼을 1초 동안 누름
-> 약 60 Frame에서 true 확인
```

이동이나 가속처럼 지속되는 동작은 현재 상태를 그대로 사용할 수 있다.

```ts
if (gamepad.buttons[0]?.pressed) {
  accelerate();
}
```

점프나 메뉴 선택처럼 한 번만 실행할 동작은 이전 Frame과 현재 Frame을 비교해 **Edge**를 감지한다.

```text
이전 false + 현재 true
-> 방금 눌림

이전 true + 현재 true
-> 계속 누르고 있음

이전 true + 현재 false
-> 방금 뗌
```

```ts
let previousPressed = false;

function readJump(gamepad: Gamepad) {
  const currentPressed =
    gamepad.buttons[0]?.pressed ?? false;

  const justPressed =
    currentPressed && !previousPressed;

  if (justPressed) {
    jump();
  }

  previousPressed = currentPressed;
}
```

## Dead Zone

Analog Stick은 손을 떼도 Hardware 오차와 마모로 정확히 `0`을 반환하지 않을 수 있다.

```text
axes[0] = 0.03
axes[1] = -0.07
```

작은 값을 그대로 반영하면 Character나 Cursor가 가만히 있어도 움직일 수 있으므로 일정 범위를 `0`으로 처리한다.

```ts
function applyDeadZone(
  value: number,
  deadZone = 0.15,
) {
  if (Math.abs(value) < deadZone) {
    return 0;
  }

  return value;
}
```

정교한 조작이 필요하면 Dead Zone을 제거한 나머지 범위를 다시 정규화하거나 원형 Dead Zone을 적용할 수 있다. 기본적인 면접 설명에서는 작은 Noise를 제거하는 목적을 이해하는 것으로 충분하다.

## Standard Mapping

`gamepad.mapping === 'standard'`라면 Browser가 장치 입력을 Standard Gamepad의 물리적 배치로 정규화했다는 의미다.

대표적인 표준 배치는 다음과 같다.

```text
buttons[0] ~ buttons[3]
-> 오른쪽 Face Button 네 개

buttons[12] ~ buttons[15]
-> D-pad

axes[0], axes[1]
-> Left Stick

axes[2], axes[3]
-> Right Stick
```

표준 Mapping은 버튼에 인쇄된 글자가 아니라 **물리적 위치**를 기준으로 한다.

```text
buttons[0]
-> 오른쪽 버튼 묶음의 아래쪽 버튼

Xbox
-> A

PlayStation
-> Cross

Nintendo
-> B
```

따라서 `buttons[0]`을 항상 `A`라고 표시하거나 항상 확인 동작으로 해석하면 안 된다. Platform 관습, Controller Profile과 사용자 설정을 통해 `confirm`, `cancel` 같은 의미로 변환한다.

`mapping`이 빈 문자열이면 Browser가 알려진 표준 배치로 정규화하지 못한 상태일 수 있다. 이 경우 버튼과 축 Index는 장치, Driver, OS와 Browser 조합에 따라 달라질 수 있다.

## Premium Controller와 추가 Button

Xbox Elite의 Paddle이나 DualSense Edge의 Back Button 같은 추가 입력은 Browser에 항상 독립된 버튼으로 노출된다고 보장할 수 없다.

### 기존 버튼으로 내부 Remapping

Paddle을 A 버튼으로 설정했다면 Controller나 Driver가 두 입력을 동일한 버튼으로 보고할 수 있다.

```text
실제 A Button
Paddle -> A로 Remapping

둘 다 buttons[0]
```

이 경우 Browser는 실제 A를 눌렀는지 Paddle을 눌렀는지 구분할 수 없다.

### 추가 Index로 노출

Driver와 Browser가 추가 입력을 전달한다면 표준 범위 이후의 `buttons` Index로 나타날 수 있다.

```ts
console.log(gamepad.buttons.length);
```

하지만 특정 추가 Index가 항상 왼쪽 Paddle이나 Touchpad Click이라는 표준은 없다. 장치와 실행 환경마다 의미가 달라질 수 있다.

### Browser에 노출되지 않음

추가 버튼이 Controller 내부 Profile의 Remapping 용도로만 사용되거나 Platform에서 입력을 감추면 Gamepad API에는 별도 입력이 나타나지 않는다.

따라서 표준 버튼만 필수 조작으로 사용하고, 추가 버튼은 지원되는 환경에서 사용자가 직접 Action에 할당할 수 있는 선택 기능으로 다루는 것이 안전하다.

## Controller 식별과 Profile

`gamepad.id`에는 Controller나 Vendor 정보가 포함될 수 있다.

```ts
const controllerInfo = {
  id: gamepad.id,
  mapping: gamepad.mapping,
  buttons: gamepad.buttons.length,
  axes: gamepad.axes.length,
};
```

하지만 `id` 문자열도 OS, Browser, Driver와 USB·Bluetooth 연결 방식에 따라 달라질 수 있다. 알려진 장치에는 Controller Profile을 적용하고 인식하지 못하면 일반 Label과 사용자 Mapping을 제공한다.

```text
Xbox Profile
-> A, B, X, Y Glyph

PlayStation Profile
-> Cross, Circle, Square, Triangle Glyph

Nintendo Profile
-> Nintendo 배치와 Label

Unknown
-> 일반 Button Label 또는 사용자 설정
```

## 사용자 Mapping

알 수 없는 Controller와 추가 버튼을 지원하려면 사용자가 Action별 버튼을 직접 등록하게 할 수 있다.

```text
“점프에 사용할 버튼을 눌러주세요”
-> 이전 Frame과 현재 Frame 비교
-> 새로 true가 된 Button Index 탐색
-> jump Action에 Index 저장
```

```ts
function findJustPressedButton(
  previous: readonly boolean[],
  gamepad: Gamepad,
) {
  return gamepad.buttons.findIndex(
    (button, index) =>
      button.pressed && !previous[index],
  );
}
```

설정은 `localStorage` 같은 저장소에 보관할 수 있다.

```json
{
  "confirm": 0,
  "cancel": 1,
  "jump": 17
}
```

추가 버튼이 Browser에 별도로 노출되는 장치라면 이 방식으로 특정 Action에 할당할 수 있다.

## 입력 장치와 Game Logic 분리

Game Logic이 Keyboard Key와 Gamepad Index를 직접 알지 않도록 공통 Action으로 변환한다.

```text
Keyboard ArrowLeft
Gamepad Left Stick
D-pad Left

-> 공통 Action: moveLeft
```

```ts
type InputState = {
  moveX: number;
  moveY: number;
  jumpPressed: boolean;
  confirmPressed: boolean;
};
```

```text
Keyboard Adapter
Gamepad Adapter
Touch Adapter
-> 공통 InputState
-> Game Logic
```

이 구조는 다음 장점이 있다.

- 입력 장치 추가와 교체가 쉬움
- 사용자 Key Mapping 지원
- Game Logic 테스트 단순화
- Controller별 Button Label 분리
- 여러 입력 장치의 동시 지원

## React에서 처리

Gamepad를 60FPS로 Polling하며 매번 React State를 변경하면 불필요한 Rendering이 많아질 수 있다.

게임 위치와 물리 상태는 다음 위치에서 관리하는 경우가 많다.

- Canvas나 Three.js Engine 내부 상태
- `useRef`
- 게임 전용 Store
- React UI에 필요한 값만 낮은 주기로 State에 반영

Effect에서는 Animation Frame을 정리한다.

```tsx
useEffect(() => {
  let frameId: number;

  function update() {
    const gamepad = navigator.getGamepads()[0];

    if (gamepad) {
      readGamepad(gamepad);
    }

    frameId = requestAnimationFrame(update);
  }

  frameId = requestAnimationFrame(update);

  return () => {
    cancelAnimationFrame(frameId);
  };
}, []);
```

연결과 해제 Listener를 Effect에서 등록했다면 해당 Listener도 Cleanup에서 제거한다.

## 진동과 특수 Controller

일부 Browser, OS와 Controller 조합에서는 `vibrationActuator`를 이용해 Haptic Feedback을 실행할 수 있다.

```ts
await gamepad.vibrationActuator?.playEffect(
  'dual-rumble',
  {
    duration: 300,
    strongMagnitude: 1,
    weakMagnitude: 0.5,
  },
);
```

Controller가 진동을 지원해도 Platform이나 Browser가 지원하지 않을 수 있으므로 선택 기능으로 다룬다.

일반 Game Controller로 인식되는 Joystick은 Gamepad API의 `buttons`와 `axes`로 읽을 수 있다. Flight Stick, Wheel과 특수 HID 장치는 표준 Mapping이 없거나 일부 입력이 노출되지 않을 수 있다. 더 낮은 수준의 장치 입력이 필요하면 WebHID를 검토할 수 있지만 명시적인 사용자 권한, 장치별 Protocol 처리와 Browser 호환성 관리가 추가된다.

## 면접에서 설명하기

> Browser에서는 Gamepad API로 Controller 입력을 처리합니다. 연결과 해제는 `gamepadconnected`, `gamepaddisconnected` Event로 감지하지만 버튼과 Analog Stick 상태는 `requestAnimationFrame`마다 `navigator.getGamepads()`로 Polling합니다. 버튼은 `pressed`와 `0~1`의 `value`, Stick은 `-1~1`의 `axes`로 표현합니다. Analog Stick에는 Dead Zone을 적용하고 한 번 누른 동작은 이전 Frame과 비교해 Edge를 감지합니다. 장치별 Mapping과 추가 버튼 노출 방식이 다르므로 표준 Mapping을 확인하고 Controller Profile과 사용자 재할당 기능으로 보완합니다.

## References

- [W3C: Gamepad](https://www.w3.org/TR/gamepad/)
- [MDN: Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API)
- [MDN: Using the Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API/Using_the_Gamepad_API)
- [MDN: Gamepad](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad)
- [MDN: Gamepad mapping](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad/mapping)

