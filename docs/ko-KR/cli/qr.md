---
summary: "iOS 페어링 QR + setup code를 생성하는 `openclaw qr` CLI 참조"
read_when:
  - iOS 앱을 게이트웨이와 빠르게 페어링하고 싶을 때
  - 원격/수동 공유용 setup code 출력이 필요할 때
title: "qr"
---

# `openclaw qr`

현재 게이트웨이 설정에서 iOS 페어링 QR과 setup code를 생성합니다.

## 사용법

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws
```

## 옵션

- `--remote`: config의 `gateway.remote.url`과 remote token/password를 사용
- `--url <url>`: payload에 사용할 게이트웨이 URL 재정의
- `--public-url <url>`: payload에 사용할 public URL 재정의
- `--token <token>`: bootstrap 흐름이 인증에 사용할 gateway token을 재정의
- `--password <password>`: bootstrap 흐름이 인증에 사용할 gateway password를 재정의
- `--setup-code-only`: setup code만 출력
- `--no-ascii`: ASCII QR 렌더링 건너뜀
- `--json`: JSON 출력 (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## 주의

- `--token`과 `--password`는 함께 사용할 수 없습니다.
- setup code 자체에는 이제 공유 gateway token/password가 아니라 불투명한 단기 `bootstrapToken`이 들어갑니다.
- `--remote`를 사용할 때, 실제로 활성인 remote 자격 증명이 SecretRef로 구성돼 있고 `--token`이나 `--password`를 넘기지 않으면, 명령은 활성 게이트웨이 스냅샷에서 이를 해석합니다. 게이트웨이를 사용할 수 없으면 즉시 실패합니다.
- `--remote` 없이 실행할 때는 CLI auth override가 없으면 로컬 게이트웨이 auth SecretRef를 해석합니다:
  - `gateway.auth.token`은 token auth가 승리할 수 있을 때 해석됩니다 (`gateway.auth.mode="token"`이 명시됐거나, 추론된 모드에서 password source가 이기지 않을 때).
  - `gateway.auth.password`는 password auth가 승리할 수 있을 때 해석됩니다 (`gateway.auth.mode="password"`가 명시됐거나, auth/env에 이기는 token source가 없을 때).
- `gateway.auth.token`과 `gateway.auth.password`가 둘 다 구성돼 있고(SecretRef 포함) `gateway.auth.mode`가 비어 있으면, mode를 명시적으로 설정할 때까지 setup-code 해석은 실패합니다.
- 게이트웨이 버전 불일치 주의: 이 명령 경로는 `secrets.resolve`를 지원하는 게이트웨이가 필요합니다. 오래된 게이트웨이는 unknown-method 오류를 반환합니다.
- 스캔 후에는 다음으로 디바이스 페어링을 승인합니다:
  - `openclaw devices list`
  - `openclaw devices approve <requestId>`
