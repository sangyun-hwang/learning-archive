# Chapter 3. Optimizing Fonts and Images

## 학습 목표

`next/font`와 `next/image`를 사용해 font와 image가 만드는 network 비용과 Cumulative Layout Shift를 줄이는 방법을 확인한다.

## Web Font와 Layout Shift

Custom Web Font는 font file이 준비되기 전까지 글자가 보이지 않거나 fallback font로 먼저 표시될 수 있다. 이후 custom font로 교체될 때 글자의 폭, 높이와 간격이 달라지면 주변 element의 위치도 함께 움직일 수 있다.

```text
fallback font로 text 표시
-> custom font load 완료
-> custom font로 교체
-> 글자 크기와 간격 변화
-> Layout Shift 발생 가능
```

Cumulative Layout Shift는 사용자가 동작하지 않았는데 화면 element가 예상하지 못하게 이동하는 정도를 나타내는 Core Web Vital 지표다.

## next/font

`next/font`는 font를 build 과정에서 준비하고 application의 static asset으로 self-host한다. Runtime마다 Browser가 Google Fonts에 직접 요청하는 과정을 없애고, font loading과 fallback 처리를 Next.js build 과정에 통합할 수 있다.

```ts
import { Inter, Lusitana } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });

export const lusitana = Lusitana({
  subsets: ['latin'],
  weight: ['400', '700'],
});
```

Primary font는 Root Layout의 `body`에 적용해 application 전체에서 사용할 수 있고, secondary font는 필요한 element에만 `className`으로 적용할 수 있다.

### Subset과 Weight

`subset`은 Latin이나 Korean처럼 포함할 문자 집합을 제한한다. `weight`는 400과 700처럼 실제 사용할 font 굵기를 선택한다. 필요하지 않은 문자와 굵기까지 제공하면 font file과 network 전송량이 커질 수 있으므로 사용하는 범위만 지정한다.

```text
subset
-> 필요한 문자 집합

weight
-> 필요한 font 굵기
```

## next/image

일반 `<img>`를 사용할 때는 responsive image, 크기별 source, lazy loading과 Layout Shift 방지를 직접 고려해야 한다. Next.js의 `<Image>`는 다음 최적화를 제공한다.

- image가 들어갈 공간을 미리 확보해 Layout Shift 완화
- viewport와 설정에 맞는 크기의 image 제공
- viewport 밖 image의 lazy loading
- Browser가 지원할 때 WebP나 AVIF 같은 format 제공

```tsx
<Image
  src="/hero-desktop.png"
  width={1000}
  height={760}
  alt="Dashboard desktop screen"
/>
```

### Width와 Height

`width`와 `height`는 화면에 표시되는 CSS 크기를 항상 고정하는 값이 아니라 image의 고유 크기와 종횡비를 알려주는 값이다. Next.js와 Browser는 이 비율을 이용해 image가 load되기 전에 공간을 확보할 수 있다.

```text
width / height
-> 고유 크기와 비율 제공
-> Layout 공간 사전 확보

CSS와 부모 공간
-> 실제 화면 표시 크기 결정
```

따라서 `width={1000}`, `height={760}`을 지정해도 mobile에서는 허용된 공간과 CSS에 맞춰 같은 비율로 더 작게 표시될 수 있다.

## Public Asset

Next.js는 project 최상위 `public` 폴더를 site의 URL root에 연결한다.

```text
file system
public/hero-desktop.png

Browser URL
/hero-desktop.png
```

URL에 `/public`을 포함하지 않는다. 실습에서는 desktop과 mobile용 image를 각각 준비하고 responsive class로 화면 크기에 맞는 UI를 구성했다.

## 구현에서 확인한 내용

- Inter를 Root Layout에 적용해 전체 기본 font로 사용했다.
- Lusitana의 Latin subset과 400, 700 weight만 준비했다.
- Lusitana는 기존 text element에 직접 적용하고 유효한 HTML 구조를 유지했다.
- Desktop과 mobile image에 각각 고유 크기, 대체 text와 반응형 class를 지정했다.
- `<Image>`가 생성하는 최적화 image 경로와 화면 크기별 표시를 확인했다.

## 핵심 정리

> Web Font는 fallback font에서 custom font로 교체될 때 글자 크기와 간격이 달라져 Layout Shift를 만들 수 있다. `next/font`는 font를 build 시 준비하고 self-host해 외부 font 요청을 줄이며, 필요한 subset과 weight만 선택할 수 있다. `next/image`의 width와 height는 image의 고유 비율과 사전 공간 확보에 사용되고 실제 표시 크기는 CSS가 결정한다. 또한 크기 조정, lazy loading과 최신 format 제공 같은 image 최적화를 지원한다.

## 참고 자료

- [Next.js Learn: Optimizing Fonts and Images](https://nextjs.org/learn/dashboard-app/optimizing-fonts-images)
- [Next.js: Font](https://nextjs.org/docs/app/getting-started/fonts)
- [Next.js: Image Optimization](https://nextjs.org/docs/app/getting-started/images)
