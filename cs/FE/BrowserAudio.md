# 브라우저의 오디오 재생과 일시정지

브라우저는 `<audio>` 또는 `<video>` 요소가 가리키는 미디어를 가져와 버퍼에 저장하고, 재생할 수 있는 부분부터 디코딩해 운영체제의 오디오 출력 장치로 전달한다.

```text
미디어 요청
-> 데이터 버퍼링
-> 재생 가능한 구간 디코딩
-> 오디오 출력
```

전체 파일을 모두 내려받을 때까지 기다릴 필요는 없다. 앞부분에 재생 가능한 데이터가 충분하면 재생을 시작하고, 재생하는 동안 뒷부분을 계속 받아 버퍼에 채운다.

## play()와 pause()

```js
const audio = document.querySelector('audio');

await audio.play();
audio.pause();
```

`play()`는 브라우저에 재생 시작을 요청한다. 미디어 준비와 브라우저의 자동 재생 정책 확인이 비동기적으로 이루어질 수 있으므로 `Promise`를 반환한다.

```js
try {
  await audio.play();
} catch (error) {
  console.error('오디오를 재생할 수 없습니다.', error);
}
```

음원이 없거나 로딩에 실패한 경우, 지원하지 않는 형식인 경우, 사용자 동작 없이 소리 있는 미디어를 자동 재생하려는 경우 등에 Promise가 reject될 수 있다.

`pause()`는 현재 재생 위치에서 진행만 중단한다. 현재 위치인 `currentTime`과 이미 받은 buffer를 자동으로 초기화하지 않으므로, 다시 `play()`를 호출하면 멈춘 위치부터 이어서 재생한다.

## 일시정지와 정지

`HTMLMediaElement`에는 별도의 `stop()`이 없다. 처음 위치로 돌아가는 정지는 `pause()`와 `currentTime`을 조합해 구현한다.

```js
audio.pause();
audio.currentTime = 0;
```

```text
일시정지
-> pause()
-> 현재 위치 유지

정지
-> pause()
-> currentTime = 0
```

## 버퍼링

재생 속도보다 데이터가 느리게 도착해 buffer가 부족해지면 재생을 잠시 기다릴 수 있다. 이때 `waiting` 이벤트가 발생할 수 있고, 데이터가 다시 준비되면 재생을 이어갈 수 있다. 이는 동영상 서비스에서 볼 수 있는 일반적인 버퍼링과 같은 원리다.

```js
audio.addEventListener('waiting', () => {
  console.log('버퍼링 중');
});

audio.addEventListener('playing', () => {
  console.log('재생 중');
});
```

`pause()`는 재생 진행을 멈추는 명령이지 이미 받은 buffer를 모두 삭제하는 명령은 아니다.

## 주요 상태와 이벤트

```js
audio.paused;      // 일시정지 여부
audio.currentTime; // 현재 재생 위치
audio.duration;    // 전체 재생 시간
audio.ended;       // 끝까지 재생했는지 여부
```

| 이벤트 | 의미 |
| --- | --- |
| `play` | 재생이 요청되어 시작 단계에 들어감 |
| `playing` | 필요한 데이터가 준비되어 실제 재생 중 |
| `pause` | 재생이 일시정지됨 |
| `waiting` | 데이터 부족 등으로 재생을 기다림 |
| `timeupdate` | 현재 재생 위치가 변경됨 |
| `ended` | 미디어가 끝까지 재생됨 |

재생 완료 UI는 중간에도 발생할 수 있는 `pause`가 아니라 `ended`를 기준으로 처리한다.

## 자동 재생 정책

브라우저는 사용자의 의도와 관계없이 소리가 재생되는 경험을 막기 위해 소리가 있는 미디어의 자동 재생을 제한할 수 있다. 일반적으로 클릭과 같은 사용자 동작 안에서 `play()`를 호출하는 것이 안전하다. 음소거된 미디어는 자동 재생이 허용될 수 있지만 브라우저 정책에 따라 달라질 수 있다.

## React 상태와 동기화

React의 `isPlaying`은 UI 상태일 뿐 실제 미디어 요소를 제어하지 않는다. 실제 재생과 정지는 미디어 요소의 `play()`와 `pause()`를 호출해야 한다.

```tsx
import { useRef, useState } from 'react';

export default function AudioPlayer() {
  const audioRef = useRef<HTMLAudioElement>(null);
  const [isPlaying, setIsPlaying] = useState(false);

  async function handlePlay() {
    try {
      await audioRef.current?.play();
    } catch (error) {
      console.error('재생 실패', error);
    }
  }

  function handlePause() {
    audioRef.current?.pause();
  }

  return (
    <>
      <audio
        ref={audioRef}
        src="/music.mp3"
        onPlaying={() => setIsPlaying(true)}
        onPause={() => setIsPlaying(false)}
        onEnded={() => setIsPlaying(false)}
      />
      <button type="button" onClick={handlePlay}>재생</button>
      <button type="button" onClick={handlePause}>일시정지</button>
      <span>{isPlaying ? '재생 중' : '정지'}</span>
    </>
  );
}
```

`play()` 호출과 동시에 UI를 재생 상태로 바꾸면 실제 재생이 실패했을 때 UI와 미디어 상태가 달라질 수 있다. 실제 미디어 이벤트를 기준으로 상태를 변경하면 두 상태를 맞출 수 있다.

## Cleanup

컴포넌트에서 별도로 `Audio` 객체를 만들었다면 제거될 때 재생을 중단해야 한다.

```tsx
useEffect(() => {
  const audio = new Audio('/music.mp3');

  return () => {
    audio.pause();
    audio.currentTime = 0;
  };
}, []);
```

`pause()`는 재생을 멈추기 위한 처리다. 미디어 요청과 관련 자원까지 적극적으로 해제해야 한다면 `src`를 제거하고 `load()`를 호출할 수 있다.

```js
audio.pause();
audio.removeAttribute('src');
audio.load();
```

## 면접 답변

> 브라우저는 음원 데이터를 버퍼링하고 재생 가능한 부분부터 디코딩해 오디오 장치로 전달합니다. `play()`는 미디어 준비와 자동 재생 정책 확인이 비동기적으로 이루어질 수 있어 Promise를 반환하고, `pause()`는 현재 위치에서 재생 진행만 멈추므로 다시 재생하면 `currentTime`부터 이어집니다. 별도의 `stop()`은 없기 때문에 `pause()` 후 `currentTime`을 0으로 설정해 구현합니다. React에서는 state만 변경하는 것이 아니라 미디어 요소의 API를 호출하고 실제 미디어 이벤트를 기준으로 UI 상태를 동기화해야 합니다.

## 참고 자료

- [MDN: HTMLMediaElement.play()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play)
- [MDN: HTMLMediaElement.pause()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause)
- [MDN: Autoplay guide for media and Web Audio APIs](https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Autoplay)
