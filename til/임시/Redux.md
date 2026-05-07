## Redux

### Redux 작동 방식

- 리덕스는 앱에 있는 하나의 중앙 데이터 저장소로, 전체 앱의 모든 상태를 저장.
- 그 저장소에 데이터를 저장하여 컴포넌트 안에서 데이터 사용가능
- 저장소를 구독하면 저장소가 업데이트 될 때 마다 컴포넌트에 알려.
- 데이터를 변경할 때 컴포넌트는 저장된 데이터를 직접 조작 X.
- 리듀서 함수를 설정해야 하고, 이 함수가 저장소 데이터를 변경하는 것을 담당.



### 액션 (Action)

상태에 변화가 필요할 때 발생시킴 (객체하나로 표현)

```javascript
{
  type: "TOGGLE_VALUE"
}
```

type을 필수로 그외의 값들은 개발자 마음대로 생성

```javascript
{
  type: "ADD_TODO",
  data: {
    id: 0,
    text: "리덕스 배우기"
  }
}
{
  type: "CHANGE_INPUT",
  text: "안녕하세요"
}
```

### 액션 생성함수 (Action Creator)

컴포넌트에서 더욱 쉽게 액션을 발생시키기 위함

```javascript
export function addTodo(data) {
  return {
    type: "ADD_TODO",
    data
  };
}

// 화살표 함수로도 만들 수 있습니다.
export const changeInput = text => ({
  type: "CHANGE_INPUT",
  text
});
```

필수 아님. 액션을 발생 시킬 때마다 직접 액션 객체를 작성할수도 있음

### 리듀서 (Reducer)

변화를 일으키는 함수

```javascript
function reducer(state, action) {
  // 상태 업데이트 로직
  return alteredState;
}
```

현재의 상태와 액션을 참조하여 `새로운 상태`를 반환

```javascript
function counter(state, action) {
  switch (action.type) {
    case 'INCREASE':
      return state + 1;
    case 'DECREASE':
      return state - 1;
    default:
      return state;
  }
}
```

### 스토어 (Store)

한 애플리케이션당 하나의 스토어
현재의 앱 상태와, 리듀서, 내장함수 포함

### 디스패치 (dispatch)

스토어의 내장함수
액션을 발생 시키는 것

#### 구독 (subscribe)

스토어의 내장함수
subscribe 함수에 특정 함수를 전달해주면, 액션이 디스패치 되었을 때 마다 전달해준 함수가 호출
(리액트에서는 connect 함수 또는 useSelector Hook 을 사용)





![redux-data-flow](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/redux-data-flow.gif)

- 초기 상태
  - 먼저 root reducer 함수를 사용하여 만들어진 리덕스 스토어가 있다.
  - 스토어는 root reducer를 한번 호출하고 리턴 값을 초기 상태로 저장한다.
  - UI가 처음 렌더링될 때, UI 컴포넌트는 리덕스 스토어의 상태에 접근하여 그것을 렌더링에 활용한다. 또한 그것들은 후에 상태의 변화가 업데이트 되는 것을 구독한다.
- 업데이트(순서)
  1. 유저가 버튼을 클릭한다.
  2. 앱은 유저의 행동에 맞는 디스패치를 실행해 액션을 일으킨다.
  3. 스토어는 이전 상태와 현재 액션으로 리듀서 함수를 실행하고, 그 리턴 값을 새로운 상태로 저장한다.
  4. 스토어는 스토어를 구독하고 있던 UI들에게 업데이트 되었다고 알려준다.
  5. 스토어의 데이터가 필요한 각각의 UI들은 필요한 상태가 업데이트 되었는지 확인한다.
  6. 데이터가 변경된 각 구성요소는 새 데이터로 강제로 다시 렌더링하므로 화면에 표시되는 내용을 업데이트 할 수 있다.



## Redux의 세가지 원칙

1. **Single source of truth**

   하나의 애플리케이션 안에는 하나의 스토어만 사용하자는 원칙이다. 이렇게 하면 애플리케이션의 디버깅이 쉬워지고 서버와의 직렬화가 될 수 있고 쉽게 클라이언트에서 데이터를 받아들여올 수 있게 된다.

2. **State is read-only**

   상태를 변화시키는 방법은 오직 액션을 일으키는 것이다. 이것은 상태를 변화시키는 의도를 정확하게 표현할 수 있고, 상태 변경에 대한 추적이 용이해지게 된다.

3. **Changes are made with pure functions**

   변화를 일으키는 리듀서 함수는 순수한 함수여야 한다. 순수 함수는 다음과 같은 조건을 만족한다.

   - 리듀서 함수는 이전 상태와 액션 객체를 파라미터로 받는다.
   - 파라미터 외의 값에는 의존하면 안된다.
   - 이전 상태는 절대로 건드리지 않고, 변화를 준 새로운 상태 객체를 만들어서 반환한다.
   - 똑같은 파라미터로 호출된 리듀서 함수는 언제나 똑같은 결과 값을 반환해야 한다.





### 카운터 구현

```js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import Counter from '../components/Counter';
import { increase, decrease, setDiff } from '../modules/counter';

function CounterContainer() {
  // useSelector는 리덕스 스토어의 상태를 조회하는 Hook입니다.
  // state의 값은 store.getState() 함수를 호출했을 때 나타나는 결과물과 동일합니다.
  const { number, diff } = useSelector(state => ({
    number: state.counter.number,
    diff: state.counter.diff
  }));

  // useDispatch 는 리덕스 스토어의 dispatch 를 함수에서 사용 할 수 있게 해주는 Hook 입니다.
  const dispatch = useDispatch();
  // 각 액션들을 디스패치하는 함수들을 만드세요
  const onIncrease = () => dispatch(increase());
  const onDecrease = () => dispatch(decrease());
  const onSetDiff = diff => dispatch(setDiff(diff));

  return (
    <Counter
      // 상태와
      number={number}
      diff={diff}
      // 액션을 디스패치 하는 함수들을 props로 넣어줍니다.
      onIncrease={onIncrease}
      onDecrease={onDecrease}
      onSetDiff={onSetDiff}
    />
  );
}

export default CounterContainer;
```

```js
import React from 'react';

function Counter({ number, diff, onIncrease, onDecrease, onSetDiff }) {
  const onChange = e => {
    // e.target.value 의 타입은 문자열이기 때문에 숫자로 변환해주어야 합니다.
    onSetDiff(parseInt(e.target.value, 10));
  };
  return (
    <div>
      <h1>{number}</h1>
      <div>
        <input type="number" value={diff} min="1" onChange={onChange} />
        <button onClick={onIncrease}>+</button>
        <button onClick={onDecrease}>-</button>
      </div>
    </div>
  );
}

export default Counter;
```
