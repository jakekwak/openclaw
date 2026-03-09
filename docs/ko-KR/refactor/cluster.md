---
summary: "LOC 감소 효과가 큰 리팩터 클러스터 백로그"
read_when:
  - 동작 변경 없이 전체 LOC를 줄이고 싶을 때
  - 다음 dedupe 또는 extraction 작업 대상을 고를 때
title: "리팩터 클러스터 백로그"
---

# 리팩터 클러스터 백로그

예상 LOC 감소량, 안전성, 적용 범위를 기준으로 정렬했습니다.

## 1. 채널 플러그인 설정 및 보안 스캐폴딩

가장 가치가 큰 클러스터입니다.

여러 채널 플러그인에서 반복되는 형태:

- `config.listAccountIds`
- `config.resolveAccount`
- `config.defaultAccountId`
- `config.setAccountEnabled`
- `config.deleteAccount`
- `config.describeAccount`
- `security.resolveDmPolicy`

대표 예시:

- `extensions/telegram/src/channel.ts`
- `extensions/googlechat/src/channel.ts`
- `extensions/slack/src/channel.ts`
- `extensions/discord/src/channel.ts`
- `extensions/matrix/src/channel.ts`
- `extensions/irc/src/channel.ts`
- `extensions/signal/src/channel.ts`
- `extensions/mattermost/src/channel.ts`

가능성 높은 추출 형태:

- `buildChannelConfigAdapter(...)`
- `buildMultiAccountConfigAdapter(...)`
- `buildDmSecurityAdapter(...)`

예상 절감:

- 약 250-450 LOC

위험도:

- 중간. 채널마다 `isConfigured`, 경고, 정규화 방식이 조금씩 다릅니다.

## 2. 확장 런타임 싱글턴 보일러플레이트

매우 안전합니다.

거의 모든 확장에 같은 런타임 홀더가 있습니다.

- `let runtime: PluginRuntime | null = null`
- `setXRuntime`
- `getXRuntime`

대표 예시:

- `extensions/telegram/src/runtime.ts`
- `extensions/matrix/src/runtime.ts`
- `extensions/slack/src/runtime.ts`
- `extensions/discord/src/runtime.ts`
- `extensions/whatsapp/src/runtime.ts`
- `extensions/imessage/src/runtime.ts`
- `extensions/twitch/src/runtime.ts`

특수 케이스 변형:

- `extensions/bluebubbles/src/runtime.ts`
- `extensions/line/src/runtime.ts`
- `extensions/synology-chat/src/runtime.ts`

가능성 높은 추출 형태:

- `createPluginRuntimeStore<T>(errorMessage)`

예상 절감:

- 약 180-260 LOC

위험도:

- 낮음

## 3. 온보딩 프롬프트와 config-patch 단계

영향 범위가 큽니다.

많은 온보딩 파일에서 다음이 반복됩니다.

- account id 해석
- allowlist 항목 프롬프트
- allowFrom 병합
- DM policy 설정
- 비밀값 프롬프트
- top-level과 account-scoped config 패치

대표 예시:

- `extensions/bluebubbles/src/onboarding.ts`
- `extensions/googlechat/src/onboarding.ts`
- `extensions/msteams/src/onboarding.ts`
- `extensions/zalo/src/onboarding.ts`
- `extensions/zalouser/src/onboarding.ts`
- `extensions/nextcloud-talk/src/onboarding.ts`
- `extensions/matrix/src/onboarding.ts`
- `extensions/irc/src/onboarding.ts`

기존 helper seam:

- `src/channels/plugins/onboarding/helpers.ts`

가능성 높은 추출 형태:

- `promptAllowFromList(...)`
- `buildDmPolicyAdapter(...)`
- `applyScopedAccountPatch(...)`
- `promptSecretFields(...)`

예상 절감:

- 약 300-600 LOC

위험도:

- 중간. 과도하게 일반화하기 쉬우므로 helper는 좁고 조합 가능하게 유지해야 합니다.

## 4. 멀티 계정 config-schema 조각

확장 전반에 반복되는 스키마 조각입니다.

공통 패턴:

- `const allowFromEntry = z.union([z.string(), z.number()])`
- account schema에 다음을 추가:
  - `accounts: z.object({}).catchall(accountSchema).optional()`
  - `defaultAccount: z.string().optional()`
- 반복되는 DM/group 필드
- 반복되는 markdown/tool policy 필드

대표 예시:

- `extensions/bluebubbles/src/config-schema.ts`
- `extensions/zalo/src/config-schema.ts`
- `extensions/zalouser/src/config-schema.ts`
- `extensions/matrix/src/config-schema.ts`
- `extensions/nostr/src/config-schema.ts`

가능성 높은 추출 형태:

- `AllowFromEntrySchema`
- `buildMultiAccountChannelSchema(accountSchema)`
- `buildCommonDmGroupFields(...)`

예상 절감:

- 약 120-220 LOC

위험도:

- 낮음에서 중간. 단순한 스키마도 있고 특별한 스키마도 있습니다.

## 5. 웹훅과 모니터 라이프사이클 시작

중간 가치가 높은 좋은 클러스터입니다.

반복되는 `startAccount` / monitor 설정 패턴:

- account 해석
- webhook path 계산
- 시작 로그 출력
- monitor 시작
- abort 대기
- 정리
- status sink 업데이트

대표 예시:

- `extensions/googlechat/src/channel.ts`
- `extensions/bluebubbles/src/channel.ts`
- `extensions/zalo/src/channel.ts`
- `extensions/telegram/src/channel.ts`
- `extensions/nextcloud-talk/src/channel.ts`

기존 helper seam:

- `src/plugin-sdk/channel-lifecycle.ts`

가능성 높은 추출 형태:

- account monitor lifecycle helper
- webhook 기반 account startup helper

예상 절감:

- 약 150-300 LOC

위험도:

- 중간에서 높음. 전송 세부 구현이 빠르게 갈라집니다.

## 6. 작은 exact-clone 정리

저위험 정리 묶음입니다.

예시:

- 중복된 gateway argv 감지:
  - `src/infra/gateway-lock.ts`
  - `src/cli/daemon-cli/lifecycle.ts`
- 중복된 port 진단 렌더링:
  - `src/cli/daemon-cli/restart-health.ts`
- 중복된 session-key 구성:
  - `src/web/auto-reply/monitor/broadcast.ts`

예상 절감:

- 약 30-60 LOC

위험도:

- 낮음

## 테스트 클러스터

### LINE webhook 이벤트 fixture

대표 예시:

- `src/line/bot-handlers.test.ts`

가능성 높은 추출:

- `makeLineEvent(...)`
- `runLineEvent(...)`
- `makeLineAccount(...)`

예상 절감:

- 약 120-180 LOC

### Telegram 네이티브 명령어 auth 매트릭스

대표 예시:

- `src/telegram/bot-native-commands.group-auth.test.ts`
- `src/telegram/bot-native-commands.plugin-auth.test.ts`

가능성 높은 추출:

- forum context builder
- denied-message assertion helper
- table-driven auth case

예상 절감:

- 약 80-140 LOC

### Zalo 라이프사이클 설정

대표 예시:

- `extensions/zalo/src/monitor.lifecycle.test.ts`

가능성 높은 추출:

- 공통 monitor setup harness

예상 절감:

- 약 50-90 LOC

### Brave llm-context unsupported-option 테스트

대표 예시:

- `src/agents/tools/web-tools.enabled-defaults.test.ts`

가능성 높은 추출:

- `it.each(...)` 매트릭스

예상 절감:

- 약 30-50 LOC

## 추천 순서

1. 런타임 싱글턴 보일러플레이트
2. 작은 exact-clone 정리
3. 설정 및 보안 builder 추출
4. 테스트 helper 추출
5. 온보딩 단계 추출
6. 모니터 라이프사이클 helper 추출
