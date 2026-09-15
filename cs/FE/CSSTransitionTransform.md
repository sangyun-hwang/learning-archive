# CSS transition과 transform

## 역할 차이

`transform`은 요소를 이동, 확대, 회전하는 등 시각적으로 변형하는 속성이다. `transition`은 속성값이 바뀔 때 중간 과정을 일정 시간에 걸쳐 표현한다.

```css
.button {
  transform: translateY(0);
  transition: transform 200ms ease;
}

.button:hover {
  transform: translateY(-4px);
}
```

마우스를 올리면 버튼이 200ms 동안 위로 4px 이동한다. 마우스를 떼면 원래 자리로 부드럽게 돌아온다. 기본 클래스에 `transition`을 두면 Hover가 해제된 상태에서도 전환 설정이 유지된다.

## transform 함수와 이동 방향

| 함수 | 역할 | 예시 |
| --- | --- | --- |
| `translate()` | 이동 | `translateX(20px)` |
| `scale()` | 확대 또는 축소 | `scale(1.2)` |
| `rotate()` | 회전 | `rotate(15deg)` |
| `skew()` | 기울이기 | `skewX(10deg)` |

다른 회전이나 좌표 변환이 없는 일반적인 화면 좌표에서 X의 양수는 오른쪽, Y의 양수는 아래쪽이다.

```css
transform: translateX(20px);  /* 오른쪽 */
transform: translateX(-20px); /* 왼쪽 */
transform: translateY(20px);  /* 아래쪽 */
transform: translateY(-20px); /* 위쪽 */
```

`transform` 자체가 이동 시간을 정하지는 않는다. Transition이나 Animation 없이 값이 변경되면 중간 움직임 없이 새 위치로 표시된다.

## transition 설정

```css
transition: transform 300ms ease 100ms;
/* 대상 속성, 진행 시간, 속도 곡선, 시작 지연 */
```

`transition`을 선언하는 것만으로 움직임이 시작되지는 않는다. Hover, 클래스 변경 등으로 대상 속성값이 달라져야 전환이 발생한다.

Transition은 두 상태 사이의 변화에 적합하다. 여러 단계나 반복 동작이 필요하면 `@keyframes`와 `animation`을 사용할 수 있다.

## Layout과 성능

`transform`은 일반적인 문서 흐름에서 요소가 차지하는 원래 배치 공간을 바꾸지 않는다. 버튼을 `scale(1.2)`로 확대하면 옆 요소를 밀어내기보다 겹칠 수 있다. 반면 `width`를 변경하면 주변 요소의 배치에도 영향을 줄 수 있다.

`transform`과 `opacity`는 Layout 재계산을 피하고 합성 중심으로 애니메이션을 처리할 수 있어 유리한 경우가 많다. 다만 항상 GPU에서 처리되거나 비용이 없는 것은 아니다. 렌더링 단계는 [브라우저 렌더링 과정](BrowserRendering.md)에서 함께 확인한다.

## 핵심 정리

> transform은 요소를 어떻게 변형할지 정하고, transition은 속성 변화가 어떤 시간과 속도로 진행될지 정한다. transform은 원래 배치 공간을 유지하므로 주변 요소와 겹칠 수 있으며, Layout 재계산을 피하는 움직임을 구현할 때 유용하다.

## 참고 자료

- [MDN: transform](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)
- [MDN: transition](https://developer.mozilla.org/en-US/docs/Web/CSS/transition)
