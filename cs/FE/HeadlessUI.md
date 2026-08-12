# Headless UI

## 개념

Headless UI는 Dialog, Select, Dropdown 같은 UI의 동작과 접근성은 제공하지만 시각적 스타일은 강제하지 않는 설계 방식이다.

```text
Headless Component
├─ 상태와 열기·닫기 동작
├─ Keyboard navigation
├─ Focus 이동과 복원
├─ ARIA role과 속성
└─ Style은 application에서 결정
```

Dialog를 직접 만들 때는 화면에 box를 표시하는 것 외에도 focus trap, `Escape` 처리, trigger로 focus 복원, 배경 interaction 제한과 ARIA semantics를 고려해야 한다. Headless library는 이런 복잡한 동작을 재사용 가능하게 제공하면서 project가 자체 design system을 적용할 수 있게 한다.

## Radix Primitives

Radix Primitives는 접근성과 동작을 제공하는 low-level React Component library다.

```tsx
<Dialog.Root>
  <Dialog.Trigger>열기</Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay />
    <Dialog.Content>내용</Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

주요 특징은 다음과 같다.

- 기본 style을 강제하지 않음
- WAI-ARIA pattern과 keyboard interaction 지원
- Focus management 같은 복잡한 동작 제공
- Component의 각 part를 조합하는 API
- Controlled와 uncontrolled 방식 지원
- `asChild`로 rendering할 element를 변경 가능

Radix는 완성된 화면보다 접근 가능한 UI를 만들기 위한 primitive에 가깝다.

## shadcn/ui

shadcn/ui는 일반적인 npm Component library처럼 내부 구현을 `node_modules`에서 감추는 방식이 아니다. CLI가 선택한 Component의 source code를 project에 추가하고 개발자가 그 code를 직접 소유하고 수정하게 한다.

```text
일반 Component library
-> package가 source를 소유
-> public API를 통해 사용

shadcn/ui
-> Component source를 project에 추가
-> project가 source를 소유하고 수정
```

shadcn/ui는 primitive를 기반으로 style과 일관된 Component API가 적용된 code를 제공한다. Radix를 기반으로 사용할 수 있지만 현재는 Base UI와 React Aria 같은 다른 primitive base도 선택할 수 있으므로 shadcn/ui와 Radix를 같은 library로 보면 안 된다.

## Radix와 shadcn/ui 비교

| 구분 | Radix Primitives | shadcn/ui |
| --- | --- | --- |
| 중심 역할 | 동작과 접근성 primitive | Style이 적용된 Component source 제공 |
| Style | 기본적으로 없음 | 선택한 theme와 CSS/Tailwind 기반 style 포함 |
| 사용 방식 | Package API import | Source code를 project에 추가 |
| 수정 범위 | 공개 API 안에서 조합 | 추가된 source를 직접 수정 |
| 유지보수 | Package update 중심 | 복사한 code와 dependency를 project가 관리 |

shadcn Component를 추가하면 design과 구현을 자유롭게 변경할 수 있지만 복사된 code의 update와 유지보수 책임도 project가 가진다. Radix 같은 하위 dependency의 취약점은 package update가 필요하고, project에서 수정한 Component는 변경 내용을 직접 검토하고 반영해야 한다.

## 면접 답변

> Headless UI는 Dialog나 Select의 상태, keyboard interaction, focus management와 접근성을 제공하되 style은 강제하지 않는 방식입니다. Radix는 이런 동작과 접근성을 제공하는 low-level primitive이고, shadcn/ui는 Radix, Base UI 또는 React Aria 같은 primitive를 기반으로 style이 적용된 Component source를 project에 추가합니다. shadcn은 code를 직접 소유하고 수정할 수 있는 대신 이후 update와 유지보수도 project가 담당합니다.

## 참고 자료

- [Radix Primitives: Introduction](https://www.radix-ui.com/primitives/docs/overview/introduction)
- [Radix Primitives: Styling](https://www.radix-ui.com/primitives/docs/guides/styling)
- [shadcn/ui](https://ui.shadcn.com/docs/new)
