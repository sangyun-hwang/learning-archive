# 브라우저 키보드 입력 처리

브라우저에서 키보드 입력을 처리할 때는 **어떤 키를 눌렀는지 감지하는 일**과 **입력값이 실제로 어떻게 변경됐는지 읽는 일**을 구분해야 한다.

## 기본 이벤트 흐름

일반적인 텍스트 입력은 대략 다음 순서로 처리된다.

```text
키 누름
-> keydown
-> beforeinput
-> 실제 값 변경
-> input
-> keyup
```

브라우저, 입력 방식과 IME 조합 여부에 따라 세부 순서는 달라질 수 있다.

### keydown

키를 누르는 순간 발생한다. Enter, Escape, 방향키와 단축키처럼 **키 자체의 동작**을 처리할 때 사용한다.

```tsx
function handleKeyDown(
  event: React.KeyboardEvent<HTMLInputElement>,
) {
  if (event.key === 'Enter') {
    search();
  }
}
```

### keyup

눌렀던 키에서 손을 뗄 때 발생한다. 키를 놓은 시점이 중요한 동작에 사용한다.

### input과 React onChange

실제 입력값 변경은 Browser의 `input` 이벤트와 연결된 React의 `onChange`로 처리하는 것이 적합하다.

```tsx
<input
  value={keyword}
  onChange={(event) => setKeyword(event.target.value)}
/>
```

`keydown`은 값이 변경되기 전에 발생할 수 있고 입력값은 키보드 외에도 다음 방법으로 변경될 수 있다.

- 붙여넣기와 잘라내기
- 자동 완성
- 모바일 가상 키보드
- 음성 입력
- 한글 등의 IME 조합

따라서 역할을 다음처럼 나눈다.

```text
어떤 키를 눌렀는가
-> onKeyDown

입력값이 무엇으로 변경됐는가
-> onChange
```

`keypress`는 더 이상 권장되지 않으므로 `keydown`, `keyup`과 `input`을 사용한다.

## event.key와 event.code

### event.key

사용자가 입력하려는 논리적인 값을 나타낸다. Keyboard Layout과 Shift 등의 Modifier 영향을 받는다.

```text
a 입력
-> key: "a"

Shift + a 입력
-> key: "A"
```

### event.code

Keyboard에서 누른 물리적인 위치를 나타낸다.

```text
a 입력
-> code: "KeyA"

Shift + a 입력
-> code: "KeyA"
```

Shift 키 자체에서는 별도의 Event가 발생하며 `ShiftLeft` 또는 `ShiftRight` 같은 `code`를 가진다.

```text
문자 입력과 일반 단축키
-> key 중심

게임처럼 물리적인 키 위치가 중요
-> code 중심
```

## Key Repeat

사용자가 키를 계속 누르면 운영체제와 Browser가 `keydown`을 자동으로 반복할 수 있다.

```text
첫 keydown
-> event.repeat === false

키를 누르고 있는 동안 반복 keydown
-> event.repeat === true

키를 놓음
-> keyup
```

방향키로 목록을 계속 이동할 때는 반복이 유용하지만, 저장이나 Dialog 열기처럼 한 번만 실행해야 하는 동작에서는 반복 입력을 무시할 수 있다.

```tsx
if (event.repeat) {
  return;
}
```

## Modifier Key와 단축키

조합 키는 다음 속성으로 확인한다.

```ts
event.ctrlKey
event.shiftKey
event.altKey
event.metaKey
```

Windows와 macOS에서 저장 단축키를 함께 지원하는 예시는 다음과 같다.

```tsx
function handleKeyDown(event: React.KeyboardEvent) {
  const commandKey = event.ctrlKey || event.metaKey;

  if (
    commandKey &&
    event.key.toLowerCase() === 's' &&
    !event.repeat
  ) {
    event.preventDefault();
    save();
  }
}
```

## preventDefault와 stopPropagation

두 Method의 목적은 다르다.

```text
preventDefault()
-> Browser의 기본 동작 방지
-> Event는 부모로 전달될 수 있음
```

```text
stopPropagation()
-> 부모 요소로 Event가 전파되는 것 방지
-> Browser 기본 동작은 실행될 수 있음
```

예를 들어 `Ctrl + S`에서 `stopPropagation()`만 호출하면 부모 Handler는 막을 수 있어도 Browser의 저장 창은 열릴 수 있다. 기본 저장 동작을 대체하려면 `preventDefault()`가 필요하다.

두 Method를 무조건 호출하면 Browser의 기본 기능과 접근성을 해칠 수 있으므로 필요한 경우에만 사용한다.

## 한글 입력과 IME

한글, 일본어와 중국어 등은 여러 입력을 조합해 하나의 문자를 완성한다.

```text
ㅎ
-> 하
-> 한
```

관련 Event는 다음과 같다.

```text
compositionstart
compositionupdate
compositionend
```

조합을 확정하기 위한 Enter를 메시지 전송으로 잘못 처리할 수 있으므로 조합 상태를 확인한다.

```tsx
function handleKeyDown(
  event: React.KeyboardEvent<HTMLInputElement>,
) {
  if (event.key !== 'Enter') {
    return;
  }

  if (event.nativeEvent.isComposing) {
    return;
  }

  sendMessage();
}
```

Browser와 실행 환경에 따라 Composition 종료 시점의 차이가 있을 수 있으므로 실제 IME 입력으로 확인해야 한다.

## 전역 키보드 이벤트

Page 전체에서 Escape나 단축키를 처리하려면 `window`에 Listener를 등록할 수 있다.

```tsx
useEffect(() => {
  function handleKeyDown(event: KeyboardEvent) {
    if (event.key === 'Escape') {
      closeModal();
    }
  }

  window.addEventListener('keydown', handleKeyDown);

  return () => {
    window.removeEventListener('keydown', handleKeyDown);
  };
}, [closeModal]);
```

Cleanup이 없으면 Component가 다시 Mount될 때 Listener가 중복되거나 제거된 Component의 Logic이 계속 실행될 수 있다.

전역 단축키는 사용자가 `input`, `textarea`나 `contenteditable`에서 글을 입력하는 중인지 확인해 방해하지 않도록 설계해야 한다.

## 키보드 접근성

클릭 가능한 UI는 키보드로도 조작할 수 있어야 한다. 클릭 가능한 `div`보다 Semantic Element를 우선한다.

```tsx
<button type="button" onClick={save}>
  저장
</button>
```

`button`은 기본적으로 다음 기능을 제공한다.

- Tab으로 Focus 가능
- Enter와 Space로 실행 가능
- 보조 기술에 Button 역할 전달

`div`에 `role`, `tabIndex`, `onKeyDown`을 추가해 기본 Button 동작을 다시 구현하기보다 목적에 맞는 HTML Element를 사용하는 것이 안전하다.

## 면접에서 설명하기

> 브라우저의 키 입력은 `keydown`과 `keyup`으로 감지하고 실제 텍스트 값 변경은 `input` 또는 React의 `onChange`로 처리합니다. `event.key`는 사용자가 입력하려는 논리적인 값이고 `event.code`는 물리적인 키 위치를 나타냅니다. 키를 계속 누를 때는 `event.repeat`으로 반복 여부를 확인하고, 한글 같은 IME 입력에서는 조합 중 Enter가 실행되지 않도록 `isComposing`을 확인해야 합니다. 전역 Listener는 Effect cleanup에서 제거하며, 접근성을 위해 직접 키보드 동작을 재구현하기보다 `button` 같은 Semantic Element를 우선합니다.

## References

- [MDN: KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)
- [MDN: Element keydown event](https://developer.mozilla.org/en-US/docs/Web/API/Element/keydown_event)
- [MDN: InputEvent](https://developer.mozilla.org/en-US/docs/Web/API/InputEvent)
- [MDN: CompositionEvent](https://developer.mozilla.org/en-US/docs/Web/API/CompositionEvent)

