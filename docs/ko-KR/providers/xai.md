---
summary: "OpenClaw에서 xAI Grok 모델 사용하기"
read_when:
  - OpenClaw에서 Grok 모델을 사용하려 할 때
  - xAI 인증 또는 모델 ID를 설정할 때
title: "xAI"
---

# xAI

OpenClaw는 Grok 모델용 번들 `xai` provider 플러그인을 포함합니다.

## 설정

1. xAI 콘솔에서 API 키를 만듭니다.
2. `XAI_API_KEY`를 설정하거나 다음을 실행합니다:

```bash
openclaw onboard --auth-choice xai-api-key
```

3. 다음과 같은 모델을 선택합니다:

```json5
{
  agents: { defaults: { model: { primary: "xai/grok-4" } } },
}
```

## 현재 번들 모델 카탈로그

OpenClaw에는 현재 다음 xAI 모델 계열이 기본 포함됩니다:

- `grok-4`, `grok-4-0709`
- `grok-4-fast-reasoning`, `grok-4-fast-non-reasoning`
- `grok-4-1-fast-reasoning`, `grok-4-1-fast-non-reasoning`
- `grok-4.20-reasoning`, `grok-4.20-non-reasoning`
- `grok-code-fast-1`

플러그인은 같은 API 형태를 따르는 더 새로운 `grok-4*` 및 `grok-code-fast*` ID도 forward resolve합니다.

## 웹 검색

번들 `grok` 웹 검색 provider도 동일하게 `XAI_API_KEY`를 사용합니다:

```bash
openclaw config set tools.web.search.provider grok
```

## 알려진 제한

- 현재 인증은 API 키만 지원합니다. OpenClaw에는 xAI OAuth/device-code 흐름이 아직 없습니다.
- `grok-4.20-multi-agent-experimental-beta-0304`는 일반 xAI provider 경로에서 지원되지 않습니다. 표준 OpenClaw xAI 전송과 다른 upstream API를 요구하기 때문입니다.
- `x_search`, `code_execution` 같은 xAI 네이티브 server-side tool은 번들 플러그인에서 아직 1급 모델 provider 기능이 아닙니다.

## 참고

- OpenClaw는 공용 runner 경로에서 xAI 전용 tool-schema 및 tool-call 호환성 수정을 자동 적용합니다.
- provider 전체 개요는 [Model providers](/ko-KR/providers/index)를 참조하세요.
