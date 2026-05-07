# State Management

## React Query

React Query는 서버 상태를 관리하기 위한 라이브러리입니다. 서버에서 가져온 데이터의 캐싱, 재요청, 로딩과 에러 상태, 백그라운드 업데이트를 다루기 좋습니다.

클라이언트 UI 상태와 서버 상태를 분리해서 생각하는 것이 핵심입니다.

- 서버 상태: API 응답, 캐시, stale 여부, 재요청
- 클라이언트 상태: 모달 열림 여부, 입력 중인 값, 선택된 탭

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
