---
title: "Google (Gemini)"
summary: "Google Gemini 설정 (API 키 + OAuth, 이미지 생성, 미디어 이해, 웹 검색)"
read_when:
  - OpenClaw에서 Google Gemini 모델을 사용하려 할 때
  - API 키 또는 OAuth 인증 흐름이 필요할 때
---

# Google (Gemini)

Google 플러그인은 Google AI Studio를 통해 Gemini 모델에 접근하고, 이미지 생성, 미디어 이해(이미지/오디오/비디오), Gemini Grounding 기반 웹 검색도 제공합니다.

- Provider: `google`
- Auth: `GEMINI_API_KEY` 또는 `GOOGLE_API_KEY`
- API: Google Gemini API
- 대체 provider: `google-gemini-cli` (OAuth)

## 빠른 시작

1. API 키를 설정합니다:

```bash
openclaw onboard --auth-choice google-api-key
```

2. 기본 모델을 설정합니다:

```json5
{
  agents: {
    defaults: {
      model: { primary: "google/gemini-3.1-pro-preview" },
    },
  },
}
```

## 비대화형 예시

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice google-api-key \
  --gemini-api-key "$GEMINI_API_KEY"
```

## OAuth (Gemini CLI)

대체 provider인 `google-gemini-cli`는 API 키 대신 PKCE OAuth를 사용합니다. 비공식 통합이며, 일부 사용자는 계정 제한을 보고했습니다. 사용 시에는 위험을 감수해야 합니다.

환경 변수:

- `OPENCLAW_GEMINI_OAUTH_CLIENT_ID`
- `OPENCLAW_GEMINI_OAUTH_CLIENT_SECRET`

또는 `GEMINI_CLI_*` 변형 이름을 사용할 수 있습니다.

## 기능

| Capability             | Supported         |
| ---------------------- | ----------------- |
| Chat completions       | Yes               |
| Image generation       | Yes               |
| Image understanding    | Yes               |
| Audio transcription    | Yes               |
| Video understanding    | Yes               |
| Web search (Grounding) | Yes               |
| Thinking/reasoning     | Yes (Gemini 3.1+) |

## 환경 변수 참고

Gateway가 daemon(launchd/systemd)으로 실행된다면, `GEMINI_API_KEY`가 해당 프로세스에서 보이도록 해야 합니다. 예를 들어 `~/.openclaw/.env`나 `env.shellEnv`를 사용할 수 있습니다.
