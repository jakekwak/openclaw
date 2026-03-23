---
title: "Volcengine (Doubao)"
summary: "Volcano Engine 설정 (Doubao 모델, 일반 + 코딩 엔드포인트)"
read_when:
  - Volcano Engine 또는 Doubao 모델을 OpenClaw에서 사용하려 할 때
  - Volcengine API 키 설정이 필요할 때
---

# Volcengine (Doubao)

Volcengine provider는 Volcano Engine에 호스팅된 Doubao 모델과 서드파티 모델에 접근할 수 있게 해 주며, 일반 작업과 코딩 작업에 대해 별도 엔드포인트를 사용합니다.

- Providers: `volcengine` (일반) + `volcengine-plan` (코딩)
- Auth: `VOLCANO_ENGINE_API_KEY`
- API: OpenAI-compatible

## 빠른 시작

1. API 키를 설정합니다:

```bash
openclaw onboard --auth-choice volcengine-api-key
```

2. 기본 모델을 설정합니다:

```json5
{
  agents: {
    defaults: {
      model: { primary: "volcengine-plan/ark-code-latest" },
    },
  },
}
```

## 비대화형 예시

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice volcengine-api-key \
  --volcengine-api-key "$VOLCANO_ENGINE_API_KEY"
```

## Providers와 엔드포인트

| Provider          | Endpoint                                  | Use case       |
| ----------------- | ----------------------------------------- | -------------- |
| `volcengine`      | `ark.cn-beijing.volces.com/api/v3`        | General models |
| `volcengine-plan` | `ark.cn-beijing.volces.com/api/coding/v3` | Coding models  |

두 provider 모두 단일 API 키로 구성됩니다. 설정 시 두 항목이 자동 등록됩니다.

## 사용 가능한 모델

- **doubao-seed-1-8** - Doubao Seed 1.8 (일반, 기본값)
- **doubao-seed-code-preview** - Doubao 코딩 모델
- **ark-code-latest** - Coding plan 기본값
- **Kimi K2.5** - Volcano Engine 경유 Moonshot AI
- **GLM-4.7** - Volcano Engine 경유 GLM
- **DeepSeek V3.2** - Volcano Engine 경유 DeepSeek

대부분의 모델은 텍스트 + 이미지 입력을 지원합니다. 컨텍스트 윈도는 128K에서 256K 토큰까지 다양합니다.

## 환경 변수 참고

Gateway가 daemon(launchd/systemd)으로 실행된다면, `VOLCANO_ENGINE_API_KEY`가 해당 프로세스에서 보이도록 해야 합니다. 예를 들어 `~/.openclaw/.env`나 `env.shellEnv`를 사용할 수 있습니다.
