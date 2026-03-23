---
title: "Plugin Entry Points"
sidebarTitle: "Entry Points"
summary: "`definePluginEntry`, `defineChannelPluginEntry`, `defineSetupPluginEntry` 레퍼런스"
read_when:
  - `definePluginEntry`나 `defineChannelPluginEntry`의 정확한 타입 시그니처가 필요할 때
  - registration mode(full vs setup)를 이해하려 할 때
  - entry point 옵션을 찾고 있을 때
---

# Plugin Entry Points

모든 plugin은 기본 entry object를 export합니다. SDK는 이를 만들기 위한 세 가지 helper를 제공합니다.

<Tip>
  **실전 예제가 필요하다면** [Channel Plugins](/ko-KR/plugins/sdk-channel-plugins) 또는 [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins)을 보세요.
</Tip>

## `definePluginEntry`

**Import:** `openclaw/plugin-sdk/plugin-entry`

provider plugin, tool plugin, hook plugin, 그 밖에 메시징 채널이 아닌 plugin에 사용합니다.

```typescript
import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry";

export default definePluginEntry({
  id: "my-plugin",
  name: "My Plugin",
  description: "Short summary",
  register(api) {
    api.registerProvider({
      /* ... */
    });
    api.registerTool({
      /* ... */
    });
  },
});
```

| Field          | Type                                                             | Required | Default             |
| -------------- | ---------------------------------------------------------------- | -------- | ------------------- |
| `id`           | `string`                                                         | Yes      | —                   |
| `name`         | `string`                                                         | Yes      | —                   |
| `description`  | `string`                                                         | Yes      | —                   |
| `kind`         | `string`                                                         | No       | —                   |
| `configSchema` | `OpenClawPluginConfigSchema \| () => OpenClawPluginConfigSchema` | No       | Empty object schema |
| `register`     | `(api: OpenClawPluginApi) => void`                               | Yes      | —                   |

- `id`는 `openclaw.plugin.json` manifest와 일치해야 합니다.
- `kind`는 `"memory"`, `"context-engine"` 같은 exclusive slot에 사용합니다.
- `configSchema`는 lazy evaluation을 위해 함수로 둘 수 있습니다.

## `defineChannelPluginEntry`

**Import:** `openclaw/plugin-sdk/core`

channel 전용 wiring을 포함한 helper입니다. 자동으로 `api.registerChannel({ plugin })`를 호출하고, registration mode에 따라 `registerFull`을 gating합니다.

```typescript
import { defineChannelPluginEntry } from "openclaw/plugin-sdk/core";

export default defineChannelPluginEntry({
  id: "my-channel",
  name: "My Channel",
  description: "Short summary",
  plugin: myChannelPlugin,
  setRuntime: setMyRuntime,
  registerFull(api) {
    api.registerCli(/* ... */);
    api.registerGatewayMethod(/* ... */);
  },
});
```

| Field          | Type                                                             | Required | Default             |
| -------------- | ---------------------------------------------------------------- | -------- | ------------------- |
| `id`           | `string`                                                         | Yes      | —                   |
| `name`         | `string`                                                         | Yes      | —                   |
| `description`  | `string`                                                         | Yes      | —                   |
| `plugin`       | `ChannelPlugin`                                                  | Yes      | —                   |
| `configSchema` | `OpenClawPluginConfigSchema \| () => OpenClawPluginConfigSchema` | No       | Empty object schema |
| `setRuntime`   | `(runtime: PluginRuntime) => void`                               | No       | —                   |
| `registerFull` | `(api: OpenClawPluginApi) => void`                               | No       | —                   |

- `setRuntime`는 registration 중 호출되며 runtime 참조를 저장하는 데 씁니다.
- `registerFull`은 `api.registrationMode === "full"`일 때만 실행됩니다.

## `defineSetupPluginEntry`

**Import:** `openclaw/plugin-sdk/core`

가벼운 `setup-entry.ts` 파일용 helper입니다. runtime이나 CLI wiring 없이 `{ plugin }`만 반환합니다.

```typescript
import { defineSetupPluginEntry } from "openclaw/plugin-sdk/core";

export default defineSetupPluginEntry(myChannelPlugin);
```

OpenClaw는 채널이 비활성화됐거나 아직 구성되지 않았거나 deferred loading이 켜져 있을 때 full entry 대신 이것을 로드합니다. 자세한 내용은 [Setup and Config](/ko-KR/plugins/sdk-setup#setup-entry)를 보세요.

## Registration mode

`api.registrationMode`는 plugin이 어떤 방식으로 로드됐는지 알려줍니다:

| Mode              | When                              | What to register              |
| ----------------- | --------------------------------- | ----------------------------- |
| `"full"`          | Normal gateway startup            | Everything                    |
| `"setup-only"`    | Disabled/unconfigured channel     | Channel registration only     |
| `"setup-runtime"` | Setup flow with runtime available | Channel + lightweight runtime |

`defineChannelPluginEntry`는 이 분기를 자동 처리합니다. 채널에 대해 `definePluginEntry`를 직접 쓴다면 mode를 직접 확인해야 합니다:

```typescript
register(api) {
  api.registerChannel({ plugin: myPlugin });
  if (api.registrationMode !== "full") return;

  // Heavy runtime-only registrations
  api.registerCli(/* ... */);
  api.registerService(/* ... */);
}
```

## Plugin shape

OpenClaw는 plugin을 registration 동작에 따라 다음처럼 분류합니다:

| Shape                 | Description                                        |
| --------------------- | -------------------------------------------------- |
| **plain-capability**  | One capability type (e.g. provider-only)           |
| **hybrid-capability** | Multiple capability types (e.g. provider + speech) |
| **hook-only**         | Only hooks, no capabilities                        |
| **non-capability**    | Tools/commands/services but no capabilities        |

plugin의 shape은 `openclaw plugins inspect <id>`로 확인할 수 있습니다.

## 관련 문서

- [SDK Overview](/ko-KR/plugins/sdk-overview) - registration API와 subpath 레퍼런스
- [Runtime Helpers](/ko-KR/plugins/sdk-runtime) - `api.runtime`와 `createPluginRuntimeStore`
- [Setup and Config](/ko-KR/plugins/sdk-setup) - manifest, setup entry, deferred loading
- [Channel Plugins](/ko-KR/plugins/sdk-channel-plugins) - `ChannelPlugin` 객체 구성
- [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins) - provider registration과 hooks
