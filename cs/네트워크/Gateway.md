# Gateway

## 개념

Gateway는 서로 다른 네트워크나 시스템 사이에서 요청이 지나가는 진입점 역할을 한다.

웹 서비스에서 Gateway는 크게 두 관점으로 볼 수 있다.

- Network Gateway
- API Gateway

둘 다 “관문”이라는 의미를 가지지만, 다루는 계층과 역할이 다르다.

## Network Gateway

Network Gateway는 서로 다른 네트워크를 연결하는 출입구다.

예를 들어 집이나 회사 내부망에서 인터넷으로 나갈 때 보통 공유기나 라우터를 거친다.

```txt
내 컴퓨터
-> 공유기 / 라우터
-> 인터넷
```

이때 공유기나 라우터는 내부 네트워크에서 외부 네트워크로 나가기 위한 gateway 역할을 한다.

Network Gateway는 다음처럼 설명할 수 있다.

```txt
다른 네트워크로 나가기 위한 관문
```

대표 예시는 다음과 같다.

- Default Gateway
- Internet Gateway
- NAT Gateway
- VPN Gateway

## API Gateway

API Gateway는 클라이언트와 여러 백엔드 서비스 사이의 API 진입점이다.

예를 들어 백엔드 서비스가 다음처럼 나뉘어 있다고 하자.

```txt
user-service
order-service
payment-service
product-service
```

클라이언트가 각 서비스의 주소와 인증 방식, 요청 정책을 모두 직접 알면 복잡해진다.

API Gateway를 두면 클라이언트는 하나의 진입점만 바라볼 수 있다.

```txt
Client
-> API Gateway
-> user-service
-> order-service
-> payment-service
```

API Gateway는 클라이언트 요청을 받아 적절한 내부 서비스로 전달한다.

## API Gateway의 역할

API Gateway는 단순히 요청을 전달하는 reverse proxy 역할만 하는 것이 아니다.

대표적으로 다음 역할을 맡을 수 있다.

### Routing

요청 경로나 method를 보고 적절한 내부 서비스로 보낸다.

```txt
/api/users -> user-service
/api/orders -> order-service
/api/payments -> payment-service
```

### Authentication

사용자가 로그인했는지 확인한다.

### Authorization

사용자가 해당 API를 호출할 권한이 있는지 확인한다.

### Rate Limiting

사용자나 IP가 너무 많은 요청을 보내지 못하게 제한한다.

### Logging and Monitoring

요청 로그, 응답 시간, 오류율 등을 수집한다.

### Request / Response Transformation

클라이언트 요청이나 백엔드 응답 형식을 변환할 수 있다.

### Load Balancing

여러 백엔드 인스턴스 중 적절한 곳으로 요청을 분산할 수 있다.

### Timeout, Retry, Circuit Breaker

백엔드 서비스가 느리거나 실패할 때 timeout, retry, circuit breaker 정책을 적용할 수 있다.

## Gateway가 있으면 편한 점

API Gateway가 인증, 인가, rate limiting 같은 공통 관심사를 처리하면 백엔드 서비스는 자신의 비즈니스 로직에 더 집중할 수 있다.

클라이언트 입장에서도 여러 서비스 주소와 정책을 직접 관리하지 않고 Gateway 하나를 기준으로 API를 호출할 수 있다.

```txt
Client:
Gateway 하나만 알면 됨

Backend Service:
비즈니스 책임에 더 집중 가능
공통 보안, 제한, 로깅 정책을 Gateway에서 일관되게 처리 가능
```

## Network Gateway와 API Gateway 차이

| 구분 | Network Gateway | API Gateway |
| --- | --- | --- |
| 관점 | 네트워크 연결 | 애플리케이션 API |
| 역할 | 다른 네트워크로 나가는 출입구 | 클라이언트와 백엔드 서비스 사이의 진입점 |
| 예시 | Default Gateway, NAT Gateway, VPN Gateway | AWS API Gateway, Kong, NGINX, Spring Cloud Gateway |
| 주요 기능 | 라우팅, NAT, 네트워크 연결 | API routing, auth, rate limit, logging |

짧게 말하면 다음과 같다.

```txt
Network Gateway는 네트워크 사이의 출입구
API Gateway는 클라이언트와 백엔드 서비스 사이의 API 진입점
```

## 주의할 점

API Gateway에 너무 많은 책임을 넣으면 Gateway가 병목이나 단일 장애 지점이 될 수 있다.

또한 비즈니스 로직이 Gateway에 과도하게 들어가면 각 서비스의 책임이 흐려질 수 있다.

Gateway는 공통 관심사를 처리하되, 핵심 도메인 로직은 각 백엔드 서비스가 담당하는 것이 좋다.

## 정리

Gateway는 시스템 사이의 관문이다.

Network Gateway는 네트워크 사이의 출입구이고, API Gateway는 클라이언트와 백엔드 서비스 사이의 API 진입점이다.

API Gateway는 routing뿐 아니라 인증, 인가, rate limiting, logging, monitoring, request/response 변환 같은 공통 관심사를 처리할 수 있다.
