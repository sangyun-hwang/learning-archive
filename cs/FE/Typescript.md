# Typescript

- [any 타입](#any-타입)
- [제네릭](#제네릭)

<br>

## 기본 타입

### 문자열, 숫자, 진위값

```ts
const str: string = "hello";
const num: number = 10;
const show: boolean = true;
```

<br>

### 배열

```ts
const arr: Array<number> = [1, 2, 3];
const heroes: Array<string> = ["Capt", "Thor", "Hulk"];

const items: number[] = [1, 2, 3];
```

- 대문자 Array를 사용하고, 각괄호 안에 내부 요소들의 type을 지정

- **literal** 방식으로 `[]` 자체가 Array를 의미
- 내부 요소의 type을 `[]` 앞에 표시

<br>

### 튜플

> 튜플: 배열의 특정 인덱스의 타입을 명시

모든 인덱스의 type이 각각 정의

```ts
const address: [string, number] = ["gangnam", 100];
```

<br>

### 객체

```ts
const obj: object = {};

const person: { name: string; age: number } = {
  name: "thor",
  age: 1000,
};
```

- Array와 달리 object 소문자로 type 정의
- 객체 내부의 특정 속성의 type을 지정 가능

<br>

## 함수 타입

### params와 return값의 type 정의

```ts
function sum(a: number, b: number): number {
  return a + b;
}
```

- return 값의 type 명시가 된다면 return의 type 뿐만 아니라 return의 유무도 확인
- return이 없다면 오류 발생

<br>

### 반환값이 없는 함수인 경우

```ts
function log(): void {
  console.log("hello");
}
```

- return type을 **void**로 둔다.

<br>

### params의 엄격한 제한

> JS의 경우 params의 인자 개수에 대한 유연함이 존재

```js
function sum(a, b) {
  return a + b;
}

sum(10, 20, 30, 40, 50); // 30
```

<br>

> TS로 params의 type이 정의되면 이런 유연함을 불허

```ts
function sum(a: number, b: number): number {
  return a + b;
}

sum(10, 20, 30, 40); // 2개의 인수가 필요한데 4개를 가져왔습니다.
```

30, 40 인자에 대해 오류 발생

<br>

### optional parameter

> 여러 params를 쓸 때도 있고, 안 쓸 때도 있는 경우에 사용

인자의 type을 미리 정의한 후, 필요한 경우 인자 생략을 허용

```ts
function log(a: string, b?: string): string {}

log("hello world");
log("hello ts", "abc");
```

- '?'를 params 옆에 바로 붙여 표시
- 만약 해당 param이 있다면 type은 ~~라는 의미

<br>

## 인터페이스

### 인터페이스의 목적

> object의 속성들의 구체적인 type(스펙)의 반복 제거

- 앞에서 object의 스펙이 복잡하면 복잡할수록 코드가 지저분해진다.
- 반복이 많으면 많을수록 더 곤란
- 때문에 한 번만 정의하고, 이를 가져다 쓰는 방식
- 깔끔한 코드 및 유지보수, 협업에 유리

<br>

### 변수 정의

```ts
interface User {
  age: number;
  name: string;
}

const seho: User = {
  age: 33,
  name: "세호",
};
```

<br>

### 함수 인자 정의

```ts
function getUser(user: User): void {
  console.log(user);
}
getUser(seho);
```

<br>

### 함수 구조 정의

> 함수 스펙(구조)에 인터페이스 활용

```ts
interface SumFunction {
  (a: number, b: number): number;
}

let sum: SumFunction;
sum = function (a, b) {
  return a + b;
};
```

- 함수 인자나 return 값의 type을 interface에 미리 정의
- 함수 선언에서 interface를 지정
- 인자의 수가 적고 구조가 간단한 경우 interface보다는 인자에 바로 지정하는 것을 권장

<br>

### 인덱싱 방식 정의

```ts
interface StringArray {
  [index: number]: string;
}

const arr: StringArray = ["a", "b", "c"];
arr[0]; // 'a'
```

- 인덱스에 숫자 type만 입력하도록 제한
- 인덱싱으로 뽑힌 결과는 문자 type이어야 한다는 의미

<br>

### 인터페이스 딕셔너리 패턴

```ts
interface StringRegexDictionary {
  [key: string]: RegExp;
}

const obj: StringRegexDictionary = {
  // sth: /abc/,
  cssFile: /\.css$/, // css 확장자를 가지는 모든 파일을 들고오는 정규표현식
  jsFile: /\.js$/,
};
```

- RegExp: Regex Expression(정규 표현식)의 약자
- 딕셔너리에서 key의 type은 string, value의 type은 RegExp가 되도록 제한하는 인터페이스

<br>

### 인터페이스 확장(상속)

```ts
interface Person {
  name: string;
  age: number;
}

// interface Developer {
//   name: string;
//   age: number;
//   language: string;
// }

interface Developer extends Person {
  language: string;
}

const capt: Developer = {
  name: "캡틴",
  age: 100,
  language: "타입스크립트",
};
```

- **interface 자식인터페이스 extends 부모인터페이스**로 인터페이스 상속

<br>

## any 타입

TypeScript에서 모든 타입의 값을 가질 수 있는 타입으로 타입스크립트의 타입검사를 무시하기 때문에 타입스크립트를 사용하지 않는 것과 같기 때문에 사용하지 않는 것을 권장한다.

```ts
let variable: any;

variable = 10; // number
variable = "hello"; // string
variable = true; // boolean
```

<br>

## unknown 타입

`unknown`은 TypeScript 3.0버전에서 도입된 타입으로 `any`와 유사하게 모든 타입의 값을 가질 수 있지만, 타입 검사를 강제하기 때문에 더 안전하게 사용할 수 있다.

```ts
let value: unknown;

value = 10;
value = "hello";
value = true;

// 타입 확인을 통해 안전하게 사용
if (typeof value === "string") {
  console.log(value.toUpperCase()); // 안전하게 문자열 메서드 사용 가능
}
```

<br>

## 제네릭

제네릭이란 타입을 마치 함수의 파라미터처럼 사용하는 것을 의미합니다.

제네릭을 사용하면 함수나 클래스 내에서 사용되는 데이터의 타입을 동적으로 지정할 수 있습니다. 이는 컴파일 시점에 타입을 체크하고 에러를 사전에 발견할 수 있어 런타임 에러를 방지하는 데 도움이 됩니다.

제네릭을 사용하는 가장 간단한 예제로는 배열의 타입 안정성을 보장하는 Array<T>를 들 수 있습니다. T는 타입 매개변수로, 배열에 저장될 요소의 타입을 동적으로 지정할 수 있게 합니다. 예를 들어, Array<number>는 숫자로 이루어진 배열을 의미하며, Array<string>은 문자열로 이루어진 배열을 의미합니다.

```typescript
function logText<T>(text: T): T {
  return text;
}

const str = logText<string>("abc");
const num = logText<number>(1);
```

인터페이스에 제네릭을 선언하는 방법

```typescript
interface Dropdown<T> {
  value: T;
  selected: boolean;
}
```

복수의 타입이 혼합된 제네릭을 선언하는 방법

```typescript
interface Dropdown<T, U> {
  value: T;
  selected: U;
}
```

#### 제네릭의 타입 제한

제네릭을 사용할 경우 동적으로 타입을 정의 할 수 있지만 그로 인해 오류가 발생할 수도 있습니다.

```typescript
function logText<T>(text: T): T {
  console.log(text.length); // Error: T doesn't have .length
  return text;
}
```

위 코드를 보면 선언한 `T`가 어떤 타입인지 구체적인 정의가 없기 때문에 `length` 코드에서 오류가 나게 됩니다. 이럴 경우 `extends`키워드를 사용하여 `length`에 대해 동작하는 인자만 넘겨받을 수 있습니다. 이러한 방식을 제네릭의 타입 제한이라고 합니다.

```typescript
interface LengthType {
  length: number;
}

function logTextLength<T extends LengthType>(text: T): T {
  console.log(text.length);
  return text;
}

logTextLength("text");
logTextLength({ length: 0, value: "hi" }); // `text.length` 코드는 객체의 속성 접근과 같이 동작하므로 오류 없음
logTextLength(true); // 'boolean' 형식의 인수는 'LengthType' 형식의 매개 변수에 할당될 수 없습니다.ts(2345)
```
