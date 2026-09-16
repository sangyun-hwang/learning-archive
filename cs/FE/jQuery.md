# jQuery 기초: 레거시 코드 읽기

jQuery는 DOM 선택과 변경, 이벤트 처리, Ajax 요청을 간편하게 작성하도록 돕는 JavaScript 라이브러리입니다. 별도의 언어나 통신 방식이 아니며, 과거 브라우저 간 동작 차이를 줄이는 역할도 했습니다.

## 요소 선택과 DOM 변경

`$`는 일반적으로 `jQuery` 함수의 별칭입니다. CSS 선택자로 요소를 찾은 뒤 메서드를 호출합니다.

```js
// 일반 JavaScript
document.querySelector('#title').textContent = '제목';

// jQuery
$('#title').text('제목');
```

`querySelector()`는 DOM 요소 또는 `null`을 반환하지만, `$()`에 선택자를 전달하면 선택된 요소들을 감싼 jQuery 객체를 반환합니다. 일치하는 요소가 없으면 `.length`가 0입니다. `$('#title')[0]`으로 첫 번째 실제 DOM 요소를 꺼낼 수 있습니다.

| 코드 | 역할 |
| --- | --- |
| `$('#title').text('제목')` | 텍스트 변경 |
| `$('#name').val()` | 입력 요소의 값 읽기 |
| `$('#name').val('홍길동')` | 입력값 설정 |
| `$('.item').addClass('active')` | 선택한 요소들에 클래스 추가 |
| `$('#panel').hide()` | 요소 숨기기 |
| `$('#item').remove()` | 요소 제거 |

`.text()`는 텍스트로 처리하지만 `.html()`은 HTML을 해석합니다. 신뢰할 수 없는 사용자 입력을 `.html()`에 그대로 넣지 않습니다.

## DOM 준비 후 이벤트 등록

```js
$(function () {
  $('#save').on('click', function () {
    const name = $('#name').val();
    $('#result').text(name);
  });
});
```

`$(function () { ... })`은 HTML이 파싱되어 DOM을 사용할 준비가 된 뒤 실행됩니다. 버튼이 아직 만들어지지 않은 시점에 선택해서 이벤트 등록을 놓치는 문제를 피합니다. 이미지 등 모든 자원의 다운로드 완료까지 기다리는 것은 아닙니다.

## 직접 등록과 이벤트 위임

```js
// 등록 시점에 선택된 버튼에 직접 등록
$('.delete-button').on('click', handler);

// 이미 존재하는 부모에 위임
$('#list').on('click', '.delete-button', function () {
  $(this).closest('li').remove();
});
```

직접 등록한 경우 나중에 추가된 버튼에는 핸들러가 자동으로 붙지 않습니다. 위임한 경우 새 버튼의 클릭도 부모까지 버블링되므로, jQuery가 지정한 선택자에 해당하는 요소를 찾아 처리할 수 있습니다.

위 일반 함수의 `this`는 위임 선택자에 일치한 버튼 요소입니다. 새 버튼을 처리할 수 있는 이유는 `this`가 아니라 **부모에 등록한 핸들러와 이벤트 버블링**입니다. 화살표 함수로 바꾸면 `this`의 의미가 달라지므로 주의합니다.

## Ajax와 React 비교

레거시 코드의 `$.ajax()`, `$.get()`, `$.post()`는 서버에 HTTP 요청을 보내는 기능입니다. 요청 주소, 메서드, 전송 데이터, 성공과 실패 처리 부분을 구분해서 읽습니다.

- jQuery: 요소를 선택하고 텍스트, 클래스 등 DOM을 직접 변경합니다.
- React: state에 따른 UI를 선언하고 React가 DOM 변경을 반영합니다.

React가 관리하는 DOM을 jQuery로 별도로 변경하면 화면과 React의 관리 상태가 어긋날 수 있습니다. 두 기술을 함께 사용하는 경우 DOM 관리 범위를 구분합니다.

## 면접 요약

> jQuery는 DOM 조작, 이벤트 처리, Ajax 요청을 편리하게 작성하는 JavaScript 라이브러리입니다. 레거시 코드를 읽을 때는 선택자가 가리키는 요소와 수행하는 동작을 먼저 확인합니다. React가 상태에 따른 UI를 선언하는 방식이라면, jQuery는 요소를 찾아 직접 변경하는 방식이 중심입니다.

## 관련 문서

- [DOM](DOM.md)
- [이벤트 위임](EventDelegation.md)
- [jQuery 객체](https://learn.jquery.com/using-jquery-core/jquery-object/)
- [ready](https://api.jquery.com/ready/)
- [on](https://api.jquery.com/on/)
- [Ajax](https://api.jquery.com/jQuery.ajax/)
