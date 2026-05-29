# CSRF

## 개념

CSRF는 Cross-Site Request Forgery의 약자다. 사용자가 로그인한 상태를 악용해, 사용자가 의도하지 않은 상태 변경 요청을 다른 사이트에서 보내게 만드는 공격이다.

중요한 점은 공격자가 쿠키 값을 직접 훔치는 공격이 아니라는 것이다.

```txt
공격자가 쿠키를 읽는 것 X
브라우저가 쿠키를 자동 첨부하는 특성을 악용 O
```

브라우저는 특정 도메인으로 요청을 보낼 때 해당 도메인의 쿠키를 자동으로 함께 보낼 수 있다. 공격자는 이 동작을 이용해 사용자의 로그인 상태로 요청이 전송되도록 유도한다.

## 공격 흐름

사용자가 `bank.com`에 로그인해 세션 쿠키를 가진 상태라고 가정한다.

공격자는 다른 사이트에 다음과 같은 form을 숨겨둘 수 있다.

```html
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="attacker" />
  <input name="amount" value="100000" />
</form>
<script>
  document.forms[0].submit();
</script>
```

사용자가 이 공격 사이트에 접속하면 브라우저는 `bank.com`으로 요청을 보낸다. 이때 `bank.com`에 대한 세션 쿠키가 자동으로 포함될 수 있다.

서버가 쿠키만 보고 요청을 신뢰하면 사용자가 의도하지 않은 송금 요청이 처리될 수 있다.

## CSRF와 XSS의 차이

CSRF는 다른 사이트에서 사용자의 브라우저를 이용해 요청을 보내게 만드는 공격이다.

```txt
공격자 사이트
-> 사용자의 브라우저가 대상 사이트로 요청
-> 쿠키가 자동 첨부됨
```

XSS는 공격자가 대상 사이트 안에서 스크립트를 실행하는 공격이다.

```txt
대상 사이트 내부에서 악성 JavaScript 실행
-> DOM, token, 사용자 입력 조작 가능
```

따라서 XSS가 성공하면 CSRF Token도 읽거나 요청에 포함시킬 수 있다. XSS는 일반적인 CSRF보다 더 강한 공격 권한을 갖는 상황이다.

## CSRF Token

CSRF Token은 서버가 발급하는 예측 불가능한 값이다. 서버는 상태 변경 요청이 들어왔을 때 세션 쿠키뿐 아니라 CSRF Token도 함께 검증한다.

정상 흐름은 다음과 같다.

```txt
1. 사용자가 정상 사이트 페이지에 접속
2. 서버가 CSRF Token을 발급
3. 페이지 또는 JavaScript가 요청에 token을 포함
4. 서버가 세션 쿠키와 CSRF Token을 함께 검증
5. 검증에 성공하면 요청 처리
```

공격 사이트는 사용자의 브라우저로 요청을 보내게 만들 수는 있지만, Same-Origin Policy 때문에 정상 사이트의 HTML이나 API 응답을 마음대로 읽기 어렵다.

즉 공격자는 쿠키가 자동으로 붙은 요청은 유도할 수 있지만, 정상 사이트가 발급한 CSRF Token 값을 알아내서 요청에 포함시키기는 어렵다.

```txt
세션 쿠키 있음
CSRF Token 없음 또는 틀림
-> 서버가 거부
```

## Token 저장 위치

CSRF Token은 구현 방식에 따라 여러 위치에 있을 수 있다.

- 서버 렌더링 HTML의 hidden input
- meta tag
- JavaScript runtime state
- CSRF 전용 cookie

예시:

```html
<form method="POST" action="/transfer">
  <input type="hidden" name="csrf_token" value="random-token-value" />
  <button type="submit">Send</button>
</form>
```

핵심은 token의 저장 위치 자체가 아니라, 공격자가 다른 origin에서 token 값을 읽거나 예측하기 어렵고 서버가 이를 검증한다는 점이다.

## 구현 방식이 공개되어도 안전한 이유

CSRF Token은 “어떤 방식으로 만들었는지”가 비밀이라서 안전한 것이 아니다.

안전해야 하는 것은 실제 token 값이다.

```txt
token 생성 방식은 공개되어도 됨
실제 token 값은 예측 불가능해야 함
```

공격자가 “서버가 랜덤 256비트 token을 만든다”는 사실을 알아도, 특정 사용자에게 발급된 실제 token 값을 모르면 요청에 넣을 수 없다.

## 뚫릴 수 있는 경우

CSRF Token도 구현이 잘못되면 우회될 수 있다.

- token 값이 예측 가능할 때
- token이 사용자 세션과 연결되어 있지 않을 때
- 상태 변경 API 일부에서 token 검증을 빠뜨렸을 때
- XSS로 대상 페이지 내부의 token을 읽을 수 있을 때
- CORS 설정 오류로 공격 사이트가 응답을 읽을 수 있을 때
- Double Submit Cookie를 세션 바인딩 없이 약하게 구현했을 때

## 다른 방어 방법

CSRF 방어는 CSRF Token만으로 끝나지 않는다.

- `SameSite` cookie 설정
- Origin / Referer header 검증
- 중요한 요청은 GET이 아니라 POST, PUT, DELETE 사용
- 위험한 작업은 재인증 또는 사용자 확인
- CORS를 필요한 origin으로만 제한

## 정리

CSRF는 공격자가 쿠키를 훔치는 공격이 아니라, 브라우저가 대상 사이트 쿠키를 자동으로 첨부하는 특성을 악용해 사용자가 의도하지 않은 요청을 보내게 만드는 공격이다.

CSRF Token은 쿠키만 자동으로 붙은 요청을 거부하기 위해, 공격자가 알기 어려운 별도 값을 요청에 포함하도록 만드는 방어 방식이다.

다만 XSS처럼 대상 사이트 자체에서 스크립트를 실행할 수 있는 공격이 있으면 CSRF Token도 우회될 수 있으므로, XSS 방어와 함께 고려해야 한다.
