---
summary: "Telegram 봇 지원 상태, 기능 및 구성"
read_when:
  - Telegram 기능 또는 웹훅 작업 중
title: "Telegram"
---

# Telegram (Bot API)

상태: grammY를 통해 봇 다이렉트 메시지 + 그룹에서 프로덕션 레디. 기본 모드는 롱 폴링이며 웹훅 모드는 선택 사항입니다.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/ko-KR/channels/pairing">
    Telegram의 기본 다이렉트 메시지 정책은 페어링입니다.
  </Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/ko-KR/channels/troubleshooting">
    크로스 채널 진단 및 수리 플레이북.
  </Card>
  <Card title="Gateway configuration" icon="settings" href="/ko-KR/gateway/configuration">
    전체 채널 구성 패턴 및 예제.
  </Card>
</CardGroup>

## 빠른 설정

<Steps>
  <Step title="BotFather에서 봇 토큰 만들기">
    Telegram을 열고 **@BotFather**와 대화하여 핸들이 정확히 `@BotFather`인지 확인합니다.

    `/newbot`을 실행하고 안내에 따라 진행하며 토큰을 저장합니다.

  </Step>

  <Step title="토큰 및 DM 정책 구성">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    환경 변수 대체: `TELEGRAM_BOT_TOKEN=...` (기본 계정에만 적용).
    Telegram은 `openclaw channels login telegram` 을 **사용하지 않습니다**; 설정/환경 변수에서 토큰을 구성한 후 게이트웨이를 시작하세요.

  </Step>

  <Step title="게이트웨이 시작 및 첫 DM 승인">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    페어링 코드는 1시간 후 만료됩니다.

  </Step>

  <Step title="봇을 그룹에 추가하기">
    봇을 그룹에 추가한 후 `channels.telegram.groups`와 `groupPolicy`를 설정하여 액세스 모델에 맞춥니다.
  </Step>
</Steps>

<Note>
토큰 해석 순서는 계정 인식에 따라 다릅니다. 실제로 구성 값이 환경 변수 대체보다 우선시되며, `TELEGRAM_BOT_TOKEN`은 기본 계정에만 적용됩니다.
</Note>

## Telegram 쪽 설정

<AccordionGroup>
  <Accordion title="프라이버시 모드 및 그룹 가시성">
    Telegram 봇은 기본적으로 **프라이버시 모드**로 설정되어 있으며, 수신할 수 있는 그룹 메시지를 제한합니다.

    봇이 모든 그룹 메시지를 수신해야 하는 경우:

    - `/setprivacy`를 통해 프라이버시 모드를 비활성화하거나,
    - 봇을 그룹 관리자로 설정합니다.

    프라이버시 모드를 전환할 때, 각 그룹에서 봇을 제거하고 다시 추가하여 Telegram이 변경 사항을 적용하도록 합니다.

  </Accordion>

  <Accordion title="그룹 권한">
    관리자 상태는 Telegram 그룹 설정에서 제어됩니다.

    관리자 봇은 모든 그룹 메시지를 수신하므로 항상 그룹 행동이 필요한 경우 유용합니다.

  </Accordion>

  <Accordion title="유용한 BotFather 토글">

    - `/setjoingroups`로 그룹 추가 허용/거부
    - `/setprivacy`로 그룹 가시성 행동 제어

  </Accordion>
</AccordionGroup>

## 액세스 제어 및 활성화

<Tabs>
  <Tab title="DM 정책">
    `channels.telegram.dmPolicy`는 다이렉트 메시지 액세스를 제어합니다:

    - `pairing` (기본)
    - `allowlist`
    - `open` (`allowFrom`에 `"*"` 포함 필요)
    - `disabled`

    `channels.telegram.allowFrom`은 Telegram 사용자 ID를 숫자로 받습니다. `telegram:` / `tg:` 접두사는 허용되고 정규화됩니다.
    온보딩 마법사는 `@username` 입력을 허용하고 숫자 ID로 변환합니다.
    업그레이드했으며 구성에 `@username` 허용 목록 항목이 있는 경우, `openclaw doctor --fix`를 실행하여 이를 해결하세요 (최대한의 노력; Telegram 봇 토큰 필요).
    한 명의 소유자만 사용하는 봇이라면 과거 페어링 승인 기록에 기대기보다 숫자 `allowFrom` ID를 명시한 `dmPolicy: "allowlist"`를 권장합니다.

    ### Telegram 사용자 ID 찾기

    더 안전한 방법 (서드파티 봇 없이):

    1. 봇에게 DM을 보냅니다.
    2. `openclaw logs --follow`를 실행합니다.
    3. `from.id`를 읽습니다.

    공식 Bot API 방법:

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    서드파티 방법 (덜 개인적인): `@userinfobot` 또는 `@getidsbot`.

  </Tab>

  <Tab title="그룹 정책 및 허용 목록">
    두 가지 독립적인 제어가 있습니다:

    1. **허용된 그룹** (`channels.telegram.groups`)
       - `groups` 설정 없음: 모든 그룹 허용
       - `groups` 설정: 허용 목록으로 작동 (명시적 ID 또는 `"*"`)

    2. **그룹에서 허용된 발신자** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (기본)
       - `disabled`

    `groupAllowFrom`은 그룹 발신자 필터링에 사용됩니다. 설정되지 않은 경우 Telegram은 `allowFrom`을 기본으로 사용합니다.
    `groupAllowFrom` 항목은 숫자로 된 Telegram 사용자 ID여야 합니다.
    런타임 참고: `channels.telegram` 이 완전히 없는 경우, 런타임은 그룹 정책 평가에 대해 `groupPolicy="allowlist"` 로 폴백합니다 (`channels.defaults.groupPolicy` 가 설정되어 있어도).

    예: 특정 그룹에서 모든 멤버를 허용하려면:

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  </Tab>

  <Tab title="언급 행동">
    그룹 응답은 기본적으로 언급을 필요로 합니다.

    언급은 다음에서 올 수 있습니다:

    - 기본 `@botusername` 언급, 또는
    - 언급 패턴에서:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`

    세션 레벨 명령어 토글:

    - `/activation always`
    - `/activation mention`

    이는 세션 상태만 업데이트합니다. 영구성을 위해 구성 설정 사용.

    영구 설정 예:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    그룹 채팅 ID 얻기:

    - 그룹 메시지를 `@userinfobot` / `@getidsbot`에 포워드하기
    - `openclaw logs --follow`에서 `chat.id` 읽기
    - 또는 Bot API `getUpdates` 검사

  </Tab>
</Tabs>

## 런타임 동작

- Telegram은 게이트웨이 프로세스에서 소유됩니다.
- 라우팅은 결정적입니다: Telegram 수신 응답은 Telegram으로 돌아갑니다 (모델이 채널을 선택하지 않음).
- 수신 메시지는 응답 메타데이터와 미디어 플레이스홀더를 포함하여 공유 채널 봉투로 정규화됩니다.
- 그룹 세션은 그룹 ID로 격리됩니다. 포럼 주제에 대해 `:topic:<threadId>`를 추가하여 주제를 격리합니다.
- DM 메시지는 `message_thread_id`를 가질 수 있으며, OpenClaw는 이를 스레드 인식 세션 키로 라우팅하고, 응답 시 스레드 ID를 유지합니다.
- 롱 폴링은 각 채팅/스레드에 대한 순서를 제공하는 grammY 러너를 사용합니다. 전체 러너 싱크 동시성은 `agents.defaults.maxConcurrent`를 사용합니다.
- Telegram Bot API는 읽음 확인 지원이 없습니다 (`sendReadReceipts`는 적용되지 않음).

## 기능 참조

<AccordionGroup>
  <Accordion title="라이브 스트림 미리보기 (메시지 수정)">
    OpenClaw는 임시 Telegram 메시지를 전송하고 텍스트가 수신되면 이를 편집하여 부분적인 답장을 스트리밍할 수 있습니다.

    요구 사항:

    - `channels.telegram.streamMode`가 `"off"`가 아닌 경우 (기본값: `"partial"`)

    모드:

    - `off`: 라이브 미리보기 없음
    - `partial`: 부분 텍스트로 빈번한 미리보기 업데이트
    - `block`: `channels.telegram.draftChunk`를 사용하는 청크 미리보기 업데이트

    `streamMode: "block"`에 대한 `draftChunk` 기본값:

    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`

    `maxChars`는 `channels.telegram.textChunkLimit`로 제한됩니다.

    이는 직접 채팅과 그룹/주제에서 작동합니다.

    텍스트 전용 응답의 경우, OpenClaw는 동일한 미리보기 메시지를 유지하고 최종 편집을 한 곳에서 수행합니다 (두 번째 메시지 없음).

    복잡한 응답 (예: 미디어 페이로드)의 경우, OpenClaw는 일반적인 최종 전달로 되돌아가고 미리보기 메시지를 정리합니다.

    `streamMode`는 블록 스트리밍과 별개입니다. Telegram에 대해 블록 스트리밍이 명시적으로 활성화된 경우, OpenClaw는 이중 스트리밍을 피하기 위해 미리보기 스트림을 건너뜁니다.

    Telegram 전용 레이블링 스트림:

    - `/reasoning stream`은 생성 중인 동안 라이브 미리보기에 이유를 보냅니다
    - 최종 해답은 이유 텍스트 없이 전송됩니다

  </Accordion>

  <Accordion title="형식 지정 및 HTML 대체">
    아웃바운드 텍스트는 Telegram `parse_mode: "HTML"`을 사용합니다.

    - Markdown 유사 텍스트는 Telegram-safe HTML로 렌더링됩니다.
    - 모델 HTML 원본은 Telegram 파싱 오류를 줄이기 위해 이스케이프됩니다.
    - Telegram이 파싱된 HTML을 거부하면, OpenClaw는 평문 텍스트로 다시 시도합니다.

    링크 미리보기는 기본적으로 활성화되어 있으며 `channels.telegram.linkPreview: false`로 비활성화할 수 있습니다.

  </Accordion>

  <Accordion title="네이티브 명령어 및 사용자 정의 명령어">
    Telegram 명령어 메뉴 등록은 `setMyCommands`로 시작 시 처리됩니다.

    네이티브 명령어 기본값:

    - `commands.native: "auto"`는 Telegram에 대해 네이티브 명령어를 활성화합니다

    사용자 정의 명령어 메뉴 항목 추가:

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git 백업" },
        { command: "generate", description: "이미지 생성" },
      ],
    },
  },
}
```

    규칙:

    - 이름은 정규화됩니다 (선행 `/` 제거, 소문자)
    - 유효한 패턴: `a-z`, `0-9`, `_`, 길이 `1..32`
    - 사용자 정의 명령어는 네이티브 명령어를 재정의할 수 없음
    - 충돌/중복은 건너뛰고 로그됨

    노트:

    - 사용자 정의 명령어는 메뉴 항목일 뿐입니다; 자동으로 동작을 구현하지 않음
    - 플러그인/스킬 명령어는 Telegram 메뉴에 표시되지 않더라도 입력할 수 있음

    네이티브 명령어가 비활성화된 경우, 내장 기능은 제거됩니다. 사용자 정의/플러그인 명령어는 구성된 경우 여전히 등록할 수 있습니다.

    일반적인 설정 실패:

    - `setMyCommands failed`는 보통 `api.telegram.org`에 대한 DNS/HTTPS 접근성 문제가 있다는 것을 의미합니다.

    ### 디바이스 페어링 명령어 (`device-pair` 플러그인)

    `device-pair` 플러그인이 설치된 경우:

    1. `/pair`는 설정 코드를 생성합니다
    2. 코드를 iOS 앱에 붙여 넣기
    3. `/pair approve`는 최신 대기 요청을 승인합니다

    더 많은 정보: [페어링](/ko-KR/channels/pairing#pair-via-telegram-recommended-for-ios).

  </Accordion>

  <Accordion title="인라인 버튼">
    인라인 키보드 범위를 설정:

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    계정별 오버라이드:

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    범위:

    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (기본)

    레거시 `capabilities: ["inlineButtons"]`는 `inlineButtons: "all"`에 매핑됩니다.

    메세지 액션 예:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "옵션을 선택하세요:",
  buttons: [
    [
      { text: "예", callback_data: "yes" },
      { text: "아니오", callback_data: "no" },
    ],
    [{ text: "취소", callback_data: "cancel" }],
  ],
}
```

    콜백 클릭은 에이전트에 텍스트로 전달됩니다:
    `callback_data: <value>`

  </Accordion>

  <Accordion title="에이전트 및 자동화를 위한 Telegram 메시지 액션">
    Telegram 도구 액션에는 다음이 포함됩니다:

    - `sendMessage` (`to`, `content`, 선택적 `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)

    채널 메시지 액션은 인체공학적인 별칭을 노출합니다 (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).

    게이팅 컨트롤:

    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (기본: 비활성화)

    반응 제거 의미 체계: [/tools/reactions](/ko-KR/tools/reactions)

  </Accordion>

  <Accordion title="응답 스레딩 태그">
    Telegram은 생성된 출력에서 명시적인 응답 스레딩 태그를 지원합니다:

    - `[[reply_to_current]]`: 트리거링 메시지에 응답
    - `[[reply_to:<id>]]`: 특정 Telegram 메시지 ID에 응답

    `channels.telegram.replyToMode`는 처리를 제어합니다:

    - `off` (기본)
    - `first`
    - `all`

    노트: `off`는 암시적 응답 스레딩을 비활성화합니다. 명시적 `[[reply_to_*]]` 태그는 여전히 존중됩니다.

  </Accordion>

  <Accordion title="포럼 주제 및 스레드 동작">
    포럼 슈퍼그룹:

    - 주제 세션 키는 `:topic:<threadId>`를 추가합니다
    - 응답 및 입력 타깃은 주제 스레드입니다
    - 주제 구성 경로:
      `channels.telegram.groups.<chatId>.topics.<threadId>`

    일반 주제 (`threadId=1`) 특별 케이스:

    - 메시지 전송은 `message_thread_id`를 생략합니다 (Telegram은 `sendMessage(...thread_id=1)`을 거부함)
    - 입력 작업은 여전히 ​​`message_thread_id`를 포함합니다

    주제 상속: 주제 항목은 재정의되지 않는 한 그룹 설정을 상속합니다 (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`).
    `agentId`는 주제 전용 필드이며 그룹 기본값에서 상속되지 않습니다.

    **주제별 에이전트 라우팅**

    각 주제는 `agentId`를 통해 다른 에이전트로 라우팅할 수 있으며, 이 경우 주제마다 고유한 워크스페이스, 메모리, 세션을 갖습니다.

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "1": { agentId: "main" },
            "3": { agentId: "zu" },
            "5": { agentId: "coder" },
          },
        },
      },
    },
  },
}
```

    예를 들어 주제 `3`의 세션 키는 `agent:zu:telegram:group:-1001234567890:topic:3`가 됩니다.

    **영속 ACP 주제 바인딩**

    포럼 주제는 최상위 typed ACP 바인딩으로 ACP harness 세션에 고정할 수 있습니다.

    - `bindings[]`에 `type: "acp"` 사용
    - `match.channel: "telegram"` 사용

```json5
{
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
    },
  ],
}
```

    현재 이 기능은 그룹/슈퍼그룹의 포럼 주제 범위에 한정됩니다.

    **채팅에서 스레드 바인딩 ACP 생성**

    - `/acp spawn <agent> --thread here|auto`로 현재 Telegram 주제를 새 ACP 세션에 바인딩할 수 있습니다.
    - 이후 주제 메시지는 `/acp steer` 없이 해당 ACP 세션으로 직접 라우팅됩니다.
    - 바인딩 성공 후 OpenClaw가 확인 메시지를 해당 주제 안에 pin 합니다.
    - `channels.telegram.threadBindings.spawnAcpSessions=true`가 필요합니다.

    템플릿 컨텍스트 포함:

    - `MessageThreadId`
    - `IsForum`

    DM 스레드 동작:

    - `message_thread_id`를 가진 프라이빗 채팅은 DM 라우팅을 유지하지만 스레드 인식 세션 키/응답 타깃을 사용합니다.

  </Accordion>

  <Accordion title="오디오, 비디오 및 스티커">
    ### 오디오 메시지

    Telegram은 음성 노트와 오디오 파일을 구분합니다.

    - 기본값: 오디오 파일 동작
    - `[[audio_as_voice]]` 태그를 에이전트 응답에 추가하여 음성 노트 전송을 강제

    메시지 액션 예:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ### 비디오 메시지

    Telegram은 비디오 파일과 비디오 노트를 구분합니다.

    메시지 액션 예:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    비디오 노트는 캡션을 지원하지 않으며, 제공된 메시지 텍스트는 별도로 전송됩니다.

    ### 스티커

    수신 스티커 처리 방법:

    - 정적 WEBP: 다운로드 및 처리됨 (플레이스홀더 `<media:sticker>`)
    - 애니메이티드 TGS: 건너뜀
    - 비디오 WEBM: 건너뜀

    스티커 컨텍스트 필드:

    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`

    스티커 캐시 파일:

    - `~/.openclaw/telegram/sticker-cache.json`

    스티커는 한 번 설명되고 (가능한 경우) 캐시되어 반복적인 비전 호출을 줄입니다.

    스티커 액션 활성화:

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    스티커 전송 액션:

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    캐시된 스티커 검색:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "고양이 휘적휘적",
  limit: 5,
}
```

  </Accordion>

  <Accordion title="반응 알림">
    Telegram 반응은 메시지 페이로드와 별도로 `message_reaction` 업데이트로 도착합니다.

    활성화 시, OpenClaw는 시스템 이벤트를 큐에 추가합니다:

    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`

    구성:

    - `channels.telegram.reactionNotifications`: `off | own | all` (기본: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (기본: `minimal`)

    노트:

    - `own`은 봇이 전송한 메시지에 대한 사용자 반응만 가리킵니다 (최대한의 노력으로; 전송된 메시지 캐시를 사용).
    - Telegram은 반응 업데이트에 스레드 ID를 제공하지 않음
      - 포럼이 아닌 그룹은 그룹 채팅 세션으로 라우팅됨
      - 포럼 그룹은 그룹 일반 주제 세션 (`:topic:1`)으로 라우팅되며, 정확한 시작 주제가 아님

    `allowed_updates`는 자동으로 polling/webhook에 `message_reaction`를 포함합니다.

  </Accordion>

  <Accordion title="Ack 반응">
    `ackReaction`은 OpenClaw가 수신 메시지를 처리하는 동안 이모지를 보내 인지합니다.

    해석 순서:

    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - 에이전트 아이덴티티 이모지 대체 (`agents.list[].identity.emoji` , 그렇지 않으면 "👀")

    노트:

    - Telegram은 유니코드 이모지를 기대합니다 (예: "👀").
    - 특정 채널이나 계정에 대해 이 반응을 비활성화하려면 `""`을 사용하세요.

  </Accordion>

  <Accordion title="Telegram 이벤트 및 명령어로 구성 쓰기">
    채널 구성 쓰기는 기본적으로 활성화되어 있습니다 (`configWrites !== false`).

    Telegram으로 인해 발생한 쓰기에는 다음이 포함됩니다:

    - 그룹 마이그레이션 이벤트 (`migrate_to_chat_id`)로 `channels.telegram.groups` 업데이트
    - `/config set` 및 `/config unset` (명령어 활성화 필요)

    비활성화:

```json5
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

  </Accordion>

  <Accordion title="롱 폴링 대 웹훅">
    기본값: 롱 폴링.

    웹훅 모드:

    - `channels.telegram.webhookUrl` 설정
    - `channels.telegram.webhookSecret` 설정 (웹훅 URL 설정 시 필수)
    - 선택적 `channels.telegram.webhookPath` (기본 `/telegram-webhook`)
    - 선택적 `channels.telegram.webhookHost` (기본 `127.0.0.1`)

    웹훅 모드에 대한 기본 로컬 리스너는 `127.0.0.1:8787`에 바인딩됩니다.

    공개 엔드포인트가 다른 경우, 프락시를 앞에 두고 `webhookUrl`을 공개 URL에 지정하십시오.
    외부 유입을 의도적으로 필요로 하는 경우 `webhookHost`를 (예: `0.0.0.0`) 설정하세요.

  </Accordion>

  <Accordion title="제한사항, 재시도 및 CLI 대상">
    - `channels.telegram.textChunkLimit` 기본은 4000입니다.
    - `channels.telegram.chunkMode="newline"`은 길이 분할 전에 단락 경계를 (빈 줄) 선호합니다.
    - `channels.telegram.mediaMaxMb` (기본값 100)는 수신/발신 Telegram 미디어 크기를 제한합니다.
    - `channels.telegram.timeoutSeconds`는 Telegram API 클라이언트 타임아웃을 재정의합니다 (설정되지 않으면 grammY 기본값이 적용됩니다).
    - 그룹 컨텍스트 히스토리는 `channels.telegram.historyLimit` 또는 `messages.groupChat.historyLimit`을 사용합니다 (기본값 50); `0`은 비활성화.
    - DM 히스토리 컨트롤:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - 아웃바운드 Telegram API 재시도는 `channels.telegram.retry`로 구성할 수 있습니다.

    CLI 전송 타깃은 숫자형 채팅 ID 또는 사용자 이름일 수 있습니다:

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

    Telegram poll은 `openclaw message poll`로 전송하며 포럼 주제도 지원합니다.

```bash
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300 --poll-public
```

    Telegram 전용 poll 플래그:

    - `--poll-duration-seconds` (5-600)
    - `--poll-anonymous`
    - `--poll-public`
    - 포럼 주제용 `--thread-id` (또는 `:topic:` 타깃 사용)

    액션 게이팅:

    - `channels.telegram.actions.sendMessage=false`는 poll을 포함한 발신 Telegram 메시지를 비활성화합니다.
    - `channels.telegram.actions.poll=false`는 일반 전송은 유지하면서 poll 생성만 막습니다.

  </Accordion>
</AccordionGroup>

## 문제 해결

<AccordionGroup>
  <Accordion title="봇이 그룹에서 언급되지 않은 메시지에 응답하지 않음">

    - `requireMention=false`인 경우, Telegram 프라이버시 모드는 전체 가시성을 허용해야 합니다.
      - BotFather: `/setprivacy` -> 비활성화
      - 그런 다음 그룹에서 봇 제거 + 재추가
    - `openclaw channels status`는 구성에서 언급되지 않은 그룹 메시지를 기대할 때 경고를 표시합니다.
    - `openclaw channels status --probe`는 명시적 숫자 그룹 ID를 확인할 수 있습니다; 와일드카드 `"*"`은 멤버십을 검사할 수 없습니다.
    - 빠른 세션 테스트: `/activation always`.

  </Accordion>

  <Accordion title="봇이 그룹 메시지를 전혀 보지 못함">

    - `channels.telegram.groups`가 있는 경우, 그룹이 나열되어 있어야 함 (또는 `"*"` 포함)
    - 그룹에서 봇의 멤버십 확인
    - 스킵 이유를 확인하려면 로그 검토: `openclaw logs --follow`

  </Accordion>

  <Accordion title="명령어가 부분적으로 작동하거나 전혀 작동하지 않음">

    - 발신자 ID 권한 부여 (페어링 및/또는 숫자 `allowFrom`)
    - 명령어 권한 부여는 그룹 정책이 `open`일 때도 계속 적용됩니다
    - `setMyCommands failed`는 보통 `api.telegram.org`에 대한 DNS/HTTPS 도달 가능성 문제를 나타냅니다

  </Accordion>

  <Accordion title="폴링 또는 네트워크 불안정">

    - Node 22+ + 사용자 정의 fetch/프록시가 임의의 중단 동작을 트리거하여 AbortSignal 유형 불일치가 발생할 수 있습니다.
    - 일부 호스트는 `api.telegram.org`를 먼저 IPv6로 해석합니다; IPv6 유출이 잘못되면 Telegram API 오류 간헐적으로 발생할 수 있습니다.
    - 로그에 `TypeError: fetch failed` 또는 `Network request for 'getUpdates' failed!` 가 포함된 경우, OpenClaw는 이제 이를 복구 가능한 네트워크 오류로 재시도합니다.
    - 불안정한 직접 이그레스/TLS가 있는 VPS 호스트에서 `channels.telegram.proxy` 를 통해 Telegram API 호출을 라우팅하세요:

```yaml
channels:
  telegram:
    proxy: socks5://user:pass@proxy-host:1080
```

    - Node 22+ 는 기본적으로 `autoSelectFamily=true` 입니다 (WSL2 제외) 그리고 `dnsResultOrder=ipv4first`.
    - 호스트가 WSL2이거나 IPv4 전용 동작이 더 잘 작동하는 경우, 패밀리 선택을 강제하세요:

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

    - 환경 변수 오버라이드 (임시):
      - `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
      - `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
      - `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
    - DNS 응답을 검증:

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  </Accordion>
</AccordionGroup>

자세한 도움말: [채널 문제 해결](/ko-KR/channels/troubleshooting).

## Telegram 구성 참조 포인터

주요 참조:

- `channels.telegram.enabled`: 채널 시작을 활성화/비활성화.
- `channels.telegram.botToken`: 봇 토큰 (BotFather).
- `channels.telegram.tokenFile`: 파일 경로에서 토큰 읽기.
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (기본값: pairing).
- `channels.telegram.allowFrom`: DM 허용 목록 (숫자 Telegram 사용자 ID). `open`은 `"*"`이 필요합니다. `openclaw doctor --fix`는 레거시 `@username` 항목을 ID로 해결할 수 있습니다.
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (기본값: allowlist).
- `channels.telegram.groupAllowFrom`: 그룹 발신자 허용 목록 (숫자 Telegram 사용자 ID). `openclaw doctor --fix`는 레거시 `@username` 항목을 ID로 해결할 수 있습니다.
- `channels.telegram.groups`: 그룹별 기본값 + 허용 목록 (전역 기본값은 `"*"` 사용).
  - `channels.telegram.groups.<id>.groupPolicy`: 그룹별 groupPolicy 오버라이드 (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: 언급 게이팅 기본값.
  - `channels.telegram.groups.<id>.skills`: 스킬 필터 (생략 = 모든 스킬, 빈 값 = 없음).
  - `channels.telegram.groups.<id>.allowFrom`: 그룹별 발신자 허용 목록 오버라이드.
  - `channels.telegram.groups.<id>.systemPrompt`: 그룹에 대한 추가 시스템 프롬프트.
  - `channels.telegram.groups.<id>.enabled`: 그룹을 비활성화할 때 `false`.
  - `channels.telegram.actions.poll`: Telegram poll 생성 허용/비활성화 (기본값: 활성화, 단 `sendMessage`도 필요).
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: 주제별 오버라이드 (그룹 필드 + 주제 전용 `agentId`).
  - `channels.telegram.groups.<id>.topics.<threadId>.agentId`: 이 주제를 특정 에이전트로 라우팅합니다.
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: 주제별 groupPolicy 오버라이드 (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: 주제별 언급 게이팅 오버라이드.
  - 최상위 `bindings[]`에서 `type: "acp"`와 canonical topic id(`chatId:topic:topicId`)를 `match.peer.id`에 쓰면 영속 ACP 주제 바인딩을 구성할 수 있습니다.
- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (기본값: allowlist).
- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: 계정별 오버라이드.
- `channels.telegram.replyToMode`: `off | first | all` (기본값: `off`).
- `channels.telegram.textChunkLimit`: 아웃바운드 청크 크기 (문자 수).
- `channels.telegram.chunkMode`: `length` (기본값) 또는 `newline`으로 빈 줄 (단락 경계)로 분할하려면.
- `channels.telegram.linkPreview`: 아웃바운드 메시지 링크 미리보기 토글 (기본: true).
- `channels.telegram.streamMode`: `off | partial | block` (라이브 스트림 미리보기). Telegram 미리보기 스트리밍은 하나의 미리보기 메시지를 in-place로 편집합니다.
- `channels.telegram.mediaMaxMb`: 인바운드/아웃바운드 Telegram 미디어 한도 (MB, 기본값 `100`).
- `channels.telegram.retry`: 아웃바운드 Telegram API 호출에 대한 재시도 정책 (시도 횟수, minDelayMs, maxDelayMs, 지터).
- `channels.telegram.network.autoSelectFamily`: Node autoSelectFamily 재정의 (true=활성화, false=비활성화). Node 22+에서 기본적으로 활성화되며, WSL2는 기본적으로 비활성화됩니다.
- `channels.telegram.network.dnsResultOrder`: DNS 결과 순서 재정의 (`ipv4first` 또는 `verbatim`). Node 22+에서 기본값은 `ipv4first`.
- `channels.telegram.proxy`: Bot API 호출에 대한 프록시 URL (SOCKS/HTTP).
- `channels.telegram.webhookUrl`: 웹훅 모드 활성화 (requires `channels.telegram.webhookSecret`).
- `channels.telegram.webhookSecret`: 웹훅 비밀 (webhookUrl이 설정된 경우 필수).
- `channels.telegram.webhookPath`: 로컬 웹훅 경로 (기본 `/telegram-webhook`).
- `channels.telegram.webhookHost`: 로컬 웹훅 바인드 호스트 (기본 `127.0.0.1`).
- `channels.telegram.actions.reactions`: Telegram 도구 반응 게이트.
- `channels.telegram.actions.sendMessage`: Telegram 도구 메시지 전송 게이트.
- `channels.telegram.actions.deleteMessage`: Telegram 도구 메시지 삭제 게이트.
- `channels.telegram.actions.sticker`: Telegram 스티커 액션 게이트 — 전송 및 검색 (기본: false).
- `channels.telegram.reactionNotifications`: `off | own | all` — 어떤 반응이 시스템 이벤트를 트리거하는지 제어 (기본: `own` 사용하지 않을 때).
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — 에이전트의 반응 기능 제어 (기본: `minimal` 사용하지 않을 때).

- [구성 참조 - Telegram](/ko-KR/gateway/configuration-reference#telegram)

Telegram 전용 특징적 필드:

- 시작/인증: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- 액세스 제어: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`, 최상위 `bindings[]` (`type: "acp"`)
- 명령/메뉴: `commands.native`, `customCommands`
- 스레딩/응답: `replyToMode`
- 스트리밍: `streamMode` (미리보기), `draftChunk`, `blockStreaming`
- 형식/배달: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- 미디어/네트워크: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `network.dnsResultOrder`, `proxy`
- 웹훅: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- 액션/기능: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- 반응: `reactionNotifications`, `reactionLevel`
- 쓰기/히스토리: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## 관련 사항

- [페어링](/ko-KR/channels/pairing)
- [채널 라우팅](/ko-KR/channels/channel-routing)
- [멀티 에이전트 라우팅](/ko-KR/concepts/multi-agent)
- [문제 해결](/ko-KR/channels/troubleshooting)
