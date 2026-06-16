# Database Relationships

## 개념

관계형 데이터베이스에서는 테이블 사이의 관계를 foreign key로 표현한다.

대표적인 관계는 다음 세 가지다.

- 1:1
- 1:N
- N:M

관계 설계의 핵심은 어떤 테이블이 다른 테이블을 참조해야 하는지, 그리고 중복 없이 관계를 표현하려면 어떤 제약이 필요한지 판단하는 것이다.

## 1:1 관계

1:1 관계는 한 row가 다른 테이블의 row 하나와만 연결되는 관계다.

예시:

```txt
User 1명 <-> UserProfile 1개
```

테이블 구조는 다음처럼 만들 수 있다.

```txt
users
- id
- email

user_profiles
- id
- user_id
- address
```

`user_profiles.user_id`가 `users.id`를 참조하고, `user_id`에 unique 제약을 걸면 한 사용자에게 하나의 profile만 연결된다.

```sql
user_profiles.user_id UNIQUE
```

## 1:N 관계

1:N 관계는 한 row가 다른 테이블의 여러 row와 연결되는 관계다.

예시:

```txt
User 1명 -> Order 여러 개
```

테이블 구조는 다음과 같다.

```txt
users
- id
- email

orders
- id
- user_id
- total_price
```

foreign key는 보통 N 쪽 테이블에 둔다.

```txt
orders.user_id -> users.id
```

이유는 하나의 사용자가 여러 주문을 가질 수 있기 때문이다. 주문 row마다 자신이 어떤 사용자에게 속하는지 저장하면 1:N 관계를 자연스럽게 표현할 수 있다.

## N:M 관계

N:M 관계는 여러 row가 여러 row와 서로 연결되는 관계다.

예시:

```txt
Student 여러 명 <-> Course 여러 개
```

학생은 여러 강의를 들을 수 있고, 강의도 여러 학생을 가질 수 있다.

관계형 DB에서는 N:M을 직접 컬럼 하나로 표현하기보다 중간 테이블로 풀어낸다.

```txt
students
- id
- name

courses
- id
- title

student_courses
- student_id
- course_id
```

중간 테이블은 junction table, join table, mapping table이라고 부르기도 한다.

이 구조는 사실상 다음 두 개의 1:N 관계로 N:M을 표현한다.

```txt
Student 1명 -> student_courses 여러 row
Course 1개 -> student_courses 여러 row
```

## Order와 Product 예시

쇼핑몰에서 `Order`와 `Product`는 N:M 관계처럼 보인다.

```txt
Order 여러 개 <-> Product 여러 개
```

하나의 주문에는 여러 상품이 들어갈 수 있고, 하나의 상품은 여러 주문에 포함될 수 있다.

이 관계는 보통 `order_items` 같은 중간 테이블로 표현한다.

```txt
orders
- id
- user_id

products
- id
- name
- current_price

order_items
- order_id
- product_id
- quantity
- price_at_order
```

`quantity`는 해당 주문에서 상품을 몇 개 샀는지 저장한다.

`price_at_order`는 주문 당시 가격을 저장한다. 상품의 현재 가격은 나중에 바뀔 수 있으므로, 주문 이력을 정확히 남기려면 구매 당시 가격을 snapshot으로 저장해야 한다.

예를 들어 현재 상품 가격이 바뀌더라도 과거 주문 내역의 금액은 바뀌면 안 된다.

```txt
Product.current_price = 현재 가격
OrderItem.price_at_order = 주문 당시 가격
```

## 정리

1:1 관계는 foreign key와 unique 제약으로 표현할 수 있다.

1:N 관계에서는 보통 N 쪽 테이블이 1 쪽 테이블을 foreign key로 참조한다.

N:M 관계는 중간 테이블로 풀어내며, 중간 테이블에는 관계 자체의 속성도 함께 저장할 수 있다.

주문과 상품 관계에서 `order_items`가 필요한 이유는 단순히 두 테이블을 연결하기 위해서뿐 아니라, 수량과 주문 당시 가격처럼 관계에 속한 데이터를 저장하기 위해서다.
