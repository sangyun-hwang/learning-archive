# Keep-Alive

HTTP Keep-Alive는 하나의 network connection을 여러 HTTP request와 response에서 재사용하는 방식입니다. Persistent Connection이라고도 합니다.

## Keep-Alive가 없는 경우

```text
요청 1
-> TCP connection 생성
-> TLS handshake
-> HTTP request와 response
-> connection 종료

요청 2
-> connection 생성부터 다시 시작
```

Request마다 connection을 새로 만들면 TCP와 TLS handshake에 필요한 시간과 자원이 반복적으로 발생합니다.

## Keep-Alive를 사용하는 경우

```text
TCP/TLS connection 생성
-> HTTP request 1과 response 1
-> 같은 connection으로 request 2와 response 2
-> 일정 시간 사용하지 않으면 connection 종료
```

기존 connection을 재사용하면 다음 비용을 줄일 수 있습니다.

- TCP/TLS handshake 횟수
- 후속 request latency
- Client와 server의 CPU 및 network 비용

웹페이지는 HTML, JavaScript, CSS, image 등 여러 resource를 요청하므로 connection 재사용 효과가 큽니다.

## HTTP/1.0과 HTTP/1.1

HTTP/1.0은 request와 response가 끝나면 connection을 닫는 것이 기본이었습니다. Connection을 유지하려면 명시해야 했습니다.

```http
Connection: keep-alive
```

HTTP/1.1은 persistent connection이 기본입니다. Connection을 재사용하지 않으려면 다음과 같이 명시합니다.

```http
Connection: close
```

따라서 HTTP/1.1에서는 일반적으로 `Connection: keep-alive`를 직접 작성하지 않아도 됩니다.

## Keep-Alive Timeout

Server가 HTTP/1.x connection 유지 조건을 hint로 전달할 수 있습니다.

```http
Connection: keep-alive
Keep-Alive: timeout=5, max=100
```

- `timeout=5`: idle connection을 유지할 시간에 대한 hint
- `max=100`: 한 connection에서 처리할 request 수에 대한 hint

Keep-Alive connection은 영원히 유지되지 않습니다. Client, server 또는 proxy는 timeout이나 자원 상황에 따라 언제든 connection을 종료할 수 있으므로 설정된 시간 동안 유지된다고 보장할 수 없습니다.

Timeout이 너무 짧으면 connection 재사용 효과가 줄어들고, 너무 길면 idle connection이 server의 socket과 memory를 계속 점유할 수 있습니다.

## HTTP/2와 HTTP/3

HTTP/2와 HTTP/3도 connection을 재사용하며, 한 connection에서 여러 stream을 동시에 처리하는 multiplexing을 지원합니다.

```text
HTTP/1.1 Keep-Alive
-> 같은 connection을 여러 request에서 재사용

HTTP/2와 HTTP/3
-> connection 재사용
-> 여러 stream을 동시에 multiplexing
```

`Connection`과 `Keep-Alive`는 HTTP/1.x의 connection별 header이므로 HTTP/2와 HTTP/3에서는 사용하면 안 됩니다. HTTP/2는 TCP connection을 사용하고 HTTP/3는 QUIC connection을 사용합니다.

## HTTP는 계속 Stateless하다

Keep-Alive가 connection을 재사용해도 각 HTTP request는 독립적입니다.

```text
Connection
-> 재사용

각 HTTP request
-> 독립적으로 처리
```

로그인 상태는 Keep-Alive가 아니라 cookie, session, token 등으로 유지합니다. Connection이 종료되거나 새로 만들어져도 인증 정보가 request에 포함되면 로그인 상태를 유지할 수 있습니다.

## TCP Keepalive와 차이

```text
HTTP Keep-Alive
-> 여러 HTTP request에 connection 재사용
-> 성능 최적화

TCP Keepalive
-> 오랫동안 통신이 없는 상대가 살아 있는지 probe
-> 끊어진 connection 감지
```

이름은 비슷하지만 HTTP Keep-Alive는 connection 재사용, TCP Keepalive는 connection 상태 확인이 목적입니다.

## WebSocket과 차이

```text
HTTP Keep-Alive
-> 여러 HTTP request와 response에서 connection 재사용
-> request-response 구조 유지

WebSocket
-> connection을 계속 유지
-> client와 server가 양방향으로 message 전송
```

Keep-Alive는 기존 HTTP request-response 구조를 효율적으로 사용하는 기능이고 WebSocket은 지속적인 양방향 통신을 위한 protocol입니다.

## 면접 답변

> HTTP Keep-Alive는 하나의 connection을 여러 HTTP request와 response에서 재사용하는 방식입니다. 매번 TCP와 TLS handshake를 반복하지 않아 latency와 server 비용을 줄일 수 있습니다. HTTP/1.1에서는 persistent connection이 기본이고, HTTP/2와 HTTP/3는 connection 재사용과 함께 multiplexing도 지원합니다. Connection을 재사용하더라도 각 request는 독립적이므로 HTTP의 stateless 특성은 유지됩니다.

## 참고

- [RFC 9112: HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112.html)
- [MDN: Connection Management in HTTP/1.x](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Connection_management_in_HTTP_1.x)
- [MDN: Keep-Alive Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Keep-Alive)
