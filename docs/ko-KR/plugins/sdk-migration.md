---
title: "Plugin SDK Migration"
sidebarTitle: "Migrate to SDK"
summary: "레거시 하위 호환 레이어에서 현대적인 plugin SDK로 이전하기"
read_when:
  - `OPENCLAW_PLUGIN_SDK_COMPAT_DEPRECATED` 경고를 볼 때
  - `OPENCLAW_EXTENSION_API_DEPRECATED` 경고를 볼 때
  - plugin을 최신 plugin 아키텍처로 업데이트하려 할 때
  - 외부 OpenClaw plugin을 유지보수할 때
---

# Plugin SDK Migration

OpenClaw는 넓은 하위 호환 레이어에서, 목적이 분명하고 문서화된 import를 갖는 현대적인 plugin 아키텍처로 이동했습니다. 새 아키텍처 이전에 만들어진 plugin이라면 이 가이드가 이전을 도와줍니다.

## 무엇이 바뀌는가

기존 plugin 시스템은 plugin이 단일 진입점에서 필요한 거의 모든 것을 import할 수 있도록 두 가지 넓은 surface를 제공했습니다:

- **`openclaw/plugin-sdk/compat`** - 수십 개 helper를 한 번에 re-export하는 단일 import
- **`openclaw/extension-api`** - embedded agent runner 같은 host-side helper에 직접 접근하게 해 주는 bridge

두 surface 모두 이제 **deprecated**입니다. 런타임에서는 아직 동작하지만, 새 plugin은 사용하면 안 되고, 기존 plugin도 다음 major release 전에 이전해야 합니다.

<Warning>
  하위 호환 레이어는 향후 major release에서 제거됩니다. 여전히 이 surface를 import하는 plugin은 그 시점에 깨지게 됩니다.
</Warning>

## 왜 바뀌었는가

예전 방식에는 문제가 있었습니다:

- **느린 startup** - helper 하나를 import해도 관련 없는 모듈까지 많이 로드됨
- **순환 의존성** - 폭넓은 re-export 때문에 import cycle을 만들기 쉬움
- **불명확한 API surface** - 어떤 export가 안정적인지, 어떤 것이 internal인지 구분이 어려움

현대적인 plugin SDK는 이를 해결합니다. 각 import path(`openclaw/plugin-sdk/<subpath>`)는 목적이 분명하고 문서화된 작은 독립 모듈입니다.

## 이전 방법

<Steps>
  <Step title="deprecated import 찾기">
    plugin에서 두 deprecated surface를 검색합니다:

    ```bash
    grep -r "plugin-sdk/compat" my-plugin/
    grep -r "openclaw/extension-api" my-plugin/
    ```

  </Step>

  <Step title="집중된 import로 교체">
    예전 surface의 export는 현대적인 import path로 매핑됩니다:

    ```typescript
    // Before (deprecated backwards-compatibility layer)
    import {
      createChannelReplyPipeline,
      createPluginRuntimeStore,
      resolveControlCommandGate,
    } from "openclaw/plugin-sdk/compat";

    // After (modern focused imports)
    import { createChannelReplyPipeline } from "openclaw/plugin-sdk/channel-reply-pipeline";
    import { createPluginRuntimeStore } from "openclaw/plugin-sdk/runtime-store";
    import { resolveControlCommandGate } from "openclaw/plugin-sdk/command-auth";
    ```

    host-side helper는 직접 import하지 말고 주입된 plugin runtime을 사용하세요:

    ```typescript
    // Before (deprecated extension-api bridge)
    import { runEmbeddedPiAgent } from "openclaw/extension-api";
    const result = await runEmbeddedPiAgent({ sessionId, prompt });

    // After (injected runtime)
    const result = await api.runtime.agent.runEmbeddedPiAgent({ sessionId, prompt });
    ```

    다른 레거시 bridge helper도 같은 방식으로 옮깁니다:

    | Old import | Modern equivalent |
    | --- | --- |
    | `resolveAgentDir` | `api.runtime.agent.resolveAgentDir` |
    | `resolveAgentWorkspaceDir` | `api.runtime.agent.resolveAgentWorkspaceDir` |
    | `resolveAgentIdentity` | `api.runtime.agent.resolveAgentIdentity` |
    | `resolveThinkingDefault` | `api.runtime.agent.resolveThinkingDefault` |
    | `resolveAgentTimeoutMs` | `api.runtime.agent.resolveAgentTimeoutMs` |
    | `ensureAgentWorkspace` | `api.runtime.agent.ensureAgentWorkspace` |
    | session store helpers | `api.runtime.agent.session.*` |

  </Step>

  <Step title="빌드와 테스트">
    ```bash
    pnpm build
    pnpm test -- my-plugin/
    ```
  </Step>
</Steps>

## Import path 레퍼런스

| Import path                         | Purpose                                                 | Key exports                                           |
| ----------------------------------- | ------------------------------------------------------- | ----------------------------------------------------- |
| `plugin-sdk/plugin-entry`           | Canonical plugin entry helper                           | `definePluginEntry`                                   |
| `plugin-sdk/core`                   | Channel entry definitions, channel builders, base types | `defineChannelPluginEntry`, `createChatChannelPlugin` |
| `plugin-sdk/channel-setup`          | Setup wizard adapters                                   | `createOptionalChannelSetupSurface`                   |
| `plugin-sdk/channel-pairing`        | DM pairing primitives                                   | `createChannelPairingController`                      |
| `plugin-sdk/channel-reply-pipeline` | Reply prefix + typing wiring                            | `createChannelReplyPipeline`                          |
| `plugin-sdk/runtime-store`          | Persistent plugin storage                               | `createPluginRuntimeStore`                            |
| `plugin-sdk/command-auth`           | Command gating                                          | `resolveControlCommandGate`                           |
| `plugin-sdk/testing`                | Test utilities                                          | Test helpers and mocks                                |

가능한 한 가장 좁은 import를 사용하세요. 원하는 export를 못 찾겠다면 `src/plugin-sdk/`를 확인하거나 Discord에서 물어보는 것이 가장 빠릅니다.

## 제거 일정

| When                   | What happens                                     |
| ---------------------- | ------------------------------------------------ |
| **Now**                | Deprecated surface가 런타임 경고를 출력          |
| **Next major release** | Deprecated surface 제거, 아직 쓰는 plugin은 실패 |

core plugin은 이미 모두 이전되었습니다. 외부 plugin도 다음 major release 전에 옮기는 것이 안전합니다.

## 경고를 임시로 숨기기

이전 작업 중 잠시 경고를 숨기려면:

```bash
OPENCLAW_SUPPRESS_PLUGIN_SDK_COMPAT_WARNING=1 openclaw gateway run
OPENCLAW_SUPPRESS_EXTENSION_API_WARNING=1 openclaw gateway run
```

임시 escape hatch일 뿐, 영구 해결책은 아닙니다.

## 관련 문서

- [Getting Started](/ko-KR/plugins/building-plugins) - 첫 plugin 만들기
- [SDK Overview](/ko-KR/plugins/sdk-overview) - 전체 subpath import 레퍼런스
- [Channel Plugins](/ko-KR/plugins/sdk-channel-plugins) - channel plugin 작성
- [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins) - provider plugin 작성
- [Plugin Internals](/ko-KR/plugins/architecture) - 아키텍처 deep dive
- [Plugin Manifest](/ko-KR/plugins/manifest) - manifest 스키마 레퍼런스
