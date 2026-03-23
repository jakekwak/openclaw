---
title: "Plugin SDK Overview"
sidebarTitle: "SDK Overview"
summary: "import map, registration API reference, SDK 아키텍처"
read_when:
  - 어떤 SDK subpath를 import해야 하는지 알고 싶을 때
  - OpenClawPluginApi의 registration 메서드 전체를 참고하려 할 때
  - 특정 SDK export를 찾고 있을 때
---

# Plugin SDK Overview

plugin SDK는 plugin과 core 사이의 타입 계약입니다. 이 페이지는 **무엇을 import해야 하는지**와 **무엇을 등록할 수 있는지**를 정리한 레퍼런스입니다.

<Tip>
  **실전 가이드가 필요하다면**
  - 첫 plugin이라면 [Getting Started](/ko-KR/plugins/building-plugins)
  - channel plugin이라면 [Channel Plugins](/ko-KR/plugins/sdk-channel-plugins)
  - provider plugin이라면 [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins)
</Tip>

## import 규칙

항상 구체적인 subpath에서 import하세요:

```typescript
import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry";
import { defineChannelPluginEntry } from "openclaw/plugin-sdk/core";

// Deprecated — 다음 major release에서 제거 예정
import { definePluginEntry } from "openclaw/plugin-sdk";
```

각 subpath는 작고 독립적인 모듈입니다. 이렇게 해야 startup이 빨라지고 circular dependency 문제를 줄일 수 있습니다.

## Subpath 레퍼런스

가장 자주 쓰는 subpath를 용도별로 묶었습니다. 100개가 넘는 전체 목록은 `scripts/lib/plugin-sdk-entrypoints.json`에 있습니다.

### Plugin entry

| Subpath                   | Key exports                                                                                                                            |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `plugin-sdk/plugin-entry` | `definePluginEntry`                                                                                                                    |
| `plugin-sdk/core`         | `defineChannelPluginEntry`, `createChatChannelPlugin`, `createChannelPluginBase`, `defineSetupPluginEntry`, `buildChannelConfigSchema` |

세부 subpath와 registration API 전체 설명은 영문 SDK 문서와 실제 SDK 소스를 함께 보는 것이 가장 정확합니다. 특히 `plugin-sdk/channel-*`, `plugin-sdk/provider-*`, `plugin-sdk/runtime-*` 계열이 핵심입니다.

## 등록 메서드 개요

plugin은 `api.register*` 메서드로 capability를 등록합니다.

### 일반 capability

| Method                                      | What it registers      |
| ------------------------------------------- | ---------------------- |
| `api.registerProvider(params)`              | Model/provider plugin  |
| `api.registerTool(tool)`                    | Tool                   |
| `api.registerChannel(params)`               | Messaging channel      |
| `api.registerSpeechProvider(params)`        | Speech provider        |
| `api.registerOnboardAuthProvider(provider)` | Onboarding auth method |
| `api.registerHook(events, handler, opts?)`  | Event hook             |

### 시스템/런타임 등록

| Method                                         | What it registers     |
| ---------------------------------------------- | --------------------- |
| `api.registerHttpRoute(params)`                | Gateway HTTP endpoint |
| `api.registerGatewayMethod(name, handler)`     | Gateway RPC method    |
| `api.registerCli(registrar, opts?)`            | CLI subcommand        |
| `api.registerService(service)`                 | Background service    |
| `api.registerInteractiveHandler(registration)` | Interactive handler   |

### 단일 슬롯

| Method                                     | What it registers                    |
| ------------------------------------------ | ------------------------------------ |
| `api.registerContextEngine(id, factory)`   | Context engine (한 번에 하나만 활성) |
| `api.registerMemoryPromptSection(builder)` | Memory prompt section builder        |

### 이벤트/라이프사이클

| Method                                       | What it does                  |
| -------------------------------------------- | ----------------------------- |
| `api.on(hookName, handler, opts?)`           | Typed lifecycle hook          |
| `api.onConversationBindingResolved(handler)` | Conversation binding callback |

### API 객체 필드

| Field                    | Type                      | Description                                               |
| ------------------------ | ------------------------- | --------------------------------------------------------- |
| `api.id`                 | `string`                  | Plugin id                                                 |
| `api.name`               | `string`                  | Display name                                              |
| `api.version`            | `string?`                 | Plugin version (optional)                                 |
| `api.description`        | `string?`                 | Plugin description (optional)                             |
| `api.source`             | `string`                  | Plugin source path                                        |
| `api.rootDir`            | `string?`                 | Plugin root directory (optional)                          |
| `api.config`             | `OpenClawConfig`          | Current config snapshot                                   |
| `api.pluginConfig`       | `Record<string, unknown>` | Plugin-specific config from `plugins.entries.<id>.config` |
| `api.runtime`            | `PluginRuntime`           | [Runtime helpers](/ko-KR/plugins/sdk-runtime)             |
| `api.logger`             | `PluginLogger`            | Scoped logger (`debug`, `info`, `warn`, `error`)          |
| `api.registrationMode`   | `PluginRegistrationMode`  | `"full"`, `"setup-only"`, or `"setup-runtime"`            |
| `api.resolvePath(input)` | `(string) => string`      | Resolve path relative to plugin root                      |

## 내부 모듈 규칙

plugin 내부에서는 로컬 barrel 파일을 통해 import하는 편이 좋습니다:

```text
my-plugin/
  api.ts            # 외부 소비자를 위한 public exports
  runtime-api.ts    # 내부 전용 runtime exports
  index.ts          # plugin entry point
  setup-entry.ts    # 가벼운 setup-only entry (선택 사항)
```

<Warning>
  production code에서 자신의 plugin을 `openclaw/plugin-sdk/<your-plugin>`으로 다시 import하지 마세요. 내부 import는 `./api.ts` 또는 `./runtime-api.ts`로 정리해야 합니다. SDK 경로는 외부 계약을 위한 것뿐입니다.
</Warning>

## 관련 문서

- [Entry Points](/ko-KR/plugins/sdk-entrypoints) - `definePluginEntry`, `defineChannelPluginEntry` 옵션
- [Runtime Helpers](/ko-KR/plugins/sdk-runtime) - `api.runtime` 네임스페이스 레퍼런스
- [Setup and Config](/ko-KR/plugins/sdk-setup) - 패키징, manifest, config schema
- [Testing](/ko-KR/plugins/sdk-testing) - 테스트 유틸리티와 lint 규칙
- [SDK Migration](/ko-KR/plugins/sdk-migration) - deprecated surface에서 이전하기
- [Plugin Internals](/ko-KR/plugins/architecture) - 아키텍처와 capability 모델
