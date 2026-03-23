---
summary: "Context engine: 플러그형 컨텍스트 조립, compaction, subagent lifecycle"
read_when:
  - OpenClaw가 모델 컨텍스트를 어떻게 조립하는지 이해하려 할 때
  - legacy engine과 plugin engine 사이를 전환하려 할 때
  - context engine plugin을 만들고 있을 때
title: "Context Engine"
---

# Context Engine

**context engine**은 OpenClaw가 각 실행마다 모델 컨텍스트를 어떻게 구성할지 제어합니다. 어떤 메시지를 포함할지, 오래된 히스토리를 어떻게 요약할지, subagent 경계에서 컨텍스트를 어떻게 관리할지를 결정합니다.

OpenClaw는 내장 `legacy` engine을 제공하며, plugin은 이를 대체하는 다른 engine을 등록할 수 있습니다.

## 빠른 시작

활성 engine 확인:

```bash
openclaw doctor
cat ~/.openclaw/openclaw.json | jq '.plugins.slots.contextEngine'
```

### Context engine plugin 설치

다른 OpenClaw plugin과 같은 방식으로 설치합니다:

```bash
openclaw plugins install @martian-engineering/lossless-claw
openclaw plugins install -l ./my-context-engine
```

설치 후 config에서 활성 engine을 선택합니다:

```json5
{
  plugins: {
    slots: {
      contextEngine: "lossless-claw",
    },
    entries: {
      "lossless-claw": {
        enabled: true,
      },
    },
  },
}
```

설정 후 Gateway를 재시작하세요. 내장 engine으로 돌아가려면 `contextEngine`을 `"legacy"`로 두거나 키를 제거하면 됩니다.

## 동작 방식

OpenClaw는 각 모델 실행마다 context engine을 네 lifecycle 지점에서 호출합니다:

1. **Ingest** - 새 메시지를 세션에 추가할 때
2. **Assemble** - 모델 실행 직전, token budget 안의 메시지를 조립할 때
3. **Compact** - 컨텍스트 창이 가득 찼거나 `/compact`를 실행할 때
4. **After turn** - 실행이 끝난 뒤 후처리를 할 때

### Subagent lifecycle

현재는 다음 hook만 실제 호출됩니다:

- **onSubagentEnded** - subagent 세션이 끝나거나 sweep될 때 정리

`prepareSubagentSpawn`은 인터페이스에는 있지만 아직 런타임이 호출하지 않습니다.

### System prompt addition

`assemble`은 `systemPromptAddition` 문자열을 반환할 수 있습니다. OpenClaw는 이를 실행용 system prompt 앞에 붙여서 동적 회상 지침이나 context-aware 힌트를 주입할 수 있게 합니다.

## Legacy engine

내장 `legacy` engine은 원래 OpenClaw 동작을 그대로 유지합니다:

- **Ingest**: no-op
- **Assemble**: pass-through
- **Compact**: 내장 요약 compaction에 위임
- **After turn**: no-op

`plugins.slots.contextEngine`이 없거나 `"legacy"`일 때 자동 사용됩니다.

## Plugin engine

plugin API를 통해 등록할 수 있습니다:

```ts
export default function register(api) {
  api.registerContextEngine("my-engine", () => ({
    info: {
      id: "my-engine",
      name: "My Context Engine",
      ownsCompaction: true,
    },
    async ingest() {
      return { ingested: true };
    },
    async assemble({ messages, tokenBudget }) {
      return {
        messages: buildContext(messages, tokenBudget),
        estimatedTokens: countTokens(messages),
        systemPromptAddition: "Use lcm_grep to search history...",
      };
    },
    async compact() {
      return { ok: true, compacted: true };
    },
  }));
}
```

## 인터페이스 핵심

필수 멤버:

| Member             | Purpose                       |
| ------------------ | ----------------------------- |
| `info`             | Engine id, name, version 정보 |
| `ingest(params)`   | 단일 메시지 저장              |
| `assemble(params)` | 모델 실행용 컨텍스트 구성     |
| `compact(params)`  | 컨텍스트 요약/축소            |

선택 멤버:

- `bootstrap(params)`
- `ingestBatch(params)`
- `afterTurn(params)`
- `prepareSubagentSpawn(params)`
- `onSubagentEnded(params)`
- `dispose()`

### ownsCompaction

`ownsCompaction`은 Pi의 내장 auto-compaction을 유지할지 결정합니다:

- `true` - engine이 compaction을 직접 책임짐
- `false` 또는 미설정 - 내장 auto-compaction이 계속 동작할 수 있음

## Compaction과 memory의 관계

- **Compaction**은 context engine 책임 중 하나입니다.
- **Memory plugin**은 검색/검색 결과를 담당하고, context engine은 모델이 실제로 볼 내용을 결정합니다.
- **Session pruning**은 어떤 engine이 활성인지와 무관하게 계속 동작합니다.

## 팁

- `openclaw doctor`로 engine 로딩 상태를 확인하세요.
- engine을 바꿔도 기존 세션 히스토리는 유지되고, 새 engine은 이후 실행부터 적용됩니다.
- engine 오류는 로그와 diagnostics에 표시됩니다.
