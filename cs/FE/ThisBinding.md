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

## 면접 요약

> 일반 함수의 this는 호출 방식에 따라 정해진다. 메서드 호출에서는 호출한 객체, strict mode의 단독 호출에서는 undefined이며, call과 apply로 명시하거나 bind로 고정한 함수를 만들 수 있다. 화살표 함수는 자체 this 없이 바깥 this를 사용한다. 따라서 메서드를 꺼내 콜백으로 전달할 때 원래 객체와의 관계가 유지되는지 확인해야 한다.

## 관련 자료

- [JavaScript: 스코프와 구조 분해](Javascript.md)
- [Next.js Dashboard: bind를 사용한 수정 액션](Next.js/dashboard-app/chapter-11-mutating-data.md)
- [MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [MDN: bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
