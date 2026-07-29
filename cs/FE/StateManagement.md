# State Management

## Client State와 Server State

상태가 저장된 위치보다 누가 원본을 소유하는지를 기준으로 구분합니다.

### Client State

Application이 직접 만들고 소유하는 상태입니다.

```text
Modal 열림 여부
선택된 Tab
Sidebar 접힘 여부
작성 중인 Form
Theme
```

Component 하나에서만 사용하는 상태는 `useState`로 관리할 수 있습니다. 여러 Component가 공유하거나 props drilling이 커지는 client state에는 Zustand 같은 상태 관리 도구를 사용할 수 있습니다.

### Server State

Server가 원본을 소유하고 client는 가져온 값을 snapshot 또는 cache로 보관합니다.

```text
사용자 Profile
상품 목록
게시글과 댓글
주문 내역
```

Server state에는 비동기 조회와 변경 외에도 loading, error, cache, stale 여부, refetch, retry와 같은 lifecycle 관리가 필요합니다.

## Zustand

Zustand는 application이 소유하는 공유 client state를 관리하는 범용 store입니다.

```tsx
import { create } from 'zustand';

const useUIStore = create((set) => ({
  isSidebarOpen: true,
  toggleSidebar: () =>
    set((state) => ({
      isSidebarOpen: !state.isSidebarOpen,
    })),
}));
```

주로 다음 상태에 적합합니다.

- 여러 Component가 공유하는 UI 상태
- Client가 직접 변경하고 소유하는 동기 상태
- 전역에서 접근해야 하는 상태와 action

Zustand에도 API response를 저장할 수 있지만 loading, error, cache 만료, refetch와 server 동기화 시점을 직접 구현해야 합니다.

모든 client state를 Zustand에 넣을 필요는 없습니다. 한 Component나 가까운 부모와 자식만 사용하는 상태는 `useState`로 충분할 수 있습니다.

## TanStack Query

TanStack Query는 server state를 조회하고 동기화하기 위한 도구입니다. React Query라는 이름으로도 알려져 있습니다.

```tsx
const { data, isPending, error } = useQuery({
  queryKey: ['products'],
  queryFn: getProducts,
});
```

다음과 같은 server state lifecycle을 관리합니다.

- Query 결과 cache
- Loading과 error 상태
- 같은 query의 중복 request 감소
- `staleTime`을 통한 freshness 판단
- Background refetch와 retry
- Mutation 이후 query invalidation

### queryKey

`queryKey`는 어떤 데이터를 같은 query와 cache로 관리할지 식별합니다.

```tsx
// 전체 상품 목록
queryKey: ['products'];

// 상품 10의 상세 정보
queryKey: ['product', 10];

// Category별 상품 목록
queryKey: ['products', { category: 'book' }];
```

서로 다른 데이터에 같은 key를 사용하면 cache가 충돌하거나 잘못된 데이터가 표시될 수 있습니다.

### Mutation과 Invalidation

Mutation이 성공해 server 원본이 변경되어도 query cache에는 이전 값이 남아 있을 수 있습니다.

```tsx
const queryClient = useQueryClient();

const mutation = useMutation({
  mutationFn: updateProduct,
  onSuccess: () => {
    queryClient.invalidateQueries({
      queryKey: ['products'],
    });
  },
});
```

`invalidateQueries`는 관련 query를 stale 상태로 표시합니다. 현재 사용 중인 query는 일반적으로 다시 조회되어 server의 최신 상태를 반영합니다.

TanStack Query가 server에서 발생한 모든 변경을 자동으로 아는 것은 아닙니다. `queryKey`, `staleTime`, refetch 조건과 mutation 이후 invalidation 등을 통해 동기화 기준을 설정해야 합니다.

## Zustand와 TanStack Query 비교

| 구분 | Zustand | TanStack Query |
| --- | --- | --- |
| 대상 | Client state | Server state |
| 원본 소유자 | Client application | Remote server |
| 대표 데이터 | Modal, Tab, UI 설정 | 상품, 사용자, 주문 |
| 비동기 요청 | 직접 구현 | Query와 Mutation 제공 |
| Cache와 freshness | 직접 관리 | 관련 기능 제공 |
| Refetch와 retry | 직접 구현 | 관련 기능 제공 |

두 도구는 경쟁 관계가 아니라 서로 다른 상태를 담당합니다.

```text
Zustand
-> 장바구니 Drawer가 열려 있는가?

TanStack Query
-> Server에 저장된 장바구니 상품은 무엇인가?
```

## 면접 답변

> Client state는 Modal이나 선택된 Tab처럼 application이 원본을 소유하는 상태이고, server state는 상품이나 사용자 정보처럼 server가 원본을 소유하며 client에는 cache된 snapshot만 존재하는 상태입니다. Zustand는 공유 client state에 적합하고, TanStack Query는 server state의 fetching, cache, stale 처리, refetch와 mutation을 관리하는 데 적합합니다. 두 도구는 경쟁 관계가 아니라 함께 사용할 수 있으며, Component 내부의 단순한 상태는 `useState`로 관리할 수 있습니다.

## Redux

Redux는 전역 상태를 예측 가능한 방식으로 관리하기 위한 패턴과 라이브러리입니다.

- store: 애플리케이션 상태를 보관합니다.
- action: 상태 변경 의도를 표현합니다.
- reducer: 현재 상태와 action을 받아 다음 상태를 계산합니다.
- dispatch: action을 store에 전달합니다.

## Redux Toolkit

Redux Toolkit은 Redux 사용 시 반복되는 설정과 보일러플레이트를 줄여줍니다.

- `configureStore`: store 설정을 단순화합니다.
- `createSlice`: action creator와 reducer를 함께 생성합니다.
- `createAsyncThunk`: 비동기 요청 흐름을 action lifecycle로 관리합니다.

전역 상태가 정말 필요한 값인지 먼저 판단하고, 서버 데이터는 React Query 같은 서버 상태 도구와 역할을 나누는 것이 좋습니다.

## 참고

- [Zustand](https://zustand.docs.pmnd.rs/)
- [TanStack Query](https://tanstack.com/query/latest/docs/framework/react/overview)
