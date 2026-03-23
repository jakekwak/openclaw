---
summary: "Plugin internals: capability model, ownership, contract, load pipeline, runtime helper"
read_when:
  - native OpenClaw plugin을 만들거나 디버그할 때
  - plugin capability model이나 ownership boundary를 이해하려 할 때
  - plugin load pipeline이나 registry를 다룰 때
title: "Plugin Internals"
sidebarTitle: "Internals"
---

# Plugin Internals

이 문서는 OpenClaw plugin 시스템의 deep architecture reference입니다.

## Public capability model

native plugin은 하나 이상의 capability type에 등록됩니다:

| Capability          | Registration method                           |
| ------------------- | --------------------------------------------- |
| Text inference      | `api.registerProvider(...)`                   |
| Speech              | `api.registerSpeechProvider(...)`             |
| Media understanding | `api.registerMediaUnderstandingProvider(...)` |
| Image generation    | `api.registerImageGenerationProvider(...)`    |
| Web search          | `api.registerWebSearchProvider(...)`          |
| Channel / messaging | `api.registerChannel(...)`                    |

hook, tool, service만 제공하고 capability를 0개 등록하는 plugin은 legacy hook-only plugin으로 계속 지원됩니다.

## Plugin shape

OpenClaw는 실제 registration behavior를 기준으로 plugin shape를 분류합니다:

- **plain-capability**
- **hybrid-capability**
- **hook-only**
- **non-capability**

`openclaw plugins inspect <id>`로 확인할 수 있습니다.

## 아키텍처 개요

plugin 시스템은 네 층으로 볼 수 있습니다:

1. **Manifest + discovery**
2. **Enablement + validation**
3. **Runtime loading**
4. **Surface consumption**

핵심 경계:

- discovery와 config validation은 plugin 코드를 실행하지 않고 manifest/schema metadata만으로 가능해야 함
- 실제 runtime behavior는 plugin module의 `register(api)` 경로에서 결정됨

## Ownership model

OpenClaw는 native plugin을 **회사** 또는 **기능** 단위의 ownership boundary로 봅니다. 따라서 채널은 vendor behavior를 직접 재구현하기보다 shared core capability를 소비해야 합니다.

## Channel plugin과 shared message tool

channel plugin은 일반 채팅 action용 send/edit/react tool을 따로 등록하지 않습니다. core가 shared `message` tool을 소유하고, channel plugin은 channel-specific discovery와 execution만 담당합니다.

## Load pipeline

간단히 말하면:

1. plugin 발견
2. manifest 검증
3. enable/disable/slot 선택
4. native plugin runtime 로드
5. registry에 capability/tools/hooks/routes 등록
6. 나머지 OpenClaw surface가 registry를 소비

## 실무 규칙

- 외부 plugin은 좁고 문서화된 SDK surface를 우선 사용
- legacy hook은 호환성 경로로 유지되지만 새 설계의 기본값은 아님
- vendor 코드를 channel/tool에 직접 넣지 말고 capability layer를 정의

## 관련 문서

- [Building Plugins](/ko-KR/plugins/building-plugins)
- [Channel Plugins](/ko-KR/plugins/sdk-channel-plugins)
- [Provider Plugins](/ko-KR/plugins/sdk-provider-plugins)
- [SDK Overview](/ko-KR/plugins/sdk-overview)
