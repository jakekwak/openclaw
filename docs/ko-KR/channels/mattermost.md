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
- `always`: 채널의 모든 메시지에 응답 (채널 동작은 `onmessage`와 동일).
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
- 현재 제한사항: Mattermost 플러그인 이벤트 동작(`#11797`)으로 인해 `chatmode: "onmessage"`와
  `chatmode: "always"`는 명시적인 그룹 언급 재정의를 통해 @언급 없이 반응할 수 있습니다.
  사용:

```json5
{
  channels: {
    mattermost: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

참조: [버그: Mattermost 플러그인이 WebSocket을 통해 채널 메시지 이벤트를 수신하지 않음 #11797](https://github.com/open-webui/open-webui/issues/11797).
관련 수정 범위: [수정(mattermost): 그룹 언급 게이트에서 chatmode 언급 대체 기능 존중 #14995](https://github.com/open-webui/open-webui/pull/14995).

## 액세스 제어 (다이렉트 메시지)

- 기본값: `channels.mattermost.dmPolicy = "pairing"` (알 수 없는 발신자가 페어링 코드를 받음).
- 승인 방법:
  - `openclaw pairing list mattermost`
  - `openclaw pairing approve mattermost <CODE>`
- 공개 다이렉트 메시지: `channels.mattermost.dmPolicy="open"` 및 `channels.mattermost.allowFrom=["*"]`.

## 채널 (그룹)

- 기본값: `channels.mattermost.groupPolicy = "allowlist"` (언급 제한).
- `channels.mattermost.groupAllowFrom`(사용자 ID 또는 `@username`)을 통해 발신자를 허용 목록에 추가합니다.
- 공개 채널: `channels.mattermost.groupPolicy="open"` (언급 제한).

## 아웃바운드 전송용 대상

`openclaw message send` 또는 cron/webhooks와 함께 다음 대상 형식 사용:

- `channel:<id>` - 채널
- `user:<id>` - 다이렉트 메시지
- `@username` - 다이렉트 메시지 (Mattermost API를 통해 해석)

단순 ID는 채널로 간주됩니다.

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

- 채널에서 응답 없음: 봇이 채널에 있는지 확인하고 모드 동작을 올바르게 사용합니다: 언급 (`oncall`), 트리거 접두어 사용 (`onchar`), 또는 `onmessage`/`always` 사용:
  `channels.mattermost.groups["*"].requireMention = false` (일반적으로 `groupPolicy: "open"` 사용).
- 인증 오류: 봇 토큰, 기본 URL 및 계정 활성화 상태를 확인하십시오.
- 멀티 계정 문제: 환경 변수는 `default` 계정에만 적용됩니다.