---
title: "Groq"
summary: "Groq 설정 (인증 + 모델 선택)"
read_when:
  - OpenClaw에서 Groq를 사용하려 할 때
  - API 키 환경 변수나 CLI 인증 선택지가 필요할 때
---

# Groq

[Groq](https://groq.com)는 자체 LPU 하드웨어를 사용해 오픈소스 모델(Llama, Gemma, Mistral 등)에 대한 초고속 추론을 제공합니다. OpenClaw는 OpenAI 호환 API를 통해 Groq에 연결합니다.

- Provider: `groq`
- Auth: `GROQ_API_KEY`
- API: OpenAI-compatible

## 빠른 시작

1. [console.groq.com/keys](https://console.groq.com/keys)에서 API 키를 발급받습니다.

2. API 키를 설정합니다:

```bash
export GROQ_API_KEY="gsk_..."
```

3. 기본 모델을 설정합니다:

```json5
{
  agents: {
    defaults: {
      model: { primary: "groq/llama-3.3-70b-versatile" },
    },
  },
}
```

## 설정 파일 예시

```json5
{
  env: { GROQ_API_KEY: "gsk_..." },
  agents: {
    defaults: {
      model: { primary: "groq/llama-3.3-70b-versatile" },
    },
  },
}
```

## 오디오 전사

Groq는 빠른 Whisper 기반 오디오 전사도 제공합니다. 미디어 이해 provider로 설정하면 OpenClaw는 Groq의 `whisper-large-v3-turbo` 모델로 음성 메시지를 전사합니다.

```json5
{
  media: {
    understanding: {
      audio: {
        models: [{ provider: "groq" }],
      },
    },
  },
}
```

## 환경 변수 참고

Gateway가 daemon(launchd/systemd)으로 실행된다면, `GROQ_API_KEY`가 해당 프로세스에서 보이도록 해야 합니다. 예를 들어 `~/.openclaw/.env`나 `env.shellEnv`를 사용할 수 있습니다.

## 사용 가능한 모델

Groq의 모델 카탈로그는 자주 바뀝니다. 현재 모델은 `openclaw models list | grep groq`로 확인하거나 [console.groq.com/docs/models](https://console.groq.com/docs/models)를 참조하세요.

대표적인 선택지는 다음과 같습니다:

- **Llama 3.3 70B Versatile** - 범용, 대규모 컨텍스트
- **Llama 3.1 8B Instant** - 빠르고 가벼움
- **Gemma 2 9B** - 작고 효율적
- **Mixtral 8x7B** - MoE 아키텍처, 강한 추론 성능

## 링크

- [Groq Console](https://console.groq.com)
- [API Documentation](https://console.groq.com/docs)
- [Model List](https://console.groq.com/docs/models)
- [Pricing](https://groq.com/pricing)
