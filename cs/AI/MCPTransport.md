# MCP Transport: stdio, SSE, Streamable HTTP

MCP(Model Context Protocol)에서 Transport는 MCP 클라이언트와 서버가 JSON-RPC 메시지를 어떤 통로로 주고받을지 정하는 방식이다.

도구 목록 조회, 도구 실행 요청과 결과 반환처럼 메시지가 담고 있는 의미는 JSON-RPC가 정의하고, `stdio`와 Streamable HTTP는 그 메시지를 전달하는 방법을 정의한다.

```text
AI 애플리케이션(MCP Client)
          <-> Transport
도구 제공 프로그램(MCP Server)
```

현재 MCP의 표준 Transport는 `stdio`와 Streamable HTTP다. 기존 HTTP+SSE 방식은 레거시 방식이며, Streamable HTTP가 이를 대체했다.

## stdio

`stdio`에서는 MCP 클라이언트가 MCP 서버를 로컬 자식 프로세스로 실행한다.

클라이언트는 서버의 표준 입력인 `stdin`으로 JSON-RPC 요청을 보내고, 서버의 표준 출력인 `stdout`에서 응답을 읽는다.

```text
MCP Client
  |-- 서버 프로세스 실행
  |-- stdin  -- 요청 --> MCP Server
  `-- stdout <-- 응답 -- MCP Server
```

설정은 다음과 같은 형태로 작성할 수 있다.

```json
{
  "command": "npx",
  "args": ["-y", "@example/mcp-server"]
}
```

이 방식은 별도의 서버 주소나 포트가 필요 없으며, 로컬 파일이나 개발 도구를 연결하기 편하다. 클라이언트가 서버 프로세스의 시작과 종료를 관리하는 형태가 일반적이다.

반면 하나의 서버를 여러 사용자가 공유하거나 원격에서 접속하는 용도에는 적합하지 않다.

### stdout과 stderr

`stdout`은 MCP JSON-RPC 메시지 전용으로 사용해야 한다. 일반 로그가 섞이면 클라이언트가 로그 문자열을 JSON-RPC 메시지로 해석하다가 파싱 오류를 일으킬 수 있다.

```text
stdout: MCP JSON-RPC 메시지
stderr: 로그와 디버깅 메시지
```

## 기존 HTTP+SSE

SSE(Server-Sent Events)는 서버가 HTTP 연결을 유지하면서 클라이언트로 이벤트를 계속 보내는 기술이다.

SSE는 기본적으로 서버에서 클라이언트로 데이터를 보내는 단방향 통신이므로, 기존 MCP의 HTTP+SSE 방식은 요청과 응답의 통로를 나눠 사용했다.

```text
요청: Client -- HTTP POST --> Server
응답: Client <-- SSE stream -- Server
```

SSE만으로는 클라이언트가 서버에 요청을 보낼 수 없다. 그러나 HTTP POST를 함께 사용하므로 MCP 전체로 보면 양방향 메시지 교환이 가능하다.

HTTP+SSE 서버는 클라이언트의 자식 프로세스가 아니라 독립된 네트워크 서버로 실행된다. 원격 접속, 여러 사용자와 중앙 배포에 적합하지만 다음과 같은 운영 책임이 추가된다.

- 서버 배포와 운영
- 사용자 인증과 권한 확인
- 세션 관리
- TLS와 네트워크 보안
- 연결 종료와 재연결 처리
- 여러 사용자의 요청 분리
- 장애 대응과 모니터링

## Streamable HTTP

Streamable HTTP는 기존 HTTP+SSE를 대체하는 현재의 표준 원격 Transport다.

기존 방식이 SSE와 POST 엔드포인트를 분리한 것과 달리, Streamable HTTP는 하나의 MCP 엔드포인트에서 HTTP POST와 GET을 처리한다.

```text
https://example.com/mcp
```

클라이언트는 POST로 요청을 보내고, 서버는 상황에 따라 일반 JSON 응답 또는 SSE 스트림으로 응답할 수 있다. 서버가 먼저 알림을 보내야 한다면 클라이언트가 GET 요청으로 SSE 스트림을 열 수도 있다.

따라서 Streamable HTTP가 SSE를 완전히 제거한 것은 아니다. 기존의 분리된 HTTP+SSE 전송 구조를 대체하면서, SSE는 여러 메시지나 진행 상황을 스트리밍해야 할 때 선택적으로 사용한다.

간단한 도구 실행 결과는 일반 JSON으로 한 번에 반환할 수 있으므로 항상 SSE를 사용할 필요는 없다.

## 비교

| 구분 | stdio | 기존 HTTP+SSE | Streamable HTTP |
| --- | --- | --- | --- |
| 실행 위치 | 주로 로컬 | 원격 가능 | 원격 가능 |
| 서버 실행 | 클라이언트가 자식 프로세스로 실행 | 독립 서버 | 독립 서버 |
| 클라이언트 요청 | `stdin` | HTTP POST | HTTP POST |
| 서버 응답 | `stdout` | 별도 SSE 연결 | JSON 또는 SSE |
| 엔드포인트 | 없음 | SSE와 POST 분리 | 하나의 MCP 엔드포인트 |
| 현재 위치 | 표준 | 레거시 | 표준 |
| 주요 용도 | 로컬 도구와 개인 개발 환경 | 기존 원격 서버 호환 | 원격 및 공용 서버 |

## 사용 사례에 따른 선택

로컬 파일을 읽거나 개인 개발 도구를 실행하는 MCP 서버에는 `stdio`가 적합하다. 별도 서버를 배포하지 않아도 되고 클라이언트가 필요한 서버 프로세스를 직접 실행할 수 있기 때문이다.

여러 직원이 함께 사용하는 사내 MCP 서버에는 Streamable HTTP가 적합하다. 각 사용자의 컴퓨터에서 서버를 따로 실행하지 않고, 중앙에 한 번 배포한 서버에 여러 클라이언트가 접속할 수 있기 때문이다. 대신 인증, 권한, 사용자별 요청 분리와 운영 환경을 갖춰야 한다.

## 보안

원격 MCP 서버를 인증 없이 공개하면 권한이 없는 사용자가 내부 데이터와 도구를 조회하거나 실행할 수 있다. 도구가 파일 수정, 메시지 전송이나 결제처럼 상태를 변경할 수 있다면 피해가 더 커질 수 있다.

원격 서버에는 인증과 인가, TLS, 요청 검증, 실행 권한 제한과 감사 로그가 필요하다. 로컬에서 실행하는 `stdio` 서버도 완전히 안전한 것은 아니다. 네트워크에 직접 공개되지는 않지만, MCP 서버 프로세스는 실행한 사용자의 권한으로 로컬 파일, 환경 변수와 명령어에 접근할 수 있기 때문이다.

신뢰할 수 없는 MCP 서버를 실행하면 악성 로컬 프로그램을 실행하는 것과 비슷한 위험이 생길 수 있으므로 서버의 출처와 부여된 권한을 확인해야 한다.

## 메시지 내용과 전달 방법

JSON-RPC와 Transport의 역할은 다음처럼 구분할 수 있다.

- JSON-RPC: 요청, 응답, 알림의 구조와 의미를 정의한다.
- Transport: JSON-RPC 메시지를 실제로 어떤 통로로 전달할지 정의한다.

같은 JSON-RPC 도구 실행 요청이라도 로컬에서는 `stdin`으로 전달하고, 원격 서버에는 HTTP POST로 전달할 수 있다.

## 정리

> `stdio`는 클라이언트가 로컬 MCP 서버 프로세스를 직접 실행해 표준 입출력으로 통신하는 방식이고, Streamable HTTP는 독립적으로 실행되는 원격 서버와 HTTP로 통신하며 필요하면 SSE 스트리밍을 사용하는 방식이다.

## 참고

- [MCP Transport Specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)
- [Legacy Transport Specification](https://modelcontextprotocol.io/specification/2024-11-05/basic/transports)
