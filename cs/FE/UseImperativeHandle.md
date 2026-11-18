# useImperativeHandle: 자식이 ref로 공개할 기능 정하기

`useImperativeHandle`은 부모에게 ref로 공개할 자식의 기능을 정의하는 React Hook입니다. 입력창 포커스, 스크롤, 미디어 재생처럼 특정 동작을 직접 요청할 때 사용할 수 있습니다.

## props와 ref의 역할

props는 “검색어가 무엇인가”, “모달이 열려 있는가”처럼 UI가 어떤 상태여야 하는지 전달합니다. ref는 “지금 입력창에 포커스를 줘”처럼 명령형 동작을 요청할 때 적합합니다.

props로 `shouldFocus`를 전달하고 자식의 Effect에서 포커스를 주는 것도 가능합니다. 따라서 props로 불가능해서 ref를 쓰는 것은 아닙니다. 단순한 동작 요청을 위해 상태와 Effect를 추가하는 것보다 ref가 자연스러운 경우에 사용합니다.

## 예제: 부모 버튼으로 자식 입력창에 포커스 주기

다음 코드는 React 19에서 ref를 prop으로 받는 방식입니다.

```jsx
import { useImperativeHandle, useRef } from 'react';

function SearchInput({ ref }) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current?.focus();
    },
  }), []);

  return <input ref={inputRef} />;
}

export default function App() {
  const searchRef = useRef(null);

  return (
    <>
      <SearchInput ref={searchRef} />
      <button onClick={() => searchRef.current?.focus()}>
        검색창으로 이동
      </button>
    </>
  );
}
```

자식의 `inputRef.current`는 실제 input DOM을 가리킵니다. 반면 부모의 `searchRef.current`에는 자식이 공개한 `{ focus() }` 객체가 들어갑니다. 부모가 버튼을 누르면 공개된 메서드가 내부 input의 `focus()`를 호출합니다.

부모에게 DOM 전체를 그대로 전달하는 것도 가능합니다. `useImperativeHandle`은 그중 필요한 기능만 골라 공개하고 싶을 때 사용합니다. 위 예제에서 부모는 해당 ref로 `input.value`를 직접 수정할 수 없습니다. 내부 구현과의 결합을 줄이는 API 설계이지, 보안 경계는 아닙니다.

React 18 이하에서는 ref를 받기 위해 `forwardRef`를 사용하는 예제를 볼 수 있습니다. React 19에서는 ref를 prop으로 받을 수 있습니다.

## 의존성과 리렌더링

`useImperativeHandle(ref, createHandle, dependencies)`의 의존성 배열에는 공개할 메서드에서 사용하는 props나 state 등 반응형 값을 포함해야 합니다. 이를 빠뜨리면 오래된 값을 참조할 수 있습니다.

위 예제는 안정적인 내부 ref만 사용하므로 빈 배열을 전달합니다. 의존성을 생략하면 리렌더링마다 handle을 새로 만들고, 의존성을 지정하면 React가 `Object.is`로 비교해 변경 시 다시 만듭니다.

ref의 변경이나 `focus()` 호출 자체는 React state 변경이 아니므로 리렌더링을 일으키지 않습니다. 다만 포커스로 발생한 이벤트 핸들러에서 `setState`를 호출한다면 그 상태 변경으로 리렌더링될 수 있습니다.

## 사용 기준

- 입력창 포커스, 스크롤, 미디어 제어처럼 직접 실행할 동작에 제한적으로 사용합니다.
- 검색어, 선택한 상품, 모달 열림 여부처럼 상태로 표현할 수 있는 것은 props를 우선합니다.
- 부모가 자식의 내부 DOM과 상태를 모두 조작하는 통로로 남용하지 않습니다.

## 면접에서 설명하기

> useImperativeHandle은 부모에게 ref로 공개할 자식의 기능을 정의하는 Hook입니다. 입력창 포커스나 스크롤처럼 명령형 동작이 필요할 때 사용합니다. DOM 전체 대신 필요한 메서드만 노출해 내부 구현과의 결합을 줄일 수 있고, 일반적인 상태 전달에는 props를 우선합니다.

## 참고

- [React: useImperativeHandle](https://react.dev/reference/react/useImperativeHandle)
- [React: forwardRef](https://react.dev/reference/react/forwardRef)

개념 학습용 예제이며 별도의 애플리케이션 실행 검증은 하지 않았습니다.
