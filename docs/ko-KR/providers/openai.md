---
summary: "OpenClaw에서 API 키 또는 Codex 구독으로 OpenAI를 사용하는 방법"
read_when:
  - OpenClaw에서 OpenAI 모델을 쓰고 싶을 때
  - API 키 대신 Codex 구독 인증을 사용하고 싶을 때
title: "OpenAI"
---

# OpenAI

OpenAI는 GPT 모델용 개발자 API를 제공합니다. Codex는 구독형 접근을 위한 **ChatGPT 로그인**과 사용량 기반 접근을 위한 **API 키 로그인**을 모두 지원합니다. Codex cloud는 ChatGPT 로그인이 필요합니다. OpenAI는 OpenClaw 같은 외부 도구/워크플로에서의 구독 OAuth 사용을 명시적으로 지원합니다.

## Option A: OpenAI API key (OpenAI Platform)

직접 API 접근과 사용량 기반 청구에 적합합니다.

### CLI 설정

```bash
openclaw onboard --auth-choice openai-api-key
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### 구성 예시

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.4" } } },
}
```

OpenAI의 현재 API 모델 문서는 직접 OpenAI API 사용 모델로 `gpt-5.4`와 `gpt-5.4-pro`를 안내합니다. OpenClaw는 둘 다 `openai/*` Responses 경로로 전달합니다.

## Option B: OpenAI Code (Codex) subscription

API 키 대신 ChatGPT/Codex 구독 접근을 사용할 때 적합합니다.

### CLI 설정 (Codex OAuth)

```bash
openclaw onboard --auth-choice openai-codex
openclaw models auth login --provider openai-codex
```

### 구성 예시

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.4" } } },
}
```

OpenAI의 현재 Codex 문서는 `gpt-5.4`를 현재 Codex 모델로 안내합니다. OpenClaw에서는 이를 ChatGPT/Codex OAuth용 `openai-codex/gpt-5.4`로 매핑합니다.

### 전송 기본값

OpenClaw는 모델 스트리밍에 `pi-ai`를 사용합니다. `openai/*`와 `openai-codex/*` 모두 기본 전송은 `"auto"`입니다. 즉, WebSocket을 먼저 시도하고 실패하면 SSE로 폴백합니다.

설정 위치:

- `agents.defaults.models.<provider/model>.params.transport`

지원 값:

- `"sse"`
- `"websocket"`
- `"auto"`

예:

```json5
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-5.4" },
      models: {
        "openai-codex/gpt-5.4": {
          params: {
            transport: "auto",
          },
        },
      },
    },
  },
}
```

### OpenAI WebSocket warm-up

OpenAI 문서는 warm-up을 선택 사항으로 설명하지만, OpenClaw는 WebSocket 사용 시 첫 턴 지연을 줄이기 위해 `openai/*`에 기본적으로 `openaiWsWarmup: true`를 켭니다.

비활성화:

```json5
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: false,
          },
        },
      },
    },
  },
}
```

명시적 활성화:

```json5
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: true,
          },
        },
      },
    },
  },
}
```

### OpenAI priority processing

OpenAI API는 `service_tier=priority`를 통한 우선 처리 기능을 제공합니다. OpenClaw에서는 직접 `openai/*` Responses 요청에 이 값을 전달하려면 `agents.defaults.models["openai/<model>"].params.serviceTier`를 설정합니다.

```json5
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            serviceTier: "priority",
          },
        },
      },
    },
  },
}
```

지원 값:

- `auto`
- `default`
- `flex`
- `priority`

### OpenAI Responses 서버 측 compaction

직접 OpenAI Responses 모델(`openai/*`, `api: "openai-responses"`, `api.openai.com` baseUrl)에서는 OpenClaw가 OpenAI 서버 측 compaction 힌트를 자동 활성화합니다.

- `store: true` 강제 (`supportsStore: false`인 호환성 모델 제외)
- `context_management: [{ type: "compaction", compact_threshold: ... }]` 주입

기본 `compact_threshold`는 모델 `contextWindow`의 70%이며, 알 수 없으면 `80000`입니다.

명시적 활성화 예:

```json5
{
  agents: {
    defaults: {
      models: {
        "azure-openai-responses/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
          },
        },
      },
    },
  },
}
```

사용자 지정 threshold:

```json5
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
            responsesCompactThreshold: 120000,
          },
        },
      },
    },
  },
}
```

비활성화:

```json5
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: false,
          },
        },
      },
    },
  },
}
```

## Notes

- 모델 참조는 항상 `provider/model` 형식을 사용합니다. 자세한 내용은 [/ko-KR/concepts/models](/ko-KR/concepts/models)를 참조하세요.
- 인증 세부사항과 재사용 규칙은 [/ko-KR/concepts/oauth](/ko-KR/concepts/oauth)에 있습니다.
