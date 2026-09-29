# Drag and Drop: 순서 변경과 그룹 이동

## 개념

이슈 보드에서 카드를 옮기는 기능은 화면의 위치만 바꾸는 기능이 아니다. **사용자의 이동 의도를 읽고, 상태를 변경하고, 서버에 저장하는 과정**으로 나누어 이해한다.

```text
드래그 시작: 어떤 이슈인가?
-> 드롭: 어느 그룹의 어느 위치인가?
-> React 상태 변경: 이동한 화면 표시
-> API 요청: 변경 사항 저장
-> 성공 결과 반영 또는 실패 복구
```

화면만 옮기고 저장하지 않으면 새로고침 후 서버의 기존 데이터로 돌아간다. 서버가 없는 로컬 도구라면 별도의 로컬 저장 방식을 정할 수 있다.

## 어떤 데이터가 바뀌는가

```text
이동 전
Todo:  [A, B]
Doing: [C, D]

B를 Doing의 두 번째로 이동
Todo:  [A]
Doing: [C, B, D]
```

| 정보 | 역할 |
| --- | --- |
| issueId: B | 이동할 이슈 식별 |
| targetGroupId: Doing | 도착할 그룹 식별 |
| 위치 정보 | 그룹 안에서 어디에 놓을지 표현 |

같은 그룹 안에서는 순서가 바뀌고, 다른 그룹으로 이동하면 소속 그룹과 순서가 함께 바뀐다. 배열 index는 이동할 때 달라지므로 이슈의 고유 ID를 대신할 수 없다.

저장 모델은 `id`, `groupId`, `position` 등으로 구성할 수 있다. 순서에 연속 정수를 사용한다면 이동한 이슈뿐 아니라 주변 이슈의 position도 조정해야 할 수 있다.

API에는 다음처럼 이동 의도를 보낼 수도 있다. 아래는 설명용 설계이며 정해진 표준은 아니다.

```json
{
  "issueId": "B",
  "targetGroupId": "doing",
  "beforeIssueId": "D"
}
```

`beforeIssueId`는 D 바로 앞에 넣는다는 뜻이다. `null`은 맨 뒤로 넣는다는 식으로 계약을 정할 수 있다. 서버는 대상의 존재 여부와 그룹 소속을 검증한 뒤 실제 저장 순서를 계산한다.

## 브라우저 이벤트

다음은 Native HTML Drag and Drop API 기준이다. 모든 드래그 라이브러리가 이 이벤트만 사용하는 것은 아니다.

| 이벤트/속성 | 역할 |
| --- | --- |
| draggable | 요소를 드래그 가능하게 지정 |
| dragstart | 이동 대상 ID를 DataTransfer에 기록 |
| dragenter / dragleave | 드롭 영역 진입과 이탈 표시 |
| dragover | 드롭 허용 여부 처리 |
| drop | 이동 대상과 도착 위치를 조합해 이동 요청 |
| dragend | 취소를 포함한 드래그 종료 시 표시 정리 |

`DataTransfer`는 드래그 출발점에서 도착점으로 데이터를 전달한다. ID를 전달했다고 데이터 정합성이나 권한까지 보장되지는 않는다.

일반적인 요소에 드롭을 허용하려면 `dragover`에서 `preventDefault()`를 호출한다. 이는 이벤트 전파를 막는 `stopPropagation()`과 다르다. [MDN: HTML Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API)

## React에서 이벤트 연결하기

아래 두 컴포넌트는 이벤트 연결을 보여주는 부분 예제다. `onMove`는 부모가 제공하며 상태 변경, 검증, 저장과 실패 처리를 담당한다.

```tsx
type MoveIntent = {
  issueId: string;
  targetGroupId: string;
  beforeIssueId: string | null;
};

function IssueCard({ id, title }: { id: string; title: string }) {
  return (
    <article
      draggable
      onDragStart={(event) => {
        event.dataTransfer.setData('text/plain', id);
        event.dataTransfer.effectAllowed = 'move';
      }}
    >
      {title}
    </article>
  );
}

function DropZone({
  groupId,
  beforeIssueId,
  onMove,
}: {
  groupId: string;
  beforeIssueId: string | null;
  onMove: (intent: MoveIntent) => void;
}) {
  return (
    <div
      style={{ minHeight: 24 }}
      onDragOver={(event) => event.preventDefault()}
      onDrop={(event) => {
        event.preventDefault();
        const issueId = event.dataTransfer.getData('text/plain');
        if (!issueId) return;
        onMove({ issueId, targetGroupId: groupId, beforeIssueId });
      }}
    />
  );
}
```

Doing의 D 앞에 `groupId="doing"`, `beforeIssueId="D"`인 DropZone을 두면 B의 ID와 도착 위치를 조합할 수 있다. 빈 그룹이나 맨 뒤에도 드롭 영역을 마련해야 한다.

`effectAllowed = 'move'`는 이동 작업을 허용한다는 표시이지, 배열이나 DB를 자동으로 수정하는 명령이 아니다. 외부에서 드래그한 문자열도 들어올 수 있으므로 `onMove`에서 실제 이슈인지 확인해야 한다.

실제 제품에서는 터치, 키보드 조작, 포커스와 이동 결과 안내도 고려한다. 드래그 외에 그룹 선택과 위/아래 이동 버튼을 제공할 수 있다. 위 코드는 완성된 접근성 구현이 아니다.

## React 상태와 서버 저장

React에서는 DOM을 직접 옮겨 끝내기보다 상태를 변경하고 그 상태로 다시 렌더링한다. 여러 그룹이 같은 이슈를 공유하므로 공통 부모나 Store 등 한 곳에서 이동 결과를 관리한다.

```text
1. ID로 이동할 이슈와 도착 그룹 확인
2. 기존 목록에서 이슈 제거
3. 도착 목록의 지정 위치에 삽입
4. 새 배열/객체로 상태 갱신
```

같은 목록 안에서 이동할 때도 제거 후의 목록을 기준으로 삽입 위치를 구하면 index가 밀리는 실수를 줄일 수 있다. 자기 자신 앞으로 놓는 경우는 변경 없음으로 처리한다.

저장 시점은 두 방식으로 나눌 수 있다.

| 방식 | 화면 처리 | 주의점 |
| --- | --- | --- |
| 성공 후 반영 | 저장 중 표시 후 서버 응답으로 이동 | 네트워크 대기가 보임 |
| 낙관적 업데이트 | 먼저 이동시키고 저장 요청 | 실패 시 복구 필요 |

낙관적 업데이트가 실패하면 오류를 알리고 이전 상태로 되돌리거나 서버 데이터를 다시 조회한다. 연속 이동 중 오래된 요청의 실패가 최신 상태를 덮어쓰지 않도록 주의한다. 처음 구현할 때는 저장 중 추가 이동을 막아 흐름을 단순하게 만들 수 있다.

서버는 인증, 이슈 변경 권한, 이동 가능한 그룹과 위치를 검증한다. 여러 행의 순서를 바꾼다면 트랜잭션 등으로 일부만 저장되지 않도록 처리한다. 성공 응답에 확정된 순서를 담으면 클라이언트도 서버 결과로 맞출 수 있다.

**drop이나 dragend는 서버 저장 성공 신호가 아니다.** 드래그의 종료와 비동기 저장의 완료를 별도 상태로 관리한다.

## 면접용 요약

Drag and Drop으로 순서나 그룹을 바꿀 때는 이동할 항목의 ID와 도착 그룹, 위치를 먼저 알아야 합니다. Native API에서는 dragstart에 ID를 기록하고 dragover에서 드롭을 허용한 뒤 drop에서 이동 의도를 읽습니다. 이후 React 상태와 서버 데이터를 함께 갱신하며, 화면을 먼저 바꾸는 낙관적 업데이트에서는 실패 복구가 필요합니다. 드래그가 끝났다는 것과 저장이 성공했다는 것은 별개입니다.

## 관련 학습과 참고

- [Lifting State Up](LiftingStateUp.md)
- [State Management](StateManagement.md)
- [이벤트 위임](EventDelegation.md)
- [MDN: drop event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/drop_event)
