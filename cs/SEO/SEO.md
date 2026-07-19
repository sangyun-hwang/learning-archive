# SEO

- [Canonical URL](#canonical-url)
- [Trailing Slash](#trailing-slash)
- [HTTP Redirects](#http-redirects)
- [구조화된 데이터](#구조화된-데이터)

<br>

## Canonical URL

Canonical URL은 동일하거나 매우 유사한 콘텐츠에 여러 URL로 접근할 수 있을 때 검색엔진이 대표로 평가하고 검색 결과에 표시할 URL입니다. Canonicalization은 여러 중복 URL 중 대표 URL을 선택하는 과정입니다.

```text
https://example.com/products/1
https://example.com/products/1?utm_source=google
https://example.com/products/1?ref=home
```

위 주소들이 같은 상품을 보여준다면 파라미터가 없는 `/products/1`을 대표 URL로 지정할 수 있습니다.

```html
<head>
  <link
    rel="canonical"
    href="https://example.com/products/1"
  />
</head>
```

Canonical은 중복 페이지를 삭제하거나 사용자를 다른 페이지로 이동시키는 기능이 아닙니다. 검색엔진에 선호하는 대표 URL을 알려주는 강한 힌트이며, 검색엔진이 반드시 따라야 하는 명령은 아닙니다.

### 필요한 이유

- 중복 URL에 분산될 수 있는 링크와 검색 신호를 대표 URL로 통합합니다.
- 검색 결과에 표시할 URL을 명확하게 전달합니다.
- 같은 콘텐츠의 성과 지표가 여러 URL로 나뉘는 것을 줄입니다.
- 검색엔진이 중복 URL보다 새로운 콘텐츠를 크롤링하는 데 시간을 쓰도록 돕습니다.

중복 콘텐츠가 존재한다는 사실만으로 사이트 평가가 자동으로 낮아지는 것은 아닙니다. Canonical의 목적은 중복 콘텐츠 패널티를 피하는 것이라고 단순화하기보다, 대표 URL 선택과 검색 신호 통합으로 이해하는 것이 정확합니다.

색상이나 옵션이 다른 상품도 항상 하나의 canonical로 합치는 것은 아닙니다. 각 페이지의 콘텐츠와 검색 의도가 실질적으로 다르다면 각각 자신을 대표 URL로 지정할 수 있습니다.

### Self-referencing canonical

대표 페이지 자신도 자기 URL을 canonical로 지정하는 것이 권장됩니다.

```html
<!-- https://example.com/products/1 -->
<link
  rel="canonical"
  href="https://example.com/products/1"
/>
```

대표 페이지에도 추적 파라미터 등이 붙을 수 있으므로, self-referencing canonical은 어떤 주소가 대표인지 일관되게 전달하는 데 도움이 됩니다.

### Redirect, Canonical, Sitemap 비교

| 방식 | 역할 | 적합한 경우 |
| --- | --- | --- |
| Permanent redirect | 사용자와 검색엔진을 새 URL로 실제 이동시킴 | 기존 URL을 폐기하고 완전히 이전할 때 |
| `rel="canonical"` | 접근 가능한 여러 URL 중 대표 URL을 제안함 | 필터, 정렬, 추적 파라미터처럼 중복 URL을 유지해야 할 때 |
| Sitemap | 검색엔진에 주요 URL 목록을 제출함 | 사이트에서 색인되기를 원하는 대표 URL을 알릴 때 |

Redirect와 `rel="canonical"`은 강한 신호이고 sitemap 포함은 상대적으로 약한 신호입니다. 여러 방법을 함께 사용한다면 모두 같은 대표 URL을 가리켜야 합니다.

```text
/old-product/1 -> /products/1
```

`/old-product/1`을 완전히 폐기했다면 canonical만 남기기보다 permanent redirect로 사용자를 새 주소로 이동시키는 것이 적합합니다.

### noindex와의 차이

```html
<meta name="robots" content="noindex" />
```

- `noindex`: 해당 페이지를 검색 결과에 색인하지 말라는 지시입니다.
- `canonical`: 유사한 URL 중 대표 URL을 고려하고 검색 신호를 통합해 달라는 힌트입니다.

Canonical이 지정된 중복 URL은 계속 접근하고 크롤링할 수 있습니다. 대표 페이지에 `noindex`를 지정하면서 다른 페이지에서 그 URL을 canonical로 가리키면, 대표 URL로 선택하라는 신호와 색인하지 말라는 신호가 충돌하므로 피해야 합니다.

`robots.txt`로 크롤링을 차단하는 것도 canonical 대체 수단이 아닙니다. 검색엔진이 페이지 내용을 확인하지 못해 canonical 신호를 읽지 못할 수 있습니다.

### Next.js App Router에서 설정하기

Next.js에서는 Metadata API의 `alternates.canonical`을 사용할 수 있습니다.

```tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  metadataBase: new URL('https://example.com'),
  alternates: {
    canonical: '/products/1',
  },
};
```

동적 페이지에서는 `generateMetadata`로 각 페이지의 대표 URL을 만듭니다.

```tsx
import type { Metadata } from 'next';

type Props = {
  params: Promise<{ id: string }>;
};

export async function generateMetadata({
  params,
}: Props): Promise<Metadata> {
  const { id } = await params;

  return {
    alternates: {
      canonical: `/products/${id}`,
    },
  };
}
```

`metadataBase`를 설정하면 상대 경로로 작성한 canonical을 절대 URL로 만들 수 있습니다. 최종 HTML의 `<head>`에 의도한 절대 URL이 생성됐는지 확인해야 합니다.

### 주의사항

- HTML 문서에서는 canonical link를 유효한 `<head>` 안에 작성합니다.
- 실수로 개발·테스트 도메인을 가리키지 않도록 절대 URL 사용을 권장합니다.
- Canonical 대상은 접근 가능하고 색인할 수 있는 대표 페이지여야 합니다.
- 내용이 크게 다른 페이지들을 하나의 canonical URL로 묶지 않습니다.
- Canonical, redirect, sitemap, 내부 링크가 같은 대표 URL을 가리키도록 일관성을 유지합니다.
- 사이트 내부 링크에서도 파라미터 URL보다 대표 URL을 사용합니다.
- 각 언어 페이지는 보통 같은 언어의 대표 URL을 canonical로 지정하고 `hreflang`과 역할을 구분합니다.

Canonical이 `/products/1`을 가리키는데 sitemap과 내부 링크가 계속 `/products/1?ref=home`을 사용하면 검색엔진에 상충하는 신호를 줍니다. 파라미터 URL을 반복해서 크롤링하거나 검색엔진이 다른 canonical을 선택할 가능성을 줄이려면 모든 신호를 일치시켜야 합니다.

### 정리

> Canonical URL은 중복 페이지를 삭제하는 기능이 아니라, 여러 유사 URL 중 검색 신호를 통합할 대표 URL을 검색엔진에 제안하는 방식입니다. 기존 URL을 폐기할 때는 redirect를 사용하고, URL을 유지해야 할 때는 canonical을 사용합니다. noindex는 페이지 자체의 색인을 막는 지시이므로 canonical과 목적이 다릅니다.

### 참고

- [Google Search Central: Canonical URL 지정 방법](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls)
- [Google Search Central: URL canonicalization](https://developers.google.com/search/docs/crawling-indexing/canonicalization)
- [Next.js: generateMetadata](https://nextjs.org/docs/app/api-reference/functions/generate-metadata)

## Trailing Slash

Trailing slash는 URL 경로 마지막에 붙는 `/`를 말합니다.

```text
https://example.com/about
https://example.com/about/
```

두 주소가 같은 콘텐츠를 보여주더라도 URL 관점에서는 서로 다른 경로입니다. 서버 설정에 따라 같은 리소스를 반환할 수도 있고 서로 다른 리소스를 반환할 수도 있습니다.

두 URL을 함께 사용하면 다음과 같은 문제가 생길 수 있습니다.

- 내부 링크와 외부 링크가 두 주소로 분산될 수 있습니다.
- 검색엔진이 두 URL을 각각 크롤링할 수 있습니다.
- 캐시와 분석 데이터가 별도로 쌓일 수 있습니다.
- Redirect, canonical, sitemap이 서로 다른 URL을 가리킬 수 있습니다.

Slash를 항상 붙이는 방식과 제거하는 방식 중 SEO에 절대적으로 더 유리한 것은 없습니다. 한 가지 규칙을 정하고 사이트 전체에서 일관되게 사용하는 것이 중요합니다.

```text
Slash를 제거하는 규칙
/about/ -> /about

Slash를 붙이는 규칙
/about -> /about/
```

선택하지 않은 형태는 선택한 대표 URL로 redirect하고, canonical, sitemap, 내부 링크도 모두 같은 URL을 사용합니다.

```text
Redirect:      /about/ -> /about
Canonical:     https://example.com/about
Sitemap:       https://example.com/about
Internal link: /about
```

### Next.js 설정

Next.js는 기본적으로 trailing slash가 있는 URL을 없는 형태로 redirect합니다.

```text
/about/ -> /about
```

반대로 모든 페이지에 trailing slash를 붙이려면 `next.config.ts`에서 설정합니다.

```ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  trailingSlash: true,
};

export default nextConfig;
```

이 경우 `/about`은 `/about/`으로 redirect됩니다. 확장자가 있는 정적 파일과 `.well-known` 아래의 경로 등에는 trailing slash가 추가되지 않습니다.

## HTTP Redirects

HTTP redirect는 서버가 `3xx` 응답과 `Location` 헤더를 반환해 클라이언트가 다른 URL로 다시 요청하도록 하는 방식입니다.

```http
HTTP/1.1 301 Moved Permanently
Location: https://example.com/new-page
```

Redirect는 canonical과 달리 사용자도 실제로 새로운 URL로 이동시킵니다.

### 301 Moved Permanently

`301`은 리소스가 영구적으로 다른 URL로 이동했다는 뜻입니다.

```text
/old-product/1 -> /products/1
```

다음과 같은 상황에 적합합니다.

- 페이지 URL을 영구적으로 변경한 경우
- 사이트를 새로운 도메인으로 이전한 경우
- HTTP를 HTTPS로 통일하는 경우
- www 유무나 trailing slash 규칙을 하나로 통일하는 경우
- 삭제한 페이지를 대응되는 새 페이지로 이전하는 경우

검색엔진은 permanent redirect를 redirect 대상이 대표 URL이어야 한다는 강한 신호로 사용합니다. 브라우저와 중간 캐시에 오래 남을 수 있으므로 다시 원래 URL로 되돌릴 임시 상황에는 사용하지 않습니다.

### 302 Found

`302`는 현재 다른 URL로 이동시키지만 원래 URL을 계속 사용할 예정인 임시 redirect입니다.

```text
/dashboard -> /maintenance
```

다음과 같은 상황에 적합합니다.

- 일시적인 서비스 점검
- 단기간 진행하는 이벤트나 실험
- 원래 URL의 검색 노출을 유지해야 하는 임시 이동

검색엔진은 일반적으로 원래 URL을 대표로 유지합니다. Redirect 대상도 다른 신호에 의해 별도로 색인될 수 있지만, `302` 자체는 대상을 영구적인 대표 URL로 선택하라는 신호가 아닙니다.

### 301, 302, 307, 308 비교

전통적인 `301`과 `302`는 원래 요청이 `POST`여도 redirect 이후 요청을 `GET`으로 변경할 수 있습니다. `307`과 `308`은 기존 HTTP 메서드와 request body를 그대로 유지하도록 보장합니다.

| 상태 코드 | 의미 | 영구 여부 | 메서드와 body 보존 |
| --- | --- | --- | --- |
| `301` | Moved Permanently | 영구 | 보장하지 않음 |
| `302` | Found | 임시 | 보장하지 않음 |
| `307` | Temporary Redirect | 임시 | 보장 |
| `308` | Permanent Redirect | 영구 | 보장 |

```text
302를 받은 경우
POST /old-api -> GET /new-api로 바뀔 수 있음

307을 받은 경우
POST /old-api -> POST /new-api로 유지됨
```

일반적인 `GET` 페이지 이동에서는 메서드 변경 문제가 드러나지 않지만, API 요청이나 form 제출에서는 의미가 달라질 수 있습니다.

POST 처리 완료 후 결과 페이지를 `GET`으로 조회하게 만들려는 경우에는 `303 See Other`를 사용할 수 있습니다. 이는 메서드 보존이 필요한 `307`과 목적이 다릅니다.

### Next.js에서 Redirect 사용하기

Next.js는 메서드가 의도치 않게 변경되는 문제를 피하기 위해 설정 기반 redirect에서 `307`과 `308`을 사용합니다.

```ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  async redirects() {
    return [
      {
        source: '/old-product/:id',
        destination: '/products/:id',
        permanent: true,
      },
    ];
  },
};

export default nextConfig;
```

- `permanent: false`: `307 Temporary Redirect`
- `permanent: true`: `308 Permanent Redirect`
- `redirect()`: 일반적으로 `307 Temporary Redirect`
- `permanentRedirect()`: `308 Permanent Redirect`
- Server Action의 `redirect()`: POST 처리 후 이동에 맞는 `303 See Other`

```tsx
import { permanentRedirect } from 'next/navigation';

export default async function OldProfilePage() {
  permanentRedirect('/profile/new-name');
}
```

### Redirect 설계 시 주의사항

- 영구 이동과 임시 이동을 구분해 상태 코드를 선택합니다.
- Redirect 대상은 사용자가 기대하는 대응 콘텐츠여야 합니다.
- 삭제된 모든 페이지를 관련 없는 홈페이지로 보내지 않습니다.
- Redirect 규칙과 canonical, sitemap, 내부 링크를 같은 대표 URL로 맞춥니다.
- Redirect chain과 loop를 피합니다.

```text
나쁜 예
/about/ -> /about -> /company

좋은 예
/about/ -> /company
/about  -> /company
```

Redirect가 여러 번 이어지면 요청 횟수와 응답 시간이 늘고 크롤링도 비효율적이 됩니다. 서로를 가리키는 규칙을 만들면 무한 redirect가 발생해 페이지에 접근할 수 없습니다.

### 정리

> Trailing slash의 유무 자체보다 한 가지 URL 규칙을 선택해 redirect, canonical, sitemap, 내부 링크에 일관되게 적용하는 것이 중요합니다. 영구적인 URL 이전에는 301 또는 308을, 임시 이동에는 302 또는 307을 사용합니다. 307과 308은 기존 요청 메서드와 body를 보존하며, Next.js는 이 차이를 명확히 하기 위해 설정 기반 redirect에서 307과 308을 사용합니다.

### 참고

- [Google Search Central: Redirects and Google Search](https://developers.google.com/search/docs/crawling-indexing/301-redirects)
- [Next.js: trailingSlash](https://nextjs.org/docs/app/api-reference/config/next-config-js/trailingSlash)
- [Next.js: Redirecting](https://nextjs.org/docs/app/guides/redirecting)
- [Next.js: redirects](https://nextjs.org/docs/app/api-reference/config/next-config-js/redirects)

## 구조화된 데이터

구조화된 데이터는 검색 엔진이 웹 페이지의 내용을 더 잘 이해할 수 있도록 도와주는 표준화된 형식의 데이터입니다. Google, Microsoft, Yahoo, Yandex 등이 공동으로 개발하여 구조화된 데이터를 표현하는 표준 스키마를 정의하는 프로젝트인 [schema.org](http://schema.org/)를 표준으로 삼고 있습니다. 정의된 스키마는 `Micodata, RDFa, JSON-LD` 의 세 가지 마크업 언어를 통해 지원됩니다. 그중 주로 사용하는 마크업 언어는 JSON-LD(JavaScript Object Notation for Linked Data) 이며 JSON형식을 사용하여, 웹 페이지의 데이터 구조를 정의합니다
