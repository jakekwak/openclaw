# Discord 채널 및 Telegram 토픽용 ACP 지속형 바인딩

상태: Draft

## 요약

다음을 장기 수명의 ACP 세션에 매핑하는 지속형 ACP 바인딩을 도입합니다.

- Discord 채널(필요 시 기존 스레드 포함)
- Telegram 그룹/슈퍼그룹 포럼 토픽 (`chatId:topic:topicId`)

바인딩 상태는 명시적 바인딩 타입을 가진 최상위 `bindings[]` 항목에 저장합니다.

목표는 고트래픽 메시징 채널에서도 ACP 사용을 예측 가능하고 지속 가능하게 만들어, `codex`, `claude-1`, `claude-myrepo` 같은 전용 채널/토픽을 만들 수 있게 하는 것입니다.

## 목표

- Discord 채널/스레드 및 Telegram 포럼 토픽의 durable ACP binding 지원
- binding source-of-truth를 config 중심으로 유지
- `/acp`, `/new`, `/reset`, `/focus`, 전달 동작을 Discord와 Telegram에서 일관되게 유지
- ad-hoc 용도의 기존 임시 바인딩 흐름 유지

## 비목표

- ACP 런타임/세션 내부 구조 전면 재설계
- 기존 임시 바인딩 제거
- 첫 단계에서 모든 채널로 확장
- 이 단계에서 Telegram DM topic variant 구현

## UX 방향

### 1) 두 가지 바인딩 타입

- **Persistent binding**: config에 저장되고 시작 시 reconcile되며, “이름 붙은 workspace” 채널/토픽용
- **Temporary binding**: runtime-only이며 idle/max-age 정책으로 만료

### 2) 명령 동작

- `/acp spawn ... --thread here|auto|off`는 유지
- 명시적 bind lifecycle 제어 추가:
  - `/acp bind [session|agent] [--persist]`
  - `/acp unbind [--persist]`
  - `/acp status`는 `persistent` / `temporary` 여부를 표시
- 바인딩된 대화에서 `/new`, `/reset`은 ACP 세션을 제자리에서 재설정하고 바인딩은 유지

### 3) 대화 ID

- Discord: 채널/스레드 ID
- Telegram: `chatId:topic:topicId`
- Telegram은 bare topic ID만으로 키잉하지 않음

## 제안 구성 모델

- `bindings[].type`
  - `route`: 일반 라우팅
  - `acp`: 매칭된 대화에 대한 지속형 ACP 하니스 바인딩
- `type: "acp"`에서는 `match.peer.id`가 canonical conversation key
  - Discord 채널/스레드: raw channel/thread ID
  - Telegram 토픽: `chatId:topic:topicId`

백엔드 우선순위:

1. `bindings[].acp.backend`
2. `agents.list[].runtime.acp.backend`
3. 전역 `acp.backend`

## 시스템 적합성

재사용할 기존 구성요소:

- `SessionBindingService`
- ACP spawn/bind service API
- Telegram의 topic/thread context 전달 경로

새로 필요하거나 확장할 구성요소:

- Telegram binding adapter
- route binding과 acp binding을 분리하는 typed binding resolver/index
- Telegram 인바운드의 bound-session 우선 해석
- startup / config change용 persistent binding reconciler

## 단계별 전달

### Phase 1

- `bindings[].type` 스키마 기반 추가
- `agents.list[].runtime.type`으로 ACP-native agent 표시
- route / acp binding index 분리

### Phase 2

- Discord 채널/스레드와 Telegram 포럼 토픽에 대한 top-level `type:"acp"` 해석
- Telegram binding adapter 구현
- Telegram 인바운드 bound-session override parity 확보

### Phase 3

- bound Telegram/Discord 대화에서 `/acp`, `/new`, `/reset`, `/focus` 동작 정렬
- reset 후에도 바인딩 유지

### Phase 4

- `/acp status`와 startup reconciliation 로그 개선
- conflict handling 및 health check 보강

## 가드레일

- 현재와 동일한 ACP enablement / sandbox restriction 준수
- `accountId`를 명시적으로 유지해 cross-account bleed 방지
- 모호한 라우팅은 fail closed
- 채널별 mention/access policy는 명시적으로 유지

## 테스트 계획

- unit: conversation ID normalization, reconciler create/update/delete, `/acp bind --persist`
- integration: Telegram topic → bound ACP session, Discord channel/thread → persistent binding precedence
- regression: temporary binding 유지, unbound 채널/토픽의 기존 라우팅 유지

## 오픈 질문

- Telegram topic에서 `/acp spawn --thread auto`는 `here`처럼 동작해야 하는가?
- persistent binding이 bound conversation에서 mention gating을 항상 우회해야 하는가?
- `/focus --persist`를 `/acp bind --persist`의 별칭으로 둘 것인가?
