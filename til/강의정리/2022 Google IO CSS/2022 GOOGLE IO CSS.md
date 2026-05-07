# 2022 GOOGLE IO CSS

## `accent-color`

> [공식문서](https://developer.mozilla.org/en-US/docs/Web/CSS/accent-color)

- input 태그에서 checkbox, radio, range 타입과 progress 태그의 색상을 변경 시켜 준다.

#### 예시

```html
<style>
  :root {
    accent-color: pink;
  }
</style>

<input type="checkbox" checked />
<input type="radio" checked />
<input type="range" />
<progress max="100" value="50">50%</progress>
<input type="text" />
```

![image-20220722020353289](https://raw.githubusercontent.com/shrewslampe/image_sever/master/img/image-20220722020353289.png)

```html
<style>
  :root {
    accent-color: blue;
  }
</style>
```

![image-20220722020338732](https://raw.githubusercontent.com/shrewslampe/image_sever/master/img/image-20220722020338732.png)

## `inert`

> [공식문서](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/inert)

- HTML element의 상호 작용 자체를 막아버리는 속성이다.
  - 드래그 방지, 버튼 클릭 방지등 여러 곳에 사용 가능하다.

#### 예시

```html
//inert 미사용
<div>
  <label for="button1">Button 1</label>
  <button id="button1">I am not inert</button>
</div>

//inert 사용
<div inert>
  <label for="button2">Button 2</label>
  <button id="button2">I am inert</button>
</div>
```

![Honeycam 2022-07-22 02-15-46](https://raw.githubusercontent.com/shrewslampe/image_sever/master/img/Honeycam%202022-07-22%2002-15-46.gif)
