# Cache-Control

`Cache-Control`은 browser나 CDN이 HTTP response를 저장하고 재사용하는 방식을 제어하는 header입니다.

## Cache 종류

```text
Private Cache
-> 한 사용자의 browser cache

Shared Cache
-> CDN, reverse proxy처럼 여러 사용자가 공유하는 cache
```

## Fresh와 Stale

- Fresh: 유효기간이 남아 있어 server 확인 없이 재사용할 수 있는 상태
- Stale: 유효기간이 지나 server 재검증이나 새로운 response가 필요한 상태

## 주요 Directive

### max-age

Response를 몇 초 동안 fresh하게 사용할지 지정합니다.

```http
Cache-Control: max-age=600
```

10분 동안 browser는 server에 확인하지 않고 저장된 response를 사용할 수 있습니다.

### no-cache

Cache 저장은 허용하지만 재사용하기 전에 origin server 검증을 요구합니다.

```http
Cache-Control: no-cache
```

`no-cache`는 저장을 금지한다는 뜻이 아닙니다.

### no-store

Response를 cache에 저장하지 않도록 합니다.

```http
Cache-Control: no-store
```

결제 결과나 일회용 token처럼 민감한 response에 사용할 수 있습니다.

```text
no-cache
-> 저장 가능
-> 재사용 전에 검증

no-store
-> 저장하지 않음
```

### private

Browser 같은 private cache에는 저장할 수 있지만 CDN 같은 shared cache에는 저장하지 않도록 합니다.

```http
Cache-Control: private, no-cache
```

사용자별 profile response에 적용하면 browser는 저장 후 재검증할 수 있지만 CDN은 저장할 수 없습니다.

### s-maxage

CDN 같은 shared cache에만 적용되는 유효기간입니다.

```http
Cache-Control: public, max-age=60, s-maxage=600
```

```text
Browser
-> 60초 동안 fresh

CDN
-> 600초 동안 fresh
```

### stale-while-revalidate

Response가 stale 상태가 되어도 지정된 시간 동안 기존 response를 먼저 제공하면서 background에서 최신 response로 갱신할 수 있습니다.

```http
Cache-Control: max-age=60, stale-while-revalidate=300
```

최신성이 약간 늦어도 빠른 response가 중요한 상품 목록이나 게시글에 사용할 수 있습니다.

## ETag와 304

`no-cache` response를 재검증할 때 server는 `ETag`로 content version을 전달할 수 있습니다.

```http
ETag: "post-123-v2"
```

Browser는 다음 요청에서 자신이 가진 version을 보냅니다.

```http
If-None-Match: "post-123-v2"
```

Content가 변경되지 않았다면 server는 body 없이 `304 Not Modified`를 반환합니다. Browser는 기존 cache의 response body를 재사용하므로 전체 데이터를 다시 다운로드하지 않아도 됩니다.

## 사용 예시

```http
# 자주 변경되는 HTML
Cache-Control: no-cache
```

```http
# Hash가 포함된 정적 JS, CSS, image
Cache-Control: public, max-age=31536000, immutable
```

Hash가 포함된 asset은 content가 변경되면 URL도 바뀌므로 기존 URL을 오래 cache할 수 있습니다.

```http
# 사용자별 API response
Cache-Control: private, no-cache
```

```http
# CDN은 10분, browser는 1분
Cache-Control: public, max-age=60, s-maxage=600
```

## 면접 답변

> `Cache-Control`은 browser나 CDN이 HTTP response를 저장하고 재사용하는 방식을 제어하는 header입니다. `max-age`는 server 검증 없이 cache를 사용할 수 있는 시간을 지정하고, `no-cache`는 저장은 허용하지만 재사용 전에 server 검증을 요구합니다. 반면 `no-store`는 저장 자체를 금지합니다. 사용자별 response는 `private`, CDN의 별도 cache 시간은 `s-maxage`로 제어할 수 있습니다.

## 참고

- [RFC 9111: HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
- [MDN: HTTP Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching)
