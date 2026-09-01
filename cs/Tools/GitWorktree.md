# Git Worktree

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

기존 Branch에 미커밋 변경이 있어도 별도의 Worktree에서 Hotfix나 다른 Feature를 시작할 수 있다. 각 폴더를 별도의 Editor 창으로 열거나 개발 Server를 동시에 실행하는 것도 가능하다.

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

## 핵심 정리

> Git Worktree는 하나의 Git 저장소와 Commit History를 공유하면서 Branch별 작업 폴더, HEAD와 Index를 분리한다. 기존 작업을 Stash하지 않고 다른 Branch를 동시에 다룰 수 있으며 Hotfix, 병렬 개발과 AI Agent 작업 분리에 유용하다. Worktree를 제거해도 Branch와 Commit은 자동으로 삭제되지 않으며, 작업 폴더는 가능하면 `git worktree remove`로 정리한다.

## 참고 자료

- [Git: git-worktree Documentation](https://git-scm.com/docs/git-worktree)
