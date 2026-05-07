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
