# Provider 패턴: React Context로 값과 기능 제공하기

## 개념

Provider 패턴은 상위 컴포넌트가 값이나 기능을 제공하고 필요한 하위 컴포넌트가 이를 읽어 사용하는 구조다. 여기서는 React Context를 이용한 방식을 다룬다.

테마, 로그인 사용자 정보, 언어 설정처럼 여러 곳에서 필요한 정보를 전달할 때 활용할 수 있다. 중간 컴포넌트가 사용하지 않는 props를 계속 전달하는 prop drilling을 줄인다. [React: Context로 데이터 전달](https://react.dev/learn/passing-data-deeply-with-context)

## 역할 구분

| 구성 요소 | 역할 |
| --- | --- |
| Context | 제공자와 소비자를 연결하는 통로 |
| Provider | 특정 하위 트리에 실제 값이나 기능 제공 |
| useContext | 가장 가까운 상위 Provider의 값을 읽고 구독 |
| useState | 상태를 저장하고 변경 시 다시 렌더링하도록 요청 |

Context 자체가 상태를 저장하지는 않는다. Provider가 반드시 useState를 사용하는 것도 아니다. 고정된 설정, 외부 Store나 서비스 객체를 제공할 수도 있다.

## 테마 예제

```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const context = useContext(ThemeContext);
  if (context === null) {
    throw new Error('useTheme은 ThemeProvider 안에서 사용해야 합니다.');
  }
  return context;
}

function ThemeButton() {
  const { theme, setTheme } = useTheme();

  return (
    <button
      onClick={() =>
        setTheme((previous) => previous === 'light' ? 'dark' : 'light')
      }
    >
      현재 테마: {theme}
    </button>
  );
}

export default function App() {
  return (
    <ThemeProvider>
      <ThemeButton />
    </ThemeProvider>
  );
}
```

ThemeProvider는 우리가 만든 컴포넌트이고 실제 값을 제공하는 부분은 ThemeContext.Provider다. 버튼은 제공된 변경 함수를 호출하고, Provider가 관리하는 상태가 바뀌면 새 값을 받아 다시 렌더링된다.

React 19에서는 `<ThemeContext value={...}>`로도 제공할 수 있다. 위 예제는 기존 코드에서도 볼 수 있는 `.Provider` 표기를 사용한다.

## 적용 범위와 기본값

같은 Context의 Provider가 여러 겹이라면 소비자는 **자신보다 위에 있는 가장 가까운 Provider**의 값을 읽는다.

```text
ThemeContext: dark
  -> 일반 영역: dark
  -> ThemeContext: light
       -> 안쪽 버튼: light
```

Provider가 없으면 createContext에 지정한 기본값을 읽는다. 이 기본값은 자동으로 갱신되는 상태가 아니다. 위 예제는 null을 기본값으로 두고 커스텀 Hook에서 Provider 누락을 확인한다. [React: useContext](https://react.dev/reference/react/useContext)

## 사용할 범위 선택하기

한 입력창에서만 필요한 값은 로컬 state로 관리하고, 가까운 부모와 자식 사이에서는 props로 전달할 수 있다. 여러 단계에서 공통으로 필요한 값이라면 Context를 고려한다. Provider도 필요한 하위 트리를 감싸는 위치에 둔다.

QueryClientProvider처럼 상태 값 대신 서비스 객체를 제공하는 사례도 있다. Provider는 하위 컴포넌트가 같은 QueryClient를 사용할 수 있게 연결하며, 캐시·조회·재요청 기능은 TanStack Query가 담당한다. Context 자체에 그런 기능이 생기는 것은 아니다.

## 렌더링 주의점

Provider의 value가 이전과 달라지면 해당 Context를 읽는 컴포넌트가 다시 렌더링된다. React는 Object.is로 값의 변경을 비교한다. 모든 하위 컴포넌트가 Context 변경 때문에 무조건 다시 렌더링된다는 뜻은 아니다.

위 예제의 `value={{ theme, setTheme }}`는 Provider가 렌더링될 때마다 새로운 객체를 만든다. 실제로 불필요한 업데이트가 문제가 된다면 useMemo로 객체 참조를 유지하거나, 서로 다른 목적의 Context를 분리하는 방법을 고려한다. useMemo를 사용해도 theme 자체가 바뀌면 구독 컴포넌트는 새 값을 받는다.

Provider 패턴은 데이터 전달 방식이며 모든 상태 관리 문제를 해결하지 않는다. 서로 무관한 상태를 하나의 Context에 모으거나 작은 로컬 상태까지 전역으로 올릴 필요는 없다.

## 면접용 설명

Provider 패턴은 상위에서 값이나 기능을 제공하고 하위 컴포넌트가 Context를 통해 읽는 구조입니다. 중간 컴포넌트가 props를 전달만 하는 상황을 줄일 수 있고, 소비자는 가장 가까운 상위 Provider의 값을 사용합니다. 상태는 useState나 별도 Store가 관리하며 Context는 전달을 담당합니다. 값 변경 시 구독 컴포넌트가 다시 렌더링되므로 적용 범위와 제공할 값을 적절히 나눕니다.

## 관련 학습

- [Lifting State Up](LiftingStateUp.md)
- [Client State와 Server State](StateManagement.md)
- [TanStack Query Hydration](TanStackQueryHydration.md)

예제는 개념 설명용이며 별도 애플리케이션에서 실행한 실습 기록은 아니다.
