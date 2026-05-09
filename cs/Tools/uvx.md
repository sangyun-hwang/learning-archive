# uvx

## 개념

`uvx`는 Python 패키지 매니저인 `uv`에 포함된 명령어다. 공식적으로는 `uv tool run`의 alias이며, Python 패키지가 제공하는 CLI 도구를 전역 설치 없이 실행할 수 있게 해준다.

느낌은 Node.js의 `npx`와 비슷하다.

```bash
npx prettier .
uvx black .
```

## 기존 Python CLI 실행 방식의 불편함

기존에는 Python CLI 도구를 실행하려면 보통 먼저 설치해야 했다.

```bash
pip install black
black .
```

이 방식은 간단하지만 몇 가지 문제가 있다.

- 전역 Python 환경이 오염될 수 있다.
- 프로젝트마다 필요한 버전이 다르면 충돌할 수 있다.
- 가상환경을 직접 만들고 관리해야 할 수 있다.

## uvx 방식

`uvx`를 사용하면 도구를 전역 설치하지 않고 바로 실행할 수 있다.

```bash
uvx black .
uvx ruff check .
uvx cowsay hello
```

동작 흐름은 다음과 같이 이해할 수 있다.

1. 실행할 CLI 도구를 확인한다.
2. 필요한 패키지를 격리된 환경에 준비한다.
3. 도구를 실행한다.
4. 반복 실행 비용을 줄이기 위해 uv cache를 활용한다.

`uvx`가 사용하는 환경은 전역 Python 환경과 분리된다. 따라서 일회성 CLI 도구를 실행할 때 전역 환경을 지저분하게 만들지 않는다.

## uvx와 uv tool install

`uvx`는 도구를 실행하는 명령이고, `uv tool install`은 도구를 설치해 PATH에서 계속 사용할 수 있게 만드는 명령이다.

| 목적 | 명령 |
| --- | --- |
| 일회성 실행 | `uvx black .` |
| 도구 설치 | `uv tool install black` |

`uvx`는 기본적으로 캐시된 격리 환경을 사용한다. 캐시가 지워지면 다시 환경을 만든다.

만약 `uv tool install`로 설치된 도구가 있다면, `uvx`는 기본적으로 그 설치된 버전을 사용할 수 있다. 특정 버전을 쓰고 싶다면 `uvx ruff@0.6.0`처럼 버전을 명시할 수 있다.

## npm, npx와 비교

| 생태계 | 전역 설치 | 일회성 실행 |
| --- | --- | --- |
| Node.js | `npm install -g prettier` | `npx prettier .` |
| Python 기존 | `pip install black` | 직접 관리가 번거로움 |
| Python uv | `uv tool install black` | `uvx black .` |

## 언제 사용하면 좋은가?

`uvx`는 프로젝트 의존성에 추가하지 않아도 되는 CLI 도구를 잠깐 실행할 때 적합하다.

- formatter 실행
- linter 실행
- 간단한 CLI 유틸리티 실행
- scaffolding 도구 실행

반대로 프로젝트의 의존성과 함께 실행되어야 하는 명령은 `uvx`보다 `uv run`이 더 적합할 수 있다. 예를 들어 프로젝트 내부 패키지를 import해야 하는 `pytest`, `mypy` 같은 경우에는 프로젝트 환경과 연결된 실행 방식이 필요할 수 있다.

## 정리

`uvx`는 Python CLI 도구를 전역 설치 없이 격리된 환경에서 빠르게 실행하는 명령이다. Node.js의 `npx`처럼 일회성 도구 실행을 단순하게 만들고, uv cache를 활용해 반복 실행 비용을 줄인다.
