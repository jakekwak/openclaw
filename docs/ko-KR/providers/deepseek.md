---
summary: "DeepSeek 설정 (인증 + 모델 선택)"
read_when:
  - OpenClaw에서 DeepSeek를 사용하려 할 때
  - API 키 환경 변수나 CLI 인증 선택지가 필요할 때
---

# DeepSeek

[DeepSeek](https://www.deepseek.com)는 OpenAI 호환 API로 강력한 AI 모델을 제공합니다.

- Provider: `deepseek`
- Auth: `DEEPSEEK_API_KEY`
- API: OpenAI-compatible

## 빠른 시작

API 키를 설정합니다 (권장: Gateway에 저장):

```bash
openclaw onboard --auth-choice deepseek-api-key
```

이 과정에서 API 키를 입력받고 기본 모델을 `deepseek/deepseek-chat`으로 설정합니다.

## 비대화형 예시

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice deepseek-api-key \
  --deepseek-api-key "$DEEPSEEK_API_KEY" \
  --skip-health \
  --accept-risk
```

## 환경 변수 참고

Gateway가 daemon(launchd/systemd)으로 실행된다면, `DEEPSEEK_API_KEY`가 해당 프로세스에서 보이도록 해야 합니다. 예를 들어 `~/.openclaw/.env`나 `env.shellEnv`를 사용할 수 있습니다.

## 사용 가능한 모델

| Model ID            | Name                     | Type      | Context |
| ------------------- | ------------------------ | --------- | ------- |
| `deepseek-chat`     | DeepSeek Chat (V3.2)     | General   | 128K    |
| `deepseek-reasoner` | DeepSeek Reasoner (V3.2) | Reasoning | 128K    |

- **deepseek-chat**은 비사고 모드의 DeepSeek-V3.2에 해당합니다.
- **deepseek-reasoner**는 chain-of-thought 추론이 포함된 사고 모드의 DeepSeek-V3.2에 해당합니다.

API 키는 [platform.deepseek.com](https://platform.deepseek.com/api_keys)에서 발급받을 수 있습니다.
