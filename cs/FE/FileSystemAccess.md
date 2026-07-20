# File System Access API

File System Access API는 사용자가 선택한 로컬 파일이나 디렉터리를 웹 애플리케이션이 읽고, 권한을 받은 경우 직접 수정할 수 있게 하는 Web API이다. 웹 IDE, 문서 편집기, 이미지 편집기처럼 로컬 파일을 반복해서 다루는 애플리케이션에서 활용할 수 있다.

`window.showDirectoryPicker()`는 사용자가 폴더 하나를 선택할 수 있는 picker를 열고 `FileSystemDirectoryHandle`을 반환한다.

```js
const directoryHandle = await window.showDirectoryPicker();
```

## 경로 문자열 대신 Handle을 반환하는 이유

API는 실제 절대 경로 문자열 대신 파일이나 디렉터리에 접근할 수 있는 handle을 반환한다.

- 웹 사이트가 운영체제의 실제 경로를 마음대로 탐색하지 못하게 한다.
- 사용자가 직접 선택하고 허용한 범위만 접근하게 한다.
- 브라우저가 읽기와 쓰기 권한을 중간에서 통제할 수 있다.
- 같은 handle로 파일을 다시 읽거나 권한이 있을 때 수정할 수 있다.

Handle은 파일 시스템 전체를 여는 경로가 아니라 사용자가 선택한 대상에 접근하기 위한 제한된 통로이다.

## 사용자 동작과 보안 조건

`showDirectoryPicker()`는 페이지가 로드됐다는 이유만으로 자동 실행할 수 없다. 버튼 클릭과 같은 사용자의 명시적인 동작에서 호출해야 한다.

```tsx
async function handleSelectDirectory() {
  try {
    const directoryHandle = await window.showDirectoryPicker({
      mode: 'read',
    });

    console.log(directoryHandle.name);
  } catch (error) {
    if (error instanceof DOMException && error.name === 'AbortError') {
      return;
    }

    throw error;
  }
}

<button type="button" onClick={handleSelectDirectory}>
  폴더 선택
</button>
```

사용자의 의도 없이 웹 사이트가 반복해서 파일 선택 창을 열거나 로컬 파일 접근을 유도하지 못하게 하기 위한 제한이다. Picker를 호출하기 전에 긴 비동기 작업을 수행하면 일시적인 사용자 활성화가 사라질 수 있으므로 사용자 이벤트에서 picker를 먼저 여는 편이 안전하다.

```js
// 사용자 활성화가 사라질 수 있다.
await fetch('/api/check');
await window.showDirectoryPicker();
```

이 API는 다음 조건과 제한을 가진다.

- HTTPS와 같은 secure context에서 사용한다.
- 버튼 클릭 같은 transient user activation이 필요하다.
- 사용자가 직접 선택한 대상만 접근할 수 있다.
- 브라우저가 민감한 시스템 디렉터리 선택을 제한할 수 있다.
- 모든 주요 브라우저에서 동일하게 지원되는 Baseline 기능은 아니다.

## Picker 옵션

```js
const directoryHandle = await window.showDirectoryPicker({
  id: 'project-workspace',
  mode: 'readwrite',
  startIn: 'documents',
});
```

- `mode: 'read'`: 읽기 접근을 요청한다. 기본값이다.
- `mode: 'readwrite'`: 읽기와 쓰기 접근을 요청한다.
- `startIn`: picker를 처음 열 위치를 제안한다.
- `id`: 같은 용도의 picker가 이전 위치를 기억할 수 있도록 구분한다.

`startIn`은 브라우저에 전달하는 시작 위치 제안이며 특정 경로를 강제로 여는 기능은 아니다.

`mode: 'readwrite'`를 지정해도 웹 사이트가 컴퓨터 전체 파일을 수정할 수는 없다. 사용자가 선택한 디렉터리와 그 하위 항목 중 브라우저가 허용하고 사용자가 승인한 범위에서만 작업할 수 있다.

## 디렉터리 순회

`showDirectoryPicker()`는 폴더 안의 모든 파일 내용을 한 번에 반환하지 않는다. `FileSystemDirectoryHandle`을 비동기로 순회해 필요한 항목에 접근한다.

```js
for await (const [name, handle] of directoryHandle.entries()) {
  if (handle.kind === 'file') {
    console.log('파일:', name);
  }

  if (handle.kind === 'directory') {
    console.log('폴더:', name);
  }
}
```

## FileSystemFileHandle과 File

```js
const fileHandle = await directoryHandle.getFileHandle('README.md');
const file = await fileHandle.getFile();
```

- `getFileHandle()`: 특정 파일에 다시 접근하거나 쓸 수 있는 `FileSystemFileHandle`을 반환한다.
- `getFile()`: 해당 시점의 파일 내용과 메타데이터를 담은 읽기용 `File`을 반환한다.

`FileSystemFileHandle`은 파일로 가는 통로이고 `File`은 특정 시점의 snapshot에 가깝다.

```js
console.log(file.name);
console.log(file.size);
console.log(file.lastModified);
console.log(await file.text());
```

외부 편집기에서 파일이 변경돼도 기존 `File` 객체가 자동으로 갱신되지는 않는다. 최신 내용을 읽으려면 같은 handle에서 `getFile()`을 다시 호출해야 한다.

## 파일과 디렉터리 생성

`create: true`를 사용하면 항목이 없을 때 새로 만들 수 있다.

```js
const fileHandle = await directoryHandle.getFileHandle('notes.md', {
  create: true,
});

const assetsHandle = await directoryHandle.getDirectoryHandle('assets', {
  create: true,
});
```

기존 항목을 열 때는 `create`를 생략하거나 `false`로 둔다. 존재하지 않는 항목을 생성 없이 요청하면 `NotFoundError`가 발생할 수 있다.

## 파일 쓰기

`FileSystemFileHandle.createWritable()`로 writable stream을 만든다.

```js
const fileHandle = await directoryHandle.getFileHandle('README.md', {
  create: true,
});

const writable = await fileHandle.createWritable();

await writable.write('# Hello');
await writable.close();
```

`write()`로 데이터를 전달한 뒤 `close()`를 호출해야 stream이 정상적으로 닫히고 변경 사항이 파일에 반영된다. 오류가 발생했다면 상황에 따라 `abort()`로 쓰기를 취소할 수 있다.

클라이언트 코드 실수나 악성 입력으로 중요한 파일을 덮어쓸 수 있으므로 다음과 같은 보호가 필요하다.

- 저장할 파일과 변경 내용을 화면에 명확히 표시한다.
- 덮어쓰기 전 사용자 확인이나 백업을 고려한다.
- 필요하지 않다면 읽기 권한만 요청한다.
- 입력받은 경로나 파일 이름을 그대로 신뢰하지 않는다.

## 권한 확인과 Handle 저장

Handle은 IndexedDB에 저장해 다음 방문에 같은 디렉터리를 다시 참조할 수 있다. 하지만 handle 저장과 권한 유지는 같은 의미가 아니다.

```js
const permission = await directoryHandle.queryPermission({
  mode: 'readwrite',
});

if (permission !== 'granted') {
  const result = await directoryHandle.requestPermission({
    mode: 'readwrite',
  });

  if (result !== 'granted') {
    throw new Error('폴더 쓰기 권한이 필요합니다.');
  }
}
```

- Handle 저장: 이전에 선택한 파일이나 디렉터리를 다시 참조할 수 있다.
- 권한 유지: 브라우저 정책과 세션 상태에 따라 달라지며 다시 승인이 필요할 수 있다.

`queryPermission()`은 현재 권한을 확인하고 `requestPermission()`은 필요한 권한을 사용자에게 다시 요청한다.

## 기존 방식과 비교

| 기능 | `<input type="file">` | `webkitdirectory` | `showDirectoryPicker()` |
| --- | --- | --- | --- |
| 선택 대상 | 파일 | 폴더 안의 파일 목록 | 디렉터리 handle |
| 폴더 구조 | 없음 | 상대 경로로 일부 복원 | handle로 직접 순회 |
| 파일 읽기 | 가능 | 가능 | 가능 |
| 원본 파일에 쓰기 | 불가능 | 불가능 | 권한이 있으면 가능 |
| 파일과 폴더 생성 | 불가능 | 불가능 | 권한이 있으면 가능 |
| 원본 대상 handle 저장 | 불가능 | 불가능 | handle을 IndexedDB에 저장 가능 |
| 브라우저 지원 | 가장 넓음 | 비교적 넓음 | 제한적 |

### input type=file

```html
<input id="files" type="file" multiple />
```

사용자가 선택한 `FileList`를 읽거나 서버에 업로드하는 방식이다. 프로필 이미지나 첨부 파일처럼 일회성 업로드가 목적이라면 지원 범위가 넓고 권한 모델이 단순한 `<input type="file">`이 더 적합하다.

### webkitdirectory

```html
<input id="directory" type="file" webkitdirectory multiple />
```

```js
for (const file of input.files) {
  console.log(file.webkitRelativePath);
}
```

사용자가 선택한 폴더의 파일 목록과 상대 경로를 받을 수 있어 폴더 단위 업로드에 적합하다. 그러나 디렉터리 handle을 받지 않으므로 새 파일을 만들거나 수정한 내용을 원본 파일에 다시 쓸 수 없다.

### Blob과 다운로드

기존 웹 앱은 변경된 내용을 새 파일로 다운로드하게 할 수 있다.

```js
const blob = new Blob(['수정된 내용'], {
  type: 'text/plain',
});

const url = URL.createObjectURL(blob);
const anchor = document.createElement('a');

anchor.href = url;
anchor.download = 'README.md';
anchor.click();

URL.revokeObjectURL(url);
```

이 방식은 기존 파일을 직접 수정하지 않고 다운로드 위치에 새 파일을 만든다. File System Access API는 사용자의 권한을 받아 기존 파일에 변경 내용을 다시 저장할 수 있다는 차이가 있다.

### Origin Private File System

Origin Private File System은 브라우저가 사이트 origin별로 제공하는 앱 전용 저장 공간이다. 사용자가 picker로 로컬 폴더를 선택하지 않아도 사용할 수 있지만, 일반 파일 탐색기에서 사용자가 직접 관리하는 문서 폴더와는 목적이 다르다.

- 사용자에게 보이는 로컬 파일을 열고 수정해야 함: File System Access API
- 앱 내부 캐시, 임시 파일, 작업 데이터를 저장함: Origin Private File System

## 브라우저 지원과 기능 감지

`showDirectoryPicker()`는 제한적으로 지원되므로 사용 전에 기능을 감지하고 대체 UI를 제공해야 한다.

```js
if ('showDirectoryPicker' in window) {
  // File System Access API 사용
} else {
  // input type=file 또는 webkitdirectory 제공
}
```

읽기와 업로드만 필요하다면 기존 input이 좋은 fallback이 될 수 있다. 그러나 원본 파일에 다시 쓰는 기능까지 동일하게 대체하지는 못한다.

## 정리

> `showDirectoryPicker()`는 실제 경로 문자열이 아니라 사용자가 허용한 디렉터리에 접근할 수 있는 `FileSystemDirectoryHandle`을 반환한다. `FileSystemFileHandle`은 파일에 다시 접근하고 쓰기 위한 통로이며, `getFile()`이 반환하는 `File`은 특정 시점의 읽기용 snapshot이다. 기존 input 방식은 업로드 호환성이 넓지만 원본 파일을 수정할 수 없고, File System Access API는 더 강력한 대신 명시적인 사용자 동작, 권한 관리와 브라우저 지원 확인이 필요하다.

## 참고

- [File System Access specification](https://wicg.github.io/file-system-access/)
- [MDN: showDirectoryPicker](https://developer.mozilla.org/en-US/docs/Web/API/Window/showDirectoryPicker)
- [File System Standard](https://fs.spec.whatwg.org/)
