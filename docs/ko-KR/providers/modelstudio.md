---
title: "Model Studio"
summary: "Alibaba Cloud Model Studio 설정 (Coding Plan, 이중 리전 엔드포인트)"
read_when:
  - OpenClaw에서 Alibaba Cloud Model Studio를 사용하려 할 때
  - Model Studio용 API 키 환경 변수가 필요할 때
---

# Model Studio (Alibaba Cloud)

Model Studio provider는 Alibaba Cloud의 Coding Plan 모델과, 플랫폼에 호스팅된 Qwen 및 서드파티 모델에 접근할 수 있게 해 줍니다.

- Provider: `modelstudio`
- Auth: `MODELSTUDIO_API_KEY`
- API: OpenAI-compatible

## 빠른 시작

1. API 키를 설정합니다:

```bash
openclaw onboard --auth-choice modelstudio-api-key
```

2. 기본 모델을 설정합니다:

```json5
{
  agents: {
    defaults: {
      model: { primary: "modelstudio/qwen3.5-plus" },
    },
  },
}
```

## 리전 엔드포인트

Model Studio는 리전에 따라 두 개의 엔드포인트를 사용합니다:

| Region     | Endpoint                             |
| ---------- | ------------------------------------ |
| China (CN) | `coding.dashscope.aliyuncs.com`      |
| Global     | `coding-intl.dashscope.aliyuncs.com` |

provider는 인증 선택지에 따라 자동으로 엔드포인트를 고릅니다 (`modelstudio-api-key`는 global, `modelstudio-api-key-cn`은 China). 필요하면 config의 `baseUrl`로 덮어쓸 수 있습니다.

## 사용 가능한 모델

- **qwen3.5-plus** (기본값) - Qwen 3.5 Plus
- **qwen3-max** - Qwen 3 Max
- **qwen3-coder** 시리즈 - Qwen 코딩 모델
- **GLM-5**, **GLM-4.7** - Alibaba를 통한 GLM 모델
- **Kimi K2.5** - Alibaba를 통한 Moonshot AI
- **MiniMax-M2.5** - Alibaba를 통한 MiniMax

대부분의 모델은 이미지 입력을 지원합니다. 컨텍스트 윈도는 200K에서 1M 토큰까지 다양합니다.

## 환경 변수 참고

Gateway가 daemon(launchd/systemd)으로 실행된다면, `MODELSTUDIO_API_KEY`가 해당 프로세스에서 보이도록 해야 합니다. 예를 들어 `~/.openclaw/.env`나 `env.shellEnv`를 사용할 수 있습니다.
