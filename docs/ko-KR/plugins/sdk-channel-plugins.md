---
title: "Building Channel Plugins"
sidebarTitle: "Channel Plugins"
summary: "OpenClaw용 messaging channel plugin을 만드는 단계별 가이드"
read_when:
  - 새로운 messaging channel plugin을 만들 때
  - OpenClaw를 메시징 플랫폼에 연결하려 할 때
  - `ChannelPlugin` adapter surface를 이해해야 할 때
---

# Building Channel Plugins

이 가이드는 OpenClaw를 메시징 플랫폼에 연결하는 channel plugin 제작 흐름을 설명합니다. 완성되면 DM security, pairing, reply threading, outbound messaging이 포함된 channel을 갖게 됩니다.

## Channel plugin의 역할

channel plugin은 자체 send/edit/react tool을 따로 만들 필요가 없습니다. OpenClaw core가 공유 `message` tool을 유지하고, channel plugin은 다음을 담당합니다:

- **Config** - account resolution과 setup wizard
- **Security** - DM policy와 allowlist
- **Pairing** - DM 승인 흐름
- **Outbound** - 텍스트/미디어/투표 전송
- **Threading** - reply threading 방식

core는 shared message tool, prompt wiring, session bookkeeping, dispatch를 맡습니다.

## 기본 흐름

1. `package.json`과 `openclaw.plugin.json`으로 channel plugin package/manifest 정의
2. `createChatChannelPlugin(...)`으로 `ChannelPlugin` 객체 구성
3. `defineChannelPluginEntry(...)`로 entry point 연결
4. `setup-entry.ts` 추가
5. inbound 메시지를 webhook이나 SDK handler를 통해 OpenClaw로 전달
6. colocated test 작성

## 최소 구조

```text
extensions/acme-chat/
├── package.json
├── openclaw.plugin.json
├── index.ts
├── setup-entry.ts
└── src/
    ├── channel.ts
    ├── channel.test.ts
    ├── client.ts
    └── runtime.ts
```

## 핵심 adapter surface

대표적으로 다음 surface를 조합합니다:

- `setup.resolveAccount(...)`
- `setup.inspectAccount(...)`
- `security.dm`
- `pairing.text`
- `threading`
- `outbound.attachedResults.sendText(...)`
- `outbound.base.sendMedia(...)`

필요하면 low-level adapter를 직접 구현할 수도 있지만, 대부분은 `createChatChannelPlugin(...)`의 선언형 옵션으로 충분합니다.

## Inbound 처리

channel plugin은 플랫폼 메시지를 받아 OpenClaw로 forward해야 합니다. 흔한 패턴은 plugin-managed webhook route를 등록하고, 서명 검증 뒤 inbound handler를 호출하는 방식입니다.

## 테스트

보통 다음을 검증합니다:

- config에서 account가 올바르게 해석되는지
- `inspectAccount`가 secret을 노출하지 않는지
- 누락된 config를 올바르게 보고하는지

공용 helper는 [Testing](/ko-KR/plugins/sdk-testing)을 보세요.

## 다음 단계

- [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins)
- [SDK Overview](/ko-KR/plugins/sdk-overview)
- [SDK Testing](/ko-KR/plugins/sdk-testing)
- [Plugin Manifest](/ko-KR/plugins/manifest)
