---
summary: "채널별 라우팅 규칙 (WhatsApp, Telegram, Discord, Slack) 및 공유 컨텍스트"
read_when:
  - 채널 라우팅 또는 인박스 동작 변경
title: "채널 라우팅"
---

# 채널 및 라우팅

OpenClaw는 **메시지가 온 채널로** 응답을 라우팅합니다. 모델은 채널을 선택하지 않으며, 라우팅은 결정론적이며 호스트 구성에 의해 제어됩니다.

## 주요 용어

- **채널**: `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
- **AccountId**: 채널 별 계정 인스턴스 (지원되는 경우).
- 선택적 채널 기본 계정: `channels.<channel>.defaultAccount`는 아웃바운드 경로에 `accountId`가 지정되지 않았을 때 사용할 계정을 선택합니다.
  - 다중 계정 구성에서 계정이 2개 이상이면 명시적 기본값(`defaultAccount` 또는 `accounts.default`)을 설정하세요. 없으면 폴백 라우팅이 정규화된 첫 번째 계정 ID를 선택할 수 있습니다.
- **AgentId**: 격리된 워크스페이스 + 세션 저장소 ("브레인").
- **SessionKey**: 컨텍스트를 저장하고 동시성을 제어하는 데 사용되는 버킷 키.

## 세션 키 형태 (예시)

다이렉트 메시지는 에이전트의 **메인** 세션으로 합쳐집니다:

- `agent:<agentId>:<mainKey>` (기본값: `agent:main:main`)

그룹과 채널은 채널별로 격리되어 유지됩니다:

- 그룹: `agent:<agentId>:<channel>:group:<id>`
- 채널/룸: `agent:<agentId>:<channel>:channel:<id>`

스레드:

- Slack/Discord 스레드는 기본 키에 `:thread:<threadId>`를 추가합니다.
- Telegram 포럼 주제는 그룹 키에 `:topic:<topicId>`를 포함합니다.

예시:

- `agent:main:telegram:group:-1001234567890:topic:42`
- `agent:main:discord:channel:123456:thread:987654`

## 메인 DM 라우트 고정

`session.dmScope`가 `main`이면 다이렉트 메시지가 하나의 메인 세션을 공유할 수 있습니다.
비소유자 DM이 세션의 `lastRoute`를 덮어쓰지 않도록, OpenClaw는 아래 조건을 모두 만족하면 `allowFrom`에서 고정 소유자를 추론합니다:

- `allowFrom`에 와일드카드가 아닌 항목이 정확히 1개 있음.
- 해당 항목을 채널의 구체적인 발신자 ID로 정규화할 수 있음.
- 인바운드 DM 발신자가 그 고정 소유자와 일치하지 않음.

이 불일치 상황에서 OpenClaw는 인바운드 세션 메타데이터는 기록하지만, 메인 세션 `lastRoute` 업데이트는 건너뜁니다.

## 라우팅 규칙 (에이전트가 선택되는 방법)

라우팅은 각 인바운드 메시지에 대해 **하나의 에이전트**를 선택합니다:

1. **정확한 피어 매칭** (`peer.kind` + `peer.id`가 있는 `bindings`).
2. **상위 피어 매칭** (스레드 상속).
3. **길드 + 역할 매칭** (Discord) via `guildId` + `roles`.
4. **길드 매칭** (Discord) via `guildId`.
5. **팀 매칭** (Slack) via `teamId`.
6. **계정 매칭** (채널의 `accountId`).
7. **채널 매칭** (그 채널의 모든 계정, `accountId: "*"`).
8. **기본 에이전트** (`agents.list[].default`, 없으면 첫 번째 리스트 항목, 기본값은 `main`).

바인딩에 여러 매칭 필드(`peer`, `guildId`, `teamId`, `roles`)가 포함된 경우, **제공된 모든 필드가 일치해야** 그 바인딩이 적용됩니다.

매칭된 에이전트는 사용될 워크스페이스와 세션 저장소를 결정합니다.

## 브로드캐스트 그룹 (여러 에이전트 실행)

브로드캐스트 그룹은 **OpenClaw가 일반적으로 응답할 때** 동일한 피어에 대해 **여러 에이전트**를 실행할 수 있게 해줍니다 (예: WhatsApp 그룹에서 멘션/활성화 게이팅 후).

설정:

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"],
  },
}
```

참조: [Broadcast Groups](/ko-KR/channels/broadcast-groups).

## 구성 개요

- `agents.list`: 명명된 에이전트 정의 (워크스페이스, 모델 등).
- `bindings`: 인바운드 채널/계정/피어를 에이전트에 매핑.

예시:

```json5
{
  agents: {
    list: [{ id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }],
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" },
  ],
}
```

## 세션 저장소

세션 저장소는 상태 디렉터리 (기본값 `~/.openclaw`) 아래에 존재합니다:

- `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- JSONL 전사는 저장소 옆에 위치합니다

`session.store` 및 `{agentId}` 템플릿을 통해 저장소 경로를 재정의할 수 있습니다.

Gateway 및 ACP 세션 검색은 기본 `agents/` 루트와 템플릿이 적용된 `session.store` 루트 아래의 디스크 기반 에이전트 저장소도 함께 검사합니다. 검색된 저장소는 해결된 에이전트 루트 내부에 있어야 하며 일반 `sessions.json` 파일을 사용해야 합니다. 심볼릭 링크와 루트 밖 경로는 무시됩니다.

## WebChat 동작

WebChat은 **선택된 에이전트**에 연결되어 에이전트의 메인 세션을 기본값으로 사용합니다. 이 때문에, WebChat은 해당 에이전트의 크로스‑채널 컨텍스트를 한 곳에서 볼 수 있게 합니다.

## 응답 컨텍스트

인바운드 응답에는 다음이 포함됩니다:

- `ReplyToId`, `ReplyToBody`, 및 `ReplyToSender` (가능한 경우).
- 인용된 컨텍스트는 `[Replying to ...]` 블록으로 `Body`에 추가됩니다.

이는 채널 간에 일관됩니다.
