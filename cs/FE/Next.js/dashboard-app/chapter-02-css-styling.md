# Chapter 2. CSS Styling

## 학습 목표

Next.js application에서 Global CSS, Tailwind CSS와 CSS Modules를 적용하는 방식과 조건부 class name을 구성하는 방법을 비교한다.

## Global CSS

Global CSS는 reset, 공통 element style과 Tailwind directive처럼 application 전체에서 공유할 style을 정의할 때 사용한다.

```tsx
import '@/app/ui/global.css';
```

`global.css`가 전역으로 적용되는 이유는 파일 이름이나 `ui` 폴더의 위치 때문이 아니다. Root Layout에서 import되어 전체 Component tree에 포함되고, 내부 selector가 global scope를 사용하기 때문이다.

Root Layout은 모든 route를 감싸므로 application 전체에 필요한 style을 한 번 불러오기 적합하다.

## Tailwind CSS

Tailwind는 미리 정의된 utility class를 JSX의 `className`에 조합하는 방식이다.

```tsx
<div className="flex rounded-lg bg-blue-500 p-4" />
```

각 utility class는 하나의 제한된 역할을 담당하고 필요한 element에 명시적으로 적용한다. 별도의 class name을 만들지 않고 Component markup 가까이에서 style 조합을 확인할 수 있다.

## CSS Modules

`.module.css` 파일은 class name을 local scope로 처리한다.

```css
.shape {
  width: 0;
  height: 0;
}
```

```tsx
import styles from './home.module.css';

<div className={styles.shape} />
```

Build 과정에서 `shape`는 고유한 class name으로 변환되므로 다른 Component의 같은 이름과 충돌할 가능성을 줄인다.

```text
작성한 이름
-> shape

Browser에 적용되는 이름
-> 고유한 값이 포함된 class name
```

CSS Module은 파일을 만드는 것만으로 적용되지 않는다. Module을 import하고 반환된 mapping에서 class를 꺼내 element에 명시적으로 연결해야 한다. 일반 CSS도 파일 이름을 `home.css`로 바꾼다고 자동으로 적용되는 것은 아니며 application 어딘가에서 import해야 한다.

## Tailwind와 CSS Modules 비교

| 구분 | Tailwind | CSS Modules |
| --- | --- | --- |
| 작성 방식 | utility class를 JSX에서 조합 | CSS 파일에 class 규칙 작성 |
| 충돌 방지 | 제한된 utility를 필요한 element에 조합 | build 시 고유한 class name 생성 |
| style 위치 | markup 가까이 위치 | Component별 CSS 파일로 분리 |
| 적합한 경우 | 빠른 UI 조합과 일관된 design token 사용 | 복잡한 style 규칙이나 CSS와 markup 분리 |

두 방식은 서로 배타적이지 않으며 project 요구에 따라 함께 사용할 수 있다. 실습에서는 같은 도형을 Tailwind와 CSS Module로 각각 표현해 차이를 확인하고, 학습용 임시 UI는 최종 Page에서 제거했다.

## clsx

`clsx`는 CSS를 생성하는 library가 아니다. 조건에 따라 적용할 class name 문자열을 읽기 쉽게 조합하도록 돕는다.

```tsx
className={clsx('base', {
  'status-paid': status === 'paid',
  'status-pending': status === 'pending',
})}
```

문자열 연결이나 중첩된 삼항 연산자보다 조건과 class의 관계를 명확하게 표현할 수 있다.

## 핵심 정리

> Global CSS는 Root Layout에서 import해 application 전체에서 공유하고, Tailwind는 utility class를 element에 조합해 style을 적용한다. CSS Modules는 class name을 build 시 고유하게 만들어 Component 간 충돌을 줄이며, import한 mapping을 통해 class를 element에 명시적으로 연결해야 한다. `clsx`는 CSS를 생성하는 것이 아니라 조건부 class name 조합을 돕는다.

## 참고 자료

- [Next.js Learn: CSS Styling](https://nextjs.org/learn/dashboard-app/css-styling)
- [Next.js: CSS](https://nextjs.org/docs/app/getting-started/css)
