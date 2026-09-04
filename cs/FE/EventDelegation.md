# Event Delegation

## 개념

이벤트 위임은 여러 자식 요소에 각각 Event Handler를 등록하는 대신, 공통 부모에 하나의 Handler를 등록해 자식에서 발생한 Event를 처리하는 방식이다.

대부분의 DOM Event는 실제 대상에서 시작해 부모 방향으로 전파된다. 이를 Event Bubbling이라고 한다.

```text
button
-> li
-> ul
-> body
-> document
```

부모는 Bubbling된 Event를 받아 어떤 자식에서 시작됐는지 확인할 수 있다.

```js
const menu = document.querySelector('#menu');

menu.addEventListener('click', (event) => {
  const button = event.target.closest('button[data-action]');

  if (!button || !menu.contains(button)) return;

  if (button.dataset.action === 'edit') {
    console.log('수정');
  }

  if (button.dataset.action === 'delete') {
    console.log('삭제');
  }
});
```

## `target`과 `currentTarget`

- `event.target`: Event가 실제로 시작된 요소
- `event.currentTarget`: 현재 Handler가 등록된 요소

버튼 안의 아이콘이나 `span`을 클릭하면 `target`은 해당 자식일 수 있다. `closest('button')`을 사용하면 Event가 시작된 위치에서 가장 가까운 처리 대상을 찾을 수 있다.

찾은 요소가 위임 영역 밖에 있지 않은지 `contains()`로 확인하면 Handler가 담당하는 범위를 명확히 할 수 있다.

## 동적으로 추가된 요소

부모 Handler는 등록 이후에 추가된 자식의 Event도 처리할 수 있다. 새 자식에서 발생한 Event도 동일한 부모까지 Bubbling되기 때문이다.

따라서 목록의 항목이 자주 추가되거나 제거되는 UI에서 각 항목의 Handler를 별도로 등록하고 해제할 필요를 줄일 수 있다.

## 장점

- 반복되는 자식 요소의 Event 처리를 한곳에서 관리할 수 있다.
- 동적으로 추가된 자식도 별도 Handler 등록 없이 처리할 수 있다.
- 목록처럼 같은 동작을 가진 요소가 많은 UI에 적용하기 좋다.

현대 Browser에서는 Handler 개수만으로 항상 성능 문제가 발생한다고 보기는 어렵다. 이벤트 위임의 주요 이점은 동적 요소와 반복되는 Event Logic을 공통 부모에서 관리하기 쉽다는 점이다.

## 주의점

- `stopPropagation()`이 호출되면 Event가 위임 Handler까지 도달하지 않을 수 있다.
- 모든 Event가 같은 방식으로 Bubbling되지는 않는다.
- `focus`, `blur` 대신 Bubbling되는 `focusin`, `focusout`을 사용할 수 있다.
- 모든 Handler를 `document`에 모으기보다 기능을 소유한 가까운 부모를 위임 지점으로 선택한다.
- `target`의 Tag 이름만 가정하지 말고 `closest()`와 식별용 `data-*` 속성 등을 활용한다.

## React에서의 이벤트 위임

React에서도 부모의 `onClick`에서 `event.target`을 확인해 여러 자식의 동작을 처리할 수 있다. 다만 일반적인 목록에서는 각 버튼에 명시적으로 `onClick`을 선언하는 방식도 충분히 자연스럽다.

복잡한 목록의 동작을 중앙화하거나 동적으로 생성되는 많은 항목을 공통 규칙으로 처리해야 할 때 명시적인 이벤트 위임을 고려한다.

## 핵심 정리

> 이벤트 위임은 Event Bubbling을 이용해 여러 자식 요소의 Event를 공통 부모의 하나의 Handler에서 처리하는 방식이다. `event.target`과 `closest()`로 실제 처리 대상을 찾으며, 동적으로 추가된 자식도 같은 Handler에서 처리할 수 있다.

## 참고 자료

- [MDN: Event bubbling](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Event_bubbling)
- [MDN: Event.target](https://developer.mozilla.org/en-US/docs/Web/API/Event/target)
- [MDN: Element.closest()](https://developer.mozilla.org/en-US/docs/Web/API/Element/closest)
