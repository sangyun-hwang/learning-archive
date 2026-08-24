# Feature-Sliced Design

Feature-Sliced Design(FSD)은 Frontend code를 **업무 책임과 의존 방향**에 따라 나누는 architecture 방법론이다. React 전용 library나 고정된 folder template이 아니라, application이 커졌을 때 관련 code를 찾고 변경 범위를 통제하기 위한 기준을 제공한다.

FSD의 구조는 `Layer -> Slice -> Segment` 순서로 이해할 수 있다.

## Layer

Layer는 code가 가진 책임의 범위와 의존 방향을 나타낸다.

```text
app
pages
widgets
features
entities
shared
```

| Layer | 역할 | 예시 |
| --- | --- | --- |
| `app` | application 전체 설정 | Provider, Router, global style |
| `pages` | 하나의 완성된 화면 | 상품 상세, 장바구니 page |
| `widgets` | 독립적인 큰 UI 영역 | Header, ProductPurchasePanel |
| `features` | 사용자가 수행하는 기능 | 장바구니 추가, 로그인, 좋아요 |
| `entities` | 핵심 business 대상 | Product, User, Order |
| `shared` | domain과 무관한 공통 기반 | Button, API client, 날짜 처리 |

과거 명세에 있던 `processes` Layer는 현재 deprecated 상태다. 모든 Layer를 반드시 만들 필요는 없으며, 규모와 복잡도에 따라 필요한 구조만 사용한다.

## 의존 방향

상위 Layer는 자신보다 아래에 있는 Layer만 의존할 수 있다.

```text
pages -> widgets -> features -> entities -> shared
```

다음 의존은 허용할 수 있다.

```text
features/add-to-cart -> entities/product
entities/product -> shared/ui
```

반대 방향의 의존은 허용하지 않는다.

```text
entities/product -> features/add-to-cart // 허용하지 않음
```

`Product`는 여러 기능에서 사용할 수 있는 기본 domain이고, `AddToCart`는 Product를 이용하는 상위 사용 사례다. Entity가 Feature를 알게 되면 기본 domain이 특정 기능에 종속되어 다른 곳에서 독립적으로 사용하기 어려워진다.

## Slice

Slice는 Layer 안에서 code를 product와 business 의미로 나눈 단위다.

```text
features/
  add-to-cart/
  login/
  toggle-favorite/

entities/
  product/
  user/
  order/
```

같은 Layer의 Slice는 서로 독립적으로 유지한다.

```text
features/add-to-cart -> features/apply-coupon // 허용하지 않음
```

Feature끼리 직접 의존하면 한 기능의 변경이 다른 기능에 전파되고 독립적인 재사용과 제거가 어려워진다. 여러 Feature를 함께 사용해야 한다면 상위의 Widget이나 Page에서 조합한다.

```text
widgets/product-purchase-panel
  -> entities/product
  -> features/select-quantity
  -> features/add-to-cart
  -> features/apply-coupon
```

`app`과 `shared`는 business Slice로 나누지 않고 Segment를 직접 두는 것이 일반적이다. `app`은 모든 domain을 결합하는 영역이고 `shared`는 특정 domain을 가지지 않기 때문이다.

## Segment

Segment는 Slice 내부를 기술적인 목적에 따라 나눈 단위다.

```text
features/add-to-cart/
  ui/
  model/
  api/
  lib/
  index.ts
```

- `ui`: Component와 표시 logic
- `model`: state, schema와 business logic
- `api`: Server 통신
- `lib`: Slice 목적에 한정된 보조 logic
- `config`: 설정과 feature flag

`components`, `hooks`, `types`처럼 code의 종류만 표현하는 이름보다 그 code의 목적을 드러내는 이름을 사용한다. `shared/lib`도 모든 helper를 넣는 공간이 아니라 날짜, 문자열처럼 명확한 목적별로 관리한다.

## Public API

각 Slice는 `index.ts`를 통해 외부에 공개할 항목을 정의한다.

```ts
// 권장
import { AddToCartButton } from '@/features/add-to-cart';

// Slice 내부 구조에 직접 의존
import { AddToCartButton } from '@/features/add-to-cart/ui/AddToCartButton';
```

Public API를 사용하면 내부 folder를 변경해도 외부 module이 받는 영향을 줄이고, 사용할 수 있는 기능과 내부 구현의 경계를 분명하게 만들 수 있다.

Public API 규칙과 Layer 의존 규칙은 서로 다른 문제를 다룬다.

| 규칙 | 통제하는 것 |
| --- | --- |
| Layer 의존 규칙 | 어느 Layer와 Slice를 의존할 수 있는가 |
| Public API 규칙 | 허용된 Slice의 어느 진입점으로 접근하는가 |

따라서 `index.ts`를 거쳤더라도 `entity -> feature`나 같은 Layer의 `feature -> feature` 의존이 허용되는 것은 아니다.

## 분류 예시

```text
src/
  app/
    providers/
    styles/
  pages/
    product-details/
  widgets/
    product-purchase-panel/
  features/
    select-quantity/
    add-to-cart/
  entities/
    product/
  shared/
    ui/
    api/
    lib/
```

- `shared/ui/Button`: 특정 domain을 모르는 범용 Button
- `entities/product/ui/ProductPrice`: Product라는 domain을 표현하는 UI
- `features/add-to-cart/AddToCartButton`: 장바구니 추가라는 사용자 행동
- `widgets/product-purchase-panel`: 상품 정보와 여러 기능을 결합한 독립 영역
- `pages/product-details`: Widget과 다른 영역을 조합한 완성 화면

이름만으로 위치를 고정하지 않고 책임을 기준으로 판단한다. 단순한 상품 표시 UI라면 `entities/product/ui`에 둘 수 있지만, 상품 정보와 좋아요, 장바구니 추가 같은 기능을 결합한 `ProductCard`라면 Widget이 될 수 있다.

## Atomic Design과 차이

| 구분 | Atomic Design | FSD |
| --- | --- | --- |
| 중심 기준 | UI의 조합 단계 | Domain, 기능과 의존 방향 |
| 주요 관심사 | Design system과 Component 재사용 | Application module 경계 |
| 잘 설명하는 것 | Atom에서 완성 화면까지의 UI 구성 | Product, 인증, 결제 같은 business 구조 |

두 방법을 함께 사용한다면 FSD로 application의 큰 경계를 나누고, `shared/ui`나 각 Slice의 UI 내부에서 Atomic Design의 조합 원칙을 활용할 수 있다.

## 적용할 때 주의할 점

- 작은 project에 모든 Layer를 미리 만들지 않는다.
- 모든 사용자 동작을 Feature로 분리하지 않는다.
- 재사용되지 않는 UI는 Page 안에 두어도 된다.
- 파일 크기보다 business 책임과 변경 이유를 기준으로 분류한다.
- 같은 Layer의 Slice를 직접 의존시키지 않는다.
- Public API가 잘못된 Layer 의존을 허용해 주는 것은 아니다.

## 면접에서 설명하기

> FSD는 Frontend code를 App, Pages, Widgets, Features, Entities, Shared Layer로 나누고 의존성이 상위에서 하위로만 흐르도록 관리하는 architecture 방법론입니다. 각 Layer 안에서는 business 의미에 따라 Slice를 만들고, Slice 내부는 UI, Model, API 같은 Segment로 구성합니다. 같은 Layer의 Slice는 독립적으로 유지하고 외부에는 Public API만 공개해 변경 영향을 줄입니다. 모든 Layer를 형식적으로 만드는 것이 아니라 project 규모와 재사용 범위에 맞게 적용하는 것이 중요합니다.

## 참고

- [Feature-Sliced Design: Layers](https://feature-sliced.design/docs/reference/layers)
- [Feature-Sliced Design: Slices and Segments](https://feature-sliced.design/docs/reference/slices-segments)
- [Feature-Sliced Design: Public API](https://feature-sliced.design/docs/reference/public-api)
