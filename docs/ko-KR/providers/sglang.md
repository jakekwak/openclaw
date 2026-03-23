---
summary: "SGLang(OpenAI-compatible self-hosted server)으로 OpenClaw 실행하기"
read_when:
  - 로컬 SGLang 서버에 OpenClaw를 연결하고 싶을 때
  - 자체 모델로 OpenAI-compatible `/v1` 엔드포인트를 사용하고 싶을 때
title: "SGLang"
---

# SGLang

SGLang은 **OpenAI-compatible** HTTP API를 통해 오픈소스 모델을 제공할 수 있습니다.
OpenClaw는 `openai-completions` API를 사용해 SGLang에 연결할 수 있습니다.

또한 `SGLANG_API_KEY`로 opt-in 하고 `models.providers.sglang` 항목을
명시적으로 정의하지 않으면, OpenClaw가 SGLang의 사용 가능한 모델을
**자동 탐색**할 수 있습니다. 서버가 인증을 강제하지 않는다면 어떤 값이든 작동합니다.

## 빠른 시작

1. OpenAI-compatible 서버로 SGLang을 시작합니다.

기본 URL은 `/v1` 엔드포인트를 노출해야 합니다 (예: `/v1/models`,
`/v1/chat/completions`). SGLang은 보통 다음 주소에서 실행됩니다:

- `http://127.0.0.1:30000/v1`

2. opt-in 합니다 (인증이 없다면 아무 값이나 가능):

```bash
export SGLANG_API_KEY="sglang-local"
```

3. 온보딩을 실행하고 `SGLang`을 고르거나, 모델을 직접 설정합니다:

```bash
openclaw onboard
```

```json5
{
  agents: {
    defaults: {
      model: { primary: "sglang/your-model-id" },
    },
  },
}
```

## 모델 탐색 (암시적 provider)

`SGLANG_API_KEY`가 설정되어 있거나(auth profile 존재 포함),
`models.providers.sglang`을 **정의하지 않은 경우**, OpenClaw는 다음을 조회합니다:

- `GET http://127.0.0.1:30000/v1/models`

그 결과로 반환된 ID를 모델 항목으로 변환합니다.

`models.providers.sglang`을 명시적으로 설정하면 자동 탐색은 건너뛰고,
모델을 수동으로 정의해야 합니다.

## 명시적 설정 (수동 모델 정의)

다음 경우에는 명시적 설정을 사용하세요:

- SGLang이 다른 호스트/포트에서 실행될 때
- `contextWindow`/`maxTokens` 값을 고정하고 싶을 때
- 서버가 실제 API 키를 요구할 때 (또는 헤더를 직접 제어하고 싶을 때)

```json5
{
  models: {
    providers: {
      sglang: {
        baseUrl: "http://127.0.0.1:30000/v1",
        apiKey: "${SGLANG_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "Local SGLang Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 문제 해결

- 서버에 연결되는지 확인하세요:

```bash
curl http://127.0.0.1:30000/v1/models
```

- 인증 오류로 요청이 실패하면, 서버 설정에 맞는 실제 `SGLANG_API_KEY`를 설정하거나
  `models.providers.sglang` 아래에 provider를 명시적으로 구성하세요.
