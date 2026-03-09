# Kilo Gateway 프로바이더 통합 설계

## 개요

이 문서는 기존 OpenRouter 구현을 기준으로 `Kilo Gateway`를 OpenClaw의 1급 프로바이더로 통합하는 설계를 설명합니다. Kilo Gateway는 OpenAI 호환 completions API를 사용하지만 기본 URL이 다릅니다.

## 설계 결정

### 1. 프로바이더 이름

권장안: `kilocode`

이유:

- 사용자가 제공한 구성 예시(`kilocode`)와 일치합니다.
- `openrouter`, `opencode`, `moonshot` 같은 기존 네이밍 패턴과 맞습니다.
- 짧고 기억하기 쉽습니다.

### 2. 기본 모델 참조

권장안: `kilocode/anthropic/claude-opus-4.6`

### 3. Base URL 구성

- 기본 URL: `https://api.kilo.ai/api/gateway/`
- `models.providers.kilocode.baseUrl`로 재정의 가능

### 4. 모델 스캔

초기에는 전용 모델 스캔 엔드포인트를 두지 않는 방향을 권장합니다.

- Kilo Gateway는 OpenRouter를 프록시하므로 모델 목록이 동적입니다.
- 사용자가 직접 모델을 구성할 수 있습니다.
- 추후 `/models` 엔드포인트가 생기면 스캔을 추가하면 됩니다.

### 5. 특수 처리

Anthropic 모델에 대해서는 OpenRouter와 같은 동작을 상속하는 쪽을 권장합니다.

- `anthropic/*` 모델의 캐시 TTL 적격성
- `anthropic/*`용 추가 파라미터(`cacheControlTtl`)
- transcript 정책은 OpenRouter 패턴을 따름

## 수정 대상 파일

- `src/commands/onboard-auth.credentials.ts`
- `src/agents/model-auth.ts`
- `src/config/io.ts`
- `src/commands/onboard-auth.config-core.ts`
- `src/commands/onboard-types.ts`
- `src/commands/auth-choice-options.ts`

핵심 변경 내용:

- `KILOCODE_API_KEY` 환경 변수 추가
- `kilocode-api-key` 온보딩 선택지 추가
- `applyKilocodeProviderConfig()` / `applyKilocodeConfig()` 추가
- 기본 모델을 `kilocode/anthropic/claude-opus-4.6`로 설정

이 설계는 Kilo Gateway를 별도 특수 케이스로 만들기보다, 기존 OpenRouter/OpenAI 호환 프로바이더 패턴에 맞춰 최소 변경으로 통합하는 것을 목표로 합니다.
