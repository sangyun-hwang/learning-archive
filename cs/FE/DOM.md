# DOM

## 개념

DOM(Document Object Model)은 HTML 문서를 JavaScript와 Browser가 읽고 변경할 수 있도록 **객체 기반의 Tree 구조로 표현한 Programming Interface**다.

```html
<body>
  <main>
    <h1>상품</h1>
    <button>구매</button>
  </main>
</body>
```

```text
Document
└─ html
   └─ body
      └─ main
         ├─ h1
         │  └─ Text: 상품
         └─ button
            └─ Text: 구매
```

HTML은 문서의 원본 문자열이고 DOM은 Browser가 HTML을 Parsing해 만든 현재 문서의 객체 표현이다. JavaScript로 DOM을 변경해도 Server에서 받은 원본 HTML 파일 자체가 수정되는 것은 아니다.

## DOM Node

DOM Tree의 각 항목을 Node라고 한다. 대표적인 Node에는 다음 항목이 있다.

- `Document`: DOM Tree의 진입점
- `Element`: `div`, `button`, `input` 같은 HTML 요소
- `Text`: Element 안의 문자열
- `Comment`: HTML 주석

Element도 Node의 한 종류다. 모든 Element는 Node지만 모든 Node가 Element인 것은 아니다. 예를 들어 Text Node에는 `querySelector()`나 `classList` 같은 Element 전용 기능이 없다.

## 탐색과 선택

```js
const title = document.querySelector('h1');
const buttons = document.querySelectorAll('button');
```

- `querySelector()`: 조건과 일치하는 첫 번째 Element 또는 `null` 반환
- `querySelectorAll()`: 일치하는 Element를 담은 정적인 `NodeList` 반환
- `getElementById()`: 일치하는 ID의 Element 또는 `null` 반환

DOM은 Tree이므로 부모, 자식과 형제 관계로도 탐색할 수 있다.

```js
element.parentElement;
element.children;
element.firstElementChild;
element.nextElementSibling;
```

`children`은 Element만 다루고 `childNodes`는 Text와 Comment를 포함한 모든 Node를 다룬다는 차이가 있다.

## 생성과 변경

```js
const item = document.createElement('li');
item.textContent = '새 상품';
item.classList.add('product-item');

const list = document.querySelector('#product-list');
list.append(item);
```

```js
item.setAttribute('data-product-id', '42');
item.remove();
```

JavaScript는 DOM API를 통해 Element를 생성하고 속성, Text와 Class를 변경하거나 Tree에서 제거할 수 있다.

사용자 입력처럼 신뢰할 수 없는 문자열을 표시할 때는 문자열을 HTML로 해석하는 `innerHTML`보다 Text로 처리하는 `textContent`가 기본적으로 안전하다. `innerHTML`이 항상 금지되는 것은 아니지만, 외부 입력을 검증 없이 넣으면 XSS로 이어질 수 있다.

## Attribute와 Property

HTML Attribute는 Markup에 작성된 초기 설정이고 DOM Property는 현재 객체가 가진 값이다.

```html
<input value="초기값" />
```

사용자가 입력값을 바꾸면 `input.value` Property는 현재 값으로 변경되지만, `getAttribute('value')`는 원래 Attribute 값을 유지할 수 있다. Attribute와 Property는 서로 연결되기도 하지만 항상 같은 의미와 값으로 유지되는 것은 아니다.

## DOM 변경과 화면 Rendering

DOM은 화면의 픽셀 그 자체가 아니다. Browser는 DOM과 CSSOM을 이용해 Render Tree를 만들고 Layout, Paint와 Composite 과정을 거쳐 화면을 표시한다.

```text
DOM 변경
-> Style과 Layout 계산이 필요할 수 있음
-> Paint
-> Composite
```

모든 DOM 변경이 항상 전체 Page의 Layout과 Paint를 발생시키는 것은 아니다. 변경한 내용과 CSS 속성, Browser의 최적화 방식에 따라 필요한 Rendering 작업이 달라진다. 반복문에서 DOM을 계속 읽고 수정하면 불필요한 계산이 발생할 수 있으므로 변경을 모아 처리하는 편이 유리할 수 있다.

## DOM과 Virtual DOM

DOM은 Browser가 실제 문서를 표현하고 화면 Rendering에 사용하는 구조다. Virtual DOM은 React가 UI 변경을 계산하기 위해 Memory에 만드는 JavaScript 표현이다.

```text
State 변경
-> React가 새로운 UI 결과 계산
-> 이전 결과와 비교
-> 필요한 변경만 실제 DOM에 반영
```

Virtual DOM이 실제 DOM을 대체하는 것은 아니다. 최종 화면을 변경하려면 React도 실제 DOM에 결과를 반영해야 한다.

## 핵심 정리

> DOM은 Browser가 HTML 문서를 객체 기반 Tree로 표현한 Programming Interface다. JavaScript는 DOM API로 Node를 탐색하고 생성·수정·삭제할 수 있다. DOM은 원본 HTML이나 화면 픽셀 자체와는 다르며, 변경 내용은 Browser의 Rendering 과정을 거쳐 화면에 반영된다.

## 참고 자료

- [MDN: Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
- [MDN: Node](https://developer.mozilla.org/en-US/docs/Web/API/Node)
- [MDN: Element](https://developer.mozilla.org/en-US/docs/Web/API/Element)
