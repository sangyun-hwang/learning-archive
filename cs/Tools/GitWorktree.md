# Git Worktree와 Branch의 차이

## 개념

Git Worktree는 하나의 Git 저장소에서 여러 Branch를 서로 다른 작업 폴더에 동시에 Checkout할 수 있게 하는 기능이다.

일반적으로 하나의 작업 폴더에서는 현재 Checkout한 Branch 하나만 다룬다.

```text
project/
└─ main
```

Worktree를 추가하면 저장소를 다시 Clone하지 않고도 여러 작업 폴더를 함께 사용할 수 있다.

```text
project/          -> main
project-search/   -> feature/search
project-hotfix/   -> hotfix/payment
```

기존 Worktree에 미커밋 변경이 있어도 별도의 Worktree에서 Hotfix나 다른 Feature를 시작할 수 있다. 각 폴더를 별도의 Editor 창으로 열거나 개발 Server를 동시에 실행하는 것도 가능하다.

## Branch와 Worktree는 대체 관계가 아니다

Branch는 특정 Commit을 가리키는 참조다. 해당 Branch에서 Commit하면 참조가 새 Commit을 가리키도록 이동한다. Worktree는 실제 파일을 Checkout하고 수정하는 작업 공간이다.

| 구분 | Branch | Worktree |
| --- | --- | --- |
| 목적 | 작업 이력을 분기 | 작업 공간을 분리 |
| 생성 결과 | Commit을 가리키는 참조 | 별도 작업 폴더와 관리 정보 |
| 미커밋 변경 | Branch 자체가 보관하지 않음 | 작업 파일과 Index에서 관리 |
| 함께 사용하는 방법 | 기능별 Branch 생성 | 각 Branch를 서로 다른 폴더에 Checkout |

Branch를 만드는 것만으로 새 폴더가 생기지는 않는다. 하나의 폴더에서 `git switch`로 Branch를 바꿀 수도 있고, Worktree를 추가해 여러 Branch를 동시에 펼쳐 놓을 수도 있다. Worktree는 Branch에 영구적으로 고정된 폴더는 아니며, 다른 Branch로 전환하거나 detached HEAD 상태로 사용할 수도 있다.

## 미커밋 변경은 어디에 남는가

아직 Commit하지 않은 변경은 Branch 이력이 아니라 현재 Worktree의 작업 파일과 Index에 남는다. 파일을 수정하면 작업 파일이 바뀌고, `git add`를 하면 선택한 변경이 Index에 반영된다.

```text
feature/login에서 파일 수정, Commit하지 않음
-> main으로 전환 시도

변경을 유지하면서 전환 가능한 경우
-> 수정 내용이 main 작업 화면에도 남을 수 있음

전환 과정에서 변경을 덮어쓸 위험이 있는 경우
-> Git이 전환을 막음
```

따라서 "로그인 Branch에서 수정했으니 다른 Branch에서는 자동으로 숨겨지고, 돌아오면 복원된다"고 생각하면 안 된다. `git add`만으로도 Branch 이력에 저장되지는 않는다.

- Commit: 변경을 Commit 객체로 기록하고 현재 Branch 참조를 이동한다. 일반적인 Branch Checkout 상태 기준이다.
- Stash: 미커밋 작업을 별도로 보관해 전환 등에 활용한다.
- Worktree: 기존 작업 파일을 그대로 두고 다른 폴더에서 별도 작업을 진행한다.

예를 들어 로그인 작업 중 `main` 기준 Hotfix Worktree를 추가하면, 원래 폴더의 미커밋 로그인 변경은 유지되고 새 폴더에는 그 변경이 자동으로 복사되지 않는다.

## 공유하는 것과 분리되는 것

Worktree는 Git 저장소 전체를 복제하지 않는다.

### Worktree들이 공유하는 것

- Commit 객체
- Git History와 객체 저장소
- 대부분의 Branch와 Tag Ref
- Remote 정보

### Worktree마다 별도로 가지는 것

- 실제 작업 파일
- `HEAD`
- Index 또는 Staging Area
- 수정 중인 파일과 미추적 파일

`HEAD`는 해당 Worktree가 어떤 Branch 또는 Commit을 보고 있는지 나타낸다. Index는 다음 Commit에 포함할 Stage 상태를 관리한다. 따라서 Worktree A에서 `git add`를 실행해도 Worktree B의 Staging Area에는 영향을 주지 않는다.

반면 Commit 객체와 Branch Ref는 공유한다. A에서 만든 Commit을 B에서도 `git show`나 `git log --all`로 바로 조회할 수 있다. 다만 B의 `HEAD`와 작업 파일이 그 Commit으로 자동 이동하지는 않는다.

```text
Commit 객체와 Branch 위치
-> 저장소에서 공유

현재 Checkout 상태와 작업 파일
-> Worktree마다 독립
```

## Worktree 생성

현재 `main`을 기준으로 새 Branch와 Worktree를 함께 만든다.

```bash
git worktree add -b feature/search ../project-search main
```

```text
-b feature/search
-> 새 Branch 생성

../project-search
-> 새 작업 폴더

main
-> Branch를 생성할 기준
```

이미 존재하는 Branch를 새 Worktree에 연결할 수도 있다.

```bash
git worktree add ../project-hotfix hotfix/payment
```

같은 Branch를 여러 Worktree에서 동시에 Checkout하는 것은 기본적으로 제한된다. 서로 다른 작업 폴더가 같은 Branch Ref를 동시에 이동시키면 현재 작업 상태와 Branch 위치를 일관되게 이해하기 어려워지기 때문이다.

## 확인과 제거

현재 연결된 Worktree를 확인한다.

```bash
git worktree list
```

작업을 마친 Worktree는 폴더를 직접 삭제하지 않고 Git 명령으로 제거한다.

```bash
git worktree remove ../project-search
```

`git worktree remove`는 연결된 작업 폴더와 Worktree 관리 정보를 제거한다. Worktree에서 사용하던 Branch는 자동으로 삭제되지 않는다. Branch까지 정리하려면 별도의 명령을 사용한다.

```bash
git branch -d feature/search
```

탐색기에서 Worktree 폴더를 직접 삭제해 Git의 관리 정보만 남았다면 다음 명령으로 오래된 정보를 정리할 수 있다.

```bash
git worktree prune
```

`prune`은 정상적으로 존재하는 Worktree를 제거하는 명령이 아니라, 실제 작업 폴더가 사라져 더 이상 유효하지 않은 관리 정보만 정리한다.

## Clone과의 차이

새로 Clone하면 Git 객체 저장소와 History까지 별도로 내려받는다. Worktree는 기존 저장소의 Commit과 Ref를 공유하면서 작업 파일과 Checkout 상태만 별도로 관리한다.

```text
Clone
-> 독립된 Git 저장소
-> 객체와 History도 별도 보관

Worktree
-> 하나의 Git 저장소 공유
-> 작업 폴더, HEAD와 Index 분리
```

Git 객체를 공유하므로 Clone보다 중복 저장 공간을 줄일 수 있다. 그러나 실제 작업 파일과 Git이 추적하지 않는 항목까지 모두 공유되는 것은 아니다.

## `node_modules`와 환경 변수

`node_modules`, `.env`, `.next` 같은 항목은 보통 Git이 추적하지 않으므로 새 Worktree에 자동으로 Checkout되지 않는다. Branch마다 Package Version과 환경 설정이 다를 수도 있어 Worktree별로 설치하거나 준비해야 한다.

```text
project/
├─ node_modules/
└─ .env

project-search/
├─ node_modules/
└─ .env
```

개발 Server를 여러 Worktree에서 동시에 실행한다면 서로 다른 Port를 사용해야 한다.

## 활용 사례

- 기존 Feature의 미커밋 변경을 유지하면서 긴급 Hotfix 처리
- 여러 Feature Branch 동시 개발과 비교
- 한 Branch에서 개발 Server를 실행하면서 다른 Branch 검토
- 여러 버전의 Test 동시 실행
- AI Agent마다 별도의 Branch와 작업 폴더 제공

AI Agent마다 다른 Worktree를 제공하면 각 Agent가 별도의 작업 파일, `HEAD`와 Index를 사용한다. 서로의 미커밋 변경과 Stage 상태를 덮어쓸 위험이 줄고, 결과를 Commit 단위로 검토한 뒤 Merge하거나 Cherry-pick할 수 있다.

Worktree는 작업 결과를 자동으로 합치는 기능이 아니다. 각 Worktree에서 만든 Commit은 기존 Git Workflow와 동일하게 Merge, Rebase 또는 Cherry-pick으로 반영한다.

## 작업 분리와 보안 격리는 다르다

Worktree로 파일을 나누어도 Merge 충돌이나 기능 간 모순이 없어지지는 않는다. 예를 들어 한 작업이 API 응답 형식을 바꾸고 다른 작업이 이전 형식을 기준으로 UI를 만들면, 각자 작업을 마쳐도 통합 후 오류가 날 수 있다.

이런 코드 정합성 문제와 보안 격리도 별개다. Worktree는 다른 폴더의 파일 접근, 명령 실행, 외부 네트워크 통신 권한을 제한하지 않는다.

```text
작업 공간 분리
-> 각자의 파일과 Index에서 작업

보안 격리
-> 접근 가능한 파일, 실행 가능한 명령, 네트워크 등을 제한
```

AI Agent가 별도 Worktree에서 실행돼도 같은 사용자 권한으로 접근 가능한 다른 폴더의 `.env`를 읽을 수 있다. 별도 폴더를 지정했다는 사실만으로 이를 차단했다고 볼 수 없다. 접근을 제한하려면 별도의 OS 권한 설정이나 Sandbox 등 실행 환경의 통제가 필요하다.

## 핵심 정리

> Branch는 Commit을 가리키며 작업 이력을 나누고, Worktree는 실제 작업 파일과 HEAD, Index를 나눈다. Commit 객체와 Branch Ref는 공유하므로 한쪽에서 만든 Commit은 다른 쪽에서도 조회할 수 있지만 작업 파일이 자동으로 바뀌지는 않는다. 미커밋 작업을 유지한 Hotfix나 병렬 개발에 유용하지만, 코드 통합 검증과 보안 격리를 대신하지는 않는다.

## 참고 자료

- [Git: git-worktree Documentation](https://git-scm.com/docs/git-worktree)
