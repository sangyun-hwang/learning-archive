# 브라우저에서 Native Application 실행하기

Browser는 Sandbox 안에서 실행되므로 사용자의 Computer에 있는 임의의 `.exe`, Shell 명령이나 바로가기 파일을 직접 실행할 수 없다. 온라인 게임 사이트의 실행 버튼은 로컬 파일에 직접 접근하는 대신, 이미 설치된 Launcher를 **운영체제에 등록된 Protocol Handler를 통해 호출**한다.

## Custom URL Scheme

Launcher를 설치할 때 운영체제에 전용 URL Scheme을 등록할 수 있다.

```text
mygame://
```

웹사이트는 다음처럼 해당 Scheme을 가진 Link를 제공한다.

```html
<a href="mygame://launch?gameId=123">
  게임 시작
</a>
```

전체 실행 흐름은 다음과 같다.

```text
사용자가 게임 시작 클릭
-> Browser가 mygame:// URI 실행 요청
-> 운영체제가 Scheme을 처리할 Application 탐색
-> 등록된 Game Launcher 실행
-> Launcher가 Parameter와 Ticket 검증
-> Launcher가 자신이 관리하는 게임 실행 파일 시작
```

각 주체의 역할은 다음과 같다.

| 주체 | 역할 |
| --- | --- |
| Browser | 사용자 동작으로 Custom URI 실행 요청 |
| 운영체제 | URI Scheme에 연결된 Application 탐색 및 실행 |
| Launcher | 입력 검증, 설치 위치와 Version 확인, 게임 실행 |

Browser는 `C:\Games\Game\game.exe` 같은 실제 경로를 알 필요가 없다. Launcher가 `gameId`를 자신의 설치 정보와 연결해 실행 파일을 결정한다.

## Launcher 설치와 Scheme 등록

Launcher Installer는 일반적으로 다음 작업을 수행한다.

```text
Launcher 파일 설치
-> 운영체제에 mygame:// Scheme 등록
-> Scheme을 Launcher 실행 파일과 연결
```

Windows Application은 Package Manifest나 Windows App SDK 등을 통해 URI Activation Handler로 등록할 수 있다. 같은 Scheme을 처리하는 Application이 여러 개라면 사용자의 기본 Application 선택이 개입할 수 있다.

게임 설치 위치 확인, Patch와 Version 관리는 Launcher의 기능이지만 **Scheme 등록 자체의 역할은 URI 요청과 Launcher를 연결하는 것**이다.

## 일회용 Launch Ticket

게임 실행에 로그인 상태가 필요하더라도 장기간 유효한 Access Token이나 비밀번호를 URI에 넣으면 안 된다.

```text
mygame://launch
  ?gameId=123
  &ticket=짧게_유효한_일회용_값
```

권장되는 흐름은 다음과 같다.

```text
Browser가 Server에 Launch Ticket 요청
-> Server가 짧은 만료 시간을 가진 Ticket 발급
-> Browser가 Ticket을 Launcher에 전달
-> Launcher가 Server에 Ticket 검증
-> 검증 성공 후 게임 실행
```

Ticket은 다음 성질을 갖는 것이 안전하다.

- 짧은 만료 시간
- 한 번만 사용 가능
- 사용자와 실행할 게임에 연결
- Server에서 최종 검증
- 재사용과 Replay 차단

URI는 Browser History, Log나 다른 Application에 노출될 가능성이 있으므로 장기 Credential을 전달하지 않는다. 일회용 Ticket이 유출되어도 사용할 수 있는 시간과 범위를 제한할 수 있다.

## Custom Protocol의 한계

Custom Protocol 호출은 기본적으로 다음과 같은 단방향 실행 요청에 가깝다.

```text
Browser
-> 운영체제
-> Launcher
```

Browser는 Launcher가 실제로 실행됐는지, 게임 설치 여부를 확인했는지 또는 게임 실행에 성공했는지를 신뢰성 있게 응답받기 어렵다.

일정 시간이 지난 뒤 Launcher 설치 안내를 보여주는 fallback을 만들 수 있지만, Page Focus나 Timeout만으로 설치 여부와 실행 성공을 완벽하게 판별할 수는 없다.

```ts
window.location.href = 'mygame://launch?gameId=123';

setTimeout(() => {
  showLauncherInstallGuide();
}, 1500);
```

Browser는 사용자 몰래 Native Application이 반복 실행되는 것을 막기 위해 외부 Application 실행 확인창을 표시할 수 있다. 실행 요청은 사용자의 Click 같은 명시적인 동작과 연결하는 것이 좋다.

## Localhost Helper

더 풍부한 양방향 통신이 필요하면 설치된 Launcher가 Local Server를 열고 대기할 수 있다.

```text
Launcher
-> 127.0.0.1:포트에서 대기

Browser
-> Localhost Helper에 요청
-> Launcher가 설치, Patch와 실행 상태 응답
```

Custom Protocol보다 다음 작업에 유리하다.

- Launcher 설치 및 실행 상태 확인
- 게임 설치 여부 확인
- Patch 진행률 전달
- 실행 성공과 실패 결과 전달
- WebSocket 등을 이용한 지속적인 상태 통신

하지만 Localhost라고 자동으로 안전한 것은 아니다. 다른 악성 웹사이트도 Localhost 주소를 추측해 요청할 수 있으므로 다음 보호가 필요하다.

- 허용된 Origin 검증
- 인증 Token 또는 일회용 Nonce
- CORS 제한
- Replay 방지
- 허용된 명령만 수행
- 임의 파일 경로와 Shell 명령 거부

## Extension과 Native Messaging

Browser Extension을 중간 계층으로 사용하는 방법도 있다.

```text
Web Page
-> Browser Extension
-> Native Messaging
-> Native Application
```

Chrome Native Messaging은 Extension이 등록된 Native Host와 `stdin`, `stdout`을 통해 구조화된 Message를 교환하게 한다. 일반 웹페이지가 직접 호출할 수 있는 기능은 아니며 Native Messaging 권한을 가진 Extension이 필요하다.

Native Host Manifest는 실행 파일 경로와 접근을 허용할 Extension Origin을 제한한다. 설치 과정은 복잡하지만 Custom Protocol보다 구조적인 양방향 통신이 가능하다.

## File System Access API와의 차이

File System Access API는 사용자가 선택한 File과 Directory를 Handle을 통해 읽거나 수정할 수 있게 한다. 실행 권한을 제공하는 API는 아니다.

```text
File System Access API
-> 사용자 허용 범위의 File 읽기와 쓰기

Custom Protocol / Native Helper
-> 설치된 Native Application에 실행 요청
```

Browser가 `FileSystemFileHandle`을 얻었더라도 그 File이 `.exe`라는 이유로 실행할 수는 없다.

## 개인 게임 바로가기 Page

개인용 Page에서 여러 게임을 실행하려면 게임별로 제공되는 실행 방식을 구분해야 한다.

| 게임 종류 | 가능한 연결 방식 |
| --- | --- |
| Steam 게임 | `steam://` Protocol |
| 전용 Launcher 게임 | Launcher가 공개한 Custom Protocol |
| Google Play Games PC | 생성된 Windows 바로가기 또는 Launcher 기능 |
| Emulator 게임 | Emulator CLI 또는 등록된 바로가기 |
| 일반 실행 파일 | Browser에서 직접 실행할 수 없음 |
| Playnite 등록 게임 | `playnite://` Protocol |

### Steam

Steam 게임은 App ID를 사용한 URI로 실행할 수 있다.

```html
<a href="steam://rungameid/1245620">
  Steam 게임 실행
</a>
```

```text
Browser
-> steam://rungameid/{appId}
-> 운영체제가 Steam 실행
-> Steam이 로그인과 설치 상태 확인
-> 게임 실행
```

### 전용 Launcher와 PC Mobile Game

전용 Launcher가 외부 실행 Protocol을 공개했다면 해당 URI를 사용할 수 있다. Windows 바로가기의 대상이 Launcher 실행 파일과 인자로 구성되어 있다면 일반 웹페이지에서는 그 명령을 직접 실행할 수 없다.

Google Play Games PC나 Emulator 게임은 제품에서 만든 Windows 바로가기 또는 CLI를 통합 Launcher에 등록하는 방식이 현실적이다. 내부 Protocol이나 비공개 실행 인자는 Update로 변경될 수 있으므로 공식 지원 여부를 확인한다.

### Playnite를 통합 Launcher로 사용

여러 Launcher와 Emulator를 하나의 Page에서 다루려면 Playnite를 중간 계층으로 사용할 수 있다.

```text
개인 Web Page
-> playnite:// URI
-> Playnite
-> Steam / 전용 Launcher / Emulator
-> 게임 실행
```

Playnite에 등록된 게임은 Library ID로 실행할 수 있다.

```html
<a
  href="playnite://playnite/start/PLAYNITE_GAME_ID"
>
  게임 실행
</a>
```

Page는 Playnite ID만 관리하고 실제 실행 파일 경로, Launcher와 Emulator 명령은 Playnite가 담당한다. 각 Launcher의 실행 방식을 Web Page에 직접 구현하는 것보다 관리 범위와 보안 경계를 줄일 수 있다.

```ts
const games = [
  {
    id: 'steam-game',
    name: 'Steam Game',
    launchUrl: 'steam://rungameid/1245620',
  },
  {
    id: 'mobile-game',
    name: 'Mobile Game',
    launchUrl:
      'playnite://playnite/start/PLAYNITE_GAME_ID',
  },
];
```

## 직접 Helper를 만드는 경우

통합 Launcher로 해결할 수 없다면 개인용 Native Helper를 만들 수 있다. 이 경우 Web Page가 실행 경로나 명령을 직접 전달하지 않고 제한된 `gameId`만 전달해야 한다.

```text
위험
-> /launch?path=C:\anything.exe
-> /launch?command=임의 Shell 명령

상대적으로 안전
-> /launch?gameId=mobile-game
-> Helper 내부 Allowlist에서 실행 대상 결정
```

Launcher가 URI Parameter의 경로나 Shell 명령을 그대로 실행하면 Command Injection과 임의 파일 실행 취약점이 발생할 수 있다. Native Helper는 미리 등록된 식별자만 허용하고, 실제 경로와 실행 인자는 내부 설정에서 결정해야 한다.

## 방식 비교

| 방식 | 장점 | 한계 |
| --- | --- | --- |
| Custom URL Scheme | 단순하고 Native App 실행에 적합 | 실행 결과를 Browser가 알기 어려움 |
| Localhost Helper | 상태 확인과 양방향 통신 가능 | 인증, Origin과 CORS 보호 필요 |
| Extension + Native Messaging | 허용된 Extension과 구조화된 통신 | Extension과 Native Host 설치 필요 |
| Playnite Protocol | 여러 Launcher와 Emulator 통합 | Playnite 설치와 게임 등록 필요 |
| File Download | 별도 연동 없이 Installer 제공 가능 | 사용자가 직접 설치하고 실행해야 함 |

## 면접에서 설명하기

> Browser는 Sandbox 때문에 사용자의 로컬 실행 파일을 직접 실행할 수 없습니다. 온라인 게임 사이트는 Launcher 설치 과정에서 운영체제에 Custom URL Scheme을 등록하고, 사용자가 `game://launch` 같은 Link를 클릭하면 운영체제가 연결된 Launcher를 실행하게 합니다. Launcher는 전달받은 게임 ID와 짧은 일회용 Ticket을 Server에서 검증한 후 자신이 관리하는 실행 파일을 시작합니다. 양방향 상태 통신이 필요하면 Localhost Helper나 Extension의 Native Messaging을 사용할 수 있으며, 모든 방식에서 외부 입력을 신뢰하지 않고 허용된 명령만 실행해야 합니다.

## References

- [Microsoft: Handle URI activation](https://learn.microsoft.com/en-us/windows/apps/develop/launch/handle-uri-activation)
- [Microsoft: Launch the default app for a URI](https://learn.microsoft.com/en-us/windows/apps/develop/launch/launch-default-app)
- [Chrome: Native Messaging](https://developer.chrome.com/docs/extensions/develop/concepts/native-messaging)
- [MDN: registerProtocolHandler](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/registerProtocolHandler)
- [Playnite: Command line arguments and URI commands](https://api.playnite.link/docs/manual/advanced/cmdlineArguments.html)
- [Playnite: Game library FAQ](https://api.playnite.link/docs/manual/library/games/faq.html)
- [Google Play Games on PC](https://support.google.com/googleplay/answer/11358888)

