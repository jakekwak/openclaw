---
summary: "Gateway 대시보드를 위한 통합된 Tailscale Serve/Funnel"
read_when:
  - Gateway 제어 UI를 로컬호스트 외부에 노출하기
  - 테일넷 또는 공용 대시보드 접근 자동화
title: "Tailscale"
---

# Tailscale (게이트웨이 대시보드)

OpenClaw는 게이트웨이 대시보드와 WebSocket 포트를 위해 Tailscale **Serve**(tailnet) 또는 **Funnel**(public)을 자동 설정할 수 있습니다. 이는 게이트웨이를 로컬 루프백에 고정시키는 반면, Tailscale은 HTTPS, 라우팅 및 (Serve의 경우) 신원 헤더를 제공합니다.

## 모드

- `serve`: `tailscale serve`를 통한 Tailnet 전용 Serve. 게이트웨이는 `127.0.0.1`에 머뭅니다.
- `funnel`: `tailscale funnel`을 통한 공용 HTTPS. OpenClaw는 공유 비밀번호가 필요합니다.
- `off`: 기본 설정 (Tailscale 자동화 없음).

## 인증

`gateway.auth.mode`를 설정하여 핸드셰이크를 제어합니다:

- `token` (`OPENCLAW_GATEWAY_TOKEN`이 설정된 경우 기본값)
- `password` (`OPENCLAW_GATEWAY_PASSWORD` 또는 설정을 통한 공유 비밀)

`tailscale.mode = "serve"`이고 `gateway.auth.allowTailscale`이 `true`일 때, 유효한 Serve 프록시 요청은 토큰/비밀번호를 제공하지 않고도 Tailscale 신원 헤더(`tailscale-user-login`)를 통해 인증할 수 있습니다. OpenClaw는 로컬 Tailscale 데몬(`tailscale whois`)을 통해 `x-forwarded-for` 주소를 해결하고 이를 헤더와 일치시켜 신원을 확인한 후 이를 수락합니다. OpenClaw는 요청이 로컬 루프백에서 Tailscale의 `x-forwarded-for`, `x-forwarded-proto`, `x-forwarded-host` 헤더와 함께 도착할 때만 이를 Serve로 취급합니다. 명시적 자격 증명이 필요하면 `gateway.auth.allowTailscale: false`로 설정하거나 `gateway.auth.mode: "password"`을 강제하십시오.

## 설정 예제

### Tailnet 전용 (Serve)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

열기: `https://<magicdns>/` (또는 설정된 `gateway.controlUi.basePath`)

### Tailnet 전용 (Tailnet IP에 바인딩)

게이트웨이가 Tailnet IP에서 직접 리슨하도록 하려면 (Serve/Funnel 없음) 이를 사용하십시오.

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

다른 Tailnet 장치에서 연결하십시오:

- 제어 UI: `http://<tailscale-ip>:18789/`
- WebSocket: `ws://<tailscale-ip>:18789`

참고: 이 모드에서는 로컬 루프백 (`http://127.0.0.1:18789`)이 **작동하지 않습니다**.

### 공용 인터넷 (Funnel + 공유 비밀번호)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

비밀번호를 디스크에 커밋하는 것보다 `OPENCLAW_GATEWAY_PASSWORD`을 선호하십시오.

## CLI 예제

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## 주의사항

- Tailscale Serve/Funnel은 `tailscale` CLI가 설치 및 로그인되어 있어야 합니다.
- `tailscale.mode: "funnel"`은 공개 노출을 피하기 위해 인증 모드가 `password`가 아니면 시작을 거부합니다.
- OpenClaw가 종료 시 `tailscale serve` 또는 `tailscale funnel` 구성을 취소하기를 원하면 `gateway.tailscale.resetOnExit`을 설정하십시오.
- `gateway.bind: "tailnet"`은 직접 Tailnet 바인딩 (HTTPS 없음, Serve/Funnel 없음)입니다.
- `gateway.bind: "auto"`는 로컬 루프백을 선호합니다; Tailnet 전용을 원하면 `tailnet`을 사용하십시오.
- Serve/Funnel은 **게이트웨이 제어 UI + WS**만 노출합니다. 노드들은 동일한 게이트웨이 WS 엔드포인트를 통해 연결되므로, Serve는 노드 액세스에 사용할 수 있습니다.

## 브라우저 제어 (원격 게이트웨이 + 로컬 브라우저)

한 컴퓨터에서 게이트웨이를 실행하고 다른 컴퓨터에서 브라우저를 제어하고 싶다면, 브라우저 머신에서 **노드 호스트**를 실행하고 둘 다 동일한 테일넷에 유지하십시오. 게이트웨이는 브라우저 동작을 노드에 프록시합니다; 별도의 제어 서버나 Serve URL은 필요 없습니다.

브라우저 제어에는 Funnel을 피하고, 노드 페어링을 운영자 접근처럼 취급하십시오.

## Tailscale 필수 조건 + 제한

- Serve는 테일넷에 대한 HTTPS 사용을 요구합니다; CLI는 누락 시 프로세스를 유도합니다.
- Serve는 Tailscale 신원 헤더를 주입합니다; Funnel은 그렇지 않습니다.
- Funnel은 Tailscale v1.38.3+, MagicDNS, HTTPS 사용 및 펀넬 노드 속성을 요구합니다.
- Funnel은 TLS를 통해 포트 `443`, `8443`, `10000`만 지원합니다.
- macOS에서의 Funnel은 오픈소스 Tailscale 앱 변형을 요구합니다.

## 자세히 알아보기

- Tailscale Serve 개요: [https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
- `tailscale serve` 명령어: [https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
- Tailscale Funnel 개요: [https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
- `tailscale funnel` 명령어: [https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)