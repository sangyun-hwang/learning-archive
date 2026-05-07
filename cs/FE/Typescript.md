# TypeScript

## 기본 타입

```ts
const title: string = "learning archive";
const count: number = 10;
const enabled: boolean = true;
```

배열은 `number[]` 또는 `Array<number>`처럼 표현할 수 있습니다.

```ts
const scores: number[] = [1, 2, 3];
const names: Array<string> = ["React", "TypeScript"];
```

튜플은 특정 위치의 타입을 고정합니다.

```ts
const entry: [string, number] = ["age", 28];
```

## 객체와 함수 타입

```ts
type User = {
  id: string;
  name: string;
};

function getDisplayName(user: User): string {
  return user.name;
}
```

선택적 파라미터나 선택적 프로퍼티는 `?`를 사용합니다.

```ts
function log(message: string, level?: "info" | "warn") {
  console.log(level ?? "info", message);
}
```

## 인터페이스

인터페이스는 객체의 구조를 정의합니다.

```ts
interface Developer {
  name: string;
  language: string;
}

const developer: Developer = {
  name: "Sangyun",
  language: "TypeScript",
};
```

확장이 필요한 객체 모델에는 `interface`, 유니온이나 유틸리티 타입 조합에는 `type`이 편리한 경우가 많습니다.

## any, unknown, never

`any`는 타입 검사를 사실상 끄는 타입이므로 가능한 피합니다.

`unknown`은 어떤 값이든 받을 수 있지만, 사용하기 전에 타입 좁히기를 강제합니다.

```ts
function print(value: unknown) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  }
}
```

`never`는 도달할 수 없는 값의 타입입니다. 모든 분기를 처리했는지 검사할 때 사용할 수 있습니다.

```ts
type Status = "idle" | "loading" | "success" | "error";

function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}
```

props가 없어야 하는 컴포넌트를 표현할 때는 `Record<string, never>`를 사용할 수 있습니다.

## 타입 가드

타입 가드는 런타임 조건을 통해 타입을 좁히는 방식입니다.

```ts
function format(value: string | number) {
  if (typeof value === "string") {
    return value.trim();
  }

  return value.toFixed(2);
}
```

대표적인 타입 가드는 다음과 같습니다.

- `typeof`
- `instanceof`
- `in`
- 사용자 정의 타입 가드

## 제네릭

제네릭은 입력 타입과 출력 타입의 관계를 보존합니다.

```ts
function identity<T>(value: T): T {
  return value;
}

const name = identity("React");
const count = identity(1);
```

React에서는 `useState`처럼 초기값만으로 타입 추론이 애매한 경우 제네릭을 명시할 수 있습니다.

```tsx
const [keyword, setKeyword] = useState<string>("");
```

## 인덱스 시그니처

정해지지 않은 key를 갖는 객체는 인덱스 시그니처로 표현할 수 있습니다.

```ts
type Dictionary = {
  [key: string]: string;
};
```

## React를 위한 TypeScript

React에서 TypeScript를 사용할 때 핵심은 컴포넌트의 입력(props), 상태(state), 이벤트(event), API 응답 타입을 명확히 하는 것입니다.

```tsx
type ButtonProps = {
  label: string;
  disabled?: boolean;
  onClick: () => void;
};

function Button({ label, disabled = false, onClick }: ButtonProps) {
  return (
    <button disabled={disabled} onClick={onClick}>
      {label}
    </button>
  );
}
```

이벤트 타입은 DOM 요소 기준으로 지정합니다.

```tsx
function SearchInput() {
  const [value, setValue] = useState("");

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setValue(event.target.value);
  };

  return <input value={value} onChange={handleChange} />;
}
```

점진적으로 TypeScript를 도입할 때는 `tsconfig`를 엄격하게 한 번에 올리기보다, `unknown`, 타입 가드, API 응답 타입부터 정리하는 방식이 현실적입니다. JavaScript 파일에서는 JSDoc과 `@ts-check`를 이용해 일부 타입 검사의 도움을 받을 수 있습니다.
