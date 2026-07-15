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

## TypeScript 7.0

TypeScript 7.0은 2026년 7월 8일 정식 출시되었습니다.

이번 버전의 핵심은 새로운 타입 문법이 아니라 TypeScript 컴파일러와 언어 서비스를 기존 TypeScript/JavaScript 기반에서 Go 기반 네이티브 프로그램으로 포팅한 것입니다.

```text
TypeScript 6 이하
TypeScript로 작성된 컴파일러
-> JavaScript로 변환
-> Node.js에서 실행

TypeScript 7
Go로 작성된 컴파일러
-> 네이티브 바이너리로 실행
-> 공유 메모리와 멀티스레딩 활용
```

TypeScript 애플리케이션 자체가 Go로 실행되는 것은 아닙니다. TypeScript 코드를 검사하고 JavaScript로 변환하는 도구가 Go로 바뀐 것입니다. 애플리케이션은 이전과 마찬가지로 변환된 JavaScript를 브라우저나 Node.js 같은 JavaScript 런타임에서 실행합니다.

### 기존 타입 검사와의 호환성

TypeScript 7은 기존 컴파일러를 전혀 다른 방식으로 재설계하기보다 구조와 타입 검사 로직을 최대한 동일하게 포팅했습니다.

따라서 새로운 문법을 대량으로 추가하기보다 기존 타입 검사 결과를 유지하면서 컴파일러, 빌드와 에디터의 성능을 개선하는 것이 핵심입니다.

TypeScript 6에서 deprecated 설정을 정리하고 `stableTypeOrdering`을 사용한 프로젝트라면 TypeScript 7에서도 대부분 동일한 타입 검사 결과를 기대할 수 있습니다.

### 성능 개선

Go 기반 네이티브 실행과 병렬 처리로 공식 대규모 프로젝트 측정에서 전체 빌드 시간이 대체로 8배에서 12배 빨라졌습니다.

| 프로젝트 | TypeScript 6 | TypeScript 7 | 향상 |
| --- | ---: | ---: | ---: |
| VS Code | 125.7초 | 10.6초 | 11.9배 |
| Sentry | 139.8초 | 15.7초 | 8.9배 |
| Playwright | 12.8초 | 1.47초 | 8.7배 |

성능 향상은 모든 프로젝트에서 정확히 10배를 보장한다는 뜻은 아닙니다. 프로젝트 크기, 파일 구조, CPU 코어와 메모리에 따라 결과가 달라질 수 있습니다.

주요 개선 영역은 다음과 같습니다.

- 컴파일과 타입 검사 시간 단축
- 에디터의 프로젝트 로딩과 최초 오류 표시 단축
- 자동 완성, 참조 찾기와 정의 이동 응답 개선
- CI type-check 시간 단축
- 전체 빌드 과정의 메모리 사용량 감소

### 병렬 처리

TypeScript 7은 파일 파싱, 타입 검사, 출력과 Project References 빌드를 여러 CPU 코어에서 병렬로 처리할 수 있습니다.

```bash
npx tsc --checkers 8
npx tsc --builders 4
npx tsc --singleThreaded
```

- `--checkers`: 타입 검사 worker 수를 조절합니다. 기본값은 4입니다.
- `--builders`: 동시에 실행할 Project Reference builder 수를 조절합니다.
- `--singleThreaded`: 병렬 처리를 끄고 단일 스레드로 실행합니다.

worker 수를 늘린다고 항상 빨라지는 것은 아닙니다. 각 worker는 CPU를 사용하고 일부 타입 검사 정보를 별도로 가지므로 메모리 사용량도 증가할 수 있습니다.

```bash
npx tsc --checkers 4 --builders 4
```

이 설정은 상황에 따라 최대 16개의 타입 체커를 실행할 수 있습니다. 개발자의 고성능 PC에서는 유리할 수 있지만 자원이 제한된 CI에서는 오히려 느려지거나 메모리 부족이 발생할 수 있으므로 실행 환경에 맞게 조절해야 합니다.

### LSP 기반 언어 서버

TypeScript 7의 언어 서버는 LSP(Language Server Protocol)를 기반으로 다시 구현되었습니다.

```text
Editor
-> LSP
-> TypeScript 7 Language Server
```

자동 완성, 정의로 이동, 참조 찾기, 이름 변경, 자동 import와 오류 표시 같은 기능이 새 네이티브 언어 서버에서 동작합니다. LSP를 사용하므로 VS Code 외의 여러 에디터에서도 같은 언어 서버를 연결하기 쉬워졌습니다.

`tsc --watch`도 새로 구현되어 대규모 프로젝트의 파일 변경 감지 성능과 안정성이 개선되었습니다.

### 설치

정식 TypeScript 7은 기존 버전과 마찬가지로 `typescript` 패키지로 설치하고 `tsc` 명령으로 실행합니다.

```bash
npm install -D typescript
npx tsc --version
```

preview 단계에서 사용한 `@typescript/native-preview`와 `tsgo`는 정식 버전의 일반적인 설치 방식이 아닙니다.

### 새로운 기본 설정

TypeScript 7은 TypeScript 6에서 준비한 새로운 기본값을 적용합니다.

- `strict`: `true`
- `module`: `esnext`
- `noUncheckedSideEffectImports`: `true`
- `stableTypeOrdering`: `true`, 비활성화 불가
- `rootDir`: `./`
- `types`: `[]`

특히 `rootDir`과 `types` 변경은 기존 프로젝트에서 예상하지 못한 오류를 만들 수 있습니다.

```json
{
  "compilerOptions": {
    "rootDir": "./src",
    "types": ["node", "jest"]
  },
  "include": ["./src"]
}
```

`@types/node`나 `@types/jest`가 설치되어 있어도 `process`, `describe`, `expect` 같은 전역 타입을 찾지 못한다면 `compilerOptions.types`에 필요한 타입 패키지가 명시되어 있는지 확인해야 합니다.

```bash
npm install -D @types/node @types/jest
```

타입 패키지 설치와 `types` 설정에 포함하는 것은 별개의 과정입니다.

### 제거된 레거시 설정

TypeScript 6에서 deprecated된 여러 옵션은 TypeScript 7에서 오류가 되거나 더 이상 동작하지 않습니다.

| 기존 설정 | 변경 방향 |
| --- | --- |
| `target: "es5"` | 더 현대적인 ECMAScript target 사용 |
| `moduleResolution: "node"`, `"node10"` | `bundler` 또는 `nodenext` |
| `moduleResolution: "classic"` | `bundler` 또는 `nodenext` |
| `module: "amd"`, `"umd"`, `"systemjs"`, `"none"` | `esnext` 또는 `preserve` |
| `baseUrl` | 프로젝트 루트 기준의 상대적인 `paths` 사용 |
| `downlevelIteration` | 지원 종료 |
| `esModuleInterop: false` | `false` 설정 불가 |
| `allowSyntheticDefaultImports: false` | `false` 설정 불가 |

Vite처럼 번들러가 모듈을 해석하는 프론트엔드 프로젝트에서 기존 `moduleResolution: "node"`를 사용했다면 `bundler`로 변경하는 것이 자연스럽습니다.

```json
{
  "compilerOptions": {
    "target": "es2024",
    "module": "esnext",
    "moduleResolution": "bundler",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

Node.js의 ESM과 CommonJS 해석 규칙을 직접 따르는 프로젝트라면 `nodenext`를 고려할 수 있습니다.

### Unicode 처리 변경

Template Literal Type에서 문자를 추론할 때 Unicode code point를 더 자연스럽게 보존합니다.

```ts
type HeadTail<S> =
  S extends `${infer Head}${infer Tail}`
    ? [Head, Tail]
    : never;

type Result = HeadTail<"abc">;
// TypeScript 7: ["", "abc"]
```

이전에는 이모지를 UTF-16 surrogate 두 개로 나눌 수 있었지만 TypeScript 7은 사람이 기대하는 하나의 문자 단위로 처리합니다. 일반 애플리케이션보다 문자열을 타입 수준에서 조작하는 라이브러리에 더 큰 영향을 줄 수 있습니다.

### TypeScript 6과 병행해야 하는 경우

TypeScript 7.0에는 안정적인 programmatic compiler API가 없습니다. 새로운 API는 TypeScript 7.1에서 제공될 예정입니다.

일부 도구는 `tsc` 명령만 실행하지 않고 TypeScript를 라이브러리로 불러와 AST와 타입 정보에 직접 접근합니다.

```ts
import ts from "typescript";
```

다음 도구와 프레임워크는 이러한 API 의존성 때문에 TypeScript 6을 함께 사용해야 할 수 있습니다.

- `typescript-eslint`처럼 TypeScript AST와 타입 정보를 사용하는 도구
- Vue와 Volar
- Svelte
- Astro
- MDX
- Angular의 템플릿 타입 검사

따라서 TypeScript 7의 `tsc`로 빠르게 프로젝트 전체를 검사할 수 있어도, Vue의 에디터 타입 검사는 TypeScript 6 API를 요구할 수 있습니다. 이는 프로젝트의 레거시 `tsconfig` 문제와는 다른 제약입니다.

필요하다면 두 버전을 npm alias로 병행할 수 있습니다.

```json
{
  "devDependencies": {
    "@typescript/native": "npm:typescript@^7.0.2",
    "typescript": "npm:@typescript/typescript6@^6.0.2"
  }
}
```

### 마이그레이션 체크리스트

1. 먼저 TypeScript 6에서 deprecated 경고를 해결합니다.
2. `target`, `module`, `moduleResolution`과 `baseUrl`을 확인합니다.
3. `rootDir`을 명시해야 기존 출력 경로를 유지할 수 있는지 확인합니다.
4. 필요한 Node.js, 테스트 도구의 전역 타입을 `types`에 명시합니다.
5. ESLint, 프레임워크 플러그인과 에디터가 TypeScript 7을 지원하는지 확인합니다.
6. 로컬과 CI에서 `tsc --noEmit` 및 빌드를 비교합니다.
7. CI 자원에 맞춰 병렬 처리 worker 수를 조절합니다.

### 정리

> TypeScript 7은 새로운 타입 문법 중심의 업데이트가 아니라 기존 타입 검사 의미를 유지하면서 컴파일러와 언어 서비스를 Go 기반 네이티브 구조로 전환해 빌드와 에디터 성능을 크게 개선한 버전입니다. 다만 안정적인 compiler API가 아직 없어 이를 사용하는 일부 도구와 프레임워크는 TypeScript 6을 병행해야 할 수 있습니다.

### 참고

- [Announcing TypeScript 7.0](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/)
- [TypeScript 7 GitHub Repository](https://github.com/microsoft/typescript-go)
