---
summary: "WSL2 게이트웨이 + Windows Chrome 원격 CDP 및 확장 릴레이 구성을 계층별로 문제 해결"
read_when:
  - Chrome은 Windows에 있고 OpenClaw Gateway는 WSL2에서 실행할 때
  - WSL2와 Windows에 걸쳐 브라우저/control-ui 오류가 겹쳐 보일 때
  - 분리된 호스트 구성에서 순수 원격 CDP와 Chrome 확장 릴레이 중 무엇을 쓸지 정할 때
title: "WSL2 + Windows + 원격 Chrome CDP 문제 해결"
---

# WSL2 + Windows + 원격 Chrome CDP 문제 해결

이 문서는 다음과 같은 흔한 split-host 구성을 다룹니다.

- OpenClaw Gateway는 WSL2 안에서 실행됨
- Chrome은 Windows에서 실행됨
- 브라우저 제어가 WSL2/Windows 경계를 넘어야 함

또한 [issue #39369](https://github.com/openclaw/openclaw/issues/39369)에서 드러난 계층형 장애 패턴도 다룹니다. 서로 독립적인 문제가 한 번에 겹쳐 나타날 수 있어서, 실제 문제와 다른 계층이 먼저 고장 난 것처럼 보일 수 있습니다.

## 먼저 올바른 브라우저 모드를 고르기

유효한 패턴은 두 가지입니다.

### 옵션 1: 순수 원격 CDP

WSL2에서 Windows Chrome CDP 엔드포인트를 가리키는 원격 브라우저 프로파일을 사용합니다.

다음 경우에 선택하십시오.

- 브라우저 제어만 필요할 때
- Chrome 원격 디버깅 포트를 WSL2에 노출하는 것이 괜찮을 때
- Chrome 확장 릴레이가 필요 없을 때

### 옵션 2: Chrome 확장 릴레이

내장 `chrome` 프로파일과 OpenClaw Chrome 확장을 함께 사용합니다.

다음 경우에 선택하십시오.

- 툴바 버튼으로 기존 Windows Chrome 탭에 붙고 싶을 때
- 원시 `--remote-debugging-port` 대신 확장 기반 제어를 원할 때
- 릴레이 자체가 WSL2/Windows 경계를 넘어 도달 가능해야 할 때

네임스페이스를 가로질러 확장 릴레이를 쓰는 경우, [브라우저](/ko-KR/tools/browser)와 [Chrome 확장](/ko-KR/tools/chrome-extension)에서 설명한 `browser.relayBindHost`가 핵심 설정입니다.

## 동작하는 아키텍처

기준 형태는 다음과 같습니다.

- WSL2에서 게이트웨이를 `127.0.0.1:18789`로 실행
- Windows에서 일반 브라우저로 `http://127.0.0.1:18789/` Control UI를 염
- Windows Chrome이 `9222` 포트에 CDP 엔드포인트를 노출
- WSL2에서 그 Windows CDP 엔드포인트에 접근 가능
- OpenClaw는 WSL2에서 도달 가능한 주소를 브라우저 프로파일에 설정

## 왜 이 구성이 헷갈리는가

여러 실패가 동시에 겹칠 수 있습니다.

- WSL2가 Windows CDP 엔드포인트에 접근하지 못함
- Control UI를 보안되지 않은 origin에서 엶
- `gateway.controlUi.allowedOrigins`가 페이지 origin과 맞지 않음
- 토큰 또는 페어링이 빠져 있음
- 브라우저 프로파일이 잘못된 주소를 가리킴
- 실제로는 크로스-네임스페이스 접근이 필요한데 확장 릴레이가 여전히 loopback 전용임

그래서 한 계층을 고쳐도 다른 오류가 계속 보일 수 있습니다.

## Control UI의 핵심 규칙

UI를 Windows에서 여는 경우, 의도적으로 HTTPS를 구성한 것이 아니라면 Windows localhost를 사용하십시오.

사용:

`http://127.0.0.1:18789/`

Control UI에 LAN IP를 기본값처럼 사용하지 마십시오. LAN 또는 tailnet 주소에서의 평문 HTTP는 CDP 자체와 무관한 insecure-origin/device-auth 동작을 유발할 수 있습니다. [Control UI](/ko-KR/web/control-ui)를 참조하세요.

## 계층별로 검증하기

위에서 아래로 진행하십시오. 중간 단계를 건너뛰지 마십시오.

### 1단계: Windows에서 Chrome이 CDP를 제공하는지 확인

Windows에서 원격 디버깅을 켜고 Chrome을 시작합니다.

```powershell
chrome.exe --remote-debugging-port=9222
```

Windows에서 먼저 Chrome 자체를 확인합니다.

```powershell
curl http://127.0.0.1:9222/json/version
curl http://127.0.0.1:9222/json/list
```

여기서 실패하면 아직 OpenClaw 문제가 아닙니다.

### 2단계: WSL2에서 그 Windows 엔드포인트에 닿는지 확인

WSL2에서 `cdpUrl`에 넣을 정확한 주소를 테스트합니다.

```bash
curl http://WINDOWS_HOST_OR_IP:9222/json/version
curl http://WINDOWS_HOST_OR_IP:9222/json/list
```

좋은 결과:

- `/json/version`이 Browser / Protocol-Version 메타데이터가 담긴 JSON을 반환
- `/json/list`가 JSON을 반환함 (열린 페이지가 없으면 빈 배열이어도 괜찮음)

여기서 실패한다면:

- Windows가 아직 포트를 WSL2에 노출하지 않았거나
- WSL2 기준 주소가 잘못됐거나
- 방화벽 / 포트 포워딩 / 로컬 프록시 구성이 부족한 것입니다

OpenClaw 설정을 만지기 전에 이 계층부터 해결하십시오.

### 3단계: 올바른 브라우저 프로파일 설정

순수 원격 CDP를 사용할 경우, WSL2에서 실제로 접근 가능한 주소를 OpenClaw에 설정합니다.

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "remote",
    profiles: {
      remote: {
        cdpUrl: "http://WINDOWS_HOST_OR_IP:9222",
        attachOnly: true,
        color: "#00AA00",
      },
    },
  },
}
```

참고:

- Windows에서만 되는 주소가 아니라 WSL2에서 닿는 주소를 사용하십시오
- 외부에서 관리되는 브라우저에는 `attachOnly: true`를 유지하십시오
- OpenClaw가 성공하길 기대하기 전에 같은 URL을 `curl`로 먼저 확인하십시오

### 4단계: Chrome 확장 릴레이를 쓰는 경우

브라우저 머신과 게이트웨이가 서로 다른 네임스페이스에 있으면, 릴레이가 loopback이 아닌 바인드 주소를 필요로 할 수 있습니다.

예시:

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "chrome",
    relayBindHost: "0.0.0.0",
  },
}
```

정말 필요할 때만 사용하십시오.

- 기본 동작이 더 안전합니다. 릴레이가 loopback 전용으로 유지되기 때문입니다
- `0.0.0.0`은 노출 면적을 넓힙니다
- 게이트웨이 인증, 노드 페어링, 주변 네트워크를 비공개 상태로 유지하십시오

확장 릴레이가 꼭 필요하지 않다면 위의 순수 원격 CDP 프로파일을 우선하십시오.

### 5단계: Control UI 계층을 별도로 검증

Windows에서 UI를 엽니다.

`http://127.0.0.1:18789/`

그다음 다음을 확인합니다.

- 페이지 origin이 `gateway.controlUi.allowedOrigins`가 기대하는 값과 일치하는지
- 토큰 인증 또는 페어링이 올바르게 설정돼 있는지
- 사실은 Control UI 인증 문제인데 브라우저 문제처럼 디버깅하고 있지 않은지

도움이 되는 문서:

- [Control UI](/ko-KR/web/control-ui)

### 6단계: 엔드투엔드 브라우저 제어 확인

WSL2에서:

```bash
openclaw browser open https://example.com --browser-profile remote
openclaw browser tabs --browser-profile remote
```

확장 릴레이의 경우:

```bash
openclaw browser tabs --browser-profile chrome
```

좋은 결과:

- 탭이 Windows Chrome에서 열림
- `openclaw browser tabs`가 대상 탭을 반환
- 이후 동작(`snapshot`, `screenshot`, `navigate`)도 같은 프로파일에서 정상 동작

## 자주 헷갈리는 오류

각 메시지는 특정 계층의 힌트로 취급하십시오.

- `control-ui-insecure-auth`
  - CDP 전송 문제가 아니라 UI origin / secure-context 문제
- `token_missing`
  - 인증 설정 문제
- `pairing required`
  - 디바이스 승인 문제
- `Remote CDP for profile "remote" is not reachable`
  - WSL2가 설정된 `cdpUrl`에 닿지 못함
- `gateway timeout after 1500ms`
  - 여전히 CDP 접근성 문제이거나 느리거나 닿지 않는 원격 엔드포인트인 경우가 많음
- `Chrome extension relay is running, but no tab is connected`
  - 확장 릴레이 프로파일은 선택됐지만 연결된 탭이 아직 없음

## 빠른 트리아지 체크리스트

1. Windows에서 `curl http://127.0.0.1:9222/json/version`이 동작하는가?
2. WSL2에서 `curl http://WINDOWS_HOST_OR_IP:9222/json/version`이 동작하는가?
3. OpenClaw 설정의 `browser.profiles.<name>.cdpUrl`이 그 WSL2 접근 가능 주소를 정확히 쓰고 있는가?
4. Control UI를 LAN IP 대신 `http://127.0.0.1:18789/`로 열고 있는가?
5. 확장 릴레이를 쓰는 경우에만 `browser.relayBindHost`가 정말 필요한가, 필요하다면 명시적으로 설정했는가?

## 실전 요약

이 구성 자체는 대체로 충분히 실현 가능합니다. 어려운 점은 브라우저 전송, Control UI origin 보안, 토큰/페어링, 확장 릴레이 토폴로지가 각각 독립적으로 실패하면서 사용자 입장에서는 비슷하게 보인다는 점입니다.

확신이 서지 않을 때는 다음 순서를 따르십시오.

- 먼저 Windows 로컬에서 Chrome 엔드포인트를 확인
- 그다음 WSL2에서 같은 엔드포인트를 확인
- 그 후에야 OpenClaw 설정 또는 Control UI 인증을 디버깅

### 최근 업데이트 메모

- 확장 릴레이 프로파일 이름은 `chrome`이 아니라 `chrome-relay`입니다.
