---
summary: "OpenClaw에서 Venice AI의 개인 정보 중심 모델을 사용하는 방법"
read_when:
  - 개인 정보 중심 추론이 필요할 때
  - Venice AI 설정 안내가 필요할 때
title: "Venice AI"
---

# Venice AI

Venice AI는 개인 정보 중심 추론을 제공하며, 검열 없는 모델과 익명화 프록시를 통한 주요 독점 모델 접근을 지원합니다.

## 왜 Venice를 쓰나

- 로깅 없는 **private** 추론
- 필요 시 **검열 없는 모델**
- Claude, GPT, Gemini, Grok에 대한 **익명화된 접근**
- OpenAI 호환 `/v1` 엔드포인트

## 개인 정보 모드

| 모드           | 설명                                                                   | 모델                                                       |
| -------------- | ---------------------------------------------------------------------- | ---------------------------------------------------------- |
| **Private**    | 프롬프트/응답이 저장되거나 로깅되지 않습니다. 일시적으로만 유지됩니다. | Llama, Qwen, DeepSeek, Kimi, MiniMax, Venice Uncensored 등 |
| **Anonymized** | Venice 프록시가 메타데이터를 제거한 뒤 기반 프로바이더에 전달합니다.   | Claude, GPT, Gemini, Grok                                  |

## 기능

- private / anonymized 두 모드
- 검열 없는 모델
- OpenAI 호환 API
- 스트리밍 지원
- 일부 모델의 함수 호출 지원
- 비전 지원 모델 제공

## 설정

### 1. API 키 얻기

1. [venice.ai](https://venice.ai)에서 계정 생성
2. **Settings -> API Keys -> Create new key**
3. `vapi_...` 형식 키 복사

### 2. OpenClaw 구성

환경 변수:

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

대화형 설정:

```bash
openclaw onboard --auth-choice venice-api-key
```

비대화형:

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 3. 설정 확인

```bash
openclaw agent --model venice/kimi-k2-5 --message "Hello, are you working?"
```

## 모델 선택

- 기본 추천: `venice/kimi-k2-5`
- 가장 강한 익명화 옵션: `venice/claude-opus-4-6`
- 완전 private이 필요하면 private 모델 선택
- Claude/GPT/Gemini가 필요하면 anonymized 모델 선택

기본 모델 변경:

```bash
openclaw models set venice/kimi-k2-5
openclaw models set venice/claude-opus-4-6
```

전체 모델 목록:

```bash
openclaw models list | grep venice
```

## 어떤 모델을 쓰면 좋나

| 사용 사례           | 추천 모델                        | 이유                              |
| ------------------- | -------------------------------- | --------------------------------- |
| 일반 대화 기본값    | `kimi-k2-5`                      | private 모드에서 강한 추론 + 비전 |
| 최고의 전반적 품질  | `claude-opus-4-6`                | 가장 강한 anonymized 경로         |
| 개인 정보 + 코딩    | `qwen3-coder-480b-a35b-instruct` | 큰 컨텍스트의 private 코딩 모델   |
| private 비전        | `kimi-k2-5`                      | 비전을 지원하는 private 기본값    |
| 빠르고 저렴함       | `qwen3-4b`                       | 가벼운 추론 모델                  |
| 복잡한 private 작업 | `deepseek-v3.2`                  | 강한 추론                         |
| 검열 없음           | `venice-uncensored`              | 제한 없는 출력                    |

## 사용 가능한 모델 개요

### Private 모델 예시

- `kimi-k2-5`
- `kimi-k2-thinking`
- `llama-3.3-70b`
- `qwen3-coder-480b-a35b-instruct`
- `qwen3-coder-480b-a35b-instruct-turbo`
- `qwen3-5-35b-a3b`
- `qwen3-vl-235b-a22b`
- `deepseek-v3.2`
- `venice-uncensored`
- `minimax-m21`
- `minimax-m25`
- `zai-org-glm-5`

### Anonymized 모델 예시

- `claude-opus-4-6`
- `claude-opus-4-5`
- `claude-sonnet-4-6`
- `claude-sonnet-4-5`
- `openai-gpt-54`
- `openai-gpt-53-codex`
- `openai-gpt-52`
- `gemini-3-1-pro-preview`
- `gemini-3-flash-preview`
- `grok-41-fast`
- `grok-code-fast-1`

현재 카탈로그는 총 41개 모델이며, 일부는 동적으로 바뀔 수 있습니다.

## 모델 검색

`VENICE_API_KEY`가 설정되면 OpenClaw는 Venice API에서 모델을 자동 검색합니다. API 접근이 안 되면 정적 카탈로그로 폴백합니다.

## 사용 예시

```bash
openclaw agent --model venice/kimi-k2-5 --message "Quick health check"
openclaw agent --model venice/claude-opus-4-6 --message "Summarize this task"
openclaw agent --model venice/venice-uncensored --message "Draft options"
openclaw agent --model venice/qwen3-vl-235b-a22b --message "Describe this image"
openclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message "Review this diff"
```

## 문제 해결

### API 키가 인식되지 않음

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

키는 `vapi_`로 시작해야 합니다.

### 모델이 보이지 않음

`openclaw models list`를 다시 실행하세요. Venice 카탈로그는 동적으로 바뀔 수 있으며 일부 모델은 일시적으로 빠질 수 있습니다.

### 연결 문제

Venice API는 `https://api.venice.ai/api/v1`를 사용합니다. 네트워크에서 HTTPS outbound가 가능한지 확인하세요.

## 구성 예시

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/kimi-k2-5" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2-5",
            name: "Kimi K2.5",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```
