# JavaScript

## 데이터 타입과 동등 비교

JavaScript의 값은 크게 원시 타입과 객체 타입으로 나뉩니다.

원시 타입은 `undefined`, `null`, `boolean`, `number`, `bigint`, `string`, `symbol`입니다. 원시 값은 불변 값이며, 값을 변경하는 것처럼 보여도 실제로는 새로운 값이 만들어집니다.

객체 타입은 객체, 배열, 함수, 정규식, 클래스 인스턴스처럼 참조를 통해 다루는 값입니다. 객체는 참조가 같을 때 같은 값으로 판단됩니다.

```js
Object.is({}, {}); // false

const user = {};
Object.is(user, user); // true
```

비교 방식은 목적에 맞게 선택해야 합니다.

- `==`: 암묵적 타입 변환 후 비교하므로 예측이 어렵습니다.
- `===`: 타입 변환 없이 비교합니다.
- `Object.is`: `NaN`, `-0` 같은 특수 케이스까지 더 엄격하게 다룹니다.

React의 state 비교와 memoization은 얕은 비교를 기반으로 동작하는 경우가 많습니다. 객체 내부 값을 직접 수정하면 참조가 바뀌지 않아 변경을 감지하지 못할 수 있습니다.

## 함수

JavaScript에서 함수는 일급 객체입니다. 변수에 할당하거나 인자로 전달하고, 다른 함수의 반환값으로 사용할 수 있습니다.

```js
function add(a, b) {
  return a + b;
}

const multiply = function (a, b) {
  return a * b;
};

const subtract = (a, b) => a - b;
```

함수 선언문은 호이스팅되어 선언 이전에도 호출할 수 있습니다. 함수 표현식은 변수 선언 방식에 따라 초기화 이전 접근에서 문제가 생길 수 있습니다.

화살표 함수는 자신만의 `this`, `arguments`, `prototype`을 갖지 않습니다. 생성자 함수로 사용할 수 없고, 외부 lexical scope의 `this`를 사용합니다.

좋은 함수는 한 가지 일을 명확하게 수행하고, 부수 효과를 줄이며, 이름만 보고 의도를 추측할 수 있어야 합니다.

## 클래스

`class`는 프로토타입 기반 상속을 더 읽기 쉬운 문법으로 표현한 것입니다.

```js
class User {
  constructor(name) {
    this.name = name;
  }

  get displayName() {
    return this.name;
  }

  sayHello() {
    return `Hello, ${this.name}`;
  }

  static create(name) {
    return new User(name);
  }
}
```

- `constructor`: 인스턴스 생성 시 초기화합니다.
- instance method: prototype에 등록됩니다.
- static method: 클래스 자체에서 호출합니다.
- `extends`, `super`: 상속 관계에서 부모 클래스의 생성자와 메서드를 호출합니다.

## 스코프

스코프는 변수와 함수 같은 식별자에 접근할 수 있는 코드의 범위입니다.

```js
function greet() {
  const message = 'hello';
  console.log(message);
}

greet();
console.log(message); // ReferenceError
```

`message`는 `greet`의 함수 스코프 안에서만 접근할 수 있습니다.

### Lexical Scope

JavaScript는 코드를 작성한 위치를 기준으로 식별자의 접근 범위를 결정하는 lexical scope를 사용합니다. 함수를 어디에서 호출했는지가 아니라 어디에서 정의했는지가 변수 탐색에 중요합니다.

```js
const value = 'global';

function printValue() {
  console.log(value);
}

function execute() {
  const value = 'local';
  printValue();
}

execute(); // global
```

`printValue`는 전역에서 정의됐으므로 `execute` 내부의 지역 변수에는 접근하지 않고 자신이 정의된 바깥 환경에서 `value`를 찾습니다.

### 스코프의 종류

#### 전역 스코프

전역 스코프는 가장 바깥 범위입니다. 전역 변수는 여러 코드에서 접근할 수 있어 이름 충돌과 의도하지 않은 변경을 만들 수 있으므로 필요한 범위 안으로 제한하는 편이 좋습니다.

브라우저의 일반 script에서 최상위 `var`는 전역 객체인 `window`의 프로퍼티가 될 수 있지만, 최상위 `let`과 `const`는 전역 lexical binding을 만들며 `window`의 프로퍼티가 되지 않습니다.

```html
<script>
  var first = 1;
  let second = 2;

  console.log(window.first);  // 1
  console.log(window.second); // undefined
</script>
```

#### 모듈 스코프

ES Module 최상위의 식별자는 전역 객체가 아니라 해당 모듈의 스코프에 속합니다. 다른 모듈에서 사용하려면 명시적으로 `export`와 `import`를 거칩니다.

```js
// user.js
const user = 'Kim';

export function getUser() {
  return user;
}
```

```js
// app.js
import { getUser } from './user.js';
```

모듈 최상위의 `var`도 `window`의 프로퍼티가 되지 않습니다.

#### 함수 스코프

함수 안에서 선언한 식별자는 기본적으로 함수 밖에서 접근할 수 없습니다.

```js
function calculate() {
  var first = 1;
  let second = 2;
  const third = 3;
}

console.log(first); // ReferenceError
```

`var`는 블록 스코프를 따르지 않지만 자신이 선언된 함수의 스코프는 벗어나지 않습니다.

#### 블록 스코프

`let`과 `const`는 `{}` 블록을 기준으로 유효 범위를 만듭니다. `var`는 블록 스코프를 따르지 않습니다.

```js
if (true) {
  var functionScoped = 'var';
  let blockScoped = 'let';
  const alsoBlockScoped = 'const';
}

console.log(functionScoped);  // var
console.log(blockScoped);     // ReferenceError
console.log(alsoBlockScoped); // ReferenceError
```

```js
function test() {
  if (true) {
    var value = 1;
  }

  console.log(value); // 1
}

console.log(value); // ReferenceError
```

### Scope Chain

현재 lexical environment에서 식별자를 찾지 못하면 바깥 lexical environment를 순서대로 탐색합니다.

```js
const value = 'global';

function outer() {
  const value = 'outer';

  function inner() {
    console.log(value);
  }

  inner();
}

outer(); // outer
```

탐색 방향은 다음과 같습니다.

```text
inner scope -> outer scope -> global scope
```

각 lexical environment가 자신을 감싼 바깥 환경의 참조를 가지므로 안쪽에서는 바깥쪽 식별자를 찾을 수 있습니다. 반대로 바깥 환경에서 안쪽 환경을 거슬러 탐색하지는 않습니다.

### Variable Shadowing

안쪽 스코프에서 바깥쪽과 같은 이름을 선언하면 가까운 식별자가 바깥 식별자를 가립니다.

```js
const name = 'global';

function printName() {
  const name = 'local';
  console.log(name);
}

printName();       // local
console.log(name); // global
```

스코프 체인은 가장 가까운 곳에서 이름을 찾으면 탐색을 멈춥니다. Shadowing 자체가 잘못은 아니지만 중첩된 코드에서 같은 이름을 과도하게 사용하면 읽기 어려울 수 있습니다.

### Hoisting과 Temporal Dead Zone

`var` 선언은 스코프가 만들어질 때 `undefined`로 초기화되므로 선언문 전에 읽을 수 있습니다.

```js
console.log(value); // undefined
var value = 1;
```

`let`과 `const`도 스코프의 binding은 미리 만들어지지만 선언문이 평가되기 전까지 초기화되지 않습니다. 이 구간을 Temporal Dead Zone, TDZ라고 합니다.

```js
console.log(value); // ReferenceError
const value = 1;
```

TDZ에서는 변수가 스코프에 없는 것이 아니라 binding은 존재하지만 초기화 전이므로 접근할 수 없습니다.

Shadowing과 TDZ가 함께 일어나면 바깥 변수를 대신 찾지 않습니다.

```js
const value = 'global';

{
  console.log(value); // ReferenceError
  const value = 'block';
}
```

블록의 `value` binding이 블록 전체에서 전역 `value`를 가리지만 선언문 전까지 TDZ에 있기 때문에 에러가 발생합니다.

### for문의 var와 let

`var`로 선언한 반복 변수는 블록이 아니라 하나의 함수 또는 전역 binding을 사용합니다. 비동기 callback들이 같은 `i`를 closure로 참조하므로 callback 실행 시점에는 반복이 끝난 값인 `3`을 읽습니다.

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i));
}

// 3
// 3
// 3
```

`for (let i = ...)`는 순회마다 새로운 lexical binding을 만듭니다. 각 callback은 자신이 만들어진 순회의 서로 다른 `i`를 참조합니다.

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i));
}

// 0
// 1
// 2
```

이 차이는 비동기 작업이 블록별 call stack에서 실행되기 때문이 아니라 callback이 캡처한 binding이 같거나 다르기 때문에 생깁니다.

`let`을 사용한다고 항상 반복마다 binding이 생기는 것은 아닙니다. 반복문 바깥에서 하나의 `let`을 선언하면 callback들이 같은 binding을 참조합니다.

```js
let i = 0;

for (; i < 3; i++) {
  setTimeout(() => console.log(i));
}

// 3
// 3
// 3
```

### 스코프와 클로저

클로저는 함수가 자신이 정의된 lexical environment를 계속 참조할 수 있게 합니다. 함수 실행이 끝난 뒤에도 내부 함수가 바깥 변수를 참조하고 있다면 그 환경은 유지될 수 있습니다.

면접에서는 다음처럼 정리할 수 있습니다.

> JavaScript는 함수의 호출 위치가 아니라 정의 위치를 기준으로 스코프가 결정되는 lexical scope를 사용합니다. 식별자는 현재 스코프에서 시작해 바깥쪽 scope chain으로 탐색하며 가장 가까운 값을 사용합니다. `var`는 함수 스코프이고 `let`과 `const`는 블록 스코프이며, `for (let i = ...)`에서는 반복마다 새로운 binding이 만들어져 각 closure가 서로 다른 값을 기억합니다.

## 클로저

클로저는 함수와 그 함수가 선언된 lexical environment의 조합입니다. 함수가 선언된 스코프 밖에서 실행되더라도 선언 당시의 외부 변수에 접근할 수 있습니다.

```js
function createCounter() {
  let count = 0;

  return function increase() {
    count += 1;
    return count;
  };
}

const counter = createCounter();
counter(); // 1
counter(); // 2
```

클로저는 상태 은닉, 함수형 프로그래밍, React Hook의 동작 이해에 중요합니다. 다만 클로저가 큰 객체를 계속 참조하면 메모리 해제가 늦어질 수 있으므로 장기간 유지되는 콜백에서는 참조 범위를 조심해야 합니다.

## 이벤트 루프와 비동기 통신

JavaScript는 하나의 call stack에서 코드를 실행하지만, 브라우저 환경은 Web API, task queue, microtask queue를 함께 사용해 비동기 작업을 처리합니다.

실행 흐름은 다음과 같습니다.

1. call stack의 동기 코드가 실행됩니다.
2. 비동기 작업은 Web API 또는 런타임 영역으로 위임됩니다.
3. 완료된 콜백은 queue에 들어갑니다.
4. call stack이 비면 event loop가 queue의 작업을 stack으로 올립니다.

microtask는 macrotask보다 먼저 처리됩니다. `Promise.then`, `queueMicrotask`는 microtask에 들어가고, `setTimeout`, DOM event, network callback은 macrotask로 처리됩니다.

렌더링은 일반적으로 microtask 처리가 끝난 뒤 브라우저가 화면을 그릴 수 있는 시점에 일어납니다. 애니메이션 작업은 `requestAnimationFrame`을 사용해 브라우저의 repaint 주기에 맞추는 것이 좋습니다.

## React에서 자주 사용하는 JavaScript 문법

### 구조 분해 할당

```js
const user = { name: "Kim", age: 28 };
const { name, age = 0 } = user;

const colors = ["red", "blue"];
const [primary] = colors;
```

기본값은 값이 `undefined`일 때만 적용됩니다.

### rest와 spread

```js
const nextUser = { ...user, age: 29 };
const [first, ...rest] = colors;
```

React state를 갱신할 때 객체나 배열을 직접 수정하지 않고 새로운 참조를 만드는 데 자주 사용합니다.

### 배열 메서드

- `map`: 배열을 다른 형태의 배열로 변환합니다.
- `filter`: 조건에 맞는 요소만 남깁니다.
- `reduce`: 배열을 하나의 값으로 누적합니다.
- `forEach`: 반환값 없이 순회합니다.

### 조건부 렌더링

삼항 연산자와 `&&`를 사용할 수 있지만, 조건이 복잡해지면 JSX 밖에서 변수나 함수로 분리하는 편이 좋습니다.

## 클린코드

JavaScript 코드에서는 암묵적인 변환과 의미 없는 임시 변수를 줄이는 것이 중요합니다.

- `==`보다 `===`를 사용합니다.
- `Number.isNaN`처럼 의도가 분명한 API를 사용합니다.
- 복잡한 조건은 이름 있는 변수나 함수로 분리합니다.
- `min/max`, `begin/end`, `first/last`, `prefix/suffix`처럼 경계를 표현하는 이름을 일관되게 사용합니다.
- 함수는 하나의 추상화 수준에서 읽히도록 유지합니다.

## 런타임 구조

JavaScript 엔진은 소스 코드를 파싱해 AST를 만들고, 바이트코드 또는 최적화된 기계어 코드로 실행합니다. 브라우저의 대표적인 엔진인 V8은 Ignition 인터프리터와 TurboFan 최적화 컴파일러를 사용합니다.

실행 컨텍스트에는 변수 환경, scope chain, `this` 바인딩 정보가 포함됩니다. 함수가 호출될 때마다 실행 컨텍스트가 call stack에 쌓이고, 실행이 끝나면 제거됩니다.

스코프는 식별자를 참조할 수 있는 유효 범위입니다.

- 전역 스코프: 코드 전체에서 접근할 수 있는 범위
- 함수 스코프: 함수 내부에서 유효한 범위
- 블록 스코프: `let`, `const`가 `{}` 블록 단위로 갖는 범위

가비지 컬렉터는 더 이상 접근할 수 없는 객체를 메모리에서 해제합니다. 현대 JavaScript 엔진은 주로 mark-and-sweep 계열 알고리즘을 사용해 루트 객체에서 도달할 수 없는 객체를 정리합니다.

## AJAX

AJAX는 전체 페이지를 새로고침하지 않고 필요한 데이터만 비동기로 주고받는 방식입니다. 과거에는 `XMLHttpRequest`를 주로 사용했고, 현재는 `fetch` API를 많이 사용합니다.

```js
const response = await fetch("/api/todos");
const todos = await response.json();
```

## CSR vs SSR vs SSG

### CSR

Client Side Rendering은 브라우저에서 JavaScript를 실행해 화면을 구성하는 방식입니다. 초기 로딩 이후 사용자 상호작용이 빠르지만, 초기 번들 크기와 SEO를 신경 써야 합니다.

### SSR

Server Side Rendering은 서버에서 HTML을 만들어 클라이언트에 전달하는 방식입니다. 초기 화면을 빠르게 보여주고 SEO에 유리하지만 서버 렌더링 비용과 캐싱 전략을 고려해야 합니다.

### SSG

Static Site Generation은 빌드 시점에 HTML을 미리 만들어 두는 방식입니다. 변경이 잦지 않은 콘텐츠에 적합하고 CDN 캐싱과 잘 맞습니다.

## 프론트엔드 성능 측정

성능 측정은 사용자 경험과 직접 연결됩니다.

- FCP: 첫 콘텐츠가 그려지는 시점
- LCP: 가장 큰 콘텐츠가 그려지는 시점
- FID / INP: 사용자 입력에 대한 반응성
- CLS: 레이아웃 이동 안정성
- TTI: 페이지가 상호작용 가능한 상태가 되는 시점

Chrome DevTools Performance, Lighthouse, Web Vitals를 사용해 병목을 측정하고 이미지 최적화, 코드 스플리팅, 지연 로딩, 캐싱 전략을 점검합니다.
