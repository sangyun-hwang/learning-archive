# React Activity: 상태를 유지하며 화면과 Effect 관리하기

## 개념

Activity는 React 19.2에 추가된 컴포넌트다. UI를 숨길 때 state와 DOM을 보존하고 Effect를 정리하며, 다시 표시할 때 이전 상태를 유지한 채 Effect를 재연결한다. [React 19.2 발표](https://react.dev/blog/2025/10/01/react-19-2)

```jsx
import { Activity } from 'react';

<Activity mode={isOpen ? 'visible' : 'hidden'}>
  <Editor />
</Activity>
```

mode는 visible 또는 hidden이다. 생략하면 visible이다. 같은 Activity 경계가 유지되는 동안의 화면 전환을 다루는 기능이지, 새로고침 이후까지 상태를 저장하는 기능은 아니다.

## 기존 방식과 비교

| 방식 | DOM과 화면 | 내부 state | Effect |
| --- | --- | --- | --- |
| 조건부 렌더링으로 제거 | DOM 제거 | 제거 시 사라짐 | cleanup 실행 |
| CSS display: none | DOM 유지, 화면에서 숨김 | 유지 | 숨김만으로 정리되지 않음 |
| Activity hidden | DOM 유지, 화면에서 숨김 | 유지 | cleanup 실행, 표시할 때 setup 재실행 |

폼의 입력값을 로컬 state로 관리한다면 조건부 렌더링으로 제거했을 때 작성 내용이 사라질 수 있다. Activity로 숨기면 같은 경계 안의 상태를 보존할 수 있다. 상위에 별도로 저장한 값까지 조건부 렌더링으로 사라지는 것은 아니다.

## 관리자 페이지의 탭 예시

상품 검색과 주문 조회 탭이 있다고 가정한다. 상품 검색에는 검색어, 필터와 목록 상태가 있고 Effect로 데이터를 주기적으로 갱신한다.

```jsx
import { Activity, useState } from 'react';

function Dashboard() {
  const [tab, setTab] = useState('products');

  return (
    <>
      <button onClick={() => setTab('products')}>상품 검색</button>
      <button onClick={() => setTab('orders')}>주문 조회</button>

      <Activity mode={tab === 'products' ? 'visible' : 'hidden'}>
        <ProductSearch />
      </Activity>
      <Activity mode={tab === 'orders' ? 'visible' : 'hidden'}>
        <OrderSearch />
      </Activity>
    </>
  );
}
```

위 코드는 ProductSearch와 OrderSearch를 별도로 구현하는 부분 예제다. 주문 조회로 이동해도 상품 검색어와 필터를 남겨 두고, 상품 화면의 불필요한 자동 갱신은 정리하는 용도로 사용할 수 있다.

```text
상품 탭 표시
-> 검색어와 필터 선택
-> Effect에서 자동 갱신 시작

주문 탭으로 전환
-> 상품 탭의 state와 DOM 보존
-> 상품 탭 Effect의 cleanup 실행

상품 탭으로 복귀
-> 이전 검색 상태로 표시
-> Effect setup으로 조회와 자동 갱신 재시작
```

## 타이머와 구독은 cleanup으로 정리한다

아래는 refreshProducts가 현재 검색 조건으로 조회하는 안정적인 함수 참조라고 가정한 Effect 부분 예제다.

```jsx
useEffect(() => {
  refreshProducts();
  const timerId = setInterval(refreshProducts, 30_000);

  return () => clearInterval(timerId);
}, [refreshProducts]);
```

숨겨질 때 cleanup이 interval을 해제하고, 다시 보일 때 setup이 즉시 조회와 interval 등록을 수행한다. 숨기기 전에 시작된 비동기 요청은 이 cleanup만으로 취소되지 않으므로 요청 취소나 오래된 응답 무시가 필요할 수 있다.

빈 의존성 배열인 Effect도 Activity의 숨김·복원 과정에서는 cleanup과 setup이 실행될 수 있다. 따라서 '[]이면 영원히 한 번만 실행한다'고 설명하지 않는다.

Activity가 모든 외부 작업을 자동으로 멈춰 주는 것은 아니다. 타이머 해제, 구독 해지와 미디어 정지 등 필요한 작업은 cleanup에 구현해야 한다. [Activity 공식 문서](https://react.dev/reference/react/Activity)

## 언제 적합한가

| 화면 | 보존하고 싶은 상태 | 숨길 때 정리할 작업 |
| --- | --- | --- |
| 상품 검색 탭 | 검색어와 필터 | 주기적 갱신 |
| 실시간 차트 탭 | 선택 기간과 확대 상태 | 데이터 구독 |
| 설정 패널 | 펼친 항목과 작성 중인 값 | 화면 전용 이벤트 리스너 |
| 미리보기 탭 | 화면 설정 | 크기 측정과 관찰 작업 |

핵심은 돌아왔을 때 상태를 이어가되, 떠나 있는 동안 실행할 필요가 없는 Effect를 정리하는 것이다. 전체 앱이 계속 받아야 하는 알림이나 지속적인 저장 작업은 숨겨지는 영역 밖의 적절한 위치에서 관리한다.

닫으면 입력을 버려야 하는 화면은 조건부 렌더링이 자연스러울 수 있다. 숨겨진 컴포넌트도 메모리를 사용하고 새로운 props에 따른 렌더링이 낮은 우선순위로 처리될 수 있으므로 모든 화면을 무조건 Activity로 보관하지 않는다.

Activity 경계 자체를 제거하거나 컴포넌트의 key를 바꾸면 상태가 초기화될 수 있다. 페이지 새로고침 후에도 필요한 값은 저장소나 서버 등 별도 보관 방식을 사용한다.

## 말로 설명하기

Activity는 화면을 숨길 때 state와 DOM은 보존하고 Effect는 정리하는 React 컴포넌트입니다. 예를 들어 실시간 차트에서 선택한 기간과 확대 상태는 유지하면서 숨겨진 동안 구독을 해제하고, 돌아왔을 때 다시 연결할 수 있습니다. 조건부 렌더링은 제거 시 로컬 상태가 사라지고 CSS로만 숨기면 Effect는 계속 유지된다는 차이가 있습니다. 외부 작업 정지는 cleanup으로 구현해야 합니다.

## 관련 학습

- [React Effect](React.md)
- [Race Condition](RaceCondition.md)
- [AbortController](AbortController.md)
- [상태 끌어올리기](LiftingStateUp.md)

개념 설명용 예제이며 별도 Activity 애플리케이션 실행 테스트는 진행하지 않았다.
