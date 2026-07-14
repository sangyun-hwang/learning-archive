# React

## 왜 React인가?

React는 UI를 상태의 결과로 선언적으로 표현하는 라이브러리입니다. 상태가 바뀌면 React가 변경된 UI를 다시 계산하고 DOM 반영 과정을 관리합니다.

- 단방향 데이터 흐름을 통해 데이터 변경 지점을 추적하기 쉽습니다.
- JSX를 사용해 JavaScript 안에서 UI 구조와 로직을 함께 다룰 수 있습니다.
- 컴포넌트 단위로 UI를 분리해 재사용성과 유지보수성을 높일 수 있습니다.
- 생태계와 커뮤니티가 크고, Next.js 같은 프레임워크와 함께 확장하기 좋습니다.

## JSX

JSX는 JavaScript 확장 문법으로, UI를 HTML과 비슷한 형태로 작성하게 해줍니다. JSX는 브라우저가 직접 실행하는 문법이 아니라 빌드 과정에서 `React.createElement` 호출 또는 자동 JSX 런타임 코드로 변환됩니다.

- 컴포넌트 이름은 대문자로 시작합니다.
- JSX 안에서는 `{}`를 사용해 JavaScript 표현식을 넣습니다.
- 여러 조건부 렌더링이 중첩될 때는 JSX 안의 삼항 연산자를 과하게 겹치기보다 값을 미리 분리하는 편이 읽기 쉽습니다.

## Virtual DOM과 React Fiber

Virtual DOM은 실제 DOM을 직접 수정하기 전에 메모리에서 UI 구조를 계산하기 위한 표현입니다. React는 이전 UI와 새로운 UI를 비교해 필요한 변경만 실제 DOM에 반영합니다.

Fiber는 React의 렌더링 작업 단위입니다. React는 Fiber 트리를 사용해 렌더링 작업을 쪼개고 우선순위를 조절합니다.

- render phase: 변경이 필요한 UI를 계산합니다. 이 과정은 중단되거나 다시 시작될 수 있습니다.
- commit phase: 계산된 변경 사항을 실제 DOM에 반영합니다. 이 과정은 동기적으로 처리됩니다.
- current tree와 workInProgress tree를 두고 작업해, 변경 계산과 현재 화면을 분리합니다.

## 클래스 컴포넌트와 함수 컴포넌트

클래스 컴포넌트는 `React.Component`를 상속하고 `render`, `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` 같은 라이프사이클 메서드를 사용합니다.

함수 컴포넌트는 함수 호출의 결과로 JSX를 반환하고, Hook을 통해 상태와 부수 효과를 다룹니다. `this` 바인딩을 신경 쓰지 않아도 되고 로직을 Hook 단위로 분리하기 쉽습니다.

클래스 컴포넌트는 인스턴스의 `this.props`, `this.state`를 읽지만, 함수 컴포넌트는 렌더링 시점의 값을 클로저로 캡처합니다. 이 차이 때문에 비동기 콜백에서 읽는 값의 시점이 달라질 수 있습니다.

## 렌더링은 어떻게 일어나는가?

React 렌더링은 컴포넌트 함수가 실행되어 React Element를 만들고, Fiber 트리에서 이전 결과와 비교한 뒤 필요한 DOM 변경을 커밋하는 과정입니다.

렌더링이 발생하는 대표적인 경우는 다음과 같습니다.

- 최초 렌더링
- `useState` setter 호출
- `useReducer` dispatch 호출
- 부모 컴포넌트 렌더링
- props 변경
- key 변경
- 클래스 컴포넌트의 `setState`, `forceUpdate`

렌더링이 발생했다고 항상 실제 DOM 변경이 일어나는 것은 아닙니다. React가 비교한 결과 변경 사항이 없다면 commit 단계에서 반영할 내용이 없을 수 있습니다.

## 메모이제이션

메모이제이션은 이전 계산 결과를 기억해 같은 입력에 대해 반복 계산을 줄이는 기법입니다.

React에서는 주로 다음 API를 사용합니다.

- `React.memo`: props가 같으면 컴포넌트 렌더링 결과를 재사용합니다.
- `useMemo`: 비용이 큰 계산 결과를 기억합니다.
- `useCallback`: 함수를 재생성하지 않고 참조를 유지합니다.

메모이제이션은 비교 비용과 메모리 비용이 있으므로 모든 곳에 적용하기보다 렌더링 비용이 크거나 참조 안정성이 필요한 지점에 적용합니다.

## Effect

`useEffect`는 React 컴포넌트를 React 외부 시스템과 동기화하기 위한 Hook입니다.

단순히 "렌더링 후 코드를 실행하는 함수"로 이해하면 파생 값 계산이나 사용자 이벤트 처리까지 Effect에 넣기 쉽습니다. Effect의 목적은 렌더링 결과에 맞춰 브라우저 API, 네트워크 연결, 타이머, 이벤트 리스너, 외부 라이브러리 같은 시스템의 상태를 동기화하는 것입니다.

- API 요청
- DOM 직접 조작
- 타이머 등록
- 이벤트 리스너 등록
- 외부 스토리지 접근
- WebSocket과 외부 라이브러리 연결

```tsx
useEffect(() => {
  const connection = createConnection(roomId);
  connection.connect();

  return () => connection.disconnect();
}, [roomId]);
```

이 Effect는 `roomId`에 맞는 채팅 서버 연결을 시작하고, 방이 바뀌거나 컴포넌트가 사라지면 기존 연결을 종료합니다.

### Render, Event, Effect

컴포넌트 안의 로직은 실행 원인에 따라 구분할 수 있습니다.

- Render: 현재 props와 state로 UI를 계산합니다. 렌더링 코드는 순수해야 합니다.
- Event handler: 클릭이나 입력처럼 사용자의 특정 행동 때문에 실행됩니다.
- Effect: 특정 이벤트가 아니라 렌더링 결과 자체 때문에 외부 시스템과 동기화합니다.

결제나 저장 요청은 사용자의 행동으로 발생해야 하므로 Effect가 아닌 이벤트 핸들러에 작성합니다. 결제 요청을 빈 의존성 배열의 Effect에 넣으면 개발 환경의 재마운트, 페이지 재방문이나 컴포넌트의 재생성만으로 다시 실행될 위험이 있습니다.

```tsx
function handleBuy() {
  fetch('/api/buy', { method: 'POST' });
}
```

반면 채팅 서버 연결이나 `resize` 이벤트 등록은 컴포넌트가 화면에 존재하는 동안 외부 시스템과 동기화해야 하므로 Effect에 적합합니다.

### Effect의 생명주기

Effect는 컴포넌트의 mount, update, unmount에 맞춰 암기하기보다 동기화를 시작하고 중단하는 과정으로 이해할 수 있습니다.

```tsx
useEffect(() => {
  // setup: 동기화 시작

  return () => {
    // cleanup: 동기화 중단
  };
}, [dependency]);
```

실행 순서는 다음과 같습니다.

```text
최초 commit
-> setup 실행

의존성 변경
-> 이전 cleanup 실행
-> 새로운 setup 실행

컴포넌트 제거
-> 마지막 cleanup 실행
```

cleanup은 언마운트될 때만 실행되지 않습니다. 의존성이 바뀌면 이전 렌더링의 값으로 시작한 작업을 정리한 후 새로운 값으로 setup을 실행합니다.

예를 들어 채팅방이 바뀔 때 이전 연결을 끊지 않고 새 연결을 만들면 두 채팅방에 동시에 연결되거나 이벤트를 중복 수신할 수 있습니다.

setup과 cleanup은 다음처럼 서로 대응해야 합니다.

```text
connect     <-> disconnect
subscribe   <-> unsubscribe
addEvent    <-> removeEvent
setInterval <-> clearInterval
start       <-> stop
```

```tsx
useEffect(() => {
  window.addEventListener('resize', handleResize);

  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []);
```

### Strict Mode에서 두 번 실행되는 이유

개발 환경에서 Strict Mode가 활성화되어 있으면 React는 최초 마운트에 추가로 다음 과정을 실행합니다.

```text
setup -> cleanup -> setup
```

목적은 Effect를 단순히 두 번 실행하는 것이 아니라 cleanup이 setup의 작업을 제대로 되돌리는지 검사하는 것입니다. 프로덕션에는 이 추가 검사 과정이 없습니다.

두 번 실행되는 것을 막기 위해 `useRef`로 실행 여부만 가리면 연결이나 구독이 정리되지 않는 근본 문제는 남을 수 있습니다. setup을 여러 번 실행해도 cleanup을 거치면 사용자에게 동일한 결과가 보이도록 Effect를 작성해야 합니다.

### useEffect 의존성 배열

`useEffect`의 두 번째 인자인 의존성 배열은 effect가 언제 다시 실행될지 결정합니다.

의존성 배열은 크게 세 가지 경우로 나눠서 설명할 수 있습니다.

#### 1. 의존성 배열을 생략한 경우

```tsx
useEffect(() => {
  // effect
});
```

의존성 배열을 아예 전달하지 않으면 컴포넌트가 렌더링될 때마다 effect가 실행됩니다.

즉 최초 렌더링 이후에도 `state`나 `props` 변경으로 리렌더링이 발생할 때마다 다시 실행됩니다.

#### 2. 빈 배열을 전달한 경우

```tsx
useEffect(() => {
  // effect
}, []);
```

빈 배열을 전달하면 컴포넌트가 마운트된 이후 한 번만 effect가 실행됩니다.

주로 초기 데이터 요청, 이벤트 리스너 등록, 외부 라이브러리 초기화처럼 마운트 시점에 한 번 실행하면 되는 작업에 사용합니다.

#### 3. 값이 들어 있는 배열을 전달한 경우

```tsx
useEffect(() => {
  // effect
}, [value]);
```

배열에 값을 넣으면 최초 렌더링 이후 한 번 실행되고, 이후 `value`가 변경될 때마다 effect가 다시 실행됩니다.

즉 의존성 배열은 "이 값이 바뀌면 effect를 다시 실행한다"는 기준입니다.

effect 내부에서 사용하는 `props`나 `state`가 있다면 의존성 배열에 포함하는 것이 기본입니다. 빠뜨리면 이전 렌더링 시점의 값을 참조하는 stale closure 문제가 생길 수 있습니다.

cleanup 함수가 있다면 컴포넌트가 언마운트될 때 실행되고, 의존성이 바뀌어 effect가 다시 실행되기 전에도 먼저 실행됩니다.

React는 각 의존성을 이전 렌더링의 값과 `Object.is`로 비교합니다. 렌더링할 때마다 새로 생성되는 객체나 함수가 의존성에 있으면 내용이 같아 보여도 참조가 달라 Effect가 불필요하게 다시 실행될 수 있습니다.

면접에서는 다음처럼 정리해서 말할 수 있습니다.

> `useEffect`의 두 번째 인자를 생략하면 렌더링될 때마다 effect가 실행됩니다. 빈 배열을 넣으면 마운트 이후 한 번만 실행되고, 배열에 값을 넣으면 최초 실행 후 해당 의존성이 변경될 때마다 다시 실행됩니다. effect 안에서 사용하는 props나 state는 의존성 배열에 포함해야 stale closure 문제를 줄일 수 있습니다.

### 무한 렌더링

Effect가 실행될 때마다 state를 변경하고, 그 state 변경으로 Effect가 다시 실행되면 무한 반복이 발생할 수 있습니다.

```tsx
useEffect(() => {
  setCount(count + 1);
});
```

```text
render -> Effect -> state 변경 -> render -> Effect -> 반복
```

이때 의존성 배열을 무조건 `[]`로 바꾸기보다 해당 state 변경이 정말 외부 시스템과의 동기화인지 먼저 확인해야 합니다.

### Effect가 필요 없는 경우

현재 props나 state만으로 계산할 수 있는 파생 값에는 Effect와 별도 state가 필요하지 않습니다.

```tsx
// 불필요한 Effect와 추가 렌더링
const [fullName, setFullName] = useState('');

useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);
```

```tsx
// 렌더링 중 바로 계산
const fullName = `${firstName} ${lastName}`;
```

Effect로 파생 값을 만들면 이전 값으로 먼저 렌더링한 뒤 Effect에서 state를 변경해 다시 렌더링합니다. 렌더링 중 계산하면 항상 최신 값을 사용하면서 불필요한 렌더링을 피할 수 있습니다.

다음 작업도 Effect가 필요하지 않을 가능성이 큽니다.

- props나 state로부터 파생 값 계산
- 사용자의 클릭으로 실행되는 POST 요청
- 이벤트에 따른 state 변경
- 비용이 큰 값 계산: 필요한 경우 `useMemo` 사용
- props 변경 시 전체 상태 초기화: 컴포넌트의 `key` 사용 고려

### 비동기 요청과 race condition

사용자 A의 요청을 시작한 직후 B의 요청을 시작하면 B의 응답이 먼저 도착하고 A의 늦은 응답이 최신 결과를 덮어쓸 수 있습니다.

cleanup에서 이전 요청의 결과를 무시하거나 `AbortController`로 요청을 취소해 방지할 수 있습니다.

```tsx
useEffect(() => {
  const controller = new AbortController();

  async function loadUser() {
    const response = await fetch(`/api/users/${userId}`, {
      signal: controller.signal,
    });
    const user = await response.json();
    setUser(user);
  }

  loadUser();

  return () => controller.abort();
}, [userId]);
```

실제 애플리케이션에서는 캐싱, 중복 요청 제거와 서버 렌더링을 지원하는 TanStack Query나 프레임워크의 데이터 로딩 방식도 고려할 수 있습니다.

### useEffect와 useLayoutEffect

`useEffect`는 일반적인 외부 동기화에 사용합니다. `useLayoutEffect`는 브라우저가 화면을 그리기 전에 레이아웃을 측정하거나 DOM을 조정해야 할 때 사용합니다.

```text
DOM 반영 -> useLayoutEffect -> 브라우저 paint -> useEffect
```

`useLayoutEffect`는 paint를 막을 수 있으므로 화면이 그려지기 전에 반드시 처리해야 하는 경우에만 사용합니다.

### 사용 규칙

- 컴포넌트 또는 Custom Hook의 최상위에서 호출합니다.
- 조건문과 반복문 안에서 호출하지 않습니다.
- Effect는 서버 렌더링 중에는 실행되지 않고 클라이언트에서 실행됩니다.
- 의존성은 임의로 고르는 최적화 옵션이 아니라 Effect에서 사용하는 반응형 값에 따라 결정됩니다.
- 서로 다른 외부 시스템을 동기화하는 로직은 별도의 Effect로 분리합니다.

면접에서는 다음처럼 정리할 수 있습니다.

> `useEffect`는 렌더링 결과에 맞춰 컴포넌트를 외부 시스템과 동기화하는 Hook입니다. setup에서 연결이나 구독을 시작하고 cleanup에서 이를 되돌립니다. cleanup은 언마운트뿐 아니라 의존성 변경 후 새로운 setup이 실행되기 전에도 호출됩니다. props와 state만으로 계산 가능한 값이나 특정 사용자 행동은 Effect 대신 렌더링 계산이나 이벤트 핸들러로 처리합니다.

### 참고

- [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
- [Lifecycle of Reactive Effects](https://react.dev/learn/lifecycle-of-reactive-effects)

## 짧은 지식

### 멀리 떨어진 form과 button 연결하기

HTML의 `form` 속성을 사용하면 form 내부에 있지 않은 버튼도 특정 form과 연결할 수 있습니다.

```html
<form id="order-form">
  <input name="name" />
</form>

<button type="submit" form="order-form">Submit</button>
```

### React의 부수효과란?

React 컴포넌트는 같은 props와 state에 대해 같은 UI를 반환하는 순수한 계산에 가까울수록 예측하기 쉽습니다. 네트워크 요청, 타이머, 이벤트 리스너처럼 컴포넌트 외부에 영향을 주는 작업은 부수 효과로 분리해 관리합니다.
