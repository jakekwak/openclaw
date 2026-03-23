---
title: "Building Provider Plugins"
sidebarTitle: "Provider Plugins"
summary: "OpenClaw용 model provider plugin을 만드는 단계별 가이드"
read_when:
  - 새로운 model provider plugin을 만들 때
  - OpenAI-compatible proxy나 custom LLM을 OpenClaw에 추가할 때
  - provider auth, catalog, runtime hook을 이해해야 할 때
---

# Building Provider Plugins

이 가이드는 model provider(LLM)를 OpenClaw에 추가하는 provider plugin 제작 흐름을 설명합니다. 끝나면 model catalog, API key auth, dynamic model resolution을 갖춘 provider를 만들 수 있습니다.

## 기본 흐름

1. `package.json`과 `openclaw.plugin.json`으로 package/manifest 정의
2. `api.registerProvider(...)`로 provider 등록
3. `catalog`를 제공해 기본 모델 집합 노출
4. 필요하면 `resolveDynamicModel(...)` 추가
5. 필요하면 runtime hook을 점진적으로 추가
6. 테스트 작성

## 최소 provider 요소

provider에 최소한 필요한 항목:

- `id`
- `label`
- `docsPath`
- `envVars`
- `auth`
- `catalog`

`providerAuthEnvVars`와 `providerAuthChoices`를 manifest에 넣으면 OpenClaw가 plugin runtime을 로드하지 않고도 credential을 감지할 수 있습니다.

## Dynamic model resolution

router/proxy처럼 임의의 모델 ID를 받는 provider라면 `resolveDynamicModel`을 구현합니다. 네트워크 준비가 필요하면 `prepareDynamicModel`을 추가할 수 있습니다.

## Runtime hook

대부분의 provider는 `catalog`와 `resolveDynamicModel`만으로 충분합니다. 필요 시 다음 hook을 추가합니다:

- token exchange
- request normalization
- usage/accounting
- provider 전용 extra capability(speech/media/image/web search)

## 권장 helper

- `createProviderApiKeyAuthMethod(...)`
- `defineSingleProviderPluginEntry(...)`
- `provider-onboard` 계열 preset helper

## 테스트 포인트

- API 키가 있을 때 catalog가 반환되는지
- dynamic model이 올바른 provider/api/baseUrl을 갖는지
- provider-specific auth/normalization이 예상대로 동작하는지

## 다음 단계

- [Building Plugins](/ko-KR/plugins/building-plugins)
- [SDK Overview](/ko-KR/plugins/sdk-overview)
- [SDK Runtime](/ko-KR/plugins/sdk-runtime)
- [SDK Testing](/ko-KR/plugins/sdk-testing)
