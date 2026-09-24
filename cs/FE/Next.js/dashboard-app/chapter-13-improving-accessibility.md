# Chapter 13. Improving Accessibility

## 학습 목표

Server Action에서 폼 입력을 검증하고, 실패 이유를 폼 안에 표시한다. 생성 폼뿐 아니라 수정 폼에도 적용하며, 입력 요소와 오류 설명을 보조 기술이 연결할 수 있도록 구성한다.

## 전체 흐름

```text
폼 제출
-> formAction 실행
-> Server Action이 FormData 검증
-> 실패: 오류 객체 반환 -> state 갱신 -> JSX에 오류 표시
-> 성공: DB 저장 -> 경로 재검증 -> 목록으로 이동
```

## 브라우저 검증과 서버 검증

`required`는 빈 입력을 브라우저에서 안내하는 기능이다. 하지만 사용자가 HTML을 변경하거나 별도 클라이언트로 요청할 수 있으므로 서버 검증을 대신하지 못한다.

이번 실습에서는 required를 체험한 뒤 제거해 빈 입력도 서버 검증 과정으로 전달했다. 실제 서비스에서 클라이언트와 서버 검증을 함께 사용하는 것이 잘못된 것은 아니다.

## parse와 safeParse

| 메서드 | 성공 | 검증 실패 |
| --- | --- | --- |
| `parse()` | 검증된 데이터 반환 | ZodError를 throw |
| `safeParse()` | success가 true인 결과와 data 반환 | success가 false인 결과와 error 반환 |

```ts
const result = CreateInvoice.safeParse({
  customerId: formData.get('customerId'),
  amount: formData.get('amount'),
  status: formData.get('status'),
});

if (!result.success) {
  return {
    errors: result.error.flatten().fieldErrors,
    message: '입력값을 확인해주세요.',
  };
}

const { customerId, amount, status } = result.data;
```

safeParse는 검증 실패를 결과로 전달하므로 success를 기준으로 분기할 수 있다. 실패 시 조기에 반환하고 DB 쿼리를 실행하지 않는다. 성공 후에는 원본 FormData가 아니라 검증과 변환을 거친 data를 사용한다.

금액은 `z.coerce.number().gt(0)`으로 숫자 변환 후 양수인지 검사한다. 숫자로 변환 가능하다는 것과 유효한 금액이라는 것은 별개의 조건이다.

## useActionState의 역할

```tsx
const initialState: State = { message: null, errors: {} };
const [state, formAction] = useActionState(createInvoice, initialState);

return <form action={formAction}>...</form>;
```

- state: 최초에는 initialState, 이후에는 Action이 반환한 결과다.
- formAction: 폼 제출을 Action 실행과 상태 갱신에 연결한다.

이번 state는 모든 입력값을 자동 보관하는 객체가 아니라 서버가 반환한 오류 정보를 담는다. useActionState는 pending 상태도 세 번째 값으로 제공하지만 위 예제는 앞의 두 값만 사용한다.

### 수정 Action의 인자 전달

```tsx
const updateWithId = updateInvoice.bind(null, invoice.id);
const [state, formAction] = useActionState(updateWithId, initialState);
```

원래 함수가 `updateInvoice(id, prevState, formData)`라면 인자는 다음과 같이 전달된다.

| 인자 | 제공하는 곳 |
| --- | --- |
| id | bind에서 미리 지정한 invoice.id |
| prevState | useActionState가 전달하는 이전 상태. 최초에는 initialState |
| formData | 폼 제출 과정에서 이름 있는 입력 요소들로 구성 |

bind의 첫 번째 인자 null은 this 자리이며 id로 전달되지 않는다.

## return과 throw의 화면 처리 차이

`return { message: '...' }`는 정상적인 함수 반환이다. useActionState를 통해 결과를 받고, JSX에서 state.message를 사용해야 화면에 표시된다. 오류 객체를 반환했다는 사실만으로 Error Boundary가 실행되지는 않는다.

반면 Action에서 처리하지 않고 throw한 오류는 이 흐름에서 가까운 Error Boundary로 전달된다. 입력 오류처럼 사용자가 수정할 수 있는 상황은 오류 정보를 반환해 폼 안에서 안내한다.

이번 생성과 수정 Action은 DB 실패도 message로 반환한다. 성공 시에만 `revalidatePath()`를 호출하고 그다음 `redirect()`한다. 삭제 Action은 이번 폼 상태 연결 대상이 아니다.

## 오류 메시지와 접근성

```tsx
<input id="amount" name="amount" aria-describedby="amount-error" />

<div id="amount-error" aria-live="polite" aria-atomic="true">
  {state.errors?.amount?.map((error) => (
    <p key={error}>{error}</p>
  ))}
</div>
```

`aria-describedby`는 입력 요소의 설명이 어느 요소에 있는지 ID로 연결한다. 실제 id 요소가 없으면 연결할 설명 대상도 없다. 속성 자체가 오류 메시지를 생성하거나 읽기를 무조건 보장하는 것은 아니다.

`aria-live="polite"`는 메시지 갱신을 보조 기술이 사용자를 방해하지 않는 시점에 알릴 수 있게 한다. `aria-atomic="true"`는 변경 시 해당 영역 전체를 알리도록 요청한다.

고객, 금액, 상태 각각의 오류를 대응하는 필드 아래에 표시한다. DB 실패 등 전체 메시지인 state.message는 버튼 영역 위에 별도로 표시한다. 필드별 errors만 출력하면 message만 반환된 실패를 놓칠 수 있다.

## 구현과 검증 기록

- 실습 저장소: [nextjs-dashboard-practice](https://github.com/sangyun-hwang/nextjs-dashboard-practice)
- 최종 확인한 로컬 Commit: `fdad773`
- 생성과 수정 폼의 useActionState, 필드별 오류, ARIA 연결과 전체 메시지 표시 확인
- 생성과 수정 Action의 safeParse 및 오류 반환 확인
- 전체 TypeScript 검사 통과
- 전체 lint는 직전 검토에서 통과했고, 최종 메시지 추가 후 수정 폼 lint도 통과
- ESLint 10과 React 플러그인 조합의 실행 오류는 ESLint 9 계열로 변경 후 해소
- Baseline 데이터 갱신 안내는 검사 실패가 아닌 별도 경고
- 브라우저 실제 제출, DB 저장 및 스크린 리더 동작은 이번 검토에서 실행하지 않음

## 핵심 정리

> 서버 검증은 우회 가능한 브라우저 검증을 보완한다. safeParse로 검증 결과를 분기하고, 반환한 오류 정보를 useActionState와 JSX로 연결하면 폼을 유지하면서 실패 이유를 안내할 수 있다. 오류 문구를 화면에 표시하는 것과 입력 요소의 설명으로 연결하는 것은 모두 필요하다.

## 참고 자료

- [공식 Chapter 13](https://nextjs.org/learn/dashboard-app/improving-accessibility)
- [React: useActionState](https://react.dev/reference/react/useActionState)
- [Chapter 12: 오류 처리](chapter-12-handling-errors.md)
- [this 바인딩과 부분 적용](../../ThisBinding.md)
