# Forward Proxy와 Reverse Proxy

Proxy는 Client와 Server 사이에서 요청과 응답을 중계하는 Server다. Forward Proxy와 Reverse Proxy는 Network Traffic이 이동하는 방향보다 **누구를 대신하는가**를 기준으로 구분한다.

```text
Forward Proxy
-> Client를 대신해 외부 Server에 요청

Reverse Proxy
-> Backend Server를 대신해 외부 Client 요청 수신
```

## Forward Proxy

Forward Proxy는 Client 앞에 위치해 여러 Client를 대신하여 외부 Server에 요청한다.

```text
Client A ─┐
Client B ─┼─> Forward Proxy -> Internet -> Origin Server
Client C ─┘
```

Origin Server가 직접 연결된 상대는 Client가 아니라 Forward Proxy다.

```text
직접 요청
Client -> Origin Server

Forward Proxy 사용
Client -> Forward Proxy -> Origin Server
```

### 사용 사례

- 회사와 학교 Network의 인터넷 접근 통제
- 특정 Website와 Content 차단
- 외부로 나가는 요청 기록과 감사
- 동일 Resource Cache
- Origin Server에 Client 주소를 직접 노출하지 않음
- 특정 Network 위치를 통한 외부 요청

일반적으로 Browser, Operating System 또는 Client가 속한 Network에서 Forward Proxy 사용을 설정한다.

Forward Proxy가 있다고 항상 익명성이 보장되는 것은 아니다. Proxy가 원래 Client IP를 Header로 전달하거나 요청 내용을 기록할 수 있다.

## Forward Proxy와 VPN

둘 다 Traffic을 중간 Server로 전달할 수 있지만 적용 범위와 동작 계층이 다르다.

```text
Forward Proxy
-> 설정된 Application이나 Protocol의 요청 중계

VPN
-> Operating System의 Network Traffic을 가상 Tunnel로 전달
```

Forward Proxy와 VPN은 유사한 목적에 사용될 수 있지만 같은 기술은 아니다.

## Reverse Proxy

Reverse Proxy는 Backend Server 앞에 위치해 Client 요청을 대신 받는다.

```text
Browser
-> Reverse Proxy
   ├─> Next Server A
   ├─> Next Server B
   └─> Spring Server
```

Browser는 Reverse Proxy의 공개 주소만 사용한다.

```text
Browser가 아는 주소
-> https://example.com

내부 구조
-> Next Server :3000
-> Spring Server :8080
-> 여러 Container와 Instance
```

Reverse Proxy는 요청의 Path, Host와 정책을 확인해 적절한 내부 Server로 전달한다.

### 주요 역할

- 내부 Server 주소와 Port 은닉
- HTTPS와 TLS 인증서 처리
- Path 또는 Host 기반 Routing
- Load Balancing과 Health Check
- 정적 Resource와 응답 Cache
- 압축
- Rate Limiting
- 요청 크기와 Timeout 제한
- 비정상 요청 차단
- 공통 Log와 Monitoring

Nginx, HAProxy, Traefik과 Cloud Load Balancer 등이 Reverse Proxy 역할을 수행할 수 있다.

## Next.js와 Spring Routing

Nginx가 하나의 공개 Domain에서 Path에 따라 Next와 Spring으로 요청을 분리할 수 있다.

```text
Browser
-> https://example.com
-> Nginx Reverse Proxy

/          -> Next Server :3000
/api/*     -> Spring Server :8080
```

개념적인 Nginx 설정은 다음과 같다.

```nginx
location / {
  proxy_pass http://next:3000;
}

location /api/ {
  proxy_pass http://spring:8080;
}
```

Browser는 `next:3000`과 `spring:8080` 같은 내부 주소를 알 필요가 없다. 모든 요청을 `https://example.com`으로 보내고 Reverse Proxy가 실제 목적지를 선택한다.

`proxy_pass`의 경로 결합 규칙은 Slash 위치에 따라 달라질 수 있으므로 실제 Nginx 설정에서는 Backend에 전달되는 최종 URI를 확인해야 한다.

## Same Origin과 CORS

Origin은 다음 세 요소의 조합으로 판단한다.

```text
Scheme + Host + Port
```

Browser가 다음 주소를 호출한다고 가정한다.

```text
Page
-> https://example.com/products

API
-> https://example.com/api/products
```

Path는 다르지만 Scheme, Host와 Port가 같으므로 Browser 관점에서는 같은 Origin이다. Nginx가 내부에서 Next와 Spring의 서로 다른 Port로 전달하더라도 내부 주소는 Browser의 Origin 판단에 포함되지 않는다.

```text
Browser 관점
-> 같은 Origin

Reverse Proxy 내부
-> Next :3000과 Spring :8080으로 Routing
```

Proxy가 있다고 CORS가 자동으로 해결되는 것은 아니다. Browser가 `https://api.example.com`처럼 다른 Origin을 직접 호출하면 여전히 CORS 정책이 적용된다.

## TLS Termination

Reverse Proxy가 Browser와의 HTTPS 연결을 처리하고 복호화하는 것을 TLS Termination이라고 한다.

```text
Browser
-> HTTPS
-> Reverse Proxy에서 TLS 종료
-> HTTP 또는 HTTPS
-> Backend Server
```

### 장점

- 인증서를 한곳에서 관리
- Next와 Spring의 개별 인증서 관리 부담 감소
- 여러 Backend에 동일한 HTTPS 정책 적용
- 인증서 갱신과 암호화 정책 중앙화
- Backend는 Application Logic에 집중

Reverse Proxy와 Backend 사이를 HTTP로 사용할지는 Network의 신뢰 범위와 보안 요구에 따라 결정한다. 내부 Network도 신뢰할 수 없거나 규정상 End-to-End 암호화가 필요하다면 해당 구간도 HTTPS를 사용한다.

TLS가 Proxy에서 종료되면 Backend의 직접 연결은 HTTP일 수 있다. Backend가 원래 요청이 HTTPS였다는 사실을 알아야 Redirect와 Secure Cookie를 올바르게 처리할 수 있으므로 `Forwarded` 또는 `X-Forwarded-Proto` 같은 Header를 사용한다.

## Load Balancing

Reverse Proxy는 같은 Application의 여러 Instance에 요청을 분산할 수 있다.

```text
Browser
-> Reverse Proxy
   ├─> Next Instance A
   ├─> Next Instance B
   └─> Next Instance C
```

대표적인 분배 기준은 다음과 같다.

- Round Robin
- 현재 연결이 적은 Instance
- Client IP 기반
- Instance별 가중치
- Consistent Hashing

Health Check를 적용하면 장애가 발생한 Instance를 요청 대상에서 제외할 수 있다.

## 여러 Instance와 Session

Session 정보를 각 Instance Memory에만 저장하면 요청을 받은 Instance에 따라 인증 상태가 달라질 수 있다.

```text
첫 요청
-> Instance A
-> A Memory에 Session 저장

다음 요청
-> Instance B
-> B에는 Session 정보가 없음
-> 로그인 실패 또는 Session 재발급
```

해결 방법은 다음과 같다.

### 공유 Session Store

```text
Instance A ─┐
Instance B ─┼─> Redis 또는 Database
Instance C ─┘
```

모든 Instance가 동일한 Session 정보를 조회한다.

### Stateless Token

서버 Memory의 Session 대신 검증 가능한 Token에 필요한 인증 정보를 담을 수 있다. Token 폐기, 만료와 권한 변경 반영 방식은 별도로 설계해야 한다.

### Sticky Session

같은 Client 요청을 가능한 한 동일한 Instance로 보내는 방식이다. 구현은 간단할 수 있지만 해당 Instance 장애, Scaling과 부하 불균형에 취약할 수 있어 공유 상태를 완전히 대체하는 해법으로만 보기는 어렵다.

Cache, Upload 중간 상태와 WebSocket 연결도 여러 Instance 환경에서 공유 및 Routing 정책을 고려해야 한다.

## 전달 Header와 신뢰 경계

Reverse Proxy를 거치면 Backend가 직접 연결된 상대는 Browser가 아니라 Proxy다. 원래 요청 정보를 전달하기 위해 다음 Header를 사용한다.

```text
X-Forwarded-For
-> 원래 Client IP와 거쳐 온 Proxy Chain

X-Forwarded-Proto
-> 원래 Scheme, http 또는 https

X-Forwarded-Host
-> 원래 Host
```

표준화된 `Forwarded` Header도 있다.

```http
Forwarded: for=203.0.113.1;proto=https;host=example.com
```

외부 Client도 `X-Forwarded-For`를 직접 만들어 보낼 수 있으므로 Backend가 모든 값을 무조건 신뢰하면 IP 기반 접근 제어와 Rate Limit을 우회할 수 있다.

```text
Client가 조작한 Header
-> 신뢰하면 안 됨

신뢰하는 Reverse Proxy가 정리해 전달한 Header
-> 설정된 Proxy 범위 안에서 사용
```

Reverse Proxy는 외부에서 받은 위조 Header를 제거하거나 정책에 맞게 덮어쓰고, Backend는 요청이 신뢰할 수 있는 Proxy에서 왔을 때만 전달 Header를 해석해야 한다.

Backend Port를 외부 Internet에 함께 공개하면 공격자가 Reverse Proxy를 우회해 Rate Limit, TLS와 접근 통제를 건너뛸 수 있다. Firewall, Security Group과 사설 Network를 이용해 Backend가 신뢰하는 Proxy에서만 요청을 받도록 제한한다.

## Proxy Cache

Forward와 Reverse Proxy 모두 Cache를 사용할 수 있지만 목적이 다르다.

```text
Forward Proxy Cache
-> 여러 Client가 외부 Resource 공유

Reverse Proxy Cache
-> Origin 응답을 대신 제공해 Backend 부하 감소
```

Reverse Proxy에서 다음 응답을 Cache할 때는 주의해야 한다.

- 인증된 사용자별 응답
- 개인정보를 포함한 응답
- Cookie에 따라 달라지는 응답
- 언어와 압축 방식에 따라 달라지는 응답

`Cache-Control`, `Vary`, `private`와 인증 정책을 함께 구성해야 한다.

## Reverse Proxy와 BFF

Reverse Proxy와 Next BFF는 모두 요청 중간에 있지만 책임이 다르다.

```text
Reverse Proxy
-> 외부 요청의 Infra 진입점
-> TLS, Routing, Load Balancing, Cache와 공통 제한

Next BFF
-> 특정 Web UI에 맞는 Data와 인증 흐름
-> 여러 Backend 응답 조합
-> UI에 필요한 형태로 변환
```

```text
Browser
-> Nginx Reverse Proxy
-> Next BFF
-> Spring Backend
-> Database
```

Reverse Proxy에는 핵심 Domain Logic을 넣지 않고, Next BFF도 Spring이 소유한 인증·인가와 Business Rule을 대신한다고 가정하지 않는다.

## Proxy, Load Balancer와 Gateway

각 역할은 겹칠 수 있지만 중심 목적이 다르다.

| 종류 | 중심 역할 |
| --- | --- |
| Forward Proxy | Client 대신 외부 요청 |
| Reverse Proxy | Server 대신 외부 요청 수신 |
| Load Balancer | 여러 Server에 Traffic 분산 |
| API Gateway | API 인증, Rate Limit, 변환과 Routing |
| CDN | 가까운 Edge에서 Cache 응답 제공 |
| BFF | 특정 Frontend에 맞는 Backend 조합 |

하나의 제품이 여러 역할을 동시에 수행할 수 있다.

```text
Nginx
-> Reverse Proxy + Load Balancer + Cache

API Gateway
-> Reverse Proxy + 인증 + Rate Limit + API Routing
```

## Proxy 사용 시 보안

사용자가 전달한 임의 URL을 Server가 그대로 대신 호출하는 Open Proxy를 만들면 SSRF 취약점이 발생할 수 있다.

```text
위험
GET /proxy?url=http://내부관리서버
```

Proxy 대상은 허용된 Host와 Route로 제한하고 Redirect 이후의 최종 주소, 사설 IP와 Metadata Server 접근도 차단해야 한다.

Reverse Proxy는 다음 이유로 단일 장애 지점과 병목이 될 수 있다.

- Proxy 장애 시 전체 서비스 접근 불가
- 잘못된 Routing과 Rewrite 설정
- Request Body 크기 제한
- Proxy와 Backend Timeout 불일치
- WebSocket Upgrade Header 누락
- Connection과 CPU 병목
- Log에 Credential과 개인정보 저장

운영 환경에서는 Proxy 자체를 이중화하거나 Cloud Load Balancer와 관리형 Gateway를 사용하고, Log의 민감 정보를 Masking한다.

## 면접에서 설명하기

> Forward Proxy는 Client를 대신해 외부 Server에 요청하며 회사 Network의 접근 통제나 Client 주소 은닉에 사용됩니다. Reverse Proxy는 Backend Server를 대신해 외부 요청을 받고 TLS Termination, Routing, Load Balancing과 Cache를 담당합니다. 예를 들어 Nginx가 `example.com`의 `/` 요청은 Next로, `/api` 요청은 Spring으로 전달하면 Browser는 하나의 Origin만 사용하면서 내부 주소와 Port를 알 필요가 없습니다. 이때 전달 Header는 신뢰하는 Proxy에서 온 경우에만 사용하고, 여러 Instance의 Session은 Redis 같은 외부 저장소에서 공유할 수 있습니다.

## References

- [NGINX: Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [NGINX: HTTP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
- [MDN: Proxy servers and tunneling](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Proxy_servers_and_tunneling)
- [MDN: Forwarded](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Forwarded)
- [RFC 7239: Forwarded HTTP Extension](https://www.rfc-editor.org/rfc/rfc7239)

