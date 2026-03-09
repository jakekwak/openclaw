---
summary: "OpenClaw에서 Mistral 모델과 Voxtral 전사를 사용하는 방법"
read_when:
  - OpenClaw에서 Mistral 모델을 사용하고 싶을 때
  - Mistral API 키 온보딩과 모델 참조가 필요할 때
title: "Mistral"
---

# Mistral

OpenClaw는 텍스트/이미지 모델 라우팅(`mistral/...`)과 미디어 이해용 Voxtral 오디오 전사 둘 다에서 Mistral을 지원합니다.

Mistral은 메모리 임베딩(`memorySearch.provider = "mistral"`)에도 사용할 수 있습니다.

## CLI 설정

```bash
openclaw onboard --auth-choice mistral-api-key
openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
```

## 구성 예시: LLM 프로바이더

```json5
{
  env: { MISTRAL_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
}
```

## 구성 예시: Voxtral 오디오 전사

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## 참고

- Mistral 인증은 `MISTRAL_API_KEY`를 사용합니다.
- 기본 base URL은 `https://api.mistral.ai/v1`입니다.
- 온보딩 기본 모델은 `mistral/mistral-large-latest`입니다.
- 미디어 이해 기본 오디오 모델은 `voxtral-mini-latest`입니다.
- 미디어 전사 경로는 `/v1/audio/transcriptions`를 사용합니다.
- 메모리 임베딩 경로는 `/v1/embeddings`이며 기본 모델은 `mistral-embed`입니다.
