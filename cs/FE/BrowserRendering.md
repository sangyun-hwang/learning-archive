# Browser Rendering

브라우저 렌더링은 HTML, CSS, JavaScript를 해석해서 사용자가 보는 화면을 만드는 과정이다.

프론트엔드에서 렌더링 과정을 이해하면 성능 문제를 설명하거나, 어떤 코드가 화면 업데이트에 영향을 주는지 판단하기 쉬워진다.

## 전체 흐름

브라우저는 대략 다음 흐름으로 화면을 만든다.

1. HTML을 파싱해서 DOM을 만든다.
2. CSS를 파싱해서 CSSOM을 만든다.
3. DOM과 CSSOM을 합쳐 Render Tree를 만든다.
4. Layout 단계에서 각 요소의 위치와 크기를 계산한다.
5. Paint 단계에서 요소를 픽셀로 그린다.
6. Composite 단계에서 레이어를 합성해 최종 화면을 만든다.

## DOM

DOM은 HTML 문서를 브라우저가 다룰 수 있는 트리 구조로 표현한 것이다.

예를 들어 `body` 안에 `div`, `button`, `p` 같은 요소가 있으면 이 관계가 트리 형태로 구성된다.

JavaScript에서 `document.querySelector` 같은 API로 접근하는 대상도 DOM이다.

## CSSOM

CSSOM은 CSS 규칙을 브라우저가 이해할 수 있는 트리 구조로 표현한 것이다.

브라우저는 CSS 파일, `style` 태그, 인라인 스타일 등을 해석해서 각 요소에 어떤 스타일이 적용되는지 계산한다.

## Render Tree

Render Tree는 실제 화면에 그려질 요소만 모아 만든 트리이다.

DOM의 모든 요소가 Render Tree에 들어가는 것은 아니다.

- `display: none`인 요소는 화면에 그려지지 않으므로 Render Tree에 포함되지 않는다.
- `visibility: hidden`인 요소는 보이지는 않지만 공간은 차지하므로 Render Tree에 포함된다.

## Layout

Layout은 각 요소가 화면에서 어디에, 어떤 크기로 배치될지 계산하는 단계이다.

예를 들어 다음과 같은 값은 Layout에 영향을 줄 수 있다.

- `width`
- `height`
- `margin`
- `padding`
- `position`
- `top`, `left`

Layout이 다시 발생하는 것을 Reflow라고도 부른다.

## Paint

Paint는 Layout 단계에서 계산된 요소를 실제 픽셀로 그리는 단계이다.

예를 들어 다음과 같은 스타일은 Paint에 영향을 줄 수 있다.

- `color`
- `background-color`
- `border`
- `box-shadow`

## Composite

Composite는 여러 레이어를 합성해서 최종 화면을 만드는 단계이다.

`transform`이나 `opacity` 같은 속성은 상황에 따라 Layout이나 Paint를 크게 다시 하지 않고 Composite 단계에서 처리될 수 있다.

그래서 애니메이션을 만들 때 `top`, `left`보다 `transform`을 선호하는 경우가 많다.

## 정리

브라우저 렌더링은 DOM과 CSSOM을 만들고, Render Tree를 구성한 뒤 Layout, Paint, Composite를 거쳐 화면을 만드는 과정이다.

성능 관점에서는 Layout이 자주 다시 계산되는 상황을 줄이고, 가능한 경우 Composite 중심으로 처리되는 스타일을 사용하는 것이 도움이 된다.
