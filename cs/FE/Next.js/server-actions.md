# Server Actions

Server Action은 Client에서 발생한 사용자 동작을 계기로 Server에서 mutation을 실행하는 Next.js 기능이다. 일반 함수 호출처럼 작성하지만 실제로는 Browser와 Server 사이에 POST 요청이 발생한다.

## Server Function과 Server Action

- **Server Function**: `'use server'`로 표시되어 Server에서 실행되는 비동기 함수
- **Server Action**: Server Function을 `form`의 `action`, `formAction` 또는 transition 등 mutation 문맥에서 사용하는 경우

두 용어는 자주 함께 사용되지만, Server Action은 사용자 동작으로 데이터를 변경하는 용도를 강조한 표현이다.

## Server Component와 역할 비교

| 구분 | Server Component | Server Action |
| --- | --- | --- |
| 목적 | Server에서 UI와 데이터를 준비 | 사용자 동작으로 Server 작업 실행 |
| 실행 시점 | Rendering 과정 | Form 제출이나 event 발생 이후 |
| 대표 작업 | DB 조회, Server 전용 자원 접근 | DB 수정, Cookie 변경, Cache 재검증 |
| Browser event | 직접 처리할 수 없음 | Client 또는 form을 통해 호출 |

Server Component는 화면을 준비하고, Server Action은 화면이 표시된 이후 발생한 mutation을 처리한다.

## 선언 방법

별도 파일의 최상단에 `'use server'`를 선언하면 해당 파일에서 export한 비동기 함수를 Server Function으로 만들 수 있다.

```ts
// app/actions.ts
'use server';

export async function createPost(formData: FormData) {
  // Server에서 실행
}
```

Server Component 안에서 비동기 함수를 선언하고 함수 본문 첫 줄에 `'use server'`를 작성할 수도 있다. Client Component에서는 Server Function을 직접 정의하지 않고, `'use server'` 파일에서 import해 사용한다.

## FormData 전달

```tsx
import { createPost } from './actions';

export default function Page() {
  return (
    <form action={createPost}>
      <label htmlFor="title">제목</label>
      <input id="title" name="title" />
      <button type="submit">등록</button>
    </form>
  );
}
```

Form을 제출하면 React가 `name`이 있는 form control의 값을 `FormData`로 모아 Action에 전달한다. `id`는 label 연결 등에 사용되며 제출할 key를 결정하지 않는다. 따라서 `<input id="title" />`처럼 `name`이 없다면 `formData.get('title')`로 값을 받을 수 없다.

```ts
const title = formData.get('title');
// FormDataEntryValue | null
// string | File | null
```

Client가 보낸 값은 타입을 신뢰할 수 없다. `FormData.get()`의 결과를 바로 문자열로 단언하지 않고 Zod 같은 도구로 필수값, 타입과 길이 등을 런타임에 검증해야 한다.

## 실제 호출 흐름

```text
사용자가 form 제출
-> Browser가 Action 식별자와 직렬화 가능한 인자를 POST로 전송
-> Server가 Server Action 실행
-> mutation 결과와 갱신된 UI 정보를 Client에 반환
```

코드에서는 함수를 호출하는 것처럼 보이지만 Browser에서 DB 함수가 직접 실행되는 것은 아니다. React와 Next.js가 Network 요청과 결과 반영을 연결한다.

## Mutation 이후 화면 갱신

DB가 변경되어도 기존 Cache가 남아 있으면 화면에는 이전 데이터가 표시될 수 있다. Action에서 변경 범위에 맞게 `revalidatePath()` 또는 `revalidateTag()`를 호출해 관련 Cache를 무효화한다.

```ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createPost(formData: FormData) {
  // 검증과 권한 확인
  // DB mutation

  revalidatePath('/posts');
  redirect('/posts');
}
```

`redirect()`는 Next.js의 제어 흐름을 중단해 이동을 처리하므로, Cache 재검증처럼 반드시 필요한 작업은 먼저 실행한다.

## Pending과 오류 처리

- `useActionState`: Action 결과와 pending 상태를 함께 관리
- `useFormStatus`: 상위 form의 제출 상태를 읽어 제출 버튼 비활성화 등에 사용
- 예상 가능한 검증 오류: Action 결과로 반환해 UI에 표시
- 예상하지 못한 오류: Error Boundary에서 처리할 수 있도록 throw

Form 기반 Action은 JavaScript가 준비되기 전에도 제출할 수 있는 점진적 향상을 지원한다. Client Component에서 호출한 Action은 hydration이 끝날 때까지 제출이 대기할 수 있다.

## 보안

`'use server'`는 실행 위치를 지정할 뿐 인증이나 인가를 제공하지 않는다. Server Action은 직접 POST 요청으로 호출될 수 있으므로 공개 API endpoint와 같은 기준으로 보호해야 한다.

- Action 내부에서 로그인 여부 확인
- 요청한 사용자가 해당 자원을 변경할 권한이 있는지 확인
- `FormData`와 전달 인자 런타임 검증
- Client에서 전달한 ID만 믿지 않고 자원 소유권 확인
- 결제와 주문처럼 중복 실행이 위험한 작업은 멱등성 보장
- 민감한 정보는 Client에 반환하지 않기

권한이 없는 사용자에게 버튼을 숨기는 것은 UI 처리일 뿐 보안 경계가 아니다. 공격자는 화면을 거치지 않고 직접 요청을 만들 수 있으므로 Server 내부 검사가 필요하다.

## Route Handler와 비교

| Server Action | Route Handler |
| --- | --- |
| 같은 Next.js UI의 mutation에 적합 | 명시적인 HTTP API 제공에 적합 |
| Form, Cache 재검증과 UI 갱신 통합 | HTTP method, status와 response를 직접 설계 |
| React가 호출 형식을 관리 | Mobile App, webhook, 외부 service도 호출 가능 |

외부 Client가 사용할 공개 API라면 Route Handler가 더 명확하다. Server Action은 현재 Next.js Application 안의 사용자 동작과 Server mutation을 연결할 때 적합하다.

## 주의할 점

- 인자와 반환값은 React가 직렬화할 수 있는 값이어야 한다.
- Action은 mutation을 위한 기능이며 일반적인 병렬 data fetching 수단으로 사용하지 않는다.
- 요청 취소가 이미 시작된 Server mutation을 자동으로 되돌리지는 않는다.
- 인증, 인가와 입력 검증을 Client에만 두지 않는다.

## 면접에서 설명하기

> Server Action은 Client의 form 제출이나 사용자 동작을 Server의 비동기 함수와 연결해 DB 수정, Cookie 변경, Cache 재검증 같은 mutation을 처리하는 기능입니다. 일반 함수처럼 보이지만 실제로는 POST 요청이 발생하며, `'use server'`가 보안을 보장하지 않으므로 Action 내부에서 입력 검증과 인증·인가를 다시 수행해야 합니다. 변경 후에는 `revalidatePath`나 `revalidateTag`로 관련 Cache를 갱신하고, 외부 Client도 호출해야 하는 공개 API라면 Route Handler를 사용합니다.

## References

- [Next.js: Mutating Data](https://nextjs.org/docs/app/getting-started/mutating-data)
- [Next.js: Data Security](https://nextjs.org/docs/app/guides/data-security)
- [Next.js: Authentication](https://nextjs.org/docs/app/guides/authentication)
