---
summary: "Pi, Claude Code, Codex, OpenCode, Gemini CLI 등 ACP 런타임 세션 사용법"
read_when:
  - ACP를 통해 코딩 하니스(harness)를 실행할 때
  - 스레드 가능한 채널에서 thread-bound ACP 세션을 설정할 때
  - Discord 채널이나 Telegram 포럼 토픽을 지속형 ACP 세션에 바인딩할 때
  - ACP 백엔드/플러그인 배선을 디버깅할 때
  - 채팅에서 `/acp` 명령을 운영할 때
title: "ACP Agents"
---

# ACP Agents

[Agent Client Protocol (ACP)](https://agentclientprotocol.com/) 세션을 사용하면 OpenClaw가 외부 코딩 하니스(Pi, Claude Code, Codex, OpenCode, Gemini CLI 등)를 ACP 백엔드 플러그인을 통해 실행할 수 있습니다.

사용자가 자연어로 "Codex로 이거 돌려" 또는 "스레드에서 Claude Code 시작해"라고 하면, OpenClaw는 이를 네이티브 sub-agent 런타임이 아니라 ACP 런타임으로 라우팅해야 합니다.

## 빠른 운영 흐름

1. 세션 시작:
   - `/acp spawn codex --mode persistent --thread auto`
2. 바인딩된 스레드에서 작업하거나 해당 세션 키를 명시적으로 지정
3. 런타임 상태 확인:
   - `/acp status`
4. 필요 시 옵션 조정:
   - `/acp model <provider/model>`
   - `/acp permissions <profile>`
   - `/acp timeout <seconds>`
5. 컨텍스트를 버리지 않고 방향만 수정:
   - `/acp steer tighten logging and continue`
6. 작업 중지:
   - `/acp cancel` 또는
   - `/acp close`

## ACP와 sub-agents의 차이

ACP는 외부 하니스 런타임이 필요할 때 사용하고, sub-agent는 OpenClaw 네이티브 위임 실행에 사용합니다.

| 항목       | ACP 세션                           | Sub-agent 실행                      |
| ---------- | ---------------------------------- | ----------------------------------- |
| 런타임     | ACP backend plugin (예: acpx)      | OpenClaw 네이티브 sub-agent runtime |
| 세션 키    | `agent:<agentId>:acp:<uuid>`       | `agent:<agentId>:subagent:<uuid>`   |
| 주요 명령  | `/acp ...`                         | `/subagents ...`                    |
| spawn 도구 | `sessions_spawn` + `runtime:"acp"` | 기본 `sessions_spawn`               |

관련 문서: [Sub-agents](/ko-KR/tools/subagents)

## Thread-bound ACP

채널 어댑터에서 thread binding이 활성화돼 있으면 ACP 세션을 스레드에 바인딩할 수 있습니다.

- OpenClaw가 스레드를 대상 ACP 세션에 바인딩
- 후속 메시지는 같은 ACP 세션으로 라우팅
- 출력도 같은 스레드로 전달
- unfocus/close/archive/idle-timeout/max-age 만료 시 바인딩 제거

필수 플래그:

- `acp.enabled=true`
- `acp.dispatch.enabled=true` (기본값)
- 채널별 ACP thread spawn 플래그
  - Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
  - Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

## 채널별 지속형 ACP 바인딩

비일회성 워크플로에는 최상위 `bindings[]`의 `type: "acp"` 항목을 사용합니다.

### 바인딩 모델

- `bindings[].type="acp"`: 지속형 ACP 대화 바인딩
- `bindings[].match`: 대상 대화 식별
  - Discord 채널/스레드: `match.channel="discord"` + `match.peer.id="<channelOrThreadId>"`
  - Telegram 포럼 토픽: `match.channel="telegram"` + `match.peer.id="<chatId>:topic:<topicId>"`
- `bindings[].agentId`: 소유 OpenClaw agent ID
- 선택적 ACP override: `bindings[].acp`
  - `mode`
  - `label`
  - `cwd`
  - `backend`

### 에이전트별 런타임 기본값

`agents.list[].runtime`에 ACP 기본값을 둘 수 있습니다.

- `agents.list[].runtime.type="acp"`
- `agents.list[].runtime.acp.agent`
- `agents.list[].runtime.acp.backend`
- `agents.list[].runtime.acp.mode`
- `agents.list[].runtime.acp.cwd`

우선순위:

1. `bindings[].acp.*`
2. `agents.list[].runtime.acp.*`
3. 전역 `acp.*`

## `/acp spawn`

명시적 운영 제어가 필요할 때는 채팅에서 `/acp spawn`을 사용합니다.

```text
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

주요 플래그:

- `--mode persistent|oneshot`
- `--thread auto|here|off`
- `--cwd <absolute-path>`
- `--label <name>`

## 세션 대상 해석

대부분의 `/acp` 동작은 선택적 세션 대상(`session-key`, `session-id`, `session-label`)을 받습니다.

해석 순서:

1. 명시적 대상 인수 (`/acp steer`는 `--session`)
2. 현재 thread binding
3. 현재 requester session fallback

해결되지 않으면 `Unable to resolve session target: ...` 오류를 반환합니다.

## `/acp` 명령 모음

- `/acp spawn`
- `/acp cancel`
- `/acp steer`
- `/acp close`
- `/acp status`
- `/acp set-mode`
- `/acp set`
- `/acp cwd`
- `/acp permissions`
- `/acp timeout`
- `/acp model`
- `/acp reset-options`
- `/acp sessions`
- `/acp doctor`
- `/acp install`

`/acp status`는 런타임 옵션과 가능한 경우 runtime/backend 양쪽 세션 식별자를 보여줍니다.

## 런타임 옵션 매핑

- `/acp model <id>` → `model`
- `/acp permissions <profile>` → `approval_policy`
- `/acp timeout <seconds>` → `timeout`
- `/acp cwd <path>` → cwd override
- `/acp set <key> <value>` → 일반 setter
- `/acp reset-options` → 대상 세션의 모든 runtime override 제거

## acpx 지원

현재 acpx 내장 alias:

- `pi`
- `claude`
- `codex`
- `opencode`
- `gemini`
- `kimi`

일반적인 `agentId`에는 위 값을 우선 사용하십시오.

## 기본 설정

```json5
{
  acp: {
    enabled: true,
    dispatch: { enabled: true },
    backend: "acpx",
    defaultAgent: "codex",
    allowedAgents: ["pi", "claude", "codex", "opencode", "gemini", "kimi"],
    maxConcurrentSessions: 8,
    stream: {
      coalesceIdleMs: 300,
      maxChunkChars: 1200,
    },
    runtime: {
      ttlMinutes: 120,
    },
  },
}
```

Discord 예시:

```json5
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        spawnAcpSessions: true,
      },
    },
  },
}
```

## 플러그인 설정

```bash
openclaw plugins install acpx
openclaw config set plugins.entries.acpx.enabled true
```

개발 중 로컬 설치:

```bash
openclaw plugins install ./extensions/acpx
```

그 다음 `/acp doctor`로 상태를 확인합니다.

## 권한 설정

ACP 세션은 비대화형으로 실행되므로 TTY 승인이 없습니다.

### `permissionMode`

- `approve-all`
- `approve-reads`
- `deny-all`

### `nonInteractivePermissions`

- `fail` (기본값)
- `deny`

## 샌드박스 호환성

ACP 세션은 현재 OpenClaw sandbox 안이 아니라 호스트 런타임에서 실행됩니다.

- sandboxed requester session에서는 `runtime:"acp"` spawn이 차단됩니다.
- `sessions_spawn`에서 `runtime:"acp"`와 `sandbox:"require"` 조합은 지원되지 않습니다.

샌드박스 강제가 필요하면 `runtime:"subagent"`를 사용하십시오.

## 관련 문서

- [슬래시 명령어](/ko-KR/tools/slash-commands)
- [플러그인](/ko-KR/tools/plugin)
- [구성 참조](/ko-KR/gateway/configuration-reference)
- [Sub-agents](/ko-KR/tools/subagents)

### 최근 업데이트 메모

- `resumeSessionId`로 기존 ACP 세션을 이어받을 수 있으며, 이력은 `session/load`로 재생됩니다.
- `/acp sessions`는 현재 바인딩되었거나 요청자 세션 기준 session store를 읽습니다.
