# Ontology

## 개념

Ontology는 특정 도메인에서 사용하는 개념과 개념 사이의 관계를 명시적으로 구조화한 지식 모델이다.

단순한 카테고리 목록은 개념을 나열하는 데 그친다.

```txt
User
Order
Product
Payment
Delivery
```

Ontology는 여기서 한 단계 더 나아가 각 개념이 어떤 의미를 가지고, 서로 어떤 관계를 맺는지 정의한다.

```txt
User places Order
Order contains Product
Order has Payment
Order has Delivery
```

즉 Ontology는 “무엇이 있는가”뿐 아니라 “그것들이 어떻게 연결되는가”를 함께 표현한다.

## 구성 요소

Ontology는 보통 다음 요소로 구성된다.

### Class

도메인 안의 개념이나 분류다.

```txt
User
Order
Product
Payment
```

### Instance

Class에 속하는 실제 개체다.

```txt
User: sangyun
Product: MacBook
Order: order-123
```

### Relation

개념 사이의 관계다.

```txt
User places Order
Order contains Product
Order paidBy Payment
```

### Property

개념이나 개체가 가지는 속성이다.

```txt
User.email
Product.price
Order.createdAt
```

### Constraint

도메인 규칙이나 제약이다.

```txt
Order는 하나 이상의 Product를 포함해야 한다.
Payment는 하나의 Order와 연결된다.
```

## 쇼핑몰 도메인 예시

쇼핑몰 도메인에서 `User`, `Order`, `Product`를 단순히 나열하면 카테고리 목록에 가깝다.

Ontology로 표현하면 관계가 함께 드러난다.

```txt
User places Order
Order contains Product
Order connects User and Product
User pays for Order
```

이 구조를 통해 “사용자가 상품을 바로 소유한다”가 아니라, “사용자가 주문을 만들고 주문이 상품을 포함한다”는 도메인 의미를 표현할 수 있다.

## Ontology와 지식 그래프

Ontology는 지식 그래프를 설계할 때 중요한 기반이 될 수 있다.

지식 그래프는 개체와 관계를 graph 형태로 표현한다.

```txt
(User)-[places]->(Order)-[contains]->(Product)
```

Ontology는 이 graph에서 어떤 node와 edge가 가능한지, 각 관계가 어떤 의미를 가지는지 정의하는 역할을 한다.

## 어디에 사용되는가?

Ontology는 다음 영역에서 활용된다.

- 시맨틱 웹
- 지식 그래프
- 검색
- 추천 시스템
- 데이터 통합
- 도메인 모델링
- AI와 RAG의 지식 구조화

예를 들어 검색 시스템에서 단어만 비교하면 `order`, `payment`, `delivery`가 따로 분리된 텍스트처럼 보일 수 있다. Ontology를 사용하면 주문, 결제, 배송 사이의 관계를 바탕으로 더 의미 있는 검색이나 추론을 할 수 있다.

## RAG와 Ontology

RAG는 외부 문서를 검색해 LLM의 답변에 활용하는 방식이다.

단순히 chunk와 embedding만 사용하면 문서 사이의 의미 관계가 약하게 표현될 수 있다. Ontology나 지식 그래프를 함께 사용하면 개념 간 관계를 기준으로 검색 결과를 보강할 수 있다.

예를 들어 다음 질문이 있다고 하자.

```txt
주문 취소 시 환불과 배송 상태는 어떻게 처리돼?
```

Ontology가 있다면 `Order`, `Payment`, `Delivery`, `Refund`의 관계를 바탕으로 관련 문서를 더 구조적으로 찾을 수 있다.

## 정리

Ontology는 도메인의 개념과 관계를 구조화한 지식 모델이다.

카테고리 목록이 개념의 나열이라면, Ontology는 개념 사이의 관계, 속성, 제약까지 함께 표현한다.

이를 통해 검색, 추천, 지식 그래프, RAG 같은 시스템에서 단순 키워드보다 의미 중심으로 정보를 연결할 수 있다.
