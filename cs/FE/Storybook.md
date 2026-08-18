# Storybook

Storybook은 Application 전체를 실행하고 특정 화면까지 이동하지 않아도 UI Component와 Page를 **격리된 환경에서 개발, 확인, 문서화하고 테스트**할 수 있게 해주는 Frontend Workshop이다.

Component를 Application의 Routing, Backend 상태와 복잡한 Business Logic에서 분리해 Loading, Error, Empty, Disabled처럼 실제 화면에서 만들기 어려운 상태도 독립적으로 재현할 수 있다.

## Storybook이 필요한 이유

하나의 Component도 다양한 상태를 가진다.

```text
Button
├─ Primary
├─ Secondary
├─ Disabled
├─ Loading
└─ Long Label
```

Application에서 각 상태를 확인하려면 Page 이동, Login, API 응답이나 특정 사용자 동작이 필요할 수 있다. Storybook에서는 필요한 props와 mock data를 전달해 원하는 상태를 바로 Rendering한다.

주요 용도는 다음과 같다.

- UI Component를 Application과 분리해 개발
- Component가 지원하는 상태와 변형을 Story로 기록
- Props를 바꾸며 Edge Case 확인
- UI 사용법과 상태를 팀에 문서로 공유
- Interaction, Accessibility와 Visual Regression 테스트의 기반 제공

## Story

**Story는 Component의 특정 Rendering 상태를 표현하는 예제**다.

Button Component 자체가 설계라면 `Primary`, `Disabled`, `Loading` Story는 그 Component가 실제로 나타날 수 있는 각각의 상태다.

```text
Component
-> 재사용 가능한 UI 구현

Story
-> 특정 props와 환경을 적용한 Component 상태
```

Story는 보통 `*.stories.ts` 또는 `*.stories.tsx` 파일에 Component Story Format(CSF)으로 작성한다.

## Component Story Format

CSF는 ES Module의 default export와 named export를 사용해 Story를 작성하는 형식이다.

```tsx
import type { Meta, StoryObj } from '@storybook/nextjs-vite';

import { Button } from './Button';

const meta = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    label: '저장',
    variant: 'primary',
  },
};

export const Disabled: Story = {
  args: {
    label: '저장',
    disabled: true,
  },
};
```

- `meta`: Component 전체 Story에 적용되는 metadata와 설정
- `component`: Story의 대상 Component
- `title`: Storybook Sidebar에서 보이는 분류
- named export: 각각의 Story
- `StoryObj`: Story 객체의 Type
- `satisfies Meta`: Component props에 맞는 설정인지 TypeScript로 검사

Framework에 따라 `Meta`와 `StoryObj`를 import하는 package 이름은 달라질 수 있다.

## Args와 Controls

`args`는 Story에 전달할 props와 유사한 입력값이다.

```tsx
export const Loading: Story = {
  args: {
    label: '저장 중',
    loading: true,
  },
};
```

Storybook은 args를 Controls Panel과 연결해 개발자가 값을 바꾸며 Component를 확인할 수 있게 한다.

```text
Story args
-> Component props로 전달
-> Controls에서 값 변경
-> Component가 즉시 다시 Rendering
```

Controls는 새로운 UI 상태를 영구 저장하는 State Manager가 아니다. Story를 탐색하며 다양한 입력을 시험하는 개발 도구다. 중요한 상태는 Controls에서만 확인하지 않고 named Story로 남겨야 재현하고 공유할 수 있다.

`argTypes`를 사용하면 각 arg의 Control 종류, 설명과 선택 가능한 값을 설정할 수 있다.

## Parameters와 Decorators

### Parameters

Parameters는 Story가 Rendering되는 방식이나 Addon 동작을 설정하는 정적 metadata다.

예시:

- Background와 Viewport
- Layout
- Accessibility 설정
- 특정 Story의 문서 설정

### Decorators

Decorator는 Story를 Provider나 Layout으로 감쌀 때 사용한다.

```text
ThemeProvider
└─ Router Provider
   └─ Story
```

Theme, Context, Router처럼 Application에서는 상위에 존재하지만 격리된 Story에는 없는 환경을 재현할 수 있다. 모든 Provider를 무조건 넣으면 Story가 실제 Application에 다시 강하게 결합될 수 있으므로 필요한 환경만 제공한다.

## Interaction Test와 play

`play` 함수는 Story가 Rendering된 후 사용자 상호작용을 실행하고 결과를 검증한다.

```tsx
import { expect, fn } from 'storybook/test';

export const Clicked: Story = {
  args: {
    label: '저장',
    onClick: fn(),
  },
  play: async ({ args, canvas, userEvent }) => {
    await userEvent.click(
      canvas.getByRole('button', { name: '저장' }),
    );

    await expect(args.onClick).toHaveBeenCalledOnce();
  },
};
```

- `canvas`: 현재 Story의 DOM 범위
- `userEvent`: 실제 사용자와 비슷한 상호작용 실행
- `expect`: 결과 검증
- `fn`: Callback 호출을 확인하기 위한 mock function

Story를 화면에 표시하는 것만으로도 Rendering 오류를 확인하는 기본 검증이 된다. `play`를 추가하면 클릭, 입력, Dialog 열기 같은 상태 변화를 검증할 수 있다.

## Testing에서의 위치

Storybook은 단순한 Component 전시 도구를 넘어 여러 UI 테스트의 출발점이 될 수 있다.

| 종류 | 확인하는 내용 |
| --- | --- |
| Render Test | Story가 오류 없이 Rendering되는가 |
| Interaction Test | 입력과 클릭 후 올바르게 동작하는가 |
| Accessibility Test | 접근성 규칙을 위반하지 않는가 |
| Visual Test | 이전 화면과 의도하지 않은 시각적 차이가 생겼는가 |

Storybook만으로 전체 Application의 Routing, 실제 Backend와 Browser 간 통합을 모두 검증할 수는 없다. Component 상태는 Storybook으로 검증하고, 실제 사용자 흐름은 Playwright나 Cypress 같은 E2E 도구로 보완한다.

## Mocking

격리된 환경에는 실제 API, Login 상태나 Router가 없을 수 있으므로 필요한 의존성을 mock으로 제공한다.

```text
실제 API를 기다리지 않고
-> Success Story
-> Loading Story
-> Empty Story
-> Error Story
```

Mocking의 목적은 실제 Backend가 정확히 동작한다고 증명하는 것이 아니라, **각 응답 상태에서 UI가 어떻게 표현되는지 반복 가능하게 확인하는 것**이다.

## 실제 프로젝트에서의 활용

1. 공통 Button, Input과 Dialog부터 Story 작성
2. 정상 상태뿐 아니라 Loading, Empty, Error와 Disabled 상태 기록
3. args와 Controls로 props 조합 확인
4. 중요한 사용자 동작에 `play`와 assertion 추가
5. CI에서 Story Rendering과 테스트 실행
6. 배포된 Storybook이나 Visual Test 결과로 변경사항 검토

모든 `<div>`나 작은 내부 구현까지 Story를 만들 필요는 없다. 재사용되는 Component, 의미 있는 UI 상태, Regression 위험이 있는 사용자 동작을 중심으로 작성한다.

## 면접에서 설명하기

> Storybook은 UI Component와 Page를 Application의 Routing이나 Backend 상태에서 분리해 개발하고 문서화하는 도구입니다. Component의 Loading, Error, Empty 같은 상태를 각각 Story로 저장해 반복 가능하게 확인할 수 있고, args와 Controls로 props를 변경하며 Edge Case를 탐색할 수 있습니다. 또한 play 함수를 이용한 Interaction Test와 Accessibility, Visual Regression Test의 기반으로 활용할 수 있지만, 실제 Backend를 포함한 전체 사용자 흐름은 E2E 테스트로 보완해야 합니다.

## 확인할 질문

1. Storybook을 단순한 Component 전시 페이지보다 개발 도구라고 보는 이유는 무엇일까?
2. Component와 Story의 차이를 Button 예시로 설명해봐.
3. Controls에서 props를 바꿀 수 있어도 중요한 상태를 named Story로 남겨야 하는 이유는 무엇일까?
4. args와 argTypes는 각각 어떤 역할을 할까?
5. Decorator로 모든 Application Provider를 넣는 것이 항상 좋은 것은 아닌 이유는 무엇일까?
6. `play` 함수는 Story Rendering 전과 후 중 언제 실행되며 무엇을 검증할까?
7. Storybook Interaction Test와 E2E Test는 검증 범위가 어떻게 다를까?

## References

- [Storybook: Get started](https://storybook.js.org/docs)
- [Storybook: Why Storybook](https://storybook.js.org/docs/get-started/why-storybook)
- [Storybook: Writing stories](https://storybook.js.org/docs/writing-stories)
- [Storybook: Args](https://storybook.js.org/docs/writing-stories/args)
- [Storybook: Play function](https://storybook.js.org/docs/writing-stories/play-function)
- [Storybook: Testing](https://storybook.js.org/docs/writing-tests)

