---
summary: "공유 OpenCode 설정으로 OpenCode Go 카탈로그 사용하기"
read_when:
  - OpenCode Go 카탈로그를 사용하고 싶을 때
  - Go 호스팅 모델용 런타임 모델 ref가 필요할 때
title: "OpenCode Go"
---

# OpenCode Go

OpenCode Go는 [OpenCode](/ko-KR/providers/opencode) 안의 Go 카탈로그입니다.
Zen 카탈로그와 같은 `OPENCODE_API_KEY`를 사용하지만, 업스트림 모델별
라우팅이 올바르게 유지되도록 런타임 provider id는 `opencode-go`를 유지합니다.

## 지원 모델

- `opencode-go/kimi-k2.5`
- `opencode-go/glm-5`
- `opencode-go/minimax-m2.5`

## CLI 설정

```bash
openclaw onboard --auth-choice opencode-go
# 또는 비대화형
openclaw onboard --opencode-go-api-key "$OPENCODE_API_KEY"
```

## 설정 예시

```json5
{
  env: { OPENCODE_API_KEY: "YOUR_API_KEY_HERE" }, // pragma: allowlist secret
  agents: { defaults: { model: { primary: "opencode-go/kimi-k2.5" } } },
}
```

## 라우팅 동작

모델 ref가 `opencode-go/...` 형식을 사용할 때 OpenClaw가 모델별 라우팅을 자동으로 처리합니다.

## 참고

- 공유 온보딩과 카탈로그 개요는 [OpenCode](/ko-KR/providers/opencode)를 사용하세요.
- 런타임 ref는 명시적으로 유지됩니다: Zen은 `opencode/...`, Go는 `opencode-go/...`.
