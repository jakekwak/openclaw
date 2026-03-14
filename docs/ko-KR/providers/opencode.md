---
summary: "OpenClaw와 함께 OpenCode Zen 및 Go 카탈로그 사용하기"
read_when:
  - OpenCode 호스팅 모델 액세스를 원할 때
  - Zen과 Go 카탈로그 중에서 선택하고 싶을 때
title: "OpenCode"
---

# OpenCode

OpenCode는 OpenClaw에서 두 가지 호스팅 카탈로그를 제공합니다:

- **Zen** 카탈로그용 `opencode/...`
- **Go** 카탈로그용 `opencode-go/...`

두 카탈로그는 같은 OpenCode API 키를 사용합니다. OpenClaw는 업스트림 모델별 라우팅 정확성을 위해 런타임 provider id는 분리해 두지만, 온보딩과 문서에서는 하나의 OpenCode 설정으로 다룹니다.

## CLI 설정

### Zen 카탈로그

```bash
openclaw onboard --auth-choice opencode-zen
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

### Go 카탈로그

```bash
openclaw onboard --auth-choice opencode-go
openclaw onboard --opencode-go-api-key "$OPENCODE_API_KEY"
```

## 설정 코드 스니펫

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## 카탈로그

### Zen

- 런타임 provider: `opencode`
- 예시 모델: `opencode/claude-opus-4-6`, `opencode/gpt-5.2`, `opencode/gemini-3-pro`
- OpenCode의 추천 multi-model proxy를 원할 때 적합

### Go

- 런타임 provider: `opencode-go`
- 예시 모델: `opencode-go/kimi-k2.5`, `opencode-go/glm-5`, `opencode-go/minimax-m2.5`
- OpenCode 호스팅 Kimi/GLM/MiniMax 라인업을 원할 때 적합

## 참고 사항

- `OPENCODE_ZEN_API_KEY` 도 지원됩니다.
- 온보딩에서 OpenCode 키 하나를 입력하면 두 런타임 provider 모두에 대한 자격 증명이 저장됩니다.
- OpenCode에 로그인하고 결제 정보를 추가한 뒤 API 키를 복사합니다.
- 과금과 카탈로그 제공 여부는 OpenCode 대시보드에서 관리됩니다.
