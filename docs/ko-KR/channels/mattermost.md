---
summary: "Mattermost 봇 설정 및 OpenClaw 구성"
read_when:
  - Mattermost 설정하기
  - Mattermost 라우팅 디버깅
title: "Mattermost"
---

# Mattermost (plugin)

상태: 플러그인으로 지원 (봇 토큰 + WebSocket 이벤트). 채널, 그룹, 다이렉트 메시지가 지원됩니다.
Mattermost는 자체 호스팅 가능한 팀 메시징 플랫폼입니다. 제품 정보 및 다운로드는
[공식 사이트](https://mattermost.com)를 참조하세요.

## 플러그인 필요

Mattermost는 플러그인으로 제공되며 기본 설치에 포함되지 않습니다.

CLI 사용하여 설치 (npm registry):

```bash
openclaw plugins install @openclaw/mattermost
```

git 저장소에서 실행 중일 때 로컬 체크아웃:

```bash
openclaw plugins install ./extensions/mattermost
```

Mattermost를 설정/온보딩하는 동안 선택하고 git 체크아웃이 감지되면,
OpenClaw가 자동으로 로컬 설치 경로를 제공합니다.

자세한 내용: [플러그인](/ko-KR/tools/plugin)

## 빠른 설정

1. Mattermost 플러그인을 설치합니다.
2. Mattermost 봇 계정을 생성하고 **봇 토큰**을 복사합니다.
3. Mattermost **기본 URL**(예: `https://chat.example.com`)을 복사합니다.
4. OpenClaw를 구성하고 게이트웨이를 시작합니다.

최소 구성:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## 환경 변수 (기본 계정)

환경 변수를 선호하는 경우, 게이트웨이 호스트에 설정:

- `MATTERMOST_BOT_TOKEN=...`
- `MATTERMOST_URL=https://chat.example.com`

환경 변수는 **기본** 계정(`default`)에만 적용됩니다. 다른 계정은 설정 값을 사용해야 합니다.

## 채팅 모드

Mattermost는 다이렉트 메시지에 자동으로 응답합니다. 채널 동작은 `chatmode`로 제어됩니다:

- `oncall` (기본값): 채널에서 @언급될 때만 응답합니다.
- `onmessage`: 채널 메시지마다 응답합니다.
- `onchar`: 메시지가 트리거 접두어로 시작할 때 응답합니다.

설정 예시:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

참고:

- `onchar`도 명시적인 @언급에 응답합니다.
- `channels.mattermost.requireMention`은 기존 설정에 대해 적용되지만 `chatmode`가 선호됩니다.

## 액세스 제어 (다이렉트 메시지)

- 기본값: `channels.mattermost.dmPolicy = "pairing"` (알 수 없는 발신자가 페어링 코드를 받음).
- 승인 방법:
  - `openclaw pairing list mattermost`
  - `openclaw pairing approve mattermost <CODE>`
- 공개 다이렉트 메시지: `channels.mattermost.dmPolicy="open"` 및 `channels.mattermost.allowFrom=["*"]`.

## 채널 (그룹)

- 기본값: `channels.mattermost.groupPolicy = "allowlist"` (언급 제한).
- `channels.mattermost.groupAllowFrom` (사용자 ID 추천)을 통해 발신자를 허용 목록에 추가합니다.
- `@username` 매칭은 변경 가능하며 `channels.mattermost.dangerouslyAllowNameMatching: true`일 때만 활성화됩니다.
- 공개 채널: `channels.mattermost.groupPolicy="open"` (언급 제한).
- 런타임 노트: `channels.mattermost`가 완전히 없는 경우, `channels.defaults.groupPolicy`가 설정되어 있더라도 런타임은 그룹 확인에 `groupPolicy="allowlist"`로 폴백합니다.

## 아웃바운드 전송용 대상

`openclaw message send` 또는 cron/webhooks와 함께 다음 대상 형식 사용:

- `channel:<id>` - 채널
- `user:<id>` - 다이렉트 메시지
- `@username` - 다이렉트 메시지 (Mattermost API를 통해 해석)

단순 ID는 채널로 간주됩니다.

## 반응 (메시지 도구)

- `message action=react`를 `channel=mattermost`와 함께 사용합니다.
- `messageId`는 Mattermost post id입니다.
- `emoji`는 `thumbsup` 또는 `:+1:`과 같은 이름을 허용합니다 (콜론은 선택 사항).
- 반응을 제거하려면 `remove=true` (boolean)를 설정합니다.
- 반응 추가/제거 이벤트는 라우팅된 에이전트 세션에 시스템 이벤트로 전달됩니다.

예시:

```
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

설정:

- `channels.mattermost.actions.reactions`: 반응 동작 활성화/비활성화 (기본값 true).
- 계정별 재정의: `channels.mattermost.accounts.<id>.actions.reactions`.

## 인터랙티브 버튼 (메시지 도구)

클릭 가능한 버튼이 포함된 메시지를 보낼 수 있습니다. 사용자가 버튼을 누르면 에이전트가 해당 선택을 받아 응답할 수 있습니다.

채널 기능에 `inlineButtons`를 추가해 버튼을 활성화합니다.

```json5
{
  channels: {
    mattermost: {
      capabilities: ["inlineButtons"],
    },
  },
}
```

`message action=send`와 함께 `buttons` 파라미터를 사용합니다. 버튼은 2차원 배열(버튼 행의 배열)입니다.

```
message action=send channel=mattermost target=channel:<channelId> buttons=[[{"text":"Yes","callback_data":"yes"},{"text":"No","callback_data":"no"}]]
```

버튼 필드:

- `text` (필수): 표시 라벨
- `callback_data` (필수): 클릭 시 되돌아오는 값 (action ID로 사용)
- `style` (선택): `"default"`, `"primary"`, `"danger"`

사용자가 버튼을 누르면:

1. 모든 버튼이 확인 문구(예: "✓ **Yes** selected by @user")로 교체됩니다.
2. 에이전트가 해당 선택을 인바운드 메시지로 받고 응답합니다.

참고:

- 버튼 콜백은 HMAC-SHA256 검증을 사용합니다 (자동 처리되며 별도 설정 불필요).
- Mattermost는 API 응답에서 callback data를 제거하므로(보안 기능), 클릭 시 모든 버튼이 제거됩니다. 일부만 제거하는 것은 불가능합니다.
- 하이픈이나 언더스코어가 들어간 action ID는 자동으로 정리됩니다(Mattermost 라우팅 제한).

설정:

- `channels.mattermost.capabilities`: capability 문자열 배열. 에이전트 시스템 프롬프트에 버튼 도구 설명을 포함하려면 `"inlineButtons"`를 추가합니다.
- `channels.mattermost.interactions.callbackBaseUrl`: 버튼 콜백용 외부 base URL(예: `https://gateway.example.com`). Mattermost가 bind host로 직접 게이트웨이에 닿지 못할 때 사용합니다.
- 멀티 계정에서는 같은 필드를 `channels.mattermost.accounts.<id>.interactions.callbackBaseUrl` 아래에도 설정할 수 있습니다.
- `interactions.callbackBaseUrl`이 없으면 OpenClaw는 `gateway.customBindHost` + `gateway.port`에서 콜백 URL을 유도하고, 실패하면 `http://localhost:<port>`로 폴백합니다.
- 도달 가능성 규칙: 버튼 콜백 URL은 Mattermost 서버에서 접근 가능해야 합니다. `localhost`는 Mattermost와 OpenClaw가 같은 호스트/네트워크 네임스페이스에 있을 때만 동작합니다.
- 콜백 대상이 private/tailnet/internal 주소라면 Mattermost의 `ServiceSettings.AllowedUntrustedInternalConnections`에 해당 host/domain을 추가하십시오.

### Direct API integration (외부 스크립트)

외부 스크립트와 웹훅은 에이전트의 `message` 도구를 거치지 않고도 Mattermost REST API를 통해 버튼을 직접 게시할 수 있습니다. 가능하면 확장에서 `buildButtonAttachments()`를 사용하고, raw JSON으로 게시할 때는 다음 규칙을 따르십시오.

**페이로드 구조:**

```json5
{
  channel_id: "<channelId>",
  message: "Choose an option:",
  props: {
    attachments: [
      {
        actions: [
          {
            id: "mybutton01",
            type: "button",
            name: "Approve",
            style: "primary",
            integration: {
              url: "https://gateway.example.com/mattermost/interactions/default",
              context: {
                action_id: "mybutton01",
                action: "approve",
                _token: "<hmac>",
              },
            },
          },
        ],
      },
    ],
  },
}
```

**중요 규칙:**

1. attachment는 최상위 `attachments`가 아니라 `props.attachments`에 들어가야 합니다. 그렇지 않으면 조용히 무시됩니다.
2. 모든 action에는 `type: "button"`이 필요합니다. 없으면 클릭이 조용히 무시됩니다.
3. 모든 action에는 `id` 필드가 필요합니다. Mattermost는 ID가 없는 action을 무시합니다.
4. action `id`는 반드시 **영숫자만** (`[a-zA-Z0-9]`) 포함해야 합니다. 하이픈과 언더스코어는 Mattermost 서버 측 action routing을 깨뜨려 404를 유발합니다.
5. 확인 메시지가 raw ID가 아니라 버튼 이름을 표시하려면 `context.action_id`가 버튼 `id`와 같아야 합니다.
6. `context.action_id`는 필수입니다. 없으면 interaction handler가 400을 반환합니다.

**HMAC 토큰 생성:**

게이트웨이는 HMAC-SHA256으로 버튼 클릭을 검증합니다. 외부 스크립트는 게이트웨이 검증 로직과 동일한 토큰을 생성해야 합니다.

1. 봇 토큰으로부터 secret을 파생:
   `HMAC-SHA256(key="openclaw-mattermost-interactions", data=botToken)`
2. `_token`을 제외한 모든 필드로 context 객체를 만듭니다.
3. **정렬된 키**와 **공백 없는 형태**로 직렬화합니다. 게이트웨이는 정렬된 키의 compact `JSON.stringify` 결과를 사용합니다.
4. 서명:
   `HMAC-SHA256(key=secret, data=serializedContext)`
5. 결과 hex digest를 context의 `_token`으로 넣습니다.

Python 예시:

```python
import hmac, hashlib, json

secret = hmac.new(
    b"openclaw-mattermost-interactions",
    bot_token.encode(), hashlib.sha256
).hexdigest()

ctx = {"action_id": "mybutton01", "action": "approve"}
payload = json.dumps(ctx, sort_keys=True, separators=(",", ":"))
token = hmac.new(secret.encode(), payload.encode(), hashlib.sha256).hexdigest()

context = {**ctx, "_token": token}
```

자주 하는 HMAC 실수:

- Python의 `json.dumps`는 기본적으로 공백을 넣습니다 (`{"key": "val"}`). JavaScript의 compact 출력(`{"key":"val"}`)과 맞추려면 `separators=(",", ":")`를 사용하십시오.
- 반드시 `_token`을 제외한 **모든** context 필드를 서명해야 합니다. 일부만 서명하면 조용히 검증 실패합니다.
- `sort_keys=True`를 사용하십시오. 게이트웨이는 서명 전에 키를 정렬하고, Mattermost도 payload 저장 시 context 필드 순서를 바꿀 수 있습니다.
- secret은 임의 바이트가 아니라 봇 토큰에서 결정적으로 파생해야 합니다. 버튼을 만드는 프로세스와 검증하는 게이트웨이에서 같은 secret이 나와야 합니다.

## 디렉터리 어댑터

Mattermost 플러그인에는 Mattermost API를 통해 채널명과 사용자명을 해석하는 디렉터리 어댑터가 포함되어 있습니다. 이를 통해 `openclaw message send`와 cron/webhook 전송에서 `#channel-name`, `@username` 대상을 사용할 수 있습니다.

별도 설정은 필요 없습니다. 어댑터는 계정 설정의 봇 토큰을 사용합니다.

## 멀티 계정

Mattermost는 `channels.mattermost.accounts`에서 여러 계정을 지원합니다:

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## 문제 해결

- 채널에서 응답 없음: 봇이 채널에 있는지 확인하고 언급 (`oncall`), 트리거 접두어 사용 (`onchar`), 또는 `chatmode: "onmessage"` 설정을 확인하십시오.
- 인증 오류: 봇 토큰, 기본 URL 및 계정 활성화 상태를 확인하십시오.
- 멀티 계정 문제: 환경 변수는 `default` 계정에만 적용됩니다.
- 버튼이 흰 박스로 보임: 에이전트가 잘못된 버튼 데이터를 보내는 것일 수 있습니다. 각 버튼에 `text`와 `callback_data`가 모두 있는지 확인하십시오.
- 버튼은 렌더링되지만 클릭이 동작하지 않음: Mattermost 서버 설정의 `AllowedUntrustedInternalConnections`에 `127.0.0.1 localhost`가 포함되어 있는지, 그리고 `ServiceSettings.EnablePostActionIntegration`이 `true`인지 확인하십시오.
- 버튼 클릭 시 404 반환: 버튼 `id`에 하이픈 또는 언더스코어가 들어 있을 가능성이 큽니다. `[a-zA-Z0-9]`만 사용하십시오.
- 게이트웨이 로그에 `invalid _token`: HMAC 불일치입니다. 모든 context 필드를 서명했는지, 키를 정렬했는지, compact JSON(공백 없음)을 썼는지 확인하십시오.
- 게이트웨이 로그에 `missing _token in context`: 버튼의 context에 `_token` 필드가 없습니다. integration payload를 만들 때 포함했는지 확인하십시오.
- 확인 메시지에 버튼 이름 대신 raw ID가 표시됨: `context.action_id`가 버튼 `id`와 일치하지 않습니다. 둘 다 같은 정리된 값으로 설정하십시오.
- 에이전트가 버튼을 모름: Mattermost 채널 설정에 `capabilities: ["inlineButtons"]`를 추가하십시오.
