# undefined와 null: 값의 부재와 기본값 처리

`undefined`와 `null`은 모두 값이 없음을 표현할 때 사용하지만, 서로 다른 원시 값입니다. 프론트엔드에서는 선택 상태, API 응답, 기본값과 JSON 전송을 다룰 때 이 차이가 실제 동작에 영향을 줍니다.

## 1. 의미와 발생 상황

| 구분 | undefined | null |
| --- | --- | --- |
| 일반적인 의미 | 값이 아직 지정되지 않았거나 조회 결과가 없음 | 값이 없음을 명시적으로 표현 |
| 대표 상황 | 미할당 변수, 없는 프로퍼티, 생략된 인자 | 선택한 상품 없음, 조회 대상 없음 |
| 타입 | Undefined 원시 타입 | Null 원시 타입 |
| `typeof` 결과 | `'undefined'` | `'object'` |

이 의미 구분은 관례입니다. 개발자가 `undefined`를 직접 지정할 수도 있고, API마다 없음의 표현 방식도 다를 수 있습니다. `typeof null`이 `'object'`인 것은 역사적인 동작이며, null이 실제 객체라는 뜻은 아닙니다.

```js
let nickname;
console.log(nickname); // undefined

const user = {};
console.log(user.nickname); // undefined

function readName(name) {
  return name;
}
readName(); // undefined: 인자를 전달하지 않음

function doNothing() {}
doNothing(); // undefined: 반환값이 없음

const selectedProduct = null; // 선택한 상품이 없음을 명시
```

주의할 점은 **값이 undefined인 것과 선언 자체가 없는 것은 다르다**는 것입니다. 선언하지 않은 변수를 직접 읽으면 `ReferenceError`가 발생합니다. `let`과 `const`도 선언문 실행 전의 TDZ에서 읽으면 오류가 납니다.

## 2. 없는 프로퍼티와 undefined 프로퍼티

```js
const a = {};
const b = { nickname: undefined };

a.nickname === b.nickname; // true: 읽은 값은 둘 다 undefined
Object.hasOwn(a, 'nickname'); // false
Object.hasOwn(b, 'nickname'); // true
```

값을 읽는 것만으로는 프로퍼티가 아예 없는지 구분할 수 없습니다. 필드의 존재 자체가 중요하면 `Object.hasOwn()`으로 확인합니다. `obj.key = undefined`는 프로퍼티를 삭제하는 것과 다릅니다.

## 3. 비교와 falsy

```js
undefined === null; // false: 다른 타입과 값
undefined == null;  // true: 느슨한 동등 비교 규칙

Boolean(undefined); // false
Boolean(null);      // false
```

두 값은 모두 falsy이지만, 모든 falsy가 값의 부재를 뜻하지는 않습니다. `0`, `false`, `''`, `NaN` 등도 falsy입니다.

```js
const stock = 0;

if (!stock) {
  // 재고가 0이어도 실행된다. 값의 부재만 검사하는 조건이 아니다.
}

if (stock === null || stock === undefined) {
  // null과 undefined만 검사한다.
}
```

재고 0은 품절이라는 유효한 정보이고, `false`는 사용자가 기능을 끈 상태일 수 있습니다. 이들을 누락된 값과 섞지 않는 것이 중요합니다.

## 4. 기본값 처리: ||와 ??

`||`는 왼쪽이 falsy이면 오른쪽 값을 반환합니다. `??`는 왼쪽이 null 또는 undefined인 경우에만 오른쪽 값을 반환합니다. 둘 다 결과가 반드시 boolean인 것은 아닙니다.

| 왼쪽 값 | `값 \|\| 5` | `값 ?? 5` |
| --- | --- | --- |
| `0` | `5` | `0` |
| `false` | `5` | `false` |
| `''` | `5` | `''` |
| `null` | `5` | `5` |
| `undefined` | `5` | `5` |
| `NaN` | `5` | `NaN` |

```js
const price = 0;
price || 1000; // 1000: 무료 가격까지 기본값으로 변경
price ?? 1000; // 0: 유효한 무료 가격 유지

const stock = 0;
stock ?? 5; // 0: 재고 없음 유지

const notificationsEnabled = false;
notificationsEnabled ?? true; // false: 알림 끄기 설정 유지

const displayName = '';
displayName || '방문자'; // '방문자': 빈 문자열도 대체하려는 정책
displayName ?? '방문자'; // '': 빈 문자열을 유지하는 정책
```

`??`가 항상 정답인 것은 아닙니다. 빈 문자열까지 대체하려면 `||`가 의도에 맞을 수 있습니다. 반대로 수량 0이나 false 설정을 보존해야 한다면 `??`가 적합합니다. `??`는 NaN이나 잘못된 문자열을 검증해 주는 기능은 아닙니다.

두 연산자는 필요한 경우에만 오른쪽 식을 평가하는 단락 평가를 합니다. `value ?? createDefault()`는 value가 null 또는 undefined일 때만 함수를 호출합니다.

## 5. 기본 매개변수와 구조 분해

기본 매개변수와 구조 분해의 기본값은 **undefined일 때만** 적용되고 null에는 적용되지 않습니다.

```js
function greet(name = '방문자') {
  return name;
}

greet();          // '방문자'
greet(undefined); // '방문자'
greet(null);      // null
greet('');        // ''

const { name: a = '방문자' } = {};               // a: '방문자'
const { name: b = '방문자' } = { name: null };   // b: null
```

null도 기본값으로 처리해야 한다면 함수 내부에서 `name ?? '방문자'`처럼 의도를 표현합니다.

## 6. Optional Chaining과 함께 사용

```js
const user = { profile: null };
const name = user.profile?.name ?? '방문자';
// '방문자'
```

- `?.`: 접근 대상이 null 또는 undefined이면 뒤의 접근을 중단하고 undefined를 반환합니다.
- `??`: 그렇게 얻은 값이 null 또는 undefined이면 기본값을 선택합니다.

`user.profile.name`은 위 예제에서 오류가 납니다. `?.`는 이런 접근을 방어하지만, API 응답 전체의 구조나 타입을 검증하는 기능은 아닙니다.

## 7. JSON과 API 수정 요청

JSON에는 null은 있지만 undefined라는 값은 없습니다. `JSON.stringify()`는 위치에 따라 undefined를 다르게 처리합니다.

```js
JSON.stringify({ nickname: undefined, profileImage: null });
// '{"profileImage":null}'

JSON.stringify([undefined, null]);
// '[null,null]'

JSON.stringify(undefined);
// undefined: JSON 문자열 자체가 만들어지지 않음
```

프로필 수정 API에서는 다음과 같은 계약을 정할 수 있습니다.

| 전송한 JSON | 계약 예시 |
| --- | --- |
| `{}` | 닉네임을 수정하지 않음 |
| `{"nickname": null}` | 기존 닉네임을 비움 |
| `{"nickname": "새 이름"}` | 닉네임을 변경 |

이 의미는 JSON 자체가 보장하지 않습니다. 백엔드가 어떤 수정 규칙을 사용하는지 확인해야 합니다. 삭제 의도로 undefined를 넣어도 직렬화 과정에서 필드가 빠져, 서버에서는 수정 요청이 없다고 처리할 수 있습니다.

## 8. React 상태와 화면 표시

```tsx
type Product = { id: string; name: string };

const [selectedProduct, setSelectedProduct] = useState<Product | null>(null);
```

이 상태에서 null은 아직 선택한 상품이 없다는 명확한 의미입니다. 선택하면 상품 객체를 저장하고, 선택 해제 시 null로 되돌립니다.

하지만 데이터 조회에서 null 하나로 로딩, 실패, 조회 결과 없음을 모두 표현하면 화면이 모호해집니다. `status: 'loading' | 'error' | 'success'` 등으로 요청 상태를 별도로 구분하고, 성공했지만 결과가 없을 때 null이나 빈 배열을 사용하는 식으로 설계합니다.

JSX 자식으로 null과 undefined는 표시되지 않습니다. 반면 숫자 0은 표시됩니다.

```tsx
// count가 0이면 숫자 0이 화면에 표시될 수 있다.
{count && <Badge count={count} />}

// 의도를 명확히 표현한다.
{count > 0 ? <Badge count={count} /> : null}

// controlled input은 null/undefined 대신 문자열을 전달한다.
<input value={nickname ?? ''} onChange={handleChange} />
```

## 9. TypeScript에서 다루기

`strictNullChecks`를 활성화하면 null과 undefined를 별도로 다뤄야 하므로, 값이 없을 수 있는 상황을 타입과 분기에서 확인할 수 있습니다.

```ts
type User = {
  nickname?: string;             // 읽을 때 string 또는 undefined
  profileImage: string | null;   // 필수 필드지만 null 허용
};

function label(name: string | null | undefined) {
  return name ?? '방문자';
}
```

`value!` 같은 non-null assertion은 실제 데이터를 검사하지 않습니다. 외부 API 응답은 TypeScript 타입 선언만 믿지 말고 필요한 런타임 검증을 별도로 수행합니다.

## 면접 요약

> undefined는 미할당 변수나 없는 프로퍼티를 읽을 때 나타나고, null은 값이 없음을 명시할 때 주로 사용합니다. 둘 다 falsy지만 0이나 false도 falsy이므로, 값의 부재만 처리하려면 ?? 또는 명시적인 비교를 사용합니다. 기본 매개변수는 undefined에만 적용되고, JSON 객체에서 undefined 필드는 생략되지만 null은 유지됩니다. 실제 상태와 API 계약에 맞게 의미를 정하는 것이 중요합니다.

## 참고 자료

- [MDN: undefined](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/undefined)
- [MDN: null](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/null)
- [MDN: Nullish coalescing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)
- [MDN: JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)
- [TypeScript: strictNullChecks](https://www.typescriptlang.org/tsconfig/strictNullChecks.html)
- [React: 조건부 렌더링](https://react.dev/learn/conditional-rendering)
- [React: input](https://react.dev/reference/react-dom/components/input)
