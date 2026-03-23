---
title: "Building Plugins"
sidebarTitle: "Getting Started"
summary: "몇 분 안에 첫 OpenClaw plugin 만들기"
read_when:
  - 새 OpenClaw plugin을 만들고 싶을 때
  - plugin 개발용 quick start가 필요할 때
  - 새로운 channel, provider, tool, 기타 capability를 OpenClaw에 추가할 때
---

# Building Plugins

plugin은 OpenClaw에 새로운 capability를 추가합니다: channel, model provider, speech, image generation, web search, agent tool, 또는 그 조합입니다.

plugin을 OpenClaw 저장소에 직접 추가할 필요는 없습니다. [ClawHub](/ko-KR/tools/clawhub) 또는 npm에 publish하면 사용자는 `openclaw plugins install <package-name>`으로 설치할 수 있습니다. OpenClaw는 먼저 ClawHub를 확인하고, 없으면 npm으로 fallback합니다.

## 필요 조건

- Node >= 22 및 package manager(npm 또는 pnpm)
- TypeScript(ESM) 기본 이해
- 저장소 내부 plugin이라면 repository clone + `pnpm install`

## 어떤 종류의 plugin인가

<CardGroup cols={3}>
  <Card title="Channel plugin" icon="messages-square" href="/ko-KR/plugins/sdk-channel-plugins">
    OpenClaw를 새로운 메시징 플랫폼(Discord, IRC 등)에 연결
  </Card>
  <Card title="Provider plugin" icon="cpu" href="/ko-KR/plugins/sdk-provider-plugins">
    새로운 모델 provider(LLM, proxy, custom endpoint) 추가
  </Card>
  <Card title="Tool / hook plugin" icon="wrench">
    agent tool, event hook, service 등록
  </Card>
</CardGroup>

## 빠른 시작: tool plugin

최소 plugin 하나를 만들어 agent tool을 등록하는 예시입니다.

<Steps>
  <Step title="패키지와 manifest 만들기">
    <CodeGroup>
    ```json package.json
    {
      "name": "@myorg/openclaw-my-plugin",
      "version": "1.0.0",
      "type": "module",
      "openclaw": {
        "extensions": ["./index.ts"]
      }
    }
    ```

    ```json openclaw.plugin.json
    {
      "id": "my-plugin",
      "name": "My Plugin",
      "description": "Adds a custom tool to OpenClaw",
      "configSchema": {
        "type": "object",
        "additionalProperties": false
      }
    }
    ```
    </CodeGroup>

    config가 없어도 manifest는 필요합니다. 전체 스키마는 [Manifest](/ko-KR/plugins/manifest)를 보세요.

  </Step>

  <Step title="entry point 작성">
    ```typescript
    import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry";
    import { Type } from "@sinclair/typebox";

    export default definePluginEntry({
      id: "my-plugin",
      name: "My Plugin",
      description: "Adds a custom tool to OpenClaw",
      register(api) {
        api.registerTool({
          name: "my_tool",
          description: "Do a thing",
          parameters: Type.Object({ input: Type.String() }),
          async execute(_id, params) {
            return { content: [{ type: "text", text: `Got: ${params.input}` }] };
          },
        });
      },
    });
    ```

    `definePluginEntry`는 non-channel plugin용입니다. channel은 `defineChannelPluginEntry`를 사용하세요.

  </Step>

  <Step title="테스트와 배포">
    외부 plugin은 ClawHub 또는 npm에 publish한 뒤 설치합니다:

    ```bash
    openclaw plugins install @myorg/openclaw-my-plugin
    ```

    저장소 내부 plugin은 `extensions/` 아래에 두면 자동 감지됩니다.

    ```bash
    pnpm test -- extensions/my-plugin/
    ```

  </Step>
</Steps>

## Plugin capability

하나의 plugin은 `api` 객체를 통해 여러 capability를 등록할 수 있습니다:

| Capability           | Registration method                           | Detailed guide                                                                        |
| -------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------- |
| Text inference (LLM) | `api.registerProvider(...)`                   | [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins)                               |
| Channel / messaging  | `api.registerChannel(...)`                    | [Channel Plugins](/ko-KR/plugins/sdk-channel-plugins)                                 |
| Speech (TTS/STT)     | `api.registerSpeechProvider(...)`             | [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins#step-5-add-extra-capabilities) |
| Media understanding  | `api.registerMediaUnderstandingProvider(...)` | [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins#step-5-add-extra-capabilities) |
| Image generation     | `api.registerImageGenerationProvider(...)`    | [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins#step-5-add-extra-capabilities) |
| Web search           | `api.registerWebSearchProvider(...)`          | [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins#step-5-add-extra-capabilities) |
| Agent tools          | `api.registerTool(...)`                       | 아래 설명                                                                             |
| Event hooks          | `api.registerHook(...)`                       | [Entry Points](/ko-KR/plugins/sdk-entrypoints)                                        |
| HTTP routes          | `api.registerHttpRoute(...)`                  | [Internals](/ko-KR/plugins/architecture#gateway-http-routes)                          |
| CLI subcommands      | `api.registerCli(...)`                        | [Entry Points](/ko-KR/plugins/sdk-entrypoints)                                        |

## Agent tool 등록

tool은 LLM이 호출할 수 있는 typed function입니다. 항상 사용 가능한 필수 tool도 있고, 사용자가 allowlist로 켜야 하는 optional tool도 있습니다.

```typescript
register(api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({ input: Type.String() }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });

  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a workflow",
      parameters: Type.Object({ pipeline: Type.String() }),
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

사용자는 config에서 optional tool을 켭니다:

```json5
{
  tools: { allow: ["workflow_tool"] },
}
```

## Import 규칙

항상 `openclaw/plugin-sdk/<subpath>`의 좁은 경로에서 import하세요:

```typescript
import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry";
import { createPluginRuntimeStore } from "openclaw/plugin-sdk/runtime-store";
```

plugin 내부 import는 로컬 barrel 파일(`api.ts`, `runtime-api.ts`)로 정리하고, 자기 plugin을 SDK 경로로 다시 import하지 마세요.

## 제출 전 체크리스트

<Check>`package.json`에 올바른 `openclaw` metadata가 있는가</Check>
<Check>`openclaw.plugin.json` manifest가 유효한가</Check>
<Check>entry point가 `defineChannelPluginEntry` 또는 `definePluginEntry`를 쓰는가</Check>
<Check>import가 모두 `plugin-sdk/<subpath>`를 쓰는가</Check>
<Check>테스트가 통과하는가</Check>

## 다음 단계

- [Channel Plugins](/ko-KR/plugins/sdk-channel-plugins)
- [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins)
- [SDK Overview](/ko-KR/plugins/sdk-overview)
- [Runtime Helpers](/ko-KR/plugins/sdk-runtime)
- [Testing](/ko-KR/plugins/sdk-testing)
- [Plugin Manifest](/ko-KR/plugins/manifest)
