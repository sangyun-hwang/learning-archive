# Chapter 12. Handling Errors

## 학습 목표

Invoice 작업 실패와 조회 대상이 없는 상황을 구분하고, Server Action의 오류 처리와 Next.js의 오류 화면을 연결한다.

## 오류 처리 역할 구분

| 기능 | 역할 |
| --- | --- |
| `try/catch` | 예외를 잡아 반환값으로 처리하거나 다시 던짐 |
| `error.tsx` | 경계 안에서 발생한 처리되지 않은 오류에 대한 Fallback UI |
| `notFound()` | 요청한 리소스가 없음을 명시하고 해당 Rendering 중단 |
| `not-found.tsx` | 리소스가 없을 때 보여줄 안내 화면 |

DB 연결 실패는 작업을 수행할 수 없는 오류이고, 조회가 성공했지만 Invoice가 없는 것은 리소스 부재다. 후자는 `notFound()`로 구분한다.

## Server Action의 `try/catch`

실습에서는 생성, 수정, 삭제의 SQL 실행을 `try/catch`로 감싸고 실패 시 오류를 기록한 후 다시 던진다.

```tsx
export async function deleteInvoice(id: string) {
  try {
    await sql`DELETE FROM invoices WHERE id = ${id}`;
  } catch (error) {
    console.error(error);
    throw new Error('Database Error: Failed to Delete Invoice.');
  }

  revalidatePath('/dashboard/invoices');
}
```

실패하면 재검증에 도달하지 않는다. 삭제는 목록에서 실행하므로 성공 후 별도 Redirect 없이 해당 경로를 재검증한다.

## 오류 객체 반환과 예외 던지기

- `return { message: '실패' }`: 오류 정보를 일반 반환값으로 전달한다. 호출 측이 값을 받아 화면에 표시해야 한다.
- `throw new Error(...)`: 정상 흐름을 중단하고 예외를 전파한다. 이 실습의 Form Action에서 처리되지 않은 오류는 오류 경계로 연결된다.

반환값에 `message`가 있다고 자동으로 `error.tsx`가 표시되지는 않는다. 비동기 함수가 오류 객체를 반환하면 Promise는 그 객체로 이행되고, 오류를 던지면 거부된다.

공식 과정의 중간 예제는 오류 메시지 객체를 반환하지만, 현재 프로젝트의 직접 연결된 `<form action={...}>` 타입은 `void | Promise<void>`를 요구해 타입 오류가 발생했다. 이번 실습에서는 예외를 던지는 방식으로 연결했고, 반환값을 Form 상태로 다루는 방식은 다음 장에서 이어간다. 예상 가능한 입력 오류를 모두 예외로 처리해야 한다는 의미는 아니다.

## Redirect는 `try/catch` 바깥에 배치

```tsx
try {
  // DB 생성 또는 수정
} catch (error) {
  throw new Error('Database operation failed.');
}

revalidatePath('/dashboard/invoices');
redirect('/dashboard/invoices');
```

`redirect()`는 내부적으로 특별한 오류를 던져 실행 흐름을 종료한다. 일반 `catch`로 감싸면 정상 이동까지 DB 실패처럼 처리할 수 있으므로 바깥에 둔다. Redirect 이후 코드는 실행되지 않으므로 재검증도 먼저 수행한다.

## `error.tsx`와 `reset()`

`app/dashboard/invoices/error.tsx`는 Invoices의 Page와 하위 Route에서 발생하는 오류에 대한 화면을 제공한다. 같은 Segment의 Layout 오류까지 잡는 것은 아니며, 그 경우에는 더 상위 경계가 필요하다.

`error.tsx`는 Client Component이고 `error`, `reset` Props를 받는다.

- `error`: 발생한 오류 정보
- `reset()`: 오류 경계를 초기화하고 내부 UI의 Rendering을 재시도

`reset()`은 실패한 삭제 Action을 자동으로 다시 호출하거나 DB 변경을 되돌리는 함수가 아니다. 오류 원인이 남아 있으면 다시 오류 화면이 나타날 수 있다.

## Invoice가 없을 때

수정 Page는 Invoice를 조회한 후 Form을 만들기 전에 존재 여부를 확인한다.

```tsx
const [invoice, customers] = await Promise.all([
  fetchInvoiceById(id),
  fetchCustomers(),
]);

if (!invoice) {
  notFound();
}
```

이 검사가 없으면 Form에서 `invoice.id` 등에 접근하다 일반 오류가 발생할 수 있다. `notFound()`는 Rendering을 중단하고 리소스 부재에 맞는 안내로 연결한다.

관련 파일은 다음 위치에 둔다.

```text
app/dashboard/invoices/error.tsx
app/dashboard/invoices/[id]/edit/page.tsx
app/dashboard/invoices/[id]/edit/not-found.tsx
```

`create`와 `[id]/edit`는 서로 다른 Route 영역이다. `create/not-found.tsx`는 수정 Route의 경계가 아니므로 수정 화면의 리소스 부재를 처리하지 않는다. 생성 Page는 아직 Invoice ID가 없으므로 고객 목록만 조회한다.

## 구현에서 확인한 내용

- 생성 Page에 잘못 들어간 ID 조회를 제거하고 수정 Page에 `notFound()` 검사를 배치했다.
- `not-found.tsx`를 수정 Route 아래로 이동했다.
- 생성, 수정, 삭제 Action의 DB 오류 처리와 `error.tsx`의 재시도 버튼을 연결했다.
- 임시 강제 오류를 제거했고, 수정 후 TypeScript 검사를 통과했다.
- 실습 Commit: `6bfabea` (`ch12: add invoice error and not-found handling`).
- Browser에서 오류 및 Not Found 화면과 DB 변경을 직접 실행하는 테스트는 별도로 수행하지 않았다.

## 핵심 정리

> Server Action에서 오류 정보를 반환하면 호출 측이 상태로 처리해야 하고, 예외를 던지면 오류 경계로 전파할 수 있다. DB 작업 실패와 리소스 부재를 구분하고, 없는 Invoice는 notFound()로 처리한다. reset()은 화면 Rendering을 재시도하며 실패한 Mutation을 자동 재실행하지 않는다.

## 참고 자료

- [Next.js Learn: Handling Errors](https://nextjs.org/learn/dashboard-app/error-handling)
- [Next.js: error.js](https://nextjs.org/docs/app/api-reference/file-conventions/error)
