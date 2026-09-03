# Chapter 10. Adding Search and Pagination

## 학습 목표

검색어와 페이지 번호를 URL로 관리하고, Client의 입력이 Server Component의 DB 조회와 화면 갱신으로 이어지는 흐름을 이해한다.

## 검색 상태를 URL에 두는 이유

```text
/dashboard/invoices?query=lee&page=2

query -> 검색어
page  -> 현재 페이지 번호
```

검색 조건을 Component State에만 두면 새로고침하거나 URL을 공유할 때 해당 조건을 복원하기 어렵다. Search Params를 사용하면 같은 URL로 접근했을 때 동일한 검색 조건을 적용할 수 있고, Server에서도 이를 조회 조건으로 읽을 수 있다.

같은 조건이라도 DB 변경이나 사용자 권한에 따라 결과는 달라질 수 있다. URL에 검색어가 있다는 사실만으로 SEO가 좋아지는 것은 아니다.

## Client 입력에서 Server 조회까지

```text
검색창 onChange
-> 300ms Debounce
-> URLSearchParams로 query 변경, page를 1로 초기화
-> router.replace()로 Navigation
-> Next.js가 URL을 해석해 Server Page에 searchParams 제공
-> Page가 searchParams를 await하여 검색 조건 추출
-> Table이 조건에 맞는 Invoice를 DB에서 조회
-> RSC Payload를 통해 화면 갱신, 공유 Layout 유지
```

Client가 읽은 값을 직접 Server Component의 Props로 보내는 방식이 아니다. URL로 Navigation하면 Next.js가 그 요청의 정보를 Page에 제공한다.

| 위치 | 읽는 방법 | 용도 |
| --- | --- | --- |
| Client Search, Pagination | `useSearchParams()` | 현재 조건을 읽고 다음 URL 구성 |
| Server Page | `await props.searchParams` | DB 조회에 사용할 검색어와 페이지 번호 추출 |

실습의 Search는 `usePathname()`으로 현재 경로를 읽고, `URLSearchParams`로 조건을 수정한다. 검색어가 비어 있으면 `query`를 삭제한다. Input의 `defaultValue`는 URL의 검색어로 초기 입력값을 채우며, 이후 모든 URL 변경을 자동으로 동기화하는 제어 값은 아니다.

## `replace()`와 Debounce

`push()`는 방문 기록을 추가하고 `replace()`는 현재 기록을 교체한다. 검색어가 바뀔 때마다 `push()`하면 뒤로 가기를 누를 때 이전 검색어를 하나씩 거슬러 올라가게 될 수 있어, 실습에서는 `replace()`를 사용한다.

Debounce는 마지막 입력 이후 300ms 동안 추가 입력이 없을 때 검색을 실행한다. 타이핑할 때마다 Navigation과 DB 조회를 시작하는 횟수를 줄이기 위한 처리다.

```text
300ms 안에 다시 입력
-> 대기 중인 타이머를 다시 설정

300ms가 지나 이미 요청 전송
-> 이후 입력만으로 기존 Network 요청이 취소되지는 않음
```

Debounce는 아직 실행하지 않은 호출을 미루는 것이며, 이미 전송한 요청의 취소와는 다르다.

## 검색과 Pagination 연결

- 검색어가 바뀌면 `page=1`로 초기화한다. 기존 4페이지를 유지하면 새 결과의 첫 항목을 놓치거나, 결과가 적어 빈 화면을 볼 수 있다.
- 페이지 이동 시에는 기존 `query`를 유지하고 `page`만 변경한다.
- DB에서는 검색 조건을 적용한 뒤 `LIMIT`과 `OFFSET`으로 해당 페이지의 데이터만 가져온다.
- 전체 페이지 수는 같은 검색 조건으로 구한 건수를 페이지 크기로 나눈 뒤 올림한다.

실습은 페이지당 6개를 표시하므로 2페이지의 Offset은 `(2 - 1) * 6 = 6`이다. 모든 Invoice를 Browser로 내려받아 자르는 것이 아니라 Server에서 필요한 범위를 조회한다.

## 구현에서 확인한 내용

- Search에 `useDebouncedCallback(..., 300)`을 적용했다.
- Server Page에서 검색어와 페이지 번호를 읽고 Table에 전달했다.
- `fetchFilteredInvoices()`는 검색 조건과 `LIMIT`, `OFFSET`을 사용한다.
- `fetchInvoicesPages()`는 검색 결과의 전체 페이지 수를 계산한다.
- Table의 Suspense에 검색어와 페이지 번호 기반 Key를 지정했다.
- 실습 Commit: `6f8dfad` (`ch10: add invoice search and pagination`).
- Production Build와 TypeScript 검사를 통과했고, `/dashboard/invoices`는 Dynamic Route로 표시됐다. Browser에서의 실제 클릭 동작 테스트는 별도로 수행하지 않았다.

## 핵심 정리

> 검색어와 페이지 번호를 URL로 관리하면 새로고침과 공유 시 검색 조건을 복원할 수 있다. Client는 URL을 변경하고 Server Page는 그 URL의 searchParams를 읽어 DB를 조회한다. Debounce로 불필요한 조회를 줄이고, 검색 조건 변경 시 페이지를 초기화하며, 페이지 이동 시 검색 조건을 유지한다.

## 참고 자료

- [Next.js Learn: Adding Search and Pagination](https://nextjs.org/learn/dashboard-app/adding-search-and-pagination)
- [실습 Commit](https://github.com/sangyun-hwang/nextjs-dashboard-practice/commit/6f8dfad)
