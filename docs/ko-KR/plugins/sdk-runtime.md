---
title: "Plugin Runtime Helpers"
sidebarTitle: "Runtime Helpers"
summary: "api.runtime - plugin에 주입되는 runtime helper"
read_when:
  - plugin에서 core helper(TTS, STT, image gen, web search, subagent)를 호출해야 할 때
  - `api.runtime`가 무엇을 노출하는지 알고 싶을 때
  - plugin 코드에서 config, agent, media helper를 사용할 때
---

# Plugin Runtime Helpers

모든 plugin registration 시 주입되는 `api.runtime` 객체의 레퍼런스입니다. host internal을 직접 import하지 말고 이 helper를 사용하세요.

```typescript
register(api) {
  const runtime = api.runtime;
}
```

## 주요 namespace

### `api.runtime.agent`

- agent directory / workspace 경로 해석
- agent identity 조회
- default thinking/timeout 계산
- workspace 보장
- embedded Pi agent 실행
- session store helper (`api.runtime.agent.session.*`)

### `api.runtime.subagent`

- background subagent 실행
- run 완료 대기
- subagent session message 읽기
- session 삭제

### `api.runtime.tts`

- text-to-speech 변환
- telephony 최적화 TTS
- 사용 가능한 voice 목록 조회

### `api.runtime.mediaUnderstanding`

- 이미지 설명
- 오디오 전사
- 비디오 설명
- 일반 파일 분석

`api.runtime.stt.transcribeAudioFile(...)`는 호환용 alias로 남아 있습니다.

### `api.runtime.imageGeneration`

- 이미지 생성
- image generation provider 목록 조회

### `api.runtime.webSearch`

- web search provider 목록 조회
- 검색 실행

### `api.runtime.media`

- web media 로드
- MIME 감지
- media kind 판별
- voice-compatible audio 판별
- 이미지 metadata 조회
- JPEG resize

### `api.runtime.config`

- config 로드
- config 파일 쓰기

### `api.runtime.system`

- system event enqueue
- heartbeat 즉시 요청
- timeout 포함 명령 실행
- native dependency hint 포맷

### `api.runtime.events`

- agent event 구독
- session transcript update 구독

### `api.runtime.logging`

- verbose logging 여부 확인
- child logger 생성

### `api.runtime.modelAuth`

- 모델/API key 해석
- provider별 auth 해석

### `api.runtime.state`

- state directory 경로 해석

### `api.runtime.tools`

- memory tool factory
- memory CLI registration

### `api.runtime.channel`

- channel plugin이 로드된 경우 사용 가능한 channel-specific helper

## Runtime 참조 저장

`register` 바깥 코드에서 runtime이 필요하면 `createPluginRuntimeStore`를 사용하세요:

```typescript
import { createPluginRuntimeStore } from "openclaw/plugin-sdk/runtime-store";
import type { PluginRuntime } from "openclaw/plugin-sdk/runtime-store";

const store = createPluginRuntimeStore<PluginRuntime>("my-plugin runtime not initialized");
```

entry point에서는 `setRuntime: store.setRuntime`으로 연결하고, 다른 파일에서는 `store.getRuntime()` 또는 `store.tryGetRuntime()`을 씁니다.

## `api`의 다른 top-level 필드

| Field                    | Type                      | Description                                               |
| ------------------------ | ------------------------- | --------------------------------------------------------- |
| `api.id`                 | `string`                  | Plugin id                                                 |
| `api.name`               | `string`                  | Plugin display name                                       |
| `api.config`             | `OpenClawConfig`          | Current config snapshot                                   |
| `api.pluginConfig`       | `Record<string, unknown>` | Plugin-specific config from `plugins.entries.<id>.config` |
| `api.logger`             | `PluginLogger`            | Scoped logger                                             |
| `api.registrationMode`   | `PluginRegistrationMode`  | `"full"`, `"setup-only"`, or `"setup-runtime"`            |
| `api.resolvePath(input)` | `(string) => string`      | Resolve a path relative to the plugin root                |

## 관련 문서

- [SDK Overview](/ko-KR/plugins/sdk-overview)
- [SDK Entry Points](/ko-KR/plugins/sdk-entrypoints)
- [Plugin Internals](/ko-KR/plugins/architecture)
