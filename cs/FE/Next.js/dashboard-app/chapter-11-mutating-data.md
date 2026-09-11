# Chapter 11. Mutating Data

## 학습 목표

Server Action으로 Invoice를 생성, 수정, 삭제하고 FormData, 입력 검증, 경로 재검증과 Redirect의 연결을 이해한다.

## Form에서 DB 변경까지

```text
사용자가 Form 제출
-> Browser에서 Next Server로 POST 요청
-> Server Action이 FormData 추출 및 검증
-> DB 변경
-> 관련 경로 재검증
-> 갱신된 목록 표시
```

`app/lib/actions.ts`의 `'use server'`는 Export한 비동기 함수를 Server Function으로 표시한다. Form의 Action으로 사용하면 제출된 FormData를 받아 Server에서 작업한다.

```tsx
<form action={createInvoice}>
```

생성과 수정 후에는 목록으로 Redirect한다. 삭제는 목록에 있는 Form에서 실행하고 관련 경로를 재검증한다.

## FormData와 초기값

| 설정 | 역할 |
| --- | --- |
| `name="amount"` | 제출된 값을 `formData.get('amount')`로 읽을 이름 |
| `defaultValue={invoice.amount}` | 금액 입력창의 초기값 |
| `defaultChecked={invoice.status === 'paid'}` | 상태 Radio의 초기 선택 여부 |

초기값이 100이어도 사용자가 250으로 바꾸고 제출하면 현재 입력값인 문자열 `"250"`이 전달된다. `defaultValue`는 제출할 값을 고정하지 않는다.

`type="number"`인 Input도 FormData에서는 문자열로 읽힌다. 실습은 `z.coerce.number()`로 숫자로 변환하고 검증한다. 일반적으로 `FormData.get()`은 `string | File | null`을 반환하므로 TypeScript 타입만 믿지 않고 실제 입력을 검증해야 한다.

금액은 `amount * 100`으로 센트 단위로 변환해 DB에 저장한다. 수정 화면에 표시할 때는 조회 함수가 다시 달러 단위로 변환한다.

## `bind()`로 Invoice ID 미리 연결하기

수정 Action은 대상 ID와 제출한 FormData가 모두 필요하다.

```tsx
async function updateInvoice(id: string, formData: FormData) {
  // 입력 검증 후 해당 ID의 Invoice 수정
}

const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);
```

`bind()`는 함수를 즉시 실행하지 않고 `this`와 일부 인자를 미리 지정한 새 함수를 만든다. 인자를 미리 채우는 사용법을 부분 적용(Partial Application)이라고 한다.

```text
bind(null, invoice.id)
     |     |
     |     -> updateInvoice의 첫 번째 인자 id에 연결
     -> this 설정용 자리. 이번 함수에서는 this를 사용하지 않음
```

`null`이 `updateInvoice`의 첫 번째 인자로 전달되는 것은 아니다. Form 제출 시 React가 나머지 인자인 FormData를 전달한다.

```tsx
<form action={updateInvoiceWithId}>
```

```text
Form 제출
-> updateInvoiceWithId(formData)
-> updateInvoice(invoice.id, formData)
```

원래 `updateInvoice`를 그대로 Form에 연결하면 FormData가 첫 번째 인자 자리에 전달되므로 함수가 기대하는 인자 구조와 맞지 않는다. 삭제에서도 `deleteInvoice.bind(null, id)`로 대상 ID를 미리 연결한다.

## `this` 바인딩과 인자 부분 적용의 차이

일반 JavaScript 예시에서는 `this`도 실제로 사용할 수 있다.

```js
function introduce(greeting) {
  return `${greeting}, 저는 ${this.name}입니다.`;
}

const sayHello = introduce.bind({ name: '상윤' }, '안녕하세요');
sayHello(); // "안녕하세요, 저는 상윤입니다."
```

객체 전체가 `this`가 되고 `this.name`으로 해당 프로퍼티를 읽는다. `bind()`가 객체에 `name`이 존재하는지 검사하는 것은 아니다. 이번 Server Action 실습은 `this`를 활용하지 않고 ID를 미리 지정하는 기능을 사용한다.

## 재검증 후 Redirect

DB를 변경해도 관련 캐시가 남아 있으면 화면에 이전 결과가 표시될 수 있다. 실습은 변경 후 목록 경로를 재검증한다.

```tsx
revalidatePath('/dashboard/invoices');
redirect('/dashboard/invoices');
```

`redirect()`는 내부적으로 특별한 오류를 던져 현재 실행 흐름을 종료한다. 따라서 필요한 DB 작업과 재검증을 먼저 수행해야 한다.

## 구현에서 확인한 내용

- 생성 Page는 고객 목록을 조회해 Form에 전달한다.
- 수정 Page는 `params`를 await하고 Invoice와 고객 목록을 병렬 조회한다.
- 수정 Form은 기존 값을 표시하고 ID가 연결된 Action에 입력을 제출한다.
- 생성, 수정, 삭제 SQL과 목록 재검증이 연결되어 있다.
- 실습 Commit: `e9bacd1` (`ch11: add invoice creation, editing and deletion`).
- 최초 리뷰에서 발견한 수정 Form JSX 누락을 복원했고, 복원 후 TypeScript 검사를 통과했다. 이전 버전의 Production Build도 통과했으나 실제 Browser에서 DB 생성, 수정, 삭제를 수행하는 테스트는 별도로 실행하지 않았다.

## 핵심 정리

> Form 제출 시 Server Action은 FormData를 받아 검증하고 DB를 변경한다. 수정 대상 ID는 bind()의 부분 적용으로 미리 연결하고, 제출 시 FormData를 나머지 인자로 받는다. DB 변경 후에는 관련 경로를 재검증하고 필요하면 Redirect한다.

## 참고 자료

- [Next.js Learn: Mutating Data](https://nextjs.org/learn/dashboard-app/mutating-data)
- [Server Actions 정리](../server-actions.md)
