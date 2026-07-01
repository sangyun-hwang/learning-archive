# Microservice Architecture

## 개념

MSA는 Microservice Architecture의 약자다. 하나의 큰 애플리케이션을 여러 작은 서비스로 나누고, 각 서비스를 독립적으로 개발, 배포, 운영하는 아키텍처다.

쇼핑몰 서비스를 예로 들면 monolith는 하나의 서버가 여러 기능을 함께 처리한다.

```txt
shopping-api
  user
  product
  order
  payment
  delivery
```

MSA에서는 기능을 도메인 경계에 따라 여러 서비스로 나눈다.

```txt
user-service
product-service
order-service
payment-service
delivery-service
```

각 서비스는 독립적인 애플리케이션으로 존재하고, 필요한 경우 네트워크를 통해 서로 통신한다.

## Modular Monolith와의 차이

배포 단위 관점에서 가장 큰 차이가 있다.

```txt
Modular Monolith:
하나의 애플리케이션으로 배포하고, 내부를 모듈로 나눈다.

MSA:
서비스별로 독립 배포한다.
```

Modular Monolith는 내부 코드 구조에서 도메인 경계를 지킨다. 반면 MSA는 서비스가 물리적으로 분리되어 있고, 각 서비스가 별도로 빌드되고 배포될 수 있다.

## 장점

### 독립 배포

특정 서비스만 수정하고 배포할 수 있다.

```txt
payment-service 변경
-> payment-service만 배포
```

전체 시스템을 다시 배포하지 않아도 되므로 변경 범위를 줄일 수 있다.

### 독립 확장

트래픽이 많은 서비스만 따로 확장할 수 있다.

```txt
product-service는 조회가 많음
-> product-service 인스턴스만 늘림
```

### 장애 격리

한 서비스에 장애가 발생해도 다른 서비스까지 반드시 모두 중단되는 것은 아니다. 장애를 해당 서비스 범위로 제한할 수 있다.

다만 실제로 장애 격리를 얻으려면 timeout, retry, circuit breaker, fallback 같은 설계가 함께 필요하다.

### 팀과 서비스 소유권 분리

서비스별로 담당 팀을 나눌 수 있다. 각 팀이 자신이 맡은 도메인의 개발과 운영 책임을 가진다.

### 기술 스택 선택

서비스별로 다른 언어, 프레임워크, DB를 선택할 수 있다.

단, 기술 스택이 너무 다양해지면 운영 복잡도가 증가할 수 있다.

## 단점과 복잡도

MSA는 독립 배포와 확장에 유리하지만, 시스템 전체 복잡도를 크게 높인다.

### 네트워크 통신

모노리스 내부 메서드 호출은 프로세스 내부 호출이다. MSA에서는 서비스 간 통신이 네트워크 호출이 된다.

```txt
order-service -> payment-service
```

네트워크는 실패할 수 있고, 지연될 수 있으며, timeout을 고려해야 한다.

### 분산 트랜잭션

주문 생성, 결제 승인, 재고 차감처럼 여러 서비스가 함께 처리해야 하는 작업은 단일 DB 트랜잭션처럼 단순하게 묶기 어렵다.

이 때문에 eventual consistency, Saga Pattern, Outbox Pattern 같은 설계가 필요할 수 있다.

### 데이터 소유권

MSA에서는 서비스가 자신의 데이터를 소유하는 방향이 권장된다.

```txt
user-service -> user DB 소유
order-service -> order DB 소유
payment-service -> payment DB 소유
```

다른 서비스의 DB를 직접 수정하면 서비스 경계가 깨진다. 서비스 간에는 API나 event를 통해 협력해야 한다.

### 모니터링과 추적

요청 하나가 여러 서비스를 거쳐 처리될 수 있다.

```txt
client
-> api-gateway
-> order-service
-> payment-service
-> notification-service
```

문제가 생겼을 때 어느 서비스에서 실패했는지 추적하려면 centralized logging, metrics, distributed tracing이 필요하다.

### 배포와 운영 복잡도

서비스 수가 늘어나면 배포 파이프라인, 환경 변수, secret, 모니터링, 장애 대응, 버전 호환성을 모두 관리해야 한다.

## 언제 적합한가?

MSA는 다음 상황에서 적합할 수 있다.

- 서비스 규모가 커지고 도메인 경계가 명확하다.
- 팀이 여러 개로 나뉘어 각 서비스의 소유권을 가져야 한다.
- 특정 기능만 독립적으로 배포하거나 확장할 필요가 있다.
- 장애를 서비스 단위로 격리해야 한다.
- 조직이 분산 시스템 운영 역량을 갖추고 있다.

반대로 작은 팀이나 초기 서비스에서는 MSA가 오히려 과한 복잡도를 만들 수 있다. 이런 경우 Modular Monolith로 도메인 경계를 먼저 정리한 뒤, 필요할 때 일부 서비스를 분리하는 방식이 현실적일 수 있다.

## 정리

MSA는 서비스를 도메인 단위로 나누고, 각 서비스를 독립적으로 배포하고 운영하는 아키텍처다.

장점은 독립 배포, 독립 확장, 장애 격리, 팀별 소유권이다.

하지만 네트워크 통신, 분산 트랜잭션, 데이터 일관성, 모니터링, 배포 운영 복잡도가 함께 증가한다.

핵심은 서비스를 작게 나누는 것 자체가 아니라, 도메인 경계와 운영 책임을 기준으로 독립 가능한 서비스로 나누는 것이다.
