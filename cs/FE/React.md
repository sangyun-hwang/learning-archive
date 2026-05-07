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

부수 효과는 렌더링 결과 계산 자체가 아니라 외부 세계와 상호작용하는 작업입니다.

- API 요청
- DOM 직접 조작
- 타이머 등록
- 이벤트 리스너 등록
- 외부 스토리지 접근

함수 컴포넌트에서는 `useEffect`로 부수 효과를 다룹니다. cleanup 함수는 컴포넌트가 언마운트되거나 의존성이 바뀌어 effect가 다시 실행되기 전에 실행됩니다.

`useLayoutEffect`는 브라우저가 화면을 그리기 전에 동기적으로 실행되므로 레이아웃 측정처럼 화면 반영 전에 처리해야 하는 작업에 사용합니다.

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
