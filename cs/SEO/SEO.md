# SEO

- [Canonical URL](#canonical-url)
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

## 구조화된 데이터

구조화된 데이터는 검색 엔진이 웹 페이지의 내용을 더 잘 이해할 수 있도록 도와주는 표준화된 형식의 데이터입니다. Google, Microsoft, Yahoo, Yandex 등이 공동으로 개발하여 구조화된 데이터를 표현하는 표준 스키마를 정의하는 프로젝트인 [schema.org](http://schema.org/)를 표준으로 삼고 있습니다. 정의된 스키마는 `Micodata, RDFa, JSON-LD` 의 세 가지 마크업 언어를 통해 지원됩니다. 그중 주로 사용하는 마크업 언어는 JSON-LD(JavaScript Object Notation for Linked Data) 이며 JSON형식을 사용하여, 웹 페이지의 데이터 구조를 정의합니다
