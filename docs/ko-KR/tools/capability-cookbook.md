---
summary: "OpenClaw plugin 시스템에 새로운 공용 capability를 추가하는 contributor 가이드"
read_when:
  - 새로운 core capability와 plugin registration surface를 추가할 때
  - 코드가 core, vendor plugin, feature plugin 중 어디에 속해야 하는지 결정할 때
  - 채널이나 도구용 새 runtime helper를 연결할 때
title: "Adding Capabilities (Contributor Guide)"
sidebarTitle: "Adding Capabilities"
---

# Adding Capabilities

<Info>
  이 문서는 OpenClaw core 개발자를 위한 contributor guide입니다. 외부 plugin을 만드는 중이라면 [Building Plugins](/ko-KR/plugins/building-plugins)을 보세요.
</Info>

새로운 이미지 생성, 비디오 생성, 기타 vendor-backed 기능 영역이 필요할 때는 vendor를 바로 채널/도구에 연결하지 말고 capability부터 정의해야 합니다.

규칙:

- plugin = ownership boundary
- capability = shared core contract

## 언제 capability를 만들어야 하나

다음이 모두 참일 때 새로운 capability를 만드세요:

1. 둘 이상의 vendor가 구현할 수 있음
2. channel/tool/feature plugin이 vendor를 몰라도 이를 소비해야 함
3. fallback, policy, config, delivery behavior를 core가 책임져야 함

## 표준 순서

1. typed core contract 정의
2. plugin registration 추가
3. 공용 runtime helper 추가
4. 실제 vendor plugin 하나 연결
5. feature/channel consumer를 runtime helper로 이동
6. contract test 추가
7. 운영자용 config와 ownership model 문서화

## 무엇을 어디에 두나

Core:

- request/response type
- provider registry + resolution
- fallback behavior
- config schema와 help
- runtime helper surface

Vendor plugin:

- vendor API 호출
- vendor 인증 처리
- vendor 전용 request normalization
- capability 구현 등록

Feature/channel plugin:

- `api.runtime.*` 또는 대응되는 `plugin-sdk/*-runtime` helper 호출
- vendor 구현을 직접 호출하지 않음

## 파일 체크리스트

- `src/<capability>/types.ts`
- `src/<capability>/...registry/runtime.ts`
- `src/plugins/types.ts`
- `src/plugins/registry.ts`
- `src/plugins/captured-registration.ts`
- `src/plugins/contracts/registry.ts`
- `src/plugins/runtime/types-core.ts`
- `src/plugins/runtime/index.ts`
- `src/plugin-sdk/<capability>.ts`
- `src/plugin-sdk/<capability>-runtime.ts`
- 하나 이상의 `extensions/<vendor>/...`
- config/docs/tests

## 예시: 이미지 생성

1. core가 `ImageGenerationProvider` 정의
2. core가 `registerImageGenerationProvider(...)` 노출
3. core가 `runtime.imageGeneration.generate(...)` 노출
4. `openai`, `google` plugin이 vendor-backed 구현 등록
5. 이후 vendor는 채널/도구를 수정하지 않고 같은 contract를 구현 가능

config 키는 vision-analysis 라우팅과 분리합니다:

- `agents.defaults.imageModel` = 이미지 분석
- `agents.defaults.imageGenerationModel` = 이미지 생성

## 리뷰 체크리스트

- channel/tool이 vendor 코드를 직접 import하지 않는가
- runtime helper가 공용 경로인가
- 최소 하나의 contract test가 bundled ownership를 검증하는가
- config 문서가 새 모델/config 키를 설명하는가
- plugin 문서가 ownership boundary를 설명하는가

vendor 동작을 channel/tool에 하드코딩하는 PR이라면 capability layer부터 정의하도록 되돌리는 것이 맞습니다.
