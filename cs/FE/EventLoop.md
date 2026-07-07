# Event Loop

JavaScript는 기본적으로 하나의 Call Stack에서 코드를 실행한다.

하지만 브라우저에서는 타이머, 네트워크 요청, 이벤트 처리, Promise 같은 비동기 작업이 함께 동작한다. 이벤트 루프는 이런 비동기 작업들이 실행 순서에 맞게 처리되도록 조율하는 구조이다.

## 주요 구성 요소

### Call Stack

현재 실행 중인 JavaScript 코드가 쌓이는 공간이다.

동기 코드는 Call Stack에 올라가고, 실행이 끝나면 제거된다.

### Web APIs

브라우저가 제공하는 비동기 기능이다.

예를 들어 다음과 같은 작업은 브라우저의 Web APIs 영역에서 처리될 수 있다.

- `setTimeout`
- DOM 이벤트
- 네트워크 요청
- 타이머

### Task Queue

`setTimeout`, 사용자 이벤트, 네트워크 콜백 같은 작업이 대기하는 큐이다.

Task Queue에 들어간 작업은 Call Stack이 비었을 때 이벤트 루프에 의해 실행된다.

### Microtask Queue

Promise callback, `queueMicrotask`, `MutationObserver` 같은 작업이 대기하는 큐이다.

Microtask는 일반 task보다 우선순위가 높다.

## 실행 순서 예시

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve().then(() => {
  console.log("C");
});

console.log("D");
```

실행 결과는 다음과 같다.

```txt
A
D
C
B
```

이유는 다음과 같다.

1. 동기 코드인 `A`, `D`가 먼저 실행된다.
2. Promise callback은 Microtask Queue에 들어간다.
3. `setTimeout` callback은 Task Queue에 들어간다.
4. 현재 Call Stack이 비면 microtask가 먼저 실행된다.
5. 그 다음 task가 실행된다.

그래서 Promise가 `setTimeout`보다 먼저 실행된다.

## 렌더링과의 관계

브라우저는 JavaScript 실행, microtask 처리, 렌더링, task 처리를 조율한다.

무거운 동기 JavaScript가 오래 실행되면 Call Stack이 비지 않기 때문에 브라우저가 화면을 갱신할 기회를 얻기 어렵다.

또한 microtask가 너무 많이 쌓이면 다음 task뿐 아니라 렌더링 기회도 늦어질 수 있다.

즉 이벤트 루프를 이해할 때는 단순히 비동기 실행 순서만 보는 것이 아니라, JavaScript 작업이 화면 업데이트를 막을 수 있다는 점까지 함께 봐야 한다.

## 정리

이벤트 루프는 JavaScript의 동기 코드와 비동기 콜백이 어떤 순서로 실행될지 조율한다.

기본적으로 동기 코드가 먼저 실행되고, 이후 microtask가 task보다 먼저 처리된다. Promise callback이 `setTimeout`보다 먼저 실행되는 것이 대표적인 예시이다.

프론트엔드 성능 관점에서는 무거운 동기 작업이나 과도한 microtask가 렌더링을 지연시킬 수 있다는 점을 함께 이해하는 것이 중요하다.
