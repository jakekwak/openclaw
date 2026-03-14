---
summary: "iOS 노드 앱: 게이트웨이 연결, 페어링, 캔버스 및 문제 해결"
read_when:
  - iOS 노드를 페어링하거나 재연결할 때
  - 소스로부터 iOS 앱을 실행할 때
  - 게이트웨이 검색이나 캔버스 명령을 디버깅할 때
title: "iOS App"
---

# iOS App (Node)

사용 가능성: 내부 미리보기. iOS 앱은 아직 공개 배포되지 않았습니다.

## 기능

- WebSocket (LAN 또는 tailnet)을 통해 게이트웨이에 연결합니다.
- 노드 기능 공개: 캔버스, 화면 스냅샷, 카메라 캡처, 위치, 대화 모드, 음성 깨우기.
- `node.invoke` 명령을 수신하고 노드 상태 이벤트를 보고합니다.

## 요구 사항

- 다른 장치에서 실행되는 게이트웨이 (macOS, Linux 또는 WSL2를 통한 Windows).
- 네트워크 경로:
  - Bonjour를 통한 동일 LAN, **또는**
  - 유니캐스트 DNS-SD를 통한 Tailnet (예시 도메인: `openclaw.internal.`), **또는**
  - 수동 호스트/포트 (대체 경로).

## 빠른 시작 (페어링 + 연결)

1. 게이트웨이 시작:

```bash
openclaw gateway --port 18789
```

2. iOS 앱에서 설정을 열고 발견된 게이트웨이를 선택하십시오 (또는 수동 호스트를 활성화하고 호스트/포트를 입력하십시오).

3. 게이트웨이 호스트에서 페어링 요청 승인:

```bash
openclaw devices list
openclaw devices approve <requestId>
```

4. 연결 확인:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

## 공식 빌드용 relay 기반 push

공식 배포 iOS 빌드는 raw APNs 토큰을 gateway에 직접 게시하지 않고 외부 push relay를 사용합니다.

Gateway 측 요구 사항:

```json5
{
  gateway: {
    push: {
      apns: {
        relay: {
          baseUrl: "https://relay.example.com",
        },
      },
    },
  },
}
```

동작 방식:

- iOS 앱은 App Attest와 앱 receipt를 사용해 relay에 등록합니다.
- relay는 불투명한 relay handle과 등록 범위의 send grant를 반환합니다.
- iOS 앱은 페어링된 gateway identity를 가져와 relay 등록에 포함하므로, relay 등록이 특정 gateway에 위임됩니다.
- 앱은 해당 relay 기반 등록을 `push.apns.register`로 페어링된 gateway에 전달합니다.
- gateway는 저장된 relay handle을 `push.test`, 백그라운드 wake, wake nudge에 사용합니다.
- gateway relay base URL은 공식/TestFlight iOS 빌드에 bake된 relay URL과 일치해야 합니다.
- 앱이 나중에 다른 gateway나 다른 relay base URL을 가진 빌드에 연결되면, 기존 바인딩을 재사용하지 않고 relay 등록을 새로 고칩니다.

이 경로에서 gateway에 **필요하지 않은 것**:

- 배포 전체에서 공유하는 relay token 불필요
- 공식/TestFlight relay 기반 전송용 direct APNs 키 불필요

예상 운영 흐름:

1. 공식/TestFlight iOS 빌드를 설치합니다.
2. gateway에서 `gateway.push.apns.relay.baseUrl`을 설정합니다.
3. 앱을 gateway에 페어링하고 연결이 끝날 때까지 기다립니다.
4. 앱은 APNs 토큰 확보, operator session 연결, relay 등록 성공 후 자동으로 `push.apns.register`를 게시합니다.
5. 이후 `push.test`, reconnect wake, wake nudge는 저장된 relay 기반 등록을 사용할 수 있습니다.

호환성 참고:

- `OPENCLAW_APNS_RELAY_BASE_URL`은 gateway에서 임시 환경 변수 override로 계속 동작합니다.

## 디바이스 검색 경로

### Bonjour (LAN)

게이트웨이는 `local.`에 `_openclaw-gw._tcp`를 광고합니다. iOS 앱은 이를 자동으로 나열합니다.

### Tailnet (네트워크 간)

mDNS가 차단된 경우, 유니캐스트 DNS-SD 구역을 사용하십시오 (도메인 선택; 예: `openclaw.internal.`) 및 Tailscale 분할 DNS.
CoreDNS 예제를 위해 [Bonjour](/ko-KR/gateway/bonjour)을 참조하십시오.

### 수동 호스트/포트

설정에서 **수동 호스트**를 활성화하고 게이트웨이 호스트 + 포트를 입력하십시오 (기본 `18789`).

## 캔버스 + A2UI

iOS 노드는 WKWebView 캔버스를 렌더링합니다. 이를 구동하려면 `node.invoke`를 사용하세요:

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18789/__openclaw__/canvas/"}'
```

주의사항:

- 게이트웨이 캔버스 호스트는 `/__openclaw__/canvas/` 및 `/__openclaw__/a2ui/`를 제공합니다.
- 게이트웨이 HTTP 서버에서 제공됩니다 (게이트웨이 포트와 동일한 포트, 기본 `18789`).
- 캔버스 호스트 URL이 광고될 때 iOS 노드는 연결 시 자동으로 A2UI로 이동합니다.
- 내장 스캐폴드로 돌아가려면 `canvas.navigate`와 `{"url":""}`를 사용하세요.

### 캔버스 평가 / 스냅샷

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

## 음성 깨우기 + 대화 모드

- 음성 깨우기 및 대화 모드는 설정에서 사용할 수 있습니다.
- iOS는 백그라운드 오디오를 중지할 수 있습니다; 앱이 활성 상태가 아닐 때 음성 기능을 최선의 노력으로 처리하십시오.

## 일반 오류

- `NODE_BACKGROUND_UNAVAILABLE`: iOS 앱을 포그라운드로 가져오세요 (캔버스/카메라/화면 명령어가 필요합니다).
- `A2UI_HOST_NOT_CONFIGURED`: 게이트웨이가 캔버스 호스트 URL을 광고하지 않았습니다; [게이트웨이 구성](/ko-KR/gateway/configuration)에서 `canvasHost`를 확인하십시오.
- 페어링 프롬프트가 나타나지 않음: `openclaw devices list`를 실행하여 수동으로 승인하세요.
- 재설치 후 재연결 실패: 키체인 페어링 토큰이 지워졌습니다; 노드를 다시 페어링하세요.

## 관련 문서

- [Pairing](/ko-KR/channels/pairing)
- [Discovery](/ko-KR/gateway/discovery)
- [Bonjour](/ko-KR/gateway/bonjour)
