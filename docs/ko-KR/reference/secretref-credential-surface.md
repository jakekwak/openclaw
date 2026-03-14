---
summary: "정식 지원 SecretRef 자격 증명 표면과 비지원 표면"
read_when:
  - SecretRef 자격 증명 범위를 확인할 때
  - 어떤 자격 증명이 `secrets configure` 또는 `secrets apply` 대상인지 감사할 때
  - 특정 자격 증명이 지원 표면 밖인 이유를 확인할 때
title: "SecretRef 자격 증명 표면"
---

# SecretRef 자격 증명 표면

이 문서는 정식 SecretRef 자격 증명 표면을 정의합니다.

범위 의도:

- 지원 범위: OpenClaw가 직접 발급하거나 회전하지 않는, 사용자가 제공한 자격 증명
- 지원 제외: 런타임이 발급하는 자격 증명, 회전형 자격 증명, OAuth refresh material, 세션성 산출물

## 지원되는 자격 증명

### `openclaw.json` 대상 (`secrets configure` + `secrets apply` + `secrets audit`)

- `models.providers.*.apiKey`
- `models.providers.*.headers.*`
- `skills.entries.*.apiKey`
- `agents.defaults.memorySearch.remote.apiKey`
- `agents.list[].memorySearch.remote.apiKey`
- `talk.apiKey`
- `talk.providers.*.apiKey`
- `messages.tts.elevenlabs.apiKey`
- `messages.tts.openai.apiKey`
- `tools.web.search.apiKey`
- `tools.web.search.gemini.apiKey`
- `tools.web.search.grok.apiKey`
- `tools.web.search.kimi.apiKey`
- `tools.web.search.perplexity.apiKey`
- `gateway.auth.password`
- `gateway.auth.token`
- `gateway.remote.token`
- `gateway.remote.password`
- `cron.webhookToken`
- `channels.telegram.botToken`
- `channels.telegram.webhookSecret`
- `channels.telegram.accounts.*.botToken`
- `channels.telegram.accounts.*.webhookSecret`
- `channels.slack.botToken`
- `channels.slack.appToken`
- `channels.slack.userToken`
- `channels.slack.signingSecret`
- `channels.slack.accounts.*.botToken`
- `channels.slack.accounts.*.appToken`
- `channels.slack.accounts.*.userToken`
- `channels.slack.accounts.*.signingSecret`
- `channels.discord.token`
- `channels.discord.pluralkit.token`
- `channels.discord.voice.tts.elevenlabs.apiKey`
- `channels.discord.voice.tts.openai.apiKey`
- `channels.discord.accounts.*.token`
- `channels.discord.accounts.*.pluralkit.token`
- `channels.discord.accounts.*.voice.tts.elevenlabs.apiKey`
- `channels.discord.accounts.*.voice.tts.openai.apiKey`
- `channels.irc.password`
- `channels.irc.nickserv.password`
- `channels.irc.accounts.*.password`
- `channels.irc.accounts.*.nickserv.password`
- `channels.bluebubbles.password`
- `channels.bluebubbles.accounts.*.password`
- `channels.feishu.appSecret`
- `channels.feishu.verificationToken`
- `channels.feishu.accounts.*.appSecret`
- `channels.feishu.accounts.*.verificationToken`
- `channels.msteams.appPassword`
- `channels.mattermost.botToken`
- `channels.mattermost.accounts.*.botToken`
- `channels.matrix.password`
- `channels.matrix.accounts.*.password`
- `channels.nextcloud-talk.botSecret`
- `channels.nextcloud-talk.apiPassword`
- `channels.nextcloud-talk.accounts.*.botSecret`
- `channels.nextcloud-talk.accounts.*.apiPassword`
- `channels.zalo.botToken`
- `channels.zalo.webhookSecret`
- `channels.zalo.accounts.*.botToken`
- `channels.zalo.accounts.*.webhookSecret`
- `channels.googlechat.serviceAccount` via sibling `serviceAccountRef` (호환 예외)
- `channels.googlechat.accounts.*.serviceAccount` via sibling `serviceAccountRef` (호환 예외)

### `auth-profiles.json` 대상 (`secrets configure` + `secrets apply` + `secrets audit`)

- `profiles.*.keyRef` (`type: "api_key"`)
- `profiles.*.tokenRef` (`type: "token"`)

주의:

- auth-profile plan 대상에는 `agentId`가 필요합니다.
- plan 항목은 `profiles.*.key` / `profiles.*.token`을 대상으로 하고, 실제로는 sibling ref (`keyRef` / `tokenRef`)를 씁니다.
- auth-profile ref는 런타임 해석과 audit 범위에 포함됩니다.
- SecretRef-managed 모델 프로바이더의 경우, 생성된 `agents/*/agent/models.json` 항목은 `apiKey`/header 표면에 대해 해석된 비밀이 아닌 비비밀 marker를 저장합니다.
- 웹 검색의 경우:
  - 명시적 provider mode(`tools.web.search.provider` 설정)에서는 선택된 provider key만 활성입니다.
  - auto mode(`tools.web.search.provider` 미설정)에서는 `tools.web.search.apiKey`와 provider별 key가 모두 활성입니다.

## 지원되지 않는 자격 증명

- `commands.ownerDisplaySecret`
- `channels.matrix.accessToken`
- `channels.matrix.accounts.*.accessToken`
- `hooks.token`
- `hooks.gmail.pushToken`
- `hooks.mappings[].sessionKey`
- `auth-profiles.oauth.*`
- `discord.threadBindings.*.webhookToken`
- `whatsapp.creds.json`

근거:

- 이 자격 증명들은 발급형, 회전형, 세션성, 또는 OAuth 지속성 자격 증명에 속하므로 읽기 전용 외부 SecretRef 해석 모델에 맞지 않습니다.

### 최근 업데이트 메모

- `tools.web.fetch.firecrawl.apiKey`, `channels.feishu.encryptKey`, `channels.feishu.accounts.*.encryptKey`가 SecretRef 지원 표면에 포함됩니다.
- SecretRef-managed provider marker는 해석된 런타임 비밀값이 아니라 source config snapshot을 기준으로 저장됩니다.
- `tools.web.search.provider`가 unset인 auto 모드에서는 precedence상 처음 성공적으로 해석된 provider key만 활성입니다.
