# JavaScript this 바인딩

일반 함수의 `this`는 주로 **함수가 어디에 작성됐는지가 아니라 어떻게 호출됐는지**에 따라 결정된다. 바인딩은 이때 `this`가 어떤 값을 참조할지 연결하는 것이다. 변수 탐색이 정의 위치를 따르는 lexical scope와 구분한다.

## 호출 방식별 비교

아래 일반 함수 예제는 strict mode를 기준으로 한다. ES Module과 class 내부는 자동으로 strict mode가 적용된다.

| 호출 방식 | this |
| --- | --- |
| 일반 함수 `fn()` | strict mode에서 undefined |
| 메서드 `user.fn()` | 호출에 사용한 user 객체 |
| `fn.call(user, ...)` | 명시한 user 객체 |
| `fn.apply(user, args)` | 명시한 user 객체 |
| `fn.bind(user)`로 만든 함수의 일반 호출 | 미리 지정한 user 객체 |
| 일반 생성자 함수의 `new User()` | 새로 생성되는 인스턴스 |
| 화살표 함수 | 정의된 위치의 바깥 this 사용 |

non-strict 일반 함수의 단독 호출에서는 this가 전역 객체가 된다. 따라서 this가 항상 window라고 외우면 안 된다.

## 같은 함수도 호출에 따라 달라진다

```js
'use strict';

function getName() {
  return this.name;
}

const user = { name: 'Sangyun', getName };
const admin = { name: 'Admin', getName };

user.getName();  // 'Sangyun'
admin.getName(); // 'Admin'

const detached = user.getName;
// detached(); // TypeError: this가 undefined인데 name을 읽으려 함
```

객체에 함수를 저장한다고 this가 그 객체에 영구적으로 고정되지는 않는다. 함수만 꺼내 단독 호출하면 `user.`라는 호출 관계가 사라진다. 구조 분해하거나 콜백으로 전달할 때도 주의한다. 콜백의 this는 콜백을 실행하는 API의 호출 방식에 따라 달라질 수 있다.

## call, apply, bind

```js
function greet(prefix, suffix) {
  'use strict';
  return `${prefix} ${this.name}${suffix}`;
}

const person = { name: 'Sangyun' };

greet.call(person, 'Hello', '!');    // 'Hello Sangyun!'
greet.apply(person, ['Hello', '!']); // 'Hello Sangyun!'

const greetPerson = greet.bind(person, 'Hello');
// bind는 greet를 지금 실행하지 않고 새 함수를 반환한다.
greetPerson('!'); // 'Hello Sangyun!'
```

- `call`: this와 인자를 각각 전달하고 즉시 실행한다.
- `apply`: this와 인자 목록을 배열 등으로 전달하고 즉시 실행한다.
- `bind`: this와 일부 인자를 미리 지정한 새 함수를 반환한다. 원래 함수 자체를 수정하지 않는다.

bind에 전달하는 객체는 전체 this 값이다. 함수가 `this.name`을 사용하면 그 객체에서 name을 읽을 뿐, name 프로퍼티가 있는지 bind가 검증해 주지는 않는다.

### bind(null, id)의 의미

```js
function updateInvoice(id, formData) {
  // this는 사용하지 않고 id와 formData로 처리한다.
}

const updateWithId = updateInvoice.bind(null, 'invoice-1');
updateWithId(formData);
// 인자 전달 관점에서는 updateInvoice('invoice-1', formData)와 같다.
```

bind의 첫 인자 null은 this 자리이며 updateInvoice의 첫 매개변수 id로 전달되지 않는다. 두 번째 인자부터 원래 함수의 인자로 미리 고정된다. 위 코드는 this 지정이 아니라 인자 일부를 미리 정하는 부분 적용이 목적이다.

## 화살표 함수는 바깥 this를 사용한다

```js
const user = {
  name: 'Sangyun',
  makeReader() {
    return () => this.name;
  },
};

const reader = user.makeReader();
reader(); // 'Sangyun'
reader.call({ name: 'Other' }); // 'Sangyun'
```

`user.makeReader()`의 this는 user이고, 내부 화살표 함수는 그 this를 사용한다. 화살표 함수는 자신만의 this가 없으므로 call, apply, bind로 this를 바꿀 수 없다.

이 과정은 다음과 같이 구분한다.

1. 일반 메서드인 makeReader를 `user.makeReader()`로 호출해 this가 user로 결정된다.
2. 그 실행 중에 내부 화살표 함수가 만들어진다.
3. 반환된 화살표 함수는 나중에 단독 호출해도 바깥 실행의 this를 사용한다.

### 일반 함수를 반환하면 달라지는 점

```js
'use strict';

const user = {
  name: 'Sangyun',
  makeReader() {
    return function () {
      return this.name;
    };
  },
};

const reader = user.makeReader();
reader.call({ name: 'Admin' }); // 'Admin'
// reader(); // TypeError: 단독 호출의 this는 undefined
```

일반 함수는 user의 메서드 안에서 만들어졌다는 이유만으로 바깥 this를 물려받지 않는다. 반환된 함수를 나중에 어떻게 호출하는지가 중요하다. 반면 앞 예제의 화살표 함수는 바깥 this를 사용하므로 `call()`로 다른 객체를 지정해도 바뀌지 않는다.

### 문자열을 복사해서 기억하는 것은 아니다

아래 코드는 독립적으로 실행하는 화살표 함수 예제다.

```js
const user = {
  name: 'Sangyun',
  makeReader() {
    return () => this.name;
  },
};

const reader = user.makeReader();
user.name = 'Updated';

reader(); // 'Updated'
reader.call({ name: 'Admin' }); // 'Updated'
```

화살표 함수가 기억하는 것은 원래 문자열 'Sangyun'이 아니라 바깥 this를 통해 참조하는 user 객체다. 실행할 때 그 객체의 현재 name을 읽으므로 프로퍼티 변경은 결과에 반영된다.

객체 리터럴의 `getName: () => this.name`은 해당 객체를 this로 자동 지정하지 않는다. 객체 리터럴 자체는 새로운 this를 만들지 않으므로 바깥 실행 문맥을 따른다.

## new와 이벤트 핸들러

```js
function User(name) {
  this.name = name;
}

const user = new User('Sangyun');
user.name; // 'Sangyun'
```

new로 호출하면 새 인스턴스를 this로 사용한다. 화살표 함수는 생성자로 사용할 수 없다. 생성 가능한 함수를 bind한 뒤 new로 호출하는 경우에는 미리 바인딩한 this 대신 새 인스턴스를 사용한다.

DOM의 `addEventListener`에 일반 함수를 직접 전달하면 this는 해당 핸들러의 `event.currentTarget`이다. 화살표 함수는 이 규칙 대신 바깥 this를 사용하므로, 요소가 필요하면 `event.currentTarget`을 명시적으로 사용하는 방법이 있다. React 함수 컴포넌트는 class 인스턴스 this 대신 props, state, 클로저를 중심으로 작성한다.

## 현대 프론트엔드에서 this를 만나는 사용 사례

React 함수 컴포넌트에서 this를 직접 사용할 일이 적어도, 브라우저 API나 객체 기반 라이브러리의 메서드를 콜백으로 전달할 때는 호출 대상을 이해해야 한다. 대표적으로 오디오와 비디오 요소는 HTMLMediaElement의 play, pause 같은 공통 API를 사용한다.

이 섹션은 오디오·비디오 제어와 메서드 콜백, 기존 React 클래스 컴포넌트와 jQuery 코드의 유지보수를 중심으로 살펴본다. this를 새 코드에 억지로 도입하기보다, 실제로 만났을 때 호출 대상과 수정 방법을 판단하는 것이 목적이다.

### 오디오·비디오: 메서드만 전달하면 원래 호출 관계는 유지되지 않는다

아래 예제는 미디어 요소와 버튼이 DOM에 준비된 뒤 실행한다.

```js
const media = document.querySelector('audio, video');
const button = document.querySelector('#pause');

if (!(media instanceof HTMLMediaElement) || !button) {
  throw new Error('미디어 요소와 일시정지 버튼이 필요합니다.');
}

media.pause(); // media를 대상으로 호출하므로 정상 동작

// 잘못된 연결 예시: 실행하지 않는다.
// button.addEventListener('click', media.pause);
```

`media.pause`를 전달하는 것은 함수 참조를 전달하는 것이지, `media`와 연결된 호출을 함께 전달하는 것이 아니다. 일반 함수 리스너를 실행할 때 브라우저는 this로 리스너를 등록한 button을 사용한다. 하지만 pause는 HTMLMediaElement를 대상으로 실행되어야 하므로 이 방식은 호출 오류를 일으킨다.

여기서 'this가 유실된다'는 것은 **원래 객체가 사라진다는 뜻이 아니라, 원래 객체를 this로 사용하는 호출 관계가 유지되지 않는다는 뜻**이다. 콜백의 this가 무조건 undefined가 되는 것도 아니다. 콜백을 호출하는 API에 따라 다른 객체가 될 수 있다.

### bind 또는 래퍼 함수로 호출 대상을 유지한다

```js
// 방법 1: this가 media로 고정된 새 함수를 만든다.
const boundPause = media.pause.bind(media);

// 방법 2: 콜백 내부에서 media.pause()로 호출한다.
const wrappedPause = () => media.pause();
```

두 방식 중 하나를 선택한다. 두 번째 방식은 화살표 함수의 this로 media를 지정하는 것이 아니다. 화살표 함수에서는 this를 사용하지 않고, 내부의 `media.pause()`가 호출 대상을 명확히 한다.

`media.pause()`처럼 직접 호출할 때는 별도의 bind가 필요 없다. 모든 브라우저 API 메서드에 일괄적으로 bind를 붙일 필요도 없다. 메서드가 this에 의존하는지, 라이브러리가 이미 바인딩된 함수를 제공하는지 확인한다.

### 이벤트 제거에는 같은 함수 참조가 필요하다

```js
const first = media.pause.bind(media);
const second = media.pause.bind(media);

first === second; // false

button.addEventListener('click', first);
button.removeEventListener('click', second); // first는 제거되지 않음
button.removeEventListener('click', first);  // 등록한 함수로 제거
```

bind는 호출할 때마다 새 함수를 만든다. 따라서 등록할 때와 제거할 때 각각 bind를 호출하면 동작이 같아도 서로 다른 함수다. 다음처럼 화살표 함수를 각각 만드는 경우도 동일하다.

```js
// 잘못된 cleanup 예시
button.addEventListener('click', () => media.pause());
button.removeEventListener('click', () => media.pause());
```

React Effect에서 직접 DOM 리스너를 연결할 때도 같은 참조를 보관한다. 아래는 컴포넌트가 가진 mediaRef와 buttonRef를 사용하는 부분 예제다.

```jsx
useEffect(() => {
  const media = mediaRef.current;
  const button = buttonRef.current;
  if (!media || !button) return;

  const handlePause = () => media.pause();
  button.addEventListener('click', handlePause);

  return () => {
    button.removeEventListener('click', handlePause);
  };
}, []);
```

이 예제는 요소가 마운트 동안 교체되지 않는 경우를 가정한다. 이벤트 종류, 함수 참조, capture 설정을 맞춰 제거한다. 일반적인 React 버튼이라면 직접 리스너를 연결하기보다 `onClick={() => mediaRef.current?.pause()}`처럼 JSX 이벤트를 사용하는 편이 간단하다.

### 레거시 React 클래스 컴포넌트

기존 React 코드에서는 this로 컴포넌트 인스턴스의 props와 state에 접근한다. 일반 메서드를 이벤트 핸들러로 전달할 때 인스턴스와의 관계를 유지하기 위해 생성자에서 bind하는 패턴을 볼 수 있다.

```jsx
import { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState((state) => ({ count: state.count + 1 }));
  }

  render() {
    return <button onClick={this.handleClick}>{this.state.count}</button>;
  }
}
```

bind가 없으면 전달한 일반 메서드가 Counter 인스턴스를 this로 사용한다고 보장할 수 없다. 클래스 필드의 화살표 함수인 `handleClick = () => { ... }`로 인스턴스의 this를 사용하는 코드도 볼 수 있다. 이는 일반 메서드가 자동으로 바인딩되는 것과 다르다.

React는 클래스 컴포넌트를 지원하지만 신규 코드에는 함수 컴포넌트를 권장한다. 이 예제는 기존 코드의 this.state, this.props, this.setState와 이벤트 바인딩을 읽기 위한 것이다. [React: Component](https://react.dev/reference/react/Component)

### 레거시 jQuery 이벤트 핸들러

jQuery의 일반 함수 핸들러에서는 this로 이벤트에 대응하는 DOM 요소를 사용할 수 있다. 아래 위임 예제에서는 선택자에 일치한 버튼이 this가 된다.

```js
$('#list').on('click', '.select-button', function () {
  $(this).addClass('selected');
});
```

이때 this는 jQuery 객체가 아닌 DOM 요소이고, `$(this)`로 감싸 jQuery 메서드를 사용한다. 버튼 내부의 span을 클릭해도 this는 매칭된 버튼이며, 실제 이벤트 시작점인 event.target은 span일 수 있다.

이 코드를 단순히 화살표 함수로 바꾸면 jQuery가 지정하는 this 대신 바깥 this를 사용하게 된다. 화살표 함수를 사용하려면 this 의존도 함께 바꿔야 한다.

```js
$('#list').on('click', '.select-button', (event) => {
  $(event.currentTarget).addClass('selected');
});
```

여기서 currentTarget은 jQuery 위임 이벤트가 제공하는 매칭 요소다. 리스너가 붙은 부모를 currentTarget으로 제공하는 Native DOM 이벤트 위임과 혼동하지 않는다. jQuery에서 위임 리스너를 등록한 부모는 event.delegateTarget으로 확인할 수 있다. [jQuery: on](https://api.jquery.com/on/)

### Next.js의 bind 사용과 구분하기

Next.js의 `updateInvoice.bind(null, id)`는 위 사례와 목적이 다르다. 미디어나 클래스 콜백은 this를 유지하는 것이 목적이고, 해당 Server Action 코드는 id 인자를 미리 지정하는 부분 적용이 목적이다.

| 만나는 상황 | this 또는 bind의 목적 |
| --- | --- |
| 오디오·비디오 메서드를 콜백으로 전달 | 실제 미디어 요소를 호출 대상으로 유지 |
| React 클래스 컴포넌트 | 컴포넌트 인스턴스의 상태와 메서드 사용 |
| jQuery 일반 함수 이벤트 핸들러 | 이벤트에 대응하는 DOM 요소 접근 |
| Next.js Server Action의 bind(null, id) | this 활용이 아니라 인자 일부를 미리 지정 |

## 면접 요약

> 일반 함수의 this는 호출 방식에 따라 정해진다. 메서드 호출에서는 호출한 객체, strict mode의 단독 호출에서는 undefined이며, call과 apply로 명시하거나 bind로 고정한 함수를 만들 수 있다. 화살표 함수는 자체 this 없이 바깥 this를 사용한다. 따라서 메서드를 꺼내 콜백으로 전달할 때 원래 객체와의 관계가 유지되는지 확인해야 한다.

## 관련 자료

- [JavaScript: 스코프와 구조 분해](Javascript.md)
- [Next.js Dashboard: bind를 사용한 수정 액션](Next.js/dashboard-app/chapter-11-mutating-data.md)
- [MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [MDN: bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [MDN: HTMLMediaElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)
- [MDN: addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- [React: Component](https://react.dev/reference/react/Component)
- [jQuery: on](https://api.jquery.com/on/)
