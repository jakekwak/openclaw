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
OpenClaw는 오래된 `openai/gpt-5.3-codex-spark` 행을 의도적으로 숨깁니다. 현재 직접 OpenAI API 호출에서는 이 모델이 실제 트래픽에서 거부되기 때문입니다.

OpenClaw는 직접 OpenAI API 경로에서 `openai/gpt-5.3-codex-spark`를 노출하지 않습니다. `pi-ai`는 이 모델의 내장 행을 계속 가지고 있지만, 현재 live OpenAI API 요청은 이를 거부합니다. OpenClaw에서는 Spark를 Codex 전용으로 취급합니다.

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

Codex 계정에 Codex Spark entitlement가 있다면 OpenClaw는 다음도 지원합니다:

- `openai-codex/gpt-5.3-codex-spark`

OpenClaw는 Codex Spark를 Codex 전용으로 취급합니다. 직접 `openai/gpt-5.3-codex-spark` API 키 경로는 노출하지 않습니다.

또한 `pi-ai`가 `openai-codex/gpt-5.3-codex-spark`를 발견한 경우 OpenClaw는 이를 유지합니다. entitlement 의존적이며 실험적인 모델로 보세요. Codex Spark는 GPT-5.4의 `/fast`와 별개이며, 사용 가능 여부는 로그인한 Codex / ChatGPT 계정에 달려 있습니다.

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

### OpenAI fast mode

OpenClaw는 `openai/*`와 `openai-codex/*` 세션 모두에 공통 fast-mode 토글을 제공합니다:

- 채팅/UI: `/fast status|on|off`
- config: `agents.defaults.models["<provider>/<model>"].params.fastMode`

fast mode가 켜지면 OpenClaw는 낮은 지연시간용 OpenAI 프로필을 적용합니다:

- payload에 reasoning이 이미 없으면 `reasoning.effort = "low"`
- payload에 verbosity가 이미 없으면 `text.verbosity = "low"`
- 직접 `api.openai.com`으로 가는 `openai/*` Responses 호출에는 `service_tier = "priority"`

예:

```json5
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            fastMode: true,
          },
        },
        "openai-codex/gpt-5.4": {
          params: {
            fastMode: true,
          },
        },
      },
    },
  },
}
```

세션 override가 config보다 우선합니다. Sessions UI에서 세션 override를 지우면 다시 구성된 기본값으로 돌아갑니다.

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
