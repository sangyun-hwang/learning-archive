# ADE (Agentic Development Environment)

## 개념

ADE는 Agentic Development Environment의 약자로, AI Coding Agent가 개발 작업을 계획하고 실행하며 검증할 수 있도록 구성한 개발 환경을 말한다.

아직 IDE처럼 범위가 엄격하게 표준화된 용어는 아니다. 제품마다 ADE라고 부르는 기능은 다르지만, 공통적으로 개발자가 Agent에게 Task를 맡기고 실행 과정과 결과를 통제하는 환경을 지향한다.

```text
IDE
-> 사람이 Code를 직접 작성하는 작업 공간 중심

ADE
-> 사람이 목표와 제약을 제시
-> Agent가 Code 탐색, 수정과 검증 수행
-> 사람이 Diff와 실행 근거를 Review
```

ADE가 IDE를 완전히 대체한다기보다 기존 Editor, Terminal, Git과 CI를 Agent 중심 Workflow로 연결한 확장으로 볼 수 있다.

## AI Assistant와 Coding Agent

AI Assistant는 자동 완성이나 질문에 대한 Code 예시처럼 현재 작업을 보조하는 데 집중한다.

Coding Agent는 자연어로 전달받은 목표를 처리하기 위해 여러 단계를 수행할 수 있다.

1. Repository와 관련 문서를 탐색한다.
2. 변경 계획을 세운다.
3. 파일을 수정한다.
4. Build, Test와 Lint를 실행한다.
5. 실패 원인을 확인하고 다시 수정한다.
6. Diff와 검증 결과를 사람에게 전달한다.

ADE는 이 Agent Loop가 실제 개발 환경에서 반복될 수 있도록 Context, Tool, 실행 공간과 Review Surface를 제공한다.

## 주요 구성 요소

### Project Context

Agent가 Repository 구조와 개발 규칙을 이해할 수 있도록 `AGENTS.md`, 문서, Code와 Git History 등의 Context를 제공한다. Model이 React나 Spring의 일반 지식을 알고 있어도 프로젝트만의 Architecture, Build와 Test 명령, Code Style, 수정 금지 영역과 완료 조건까지 자동으로 알 수는 없다. 필요한 정보를 적절한 시점에 전달해야 프로젝트의 의도와 규칙에 맞게 작업할 수 있다.

### Tool과 실행 환경

Agent는 파일 읽기와 수정뿐 아니라 Terminal, Git, Test Runner, Browser와 Issue Tracker 같은 Tool을 사용할 수 있다. 답변만 생성하는 것이 아니라 실제 개발 작업을 수행하므로 Tool별 권한과 실행 범위를 제한해야 한다.

### 격리와 병렬 작업

여러 Agent가 같은 작업 폴더를 동시에 수정하면 파일과 Branch가 충돌할 수 있다. ADE 제품들은 Git Worktree, 별도 Branch, Container나 Remote Runtime 등으로 작업 공간을 분리할 수 있다.

```text
Agent A -> Worktree A -> feature/search
Agent B -> Worktree B -> fix/payment
```

격리된 환경은 Agent들이 같은 작업 파일과 Staging Area를 덮어쓰는 문제를 줄이지만 논리적인 Code 충돌까지 자동으로 해결하지는 않는다. 예를 들어 Agent A가 Login API Contract를 변경하고 Agent B가 기존 Contract를 기준으로 UI를 만들었다면 각 Branch에서 성공해도 Merge한 결과는 깨질 수 있다. 최종 통합과 검증은 여전히 필요하다.

### Review와 검증

Agent가 작업을 완료했다고 말하는 것만으로 결과를 신뢰해서는 안 된다. ADE에서는 다음 근거를 함께 확인할 수 있어야 한다.

- 변경된 Diff
- 실행한 Test와 결과
- Build와 Lint 상태
- Tool 호출과 작업 기록
- Commit과 Pull Request

Test 통과도 중요한 근거지만 요구사항 충족을 보장하지는 않는다. Test가 모든 요구사항을 다루지 않거나 Agent가 불필요한 파일과 기존 Test까지 잘못 변경했을 수 있으므로 Diff와 변경 범위를 함께 검토한다.

사람은 모든 Code를 처음부터 직접 작성하는 역할에서 목표, 제약과 완료 조건을 정의하고 결과를 Review하는 역할까지 맡게 된다.

## 자율성과 Guardrail

Agent의 자율성이 높아질수록 작업은 빨라질 수 있지만 잘못된 명령의 영향도 커진다.

- 읽기와 쓰기 권한을 구분한다.
- Network와 Secret 접근을 필요한 범위로 제한한다.
- 삭제, 배포와 결제처럼 영향이 큰 작업은 승인을 요구한다.
- 작업마다 종료 조건과 검증 명령을 정한다.
- Agent가 만든 Diff를 Merge 전에 Review한다.

Container나 Worktree를 사용한다고 자동으로 안전해지는 것은 아니다. 격리 범위, 전달된 Secret, Network 권한과 승인 정책까지 함께 설계해야 한다.

## IDE와 ADE의 차이

| 기준 | IDE | ADE |
| --- | --- | --- |
| 중심 사용자 | 사람이 직접 Code 작성 | 사람과 Coding Agent가 함께 작업 |
| 기본 단위 | File, Editor와 Debug Session | Task, Agent Session과 격리된 작업 공간 |
| Context | 사람이 탐색하고 이해 | Agent도 Code와 규칙을 읽도록 제공 |
| 실행 | 사람이 명령을 직접 실행 | Agent가 허용된 Tool로 실행 가능 |
| 결과 확인 | Code와 Debug 결과 | Diff, Test, 작업 기록과 Review |
| 병렬성 | 여러 Editor와 개발자 | 여러 Agent와 Worktree를 함께 운영 가능 |

IDE에 Agent 기능이 추가될 수 있으므로 두 개념의 경계가 항상 명확한 것은 아니다. 핵심 차이는 Editor에 Chat UI가 있는지가 아니라, Agent의 여러 단계 작업을 실행하고 격리하며 검증하는 Workflow가 환경의 중심에 있는지다.

## 핵심 정리

> ADE는 Coding Agent가 Repository를 탐색하고 Code를 수정하며 Test까지 수행하는 개발 Workflow를 지원하는 환경이다. IDE가 사람의 직접 편집을 중심으로 했다면 ADE는 Task, Context, Tool 실행, 격리된 작업 공간과 Review를 함께 관리한다. Agent의 자율성이 커지는 만큼 권한 제한과 검증, 사람의 최종 Review가 중요하다.

## 참고 자료

- [AWS Prescriptive Guidance: Coding agents](https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/coding-agents.html)
- [ctx: Agentic Development Environment](https://ade.ctx.rs/)
- [ADE: Lanes Overview](https://www.ade-app.dev/docs/lanes/overview)
