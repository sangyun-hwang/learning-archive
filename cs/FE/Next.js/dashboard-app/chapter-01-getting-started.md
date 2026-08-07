# Chapter 1. Getting Started

## 학습 목표

공식 starter project를 실행하고 App Router project의 기본 폴더와 파일이 어떤 책임을 갖는지 확인한다.

## 프로젝트 구조

| 위치 | 역할 |
| --- | --- |
| `app/page.tsx` | 해당 route에서 표시할 UI를 정의하고 route를 접근 가능하게 만드는 특별한 파일 |
| `app/layout.tsx` | 하위 Page와 Layout을 `children`으로 감싸 공통 UI를 제공하는 특별한 파일 |
| `app/ui` | 화면에 rendering할 Component를 모아 둔 project convention |
| `app/lib` | data 조회, type, utility와 같은 logic을 모아 둔 project convention |
| `public` | `/파일명` 형태의 URL로 접근할 수 있는 static asset 위치 |

`page.tsx`와 `layout.tsx`는 Next.js가 파일 이름에 특별한 의미를 부여하는 file convention이다. 반면 `ui`와 `lib`는 코드를 구분하기 위한 project convention이므로 이름과 구성을 팀에서 다르게 정할 수 있다.

```text
layout.tsx
├─ 공통 UI
└─ children
   └─ page.tsx
```

Layout은 navigation 중에도 유지될 수 있어 공통 UI를 다시 구성하는 작업을 줄이고, Layout 내부의 Client state를 보존하는 데 도움이 된다.

## Placeholder Data와 Type

실제 Database나 API가 준비되지 않은 단계에서는 placeholder data로 UI와 data 구조를 먼저 개발할 수 있다. 이 과정의 placeholder data는 이후 Database에 초기 data를 넣는 seed에도 사용된다.

`definitions.ts`의 TypeScript type은 개발 중 잘못된 property나 type 사용을 발견하는 데 도움을 준다. 하지만 TypeScript type은 compile 후 사라지므로 외부 API와 Database가 반환한 runtime data를 자동으로 검증하지 않는다.

Runtime 검증이 필요하면 Zod와 같은 schema 도구를 사용할 수 있다. 일반적으로 schema에서 TypeScript type을 추론하면 runtime schema와 compile-time type을 함께 관리하기 쉽다.

```text
TypeScript type
-> compile-time의 코드 사용 검사

Runtime schema
-> 실제로 들어온 data 검사
```

## TypeScript와 JSX의 실행

Browser는 TypeScript와 JSX를 직접 실행하지 않는다.

```text
TypeScript / JSX
-> Next.js 개발 서버와 build tool이 변환
-> 실행 가능한 JavaScript
-> Server 또는 Browser에서 실행
```

Server Component의 JavaScript는 Server에서 실행되며 Browser에 전달되지 않는다. 최초 접근에서는 실행 결과가 HTML 생성에 사용되고, RSC Payload에는 React가 Component tree를 구성하는 데 필요한 결과와 Client Component 참조 등이 포함된다.

```text
Server Component
-> Server에서 코드 실행
-> HTML과 RSC Payload 생성에 결과 사용
-> Component JavaScript는 Browser에 전달하지 않음

Client Component
-> 초기 HTML 생성에 참여 가능
-> JavaScript를 Browser에 전달
-> hydration으로 state와 event handler 연결
```

## 핵심 정리

> App Router의 `page.tsx`는 route의 화면을 정의하고 `layout.tsx`는 하위 route를 감싸는 공통 UI를 정의한다. `ui`와 `lib`는 Next.js의 예약 폴더가 아니라 UI와 logic을 구분하기 위한 project convention이다. Browser는 TypeScript와 JSX를 직접 실행하지 않으며, Server Component는 Server에서 실행한 결과만 전달하므로 해당 Component의 JavaScript가 Client bundle에 포함되지 않는다.

## 관련 문서

- [Server Component와 Client Component](../next.js.md#server-component와-client-component)
- [Server Component와 SSR](../next.js.md#server-component와-ssr)
