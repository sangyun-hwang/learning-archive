# Lifting State Up: 상태 끌어올리기

## 개념

여러 컴포넌트가 같은 상태를 사용하거나 함께 변경되어야 한다면, 그 상태를 가장 가까운 공통 부모로 옮겨 관리한다. 부모는 값을 props로 내려주고, 자식이 변경을 요청할 수 있도록 콜백도 전달한다.

핵심은 단순히 코드를 위로 옮기는 것이 아니라 **공유하는 상태의 원본을 한 곳에서 관리하는 것**이다. 모든 상태를 애플리케이션 최상위에 모으라는 뜻은 아니다.

## 언제 필요한가

상품 수량을 선택하는 QuantityInput과 결제 예상 금액을 표시하는 OrderSummary가 있다고 가정한다. 두 컴포넌트가 각각 quantity state를 가지면 입력창에서는 수량이 3인데 요약에는 1로 남는 문제가 생길 수 있다.

```text
ProductOrder: quantity 관리
  -> QuantityInput: quantity 표시, 변경 요청
  -> OrderSummary: quantity를 이용해 합계 표시
```

수량의 원본은 ProductOrder에 두고 두 자식이 같은 값을 받도록 구성한다.

## React 예제

아래 예제는 React 클라이언트 UI 코드다. Next.js App Router에서는 이 상호작용 영역을 Client Component로 구성한다.

```tsx
import { useState } from 'react';

type QuantityInputProps = {
  quantity: number;
  onQuantityChange: (quantity: number) => void;
};

function QuantityInput({ quantity, onQuantityChange }: QuantityInputProps) {
  return (
    <label>
      수량
      <select
        value={quantity}
        onChange={(event) => onQuantityChange(Number(event.target.value))}
      >
        {[1, 2, 3, 4, 5].map((value) => (
          <option key={value} value={value}>{value}</option>
        ))}
      </select>
    </label>
  );
}

function OrderSummary({ quantity, unitPrice }: {
  quantity: number;
  unitPrice: number;
}) {
  const total = quantity * unitPrice;
  return <p>예상 금액: {total}원</p>;
}

export default function ProductOrder() {
  const [quantity, setQuantity] = useState(1);

  return (
    <section>
      <QuantityInput quantity={quantity} onQuantityChange={setQuantity} />
      <OrderSummary quantity={quantity} unitPrice={10000} />
    </section>
  );
}
```

## 변경이 전달되는 흐름

1. 사용자가 QuantityInput에서 수량을 선택한다.
2. 자식이 부모에게 받은 onQuantityChange를 호출한다.
3. 연결된 setQuantity가 부모 상태의 갱신을 요청한다.
4. React가 부모를 다시 렌더링하며 새 quantity를 자식들에게 전달한다.
5. 입력값과 예상 금액이 같은 수량을 기준으로 표시된다.

자식이 props를 직접 수정하거나 부모 변수를 임의로 바꾸는 것은 아니다. 부모가 제공한 함수를 호출해 변경을 요청하며, 데이터는 다시 props를 통해 내려온다. 콜백을 전달한다고 React의 단방향 데이터 흐름이 양방향 바인딩으로 바뀌는 것은 아니다.

## 중복 state와 계산 가능한 값

QuantityInput에 `useState(quantity)`를 추가하면 부모 수량과 별개의 상태가 만들어진다. useState의 초기값은 이후 props 변경을 자동으로 따라가지 않는다. 편집 중 임시값처럼 별도 목적이 없다면 같은 값을 중복으로 보관하지 않는다.

### 리렌더링은 state 초기화가 아니다

```tsx
function Child({ quantity }: { quantity: number }) {
  const [localQuantity] = useState(quantity);
  return <p>{quantity} / {localQuantity}</p>;
}
```

| 상황 | quantity prop | localQuantity state | 표시 |
| --- | --- | --- | --- |
| quantity가 1일 때 최초 마운트 | 1 | 1 | 1 / 1 |
| 부모가 quantity를 5로 바꾸어 리렌더링 | 5 | 1 | 5 / 1 |

같은 컴포넌트가 유지되고 localQuantity를 갱신하지 않았다는 전제다. 자식 함수는 다시 실행되지만 React는 기존 state를 유지하며, useState의 초기값 인자를 다시 적용하지 않는다. 따라서 "자식이 리렌더링되면 props를 복사한 state도 동기화된다"는 설명은 틀리다.

같은 수량을 표시하는 목적이라면 localQuantity를 만들지 않고 quantity prop을 그대로 사용한다. 편집 후 저장하거나 취소할 수 있는 초안처럼 독립적인 값이 필요할 때만 별도 state를 두고, 초기화와 저장 시점을 명확히 정한다. 컴포넌트가 제거된 뒤 다시 마운트되거나 key가 바뀌어 새로 만들어지는 경우는 단순 리렌더링과 다르다.

예제의 total도 quantity와 unitPrice로 계산할 수 있으므로 별도 state나 Effect로 동기화하지 않는다. 실제 결제 금액은 서버가 상품 가격과 정책을 기준으로 다시 계산해야 하며, 화면의 예상 금액을 그대로 신뢰하지 않는다.

## Controlled Component 관점

QuantityInput은 수량을 자체 state로 소유하지 않고 부모의 quantity와 콜백으로 동작한다. 수량에 관해서는 부모가 제어하는 컴포넌트다. 다른 독립적인 UI 상태까지 반드시 부모로 옮길 필요는 없다.

## 어디까지 끌어올릴까

| 상황 | 선택 기준 |
| --- | --- |
| 한 컴포넌트에서만 필요한 상태 | 해당 컴포넌트의 로컬 state |
| 가까운 형제 컴포넌트가 공유 | 공통 부모로 상태 끌어올리기 |
| 여러 단계를 통과하는 props 전달이 복잡함 | 구성 변경이나 Context 검토 |
| 넓은 범위의 클라이언트 상태와 구독 관리 필요 | Zustand 같은 도구 검토 |
| 서버 데이터 조회와 캐시 관리 | TanStack Query 등 별도의 서버 상태 관리 검토 |

공유한다는 이유만으로 전역 Store가 필수는 아니다. 반대로 모든 상태를 최상위로 올리면 관계없는 영역까지 렌더링 작업이 늘 수 있다. 필요한 공통 부모에 두며, 부모가 다시 렌더링된다고 모든 DOM이 실제로 변경되는 것은 아니다.

Context는 값을 멀리 전달하는 수단이며, 그 자체가 어떤 상태를 누가 소유할지 결정해 주지는 않는다.

## 면접 요약

> Lifting State Up은 여러 컴포넌트가 공유할 상태를 가장 가까운 공통 부모에서 관리하는 방식입니다. 부모가 값과 변경 콜백을 내려주고 자식이 콜백으로 변경을 요청합니다. 같은 상태를 여러 곳에서 복사해 동기화하는 문제를 줄이며, 전역 상태 도구 없이도 가까운 컴포넌트 간 상태를 일관되게 유지할 수 있습니다.

## 관련 자료

- [React: Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
- [React: useState](https://react.dev/reference/react/useState)
- [클라이언트 상태와 서버 상태](StateManagement.md)
- [React 기초](React.md)
