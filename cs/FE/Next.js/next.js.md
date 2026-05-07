# next.js

- [React Server Component](#react-server-component)
- [Streaming](#streaming)
- [next/link](#nextlink)
- [next/router](#nextrouter)
- [next/image](#nextimage)
- [Data fetching](#data-fetching)

<br>

## React Server Component

React Server Component(줄여서 RSC)는 React18부터 도입된 개념으로, 말 그대로 서버에서 동작하는 컴포넌트를 가리킨다. 그와 반대 되는 개념인 클라이언트 컴포넌트는 React18 이전 버전에서 우리가 사용하던 모든 컴포넌트를 말한다. 가장 큰 차이점은 컴포넌트의 렌더링 장소가 서버에서 이루어지는가 클라이언트에서 이루어지는가에 차이를 가진다.

next.js 공식문서에서는 다음과 같이 구분한다.

![img](https://velog.velcdn.com/images/2ast/post/226b2cd3-4cb5-47bf-b52c-f0087b4acc3b/image.png)

다이어그램으로 표현하면 다음과 같다.

![서버컴포넌트와 클라이언트 컴포넌트](https://s3-ap-northeast-2.amazonaws.com/opentutorials-user-file/module/6341/13043.png)

### RSC의 동작 방식

사용자가 페이지에 접속 요청을 보내면 서버는 컴포넌트 트리를 실행하고, 이를 JSON 형태로 직렬화하여 클라이언트로 전달합니다. 이미 완성된 RSC는 직렬화된 결과를 그대로 클라이언트로 전송합니다. 반면, React Client Component (RCC)는 클라이언트에서 JavaScript를 통해 렌더링되므로 직렬화하지 않고 클라이언트로 넘어갑니다.

![img](https://velog.velcdn.com/images/2ast/post/465f8024-69c3-47ee-8b6f-1ee3d75b4fda/image.png)

RCC가 위치할 자리는 비워지며, 해당 자리에 대한 정보는 `module reference`라는 새로운 타입으로 지정됩니다. 이 정보는 컴포넌트의 경로를 명시하며, 클라이언트는 이를 활용하여 RCC가 나중에 렌더링될 위치를 파악합니다.

![img](https://velog.velcdn.com/images/2ast/post/7ef55ef0-4bef-417d-9e5f-08342345346f/image.png)

클라이언트는 서버로부터 결과물과 JavaScript 번들을 받아 초기 화면을 구성하며, `module reference`가 등장하면 해당 컴포넌트의 JavaScript 번들을 동적으로 가져와 렌더링합니다. 이로써 빈 공간이 RCC로 채워지고 최종적으로 완성된 화면이 사용자에게 표시됩니다.

### Next.js의 SSR

Next.js의 SSR은 기존의 클라이언트 사이드 렌더링 (CSR)과 서버 사이드 렌더링 (SSR)의 장점을 결합한 형태입니다. 초기 로딩 속도 문제를 갖고 있던 CSR의 단점을 극복하기 위해, Next.js는 초기 로딩 시에 HTML 파일을 서버 사이드 렌더링을 통해 신속하게 받아오고, 그 후에 병렬적으로 JavaScript 번들도 가져와서 클라이언트 측에서 이미 받아온 HTML과 병합하는 과정을 거칩니다. 이 과정은 보통 `hydration`이라고 불립니다. 이러한 방식으로 Next.js의 SSR은 빠른 로딩 속도를 갖는 SSR의 강점과 인터랙션에 더 적합한 CSR의 강점을 모두 활용할 수 있게 되었습니다.

### RSC 장점

- zero bundle size

RSC는 이미 서버에서 실행된 후 직렬화된 JSON 형태로 전달됩니다. 즉, 서버에서 외부 라이브러리의 실행이나 참조 모듈의 실행이 완료된 상태이기 때문에 어떠한 번들도 필요하지 않습니다. 이러한 특성이 Next의 TTI(Time To Interactive) 개선에 크게 기여할 수 있습니다. Next.js의 일반적인 SSR 방식을 사용하더라도 초기 로딩 속도만 빠를 뿐, 상호작용을 위해서는 여전히 CSR과 동일한 크기의 JavaScript 번들을 다운로드해야 하므로 TTI는 여전히 CSR과 비교했을 때 큰 메리트가 없습니다. 하지만 RSC를 도입하게 되면 다운로드해야 하는 번들 사이즈가 줄어들어 TTI가 크게 개선될 수 있습니다.

- Automatic Code Splitting

원래 코드 스플리팅을 위해서는 React.Lazy나 동적 import를 사용해야 했습니다. 그러나 RSC에서 RCC를 import하는 경우에는 자동으로 RCC가 dynamic import됩니다. 이 장점은 어떻게 보면 당연한 사실인데, RSC가 서버에서 렌더링될 때 RCC는 실행되지 않기 때문에 RCC를 즉시 import할 필요가 없습니다.

- 컴포넌트 단위 refetch

SSR의 경우 완성된 HTML 파일을 전송하기 때문에 작은 변경 사항이 발생하더라도 전체 페이지를 다시 받아와야 했습니다. 그러나 앞서 설명한대로 RSC는 최종 결과물이 HTML이 아니라 직렬화된 JSON 형태의 데이터를 받아옵니다. 클라이언트는 이 JSON을 해석하여 가상 DOM을 형성하고 화면을 갱신합니다. 따라서 화면에 변경사항이 생겨 서버에서 새로운 정보를 받아와야 할 상황이 오더라도, 기존 화면의 상태와 컨텍스트를 유지한 채로 변경된 사항만 선택적으로 반영할 수 있습니다. 이는 기존 화면을 완전히 교체하는 것이 아니라 필요한 부분만 업데이트하는 형태로 이루어지게 됩니다.

## Streaming

React 및 Next.js에서 스트리밍이 작동하는 방식을 배우기위해 사전에 SSR(서버 사이드 렌더링)과 그 한계에 대하여 이해하는 것이 좋습니다.

SSR을 통해 사용자가 페이지를 보기 전에 아래의 과정을 거칩니다.

![Chart showing Server Rendering without Streaming](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Fserver-rendering-without-streaming-chart.png&w=1920&q=75&dpl=dpl_6NUrNrbyMhexcnM4rTkyd8NkejGW)

1. 서버에서 페이지의 데이터를 가져옵니다.
2. 서버는 페이지의 HTML을 렌더링합니다.
3. HTML, CSS 및 JavaScript 파일을 클라이언트로 전송합니다.
4. 클라이언트는 먼저 HTML과 CSS를 사용하여 페이지의 정적인 부분을 렌더링합니다.
5. JavaScript 파일이 로드되면 클라이언트 측에서 페이지를 활성화하고 동적 요소를 화면에 표시합니다.

이러한 과정은 순차적으로 진행되며, Hydration이 완료되기 전까지 사용자는 페이지와 상호작용할 수 없습니다. 이로 인해 초기 렌더링은 빠르지만 TTI까지는 완전하지 않습니다.

Next.js 버전 13부터 도입된 Streaming은 이러한 문제를 해결하기 위해 Server Component를 활용해 페이지의 HTML을 작은 청크로 나누어 클라이언트에 병렬로 전송합니다.

![How Server Rendering with Streaming Works](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Fserver-rendering-with-streaming.png&w=3840&q=75&dpl=dpl_6NUrNrbyMhexcnM4rTkyd8NkejGW)

이 방법을 통해 우선순위가 높거나 데이터에 의존하지 않는 컴포넌트(예: 레이아웃)가 먼저 전송되어 브라우저에서 더 빠른 Hydration이 가능해집니다. 그리고 데이터 패칭이 필요한 컴포넌트나 낮은 우선순위의 요소는 데이터를 가져온 후 클라이언트로 전송됩니다.

![Chart showing Server Rendering with Streaming](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Fserver-rendering-with-streaming-chart.png&w=1920&q=75&dpl=dpl_6NUrNrbyMhexcnM4rTkyd8NkejGW)

따라서 Streaming을 사용하면 페이지가 렌더링되는 중에도 이미 Hydration이 완료된 컴포넌트는 사용자와 상호작용할 수 있습니다. 이것은 사용자 경험을 크게 향상시키는 데 도움이 됩니다.

## next/link

`<a>` 태그를 통해 링크를 이동하면, 대게 해당 페이지에 대한 정보를 미리 가져오지 않고 이동 시에 새로운 HTML을 받아와야 합니다. 반면, next/link 컴포넌트를 사용하면 링크가 생성된 페이지 정보를 사전에 가져와 JavaScript 파일로 미리 로드하게 됩니다. 이를 통해 페이지 이동 전에 필요한 데이터를 사전에 가져와 클라이언트 측에서 CSR과 유사한 방식으로 처리할 수 있습니다. 또한, next/link는 프리렌더링을 통해 서버에서 HTML을 생성하므로 SEO 측면에서도 문제가 발생하지 않습니다.

### Props

#### href(필수)

이동할 경로나 URL을 지정합니다.

```javascript
import Link from "next/link";

// <Link href="/링크경로">링크이름</Link>;
<Link href="/section1/getStaticProps">/getStaticProps</Link>;
```

![image-20230713231356303](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20230713231356303.png)

렌더링 후 element 상에서는 `<a>`태그로서 동작한다.

#### replace(default: false)

현재의 기록 상태(history state)를 대체하여 새로운 URL을 브라우저 기록 스택에 추가해줍니다.

- 활용 예시

게시글을 읽기 위해 리캡챠나 로그인과 같은 인증 절차가 필요한 사이트의 경우, 사용자가 게시글을 읽은 후에 게시판 목록으로 돌아가려는 경우가 종종 있습니다. 그러나 기존의 방식으로는 이미 완료한 인증 절차를 다시 거쳐야 하고, 다시 한 번 뒤로가기 버튼을 클릭해야 하는 번거로움이 생길 수 있습니다. 이러한 상황에서 replace 속성을 활용하면, 사용자가 게시글 페이지에서 뒤로가기 버튼을 클릭할 때, 이미 수행한 인증 페이지로 되돌아가지 않고 바로 게시판 목록 페이지로 이동할 수 있습니다. 이로써 사용자의 경험을 개선하고 의도한 작업을 보다 원활하게 수행할 수 있게 됩니다.

#### prefetch(default: true)

백그라운드에서 페이지(주소로 표시되는 페이지)를 프리로딩합니다.뷰포트 안에 있는 모든 <Link />는 프리로드되며 뷰포트 바깥에 있을 경우 스크롤을 통해 <Link />가 뷰포트 내에 들어온 순간 프리로드 됩니다. prefetch는 프로덕션 환경에서만 활성화 되고 개발환경에서는 동작하지 않습니다.

- 예시

```javascript
import Link from "next/link";

export default function Links() {
  return (
    <main>
      <h1>Links</h1>
      <div style={{ height: "200vh" }}></div>
      <Link href="/section1/getStaticProps">/getStaticProps</Link>
    </main>
  );
}
```

위 코드와 같이 `<Link>`태그를 페이지에 보이는 영역 바깥에 위치하게 될 경우 `<Link>`태그가 보이지 않을 때는 링크 페이지가 프리로드되지 않다가 해당 영역에 스크롤이 닿았을 때 프리로드 됩니다.

![image-20230713222227408](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20230713222227408.png)
![image-20230713222247138](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20230713222247138.png)

#### scroll(default: true)

<Link>가 새 경로로 이동할 때 페이지의 맨 위로 스크롤해줍니다. 뒤로/앞으로 탐색하는 경우에는 이전의 스크롤 위치를 유지해 줍니다. false로 할 경우 <Link>가 페이지 이동 후 페이지 맨 위로 스크롤하지 않습니다.

## next/router

> [next/link](#nextlink)의 내용 참고

`next/router`는 Next.js에서 제공하는 라우팅 관련 기능을 사용할 수 있는 모듈입니다. `next/link`와 유사한 역할을 하며, 클라이언트 사이드에서의 라우팅을 지원합니다. 그러나 `next/link`와 달리 프리로딩 기능을 제공하지는 않습니다.

- 사용법

```jsx
import { useRouter } from "next/router";

export default function Links() {
  const router = useRouter();

  return (
    <button
      onClick={() => {
        router.push("/section1/getStaticProps");
      }}
    >
      /getStaticProps
    </button>
  );
}
```

`const router = useRouter();`를 선언한 후 onClick에 콜백 함수로 페이지 링크를 push 해주면됩니다.

### useRouter 메서드

- router.push: 지정한 경로로 클라이언트 사이드 네비게이션을 수행하며, 히스토리 스택에 기록합니다.
- router.replace: 경로가 이동되며, 현재의 히스토리 스택을 이동한 경로로 교체합니다.
- router.refresh: 현재 경로를 새로고침합니다. 서버에 새로 요청을 보내 데이터를 리패치하고 서버 컴포넌트를 리렌더링합니다. 클라이언트는 업데이트 된 서버 컴포넌트 데이터를 받아와서 클라이언트의 리액트 상태(ex: useState)나 브라우저의 상태(ex: 스크롤 위치)를 초기화 하지 않고 업데이트합니다.
- router.back: 히스토리 스택에서 이전 경로로 이동합니다.
- router.forward: 히스토리 스택에서 다음 경로로 이동합니다.

## next/image

이미지는 일반적인 웹사이트의 크기의 상당히 많은 부분을 차지합니다.이는 웹사이트의 LCP 성능에 상당한 영향을 미칠수 있습니다.

Next.js에서는 `<Image/>` 컴포넌트를 사용하여 이미지를 자동으로 최적화하여 LCP 성능을 개선합니다.

#### 이미지 포멧 변경

jpeg, png등의 원본 파일을 Webp 나 AVIF 같은 최신의 이미지 형식으로 자동으로 변환하여 이미지 용량을 크게 줄여줍니다. `quality` 속성을 통해 이미지 최적화율을 조정할 수 있습니다.(default: 75)

#### 시각적인 안정

웹사이트를 이용하는 도중 메뉴를 클릭했는데 갑자기 광고나 이미지가 그자리에 로딩되어 의도하지 않은 페이지로 이동하는 경험이 있을 것이다.이러한 현상을 `Cumulative Layout Shift` 줄여서 `CLS`라고 부릅니다. Next는 이러한 현상을 방지하기 위해 기본적으로 `width`, `height`의 값을 받아 이미지가 로딩되는 동안 해당 영역을 비워 둡니다. 이를 통해 페이지 로딩이 완료되는 사이에 레이아웃이 변경되는 것을 방지합니다. `placeholder`속성을 사용하여 비어있는 영역을 blur이미지나 대체 이미지로 채워놓을 수도 있습니다.

#### lazy loading

이미지가 뷰포트 바깥에 있을 때는 로딩을 지연시키다가 뷰포트 내부로 들어왔을 경우 로드하는 기술입니다.눈에 보이는 부분만 먼저 로드하고 그외의 부분은 지연시킴으로서 페이지의 초기로딩을 개선시킬수 있습니다. `<img/>`태그에 `loading="lazy"`속성을 추가하는 것으로도 적용할 수 있으나 next/image는 이를 자동으로 적용시켜줍니다.

### 캐싱 동작

이미지는 요청 시 동적으로 최적화되어 `<distDir>/cache/images` 디렉터리에 저장됩니다. 이 최적화된 이미지 파일은 만료되기 전까지 후속 요청에 대해 제공됩니다. 이미 캐시되었지만 만료된 파일과 일치하는 요청이 발생하면, 이전 버전의 이미지가 해당 요청에 사용됩니다(이를 "stale"이라고도 합니다). 이후 이미지는 백그라운드에서 다시 최적화되며, 새로운 만료 날짜와 함께 캐시에 저장됩니다.

응답 헤더의 x-nextjs-cache 값을 읽음으로써 Next 이미지의 캐시 상태를 확인할 수 있습니다.

- MISS: 해당 경로의 이미지가 캐시에 없습니다. (첫 방문 시에 한 번만 발생)
- STALE: 해당 경로의 이미지는 캐시에 있지만, 만료 시간을 초과하여 백그라운드로 업데이트됩니다.
- HIT: 해당 경로의 이미지가 캐시에 있으며, 만료 시간동안 유효합니다.

만료시간은 `minimumCacheTTL`과 `Cache-Control`를 통해 조절할 수 있습니다.

### 사용법

```jsx
import Image from "next/image";
```

#### 로컬 이미지

로컬 이미지 는 .`jpg`, `.png`, `.webp`등의 파일을 import 구문으로 가져와서 사용합니다. Next가 사전에 이미지의 크기를 알 수 있기 때문에 `width`, `height`의 값을 생략할 수 있습니다. 이 사전에 측정된 크기를 통해 `CLS`를 방지합니다. 빌드시 분석되기 때문에 `다이나믹 import`나 `require문`은 지원하지 않습니다.

```jsx
import Image from "next/image";
import profilePic from "./me.png";

export default function Page() {
  return <Image src={profilePic} alt="Picture of the author" />;
}
```

#### 원격 이미지

`next/image`의 src속성을 문자열이나 URL로 사용하는 경우입니다. Next.js는 빌트 타입에 원격 파일에 액세스할 수 없기 때문에 `width`, `height`값을 제공해줘야 합니다. 이미지의 사이즈를 모를 경우 `fill`속성을 통해 부모의 사이즈로 대체할 수 있습니다.

```jsx
import Image from "next/image";

export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  );
}
```

Next.js는 기본적으로 외부사이트를 허용하지 않기 때문에 `next.config.js`파일에 예외로 할 URL 패턴 목록을 정의해야됩니다.

```jsx
// next.config.js

module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "s3.amazonaws.com",
        port: "",
        pathname: "/my-bucket/**",
      },
    ],
  },
};
```

만일 해당 도메인에서 제고오디는 모든 컨텐츠를 소유한 경우에는 `domains`을 사용해도 됩니다. 그외에는 앱을 보호하기 위해 `remotePatterns`으로 엄격한 구성을 권장합니다.

```jsx
// next.config.js

module.exports = {
  images: {
    domains: ["assets.acme.com"],
  },
};
```

## Data fetching

### Page router (Next 13 이전)

#### getStaticProps

SSG 방식에서 사용됩니다. SSG는 빌드 시에 페이지를 생성하며, 이후 변경이 없으면 한 번만 호출됩니다. 빌드 타임에 데이터를 가져오므로 초기 페이지 로딩 속도가 매우 빠릅니다. 데이터 변경이 필요한 경우 페이지를 재빌드해야 합니다.

#### getStaticPaths

페이지가 동적 라우팅을 사용하면서 `getStaticProps`도 사용할 경우, `getStaticPaths`를 추가하여 정적으로 렌더링할 경로를 설정합니다. 이렇게 하지 않으면 해당 경로로 접근 시 404 페이지가 표시됩니다. 따라서 동적 라우팅 시 모든 가능한 경우의 경로를 정의해야 합니다.

#### getServerSideProps

SSR 방식에서 사용됩니다. 각 페이지 요청마다 서버에서 데이터를 가져와 페이지 컴포넌트의 props로 전달합니다. 서버 사이드에서 데이터를 가져오기 때문에 항상 최신 데이터를 보여줄 수 있습니다. 실시간 업데이트가 필요한 경우에 적합합니다.

### App router

데이터를 가져오기 위해 `fetch()`를 사용합니다. 서버 컴포넌트의 경우 `async/await`를 사용하여 비동기적으로 데이터를 가져옵니다. 기존 브라우저의 `Fetch API`를 확장한 API를 Next에서 제공해 줍니다. 별도 설정을 하지않아도 Next가 자동으로 데이터를 캐시해줍니다.
