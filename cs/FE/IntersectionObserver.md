# Intersection Observer

## 개념

Intersection Observer는 대상 요소와 브라우저 viewport 또는 지정한 상위 영역의 교차 상태를 비동기로 알려주는 브라우저 API다. 무한 스크롤, 콘텐츠 지연 로딩과 화면 진입 효과 등에 활용할 수 있다.

scroll 이벤트마다 직접 요소의 위치를 반복 계산하는 대신 브라우저에 관찰을 맡기고 조건 변화에 대한 알림을 받는다. 콜백 자체는 메인 스레드에서 실행되므로 무거운 작업을 넣지 않는다. [MDN 공식 문서](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)

## 무한 스크롤의 감지용 요소

목록 마지막에 별도의 감지용 요소인 sentinel을 배치하고 관찰할 수 있다.

```text
상품 목록
상품 목록
상품 목록
sentinel
-> 관찰 영역에 들어오면 다음 페이지 요청 검토
```

Observer는 대상이 상품인지 알 필요 없이 요소의 교차 상태를 관찰한다. sentinel을 목록 끝에 두었으므로 그 요소의 진입을 목록 끝에 가까워졌다는 신호로 사용할 수 있다. 상품 목록이 추가되면 그 뒤에 있는 sentinel의 위치도 달라진다.

## 주요 옵션과 결과

| 항목 | 의미 |
| --- | --- |
| root | 관찰 기준 영역. null이면 브라우저 viewport |
| rootMargin | 교차 계산에 사용할 기준 영역을 확장하거나 축소 |
| threshold | 콜백 알림 기준이 되는 대상 요소의 교차 비율 경계 |
| entry.isIntersecting | 대상이 관찰 영역과 교차하는지 여부 |
| entry.intersectionRatio | 대상 면적 중 교차한 비율 |

rootMargin을 `0px 0px 200px 0px`로 지정하면 아래쪽 관찰 범위를 200px 확장한다. 목록 끝이 실제 화면에 들어오기 전 다음 데이터를 준비하는 데 사용할 수 있다. 실제 CSS 여백을 바꾸는 것은 아니다.

threshold가 0.5라면 대상의 절반이 교차하는 경계를 넘을 때 알림을 받는다. 콜백은 진입뿐 아니라 이탈 방향의 경계 변화에서도 호출되며, 관찰을 시작한 직후에도 초기 상태 알림을 받을 수 있다.

기본 관찰은 영역의 기하학적 교차를 다룬다. 다른 요소에 가려졌는지 등 사용자가 실제로 내용을 보고 있는지까지 항상 보장하지는 않는다.

## 기본 예제

아래는 DOM에 준비된 sentinel을 관찰하는 부분 예제다.

```js
const sentinel = document.querySelector('#sentinel');

const observer = new IntersectionObserver(
  (entries) => {
    for (const entry of entries) {
      if (entry.isIntersecting) {
        console.log('목록 끝이 관찰 영역에 들어옴');
      }
    }
  },
  {
    root: null,
    rootMargin: '0px 0px 200px 0px',
    threshold: 0,
  },
);

if (sentinel) observer.observe(sentinel);

// 해당 화면을 정리할 때 호출
// observer.disconnect();
```

## 다음 페이지 요청 조건

```js
if (entry.isIntersecting && !isFetching && hasNextPage) {
  fetchNextPage();
}
```

위 코드는 조건을 설명하는 부분 예제다. 교차했다는 이유만으로 무조건 요청하지 않는다. 현재 요청 중이면 중복 실행을 막고, 다음 페이지가 없으면 요청을 중단한다. 실제 조회 함수나 라이브러리에서도 요청 상태를 관리해야 한다.

계속 교차한 채로 머물러 있다고 매 순간 콜백이 반복되는 것은 아니다. 첫 페이지가 짧아 sentinel이 계속 보이는 상황에서 연속으로 데이터를 채울지는 별도의 로딩 정책으로 정한다.

## React에서 정리하기

React에서는 Effect에서 Observer를 생성하고 대상 DOM을 observe한 뒤 cleanup에서 disconnect한다. 특정 대상만 관찰에서 제외하려면 unobserve를 사용할 수 있다.

정리하지 않으면 불필요한 관찰과 콜백이 남고 관련 참조가 유지될 수 있다. 재마운트 시 중복 관찰이나 오래된 로직 실행을 피하기 위해 등록과 해제를 함께 관리한다.

콜백에서 사용하는 요청 상태와 함수도 최신 값인지 확인해야 한다. Effect 의존성을 빠뜨려 이전 isFetching이나 hasNextPage를 계속 읽는 stale closure를 만들지 않는다.

## 리스트 가상화와 연결

| 기능 | 역할 |
| --- | --- |
| Intersection Observer | 다음 요청 시점을 판단할 교차 신호 제공 |
| 무한 스크롤 | 다음 데이터를 조회해 목록에 추가 |
| 리스트 가상화 | 실제로 렌더링할 항목 범위 제한 |

Observer가 데이터를 직접 가져오거나 DOM 개수를 자동으로 줄이는 것은 아니다. 필요한 기능을 조합해 구현한다.

## 면접용 설명

Intersection Observer는 특정 요소가 viewport나 지정한 영역과 교차하는 상태를 비동기로 감지하는 API입니다. 무한 스크롤에서는 목록 끝의 감지용 요소를 관찰하고 rootMargin으로 미리 다음 데이터를 요청할 수 있습니다. 요청 중인지와 다음 페이지 존재 여부를 확인하고, React에서는 cleanup으로 관찰을 해제합니다.

## 관련 학습

- [리스트 가상화](ListVirtualization.md)
- [React Effect](React.md)
- [DOM](DOM.md)

개념 설명용 예제이며 별도 브라우저 구현 테스트는 진행하지 않았다.
