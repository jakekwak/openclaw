---
title: "Plugin Setup and Config"
sidebarTitle: "Setup and Config"
summary: "setup wizard, setup-entry.ts, config schema, package.json metadata"
read_when:
  - plugin에 setup wizard를 추가할 때
  - `setup-entry.ts`와 `index.ts`의 차이를 이해하려 할 때
  - plugin config schema나 `package.json`의 `openclaw` metadata를 정의할 때
---

# Plugin Setup and Config

plugin packaging(`package.json` metadata), manifest(`openclaw.plugin.json`), setup entry, config schema를 정리한 레퍼런스입니다.

## Package metadata

`package.json`에는 plugin 시스템이 plugin capability를 파악할 수 있도록 `openclaw` 필드가 필요합니다.

### Channel plugin 예시

```json
{
  "name": "@myorg/openclaw-my-channel",
  "version": "1.0.0",
  "type": "module",
  "openclaw": {
    "extensions": ["./index.ts"],
    "setupEntry": "./setup-entry.ts",
    "channel": {
      "id": "my-channel",
      "label": "My Channel",
      "blurb": "Short description of the channel."
    }
  }
}
```

### Provider plugin 예시

```json
{
  "name": "@myorg/openclaw-my-provider",
  "version": "1.0.0",
  "type": "module",
  "openclaw": {
    "extensions": ["./index.ts"],
    "providers": ["my-provider"]
  }
}
```

### 주요 필드

| Field        | Type       | Description                   |
| ------------ | ---------- | ----------------------------- |
| `extensions` | `string[]` | entry point 파일              |
| `setupEntry` | `string`   | setup-only entry              |
| `channel`    | `object`   | channel metadata              |
| `providers`  | `string[]` | plugin이 등록하는 provider ID |
| `install`    | `object`   | install hint                  |
| `startup`    | `object`   | startup 동작 플래그           |

### Deferred full load

channel plugin은 `deferConfiguredChannelFullLoadUntilAfterListen`으로 지연 로딩을 켤 수 있습니다. 이 경우 pre-listen 단계에서는 `setupEntry`만 로드되고, full entry는 gateway listen 이후 로드됩니다.

## Plugin manifest

모든 native plugin은 package root에 `openclaw.plugin.json`을 포함해야 합니다. OpenClaw는 plugin 코드를 실행하지 않고도 이 manifest로 config를 검증합니다.

config가 없어도 빈 schema는 필요합니다.

## Setup entry

`setup-entry.ts`는 onboarding, config repair, disabled channel inspection처럼 setup surface만 필요할 때 쓰는 가벼운 entry입니다.

```typescript
import { defineSetupPluginEntry } from "openclaw/plugin-sdk/core";
import { myChannelPlugin } from "./src/channel.js";

export default defineSetupPluginEntry(myChannelPlugin);
```

### 언제 사용되나

- channel이 비활성화됐지만 setup/onboarding surface가 필요할 때
- channel이 활성화됐지만 아직 구성되지 않았을 때
- deferred loading이 켜졌을 때

### 무엇을 포함해야 하나

- channel plugin object
- gateway listen 전에 꼭 필요한 HTTP route / gateway method

### 무엇을 빼야 하나

- CLI registration
- background service
- 무거운 runtime import
- startup 이후에만 필요한 항목

## Config schema

plugin config는 manifest의 JSON Schema로 검증됩니다:

```json5
{
  plugins: {
    entries: {
      "my-plugin": {
        config: {
          webhookSecret: "abc123",
        },
      },
    },
  },
}
```

plugin은 registration 중 `api.pluginConfig`로 이 값을 받습니다.

channel-specific config는 `channels.<channel-id>` 아래를 사용합니다.

## Setup wizard

channel plugin은 `openclaw onboard`용 interactive setup wizard를 제공할 수 있습니다. `ChannelSetupWizard`는 보통 `credentials`, `textInputs`, `dmPolicy`, `allowFrom`, `groupAccess`, `prepare`, `finalize` 등을 지원합니다.

단순한 allowlist prompt나 표준 status 블록은 `openclaw/plugin-sdk/setup`에 있는 shared helper를 우선 사용하는 편이 좋습니다.

## Publish와 install

외부 plugin:

```bash
openclaw plugins install @myorg/openclaw-my-plugin
openclaw plugins install clawhub:@myorg/openclaw-my-plugin
openclaw plugins install npm:@myorg/openclaw-my-plugin
```

저장소 내부 plugin은 `extensions/` 아래에 두면 build 중 자동 감지됩니다.

## 관련 문서

- [SDK Entry Points](/ko-KR/plugins/sdk-entrypoints)
- [Plugin Manifest](/ko-KR/plugins/manifest)
- [Building Plugins](/ko-KR/plugins/building-plugins)
