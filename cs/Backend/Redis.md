# Redis

## 개념

Redis는 인메모리 기반 key-value 데이터 저장소다.

일반적인 관계형 데이터베이스는 데이터를 주로 디스크에 저장하고, 필요한 데이터를 읽어와 처리한다. Redis는 데이터를 메모리에 올려두고 접근하기 때문에 읽기와 쓰기가 매우 빠르다.

```txt
key -> value
```

예시:

```txt
user:1:name -> "Kim"
session:abc123 -> "{ userId: 1 }"
product:100:stock -> "12"
```

Redis는 단순한 문자열 저장소가 아니라 다양한 자료구조를 제공하는 인메모리 데이터 구조 서버에 가깝다.

## Redis가 빠른 이유

Redis가 빠른 가장 큰 이유는 데이터를 메모리에서 처리하기 때문이다.

디스크 접근은 메모리 접근보다 훨씬 느리다. Redis는 자주 사용하는 데이터를 메모리에 저장해 디스크 기반 DB보다 빠르게 접근할 수 있다.

또한 Redis는 key-value 구조와 다양한 자료구조 명령을 제공해, 특정 작업을 애플리케이션에서 직접 구현하지 않고 Redis 명령으로 빠르게 처리할 수 있다.

## 주요 자료구조

Redis는 다음 자료구조를 지원한다.

- String
- Hash
- List
- Set
- Sorted Set
- Stream
- Bitmap
- HyperLogLog

자료구조를 잘 선택하면 cache뿐 아니라 ranking, queue, pub/sub, rate limiting 같은 기능도 구현할 수 있다.

## Cache로 사용하는 흐름

Redis는 자주 조회되는 데이터를 cache하는 데 많이 사용된다.

흐름은 다음과 같다.

```txt
1. 클라이언트가 상품 정보를 요청한다.
2. 서버가 Redis에 해당 상품 정보가 있는지 확인한다.
3. Redis에 있으면 DB를 조회하지 않고 바로 반환한다.
4. Redis에 없으면 DB에서 조회한다.
5. DB 조회 결과를 Redis에 저장한다.
6. 클라이언트에게 응답한다.
```

이를 cache-aside pattern이라고도 한다.

```txt
Cache hit
-> Redis에서 바로 반환

Cache miss
-> DB 조회
-> Redis에 저장
-> 응답
```

이 구조를 사용하면 반복되는 DB 조회를 줄이고 응답 속도를 높일 수 있다.

## 대표 사용 사례

### Cache

자주 조회되는 데이터를 Redis에 저장해 DB 부하를 줄인다.

### Session Store

로그인 세션 정보를 Redis에 저장할 수 있다. 여러 서버 인스턴스가 같은 Redis를 바라보면 세션 공유가 쉬워진다.

### Rate Limiting

특정 사용자나 IP의 요청 횟수를 Redis에 저장해 일정 시간 동안 요청 수를 제한할 수 있다.

### Pub/Sub

Redis의 publish/subscribe 기능을 사용해 메시지를 발행하고 구독할 수 있다.

### Queue

List나 Stream을 사용해 간단한 작업 queue를 만들 수 있다.

### Ranking

Sorted Set을 사용해 점수 기반 ranking을 만들 수 있다.

## 메모리 기반이라 조심할 점

Redis는 빠르지만 메모리 기반이라는 특성 때문에 주의가 필요하다.

### 용량 관리

메모리는 디스크보다 비싸고 용량이 작다. 모든 데이터를 Redis에 넣기보다 자주 조회되거나 빠른 접근이 필요한 데이터를 선별해야 한다.

### 만료 정책

cache 데이터는 TTL을 설정해 오래된 값이 계속 남아 있지 않도록 관리해야 한다.

```txt
SET product:100 "{...}" EX 60
```

### Eviction Policy

메모리가 가득 찼을 때 어떤 데이터를 제거할지 정책을 정해야 한다.

예시:

- LRU 기반 제거
- TTL이 있는 key 중 제거
- 쓰기 거부

### 영속성

Redis는 메모리 기반이지만 RDB snapshot, AOF 같은 영속화 옵션을 제공한다. 다만 Redis를 primary DB처럼 사용할지, cache로 사용할지에 따라 영속화 전략이 달라진다.

cache로 사용하는 경우 Redis 데이터가 사라져도 DB에서 다시 채울 수 있어야 한다.

### Cache Invalidation

DB의 원본 데이터가 바뀌었는데 Redis cache가 오래된 값을 가지고 있으면 문제가 생길 수 있다. 데이터 변경 시 cache를 삭제하거나 갱신하는 전략이 필요하다.

## 정리

Redis는 메모리에 데이터를 저장해 빠른 접근을 제공하는 key-value 기반 데이터 저장소다.

cache, session, rate limiting, pub/sub, queue, ranking처럼 빠른 읽기와 쓰기가 필요한 곳에서 자주 사용된다.

다만 메모리 용량, TTL, eviction policy, 영속화, cache invalidation을 함께 고려해야 안정적으로 사용할 수 있다.
