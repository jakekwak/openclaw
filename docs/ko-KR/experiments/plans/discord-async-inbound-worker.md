---
summary: "Discord listener와 장기 실행 agent turn을 분리하기 위한 Discord 전용 inbound worker 계획의 상태와 다음 단계"
owner: "openclaw"
status: "in_progress"
last_updated: "2026-03-05"
title: "Discord Async Inbound Worker 계획"
---

# Discord Async Inbound Worker 계획

## 목표

Discord listener timeout이 사용자에게 드러나는 실패 모드가 되지 않도록, Discord 인바운드 turn을 비동기화합니다.

1. Gateway listener가 인바운드 이벤트를 빠르게 수락하고 정규화
2. Discord run queue가 현재와 같은 ordering boundary로 직렬화된 job 저장
3. Worker가 실제 agent turn을 Carbon listener lifetime 밖에서 실행
4. 실행 완료 후 원래 채널/스레드로 reply 전달

이는 `channels.discord.eventQueue.listenerTimeout` 때문에 queued Discord run이 timeout되는 문제를 장기적으로 해결하기 위한 계획입니다.

## 현재 상태

부분 구현되었습니다.

이미 완료:

- Discord listener timeout과 Discord run timeout을 분리
- 수락된 인바운드 turn을 `src/discord/monitor/inbound-worker.ts`에 enqueue
- 장기 실행 turn의 소유권을 Carbon listener가 아닌 worker가 갖도록 변경
- per-route ordering 유지
- Discord worker path timeout regression 테스트 추가

아직 남은 것:

- `DiscordInboundJob`가 아직 완전한 plain data가 아니고 live runtime reference를 포함
- `stop`, `new`, `reset` 같은 command semantics가 worker-native가 아님
- worker observability와 operator status가 약함
- restart durability 없음

## 핵심 문제

현재 구조는 전체 agent turn을 listener lifetime에 묶어 두고 있습니다.

- long but healthy turn도 listener watchdog에 의해 중단될 수 있음
- downstream runtime이 실제로는 결과를 만들었어도 사용자는 응답을 못 볼 수 있음

timeout을 늘리는 것은 완화일 뿐 근본 해결이 아닙니다.

## 목표 아키텍처

### 1) Listener 단계

`DiscordMessageListener`는 계속 ingress point로 남되 다음만 담당합니다.

- preflight / policy 검사
- 수락된 입력을 serializable `DiscordInboundJob`으로 정규화
- per-session 또는 per-channel async queue에 enqueue
- enqueue 성공 즉시 Carbon으로 반환

### 2) 정규화된 job payload

job descriptor에는 이후 실행에 필요한 plain data만 담아야 합니다.

- route identity: `agentId`, `sessionKey`, `accountId`, `channel`
- delivery identity: destination channel id, reply target message id, thread id
- sender identity: sender id, label, username, tag
- channel context: guild id, channel slug/name, thread metadata, resolved system prompt override
- normalized message body: base text, effective message text, attachment descriptor
- gating decisions: mention 결과, command auth 결과, bound session/agent 메타데이터

live Carbon object나 mutable closure는 금지합니다.

### 3) Worker 단계

Discord 전용 worker runner가 다음을 담당합니다.

- `DiscordInboundJob`로부터 turn context 재구성
- media와 추가 채널 메타데이터 로드
- agent turn dispatch
- 최종 reply payload 전달
- status와 diagnostics 업데이트

### 4) Ordering 모델

지금과 동일한 queue key 논리를 유지합니다.

- 같은 bound agent conversation은 서로 interleave되지 않음
- 서로 다른 Discord 채널은 병렬 진행 가능

### 5) Timeout 모델

- listener timeout: 정규화와 enqueue만 담당, 짧아야 함
- run timeout: worker 소유이며 명시적이고 사용자에게 보이는 timeout

## 구현 단계

### Phase 1: normalization boundary

- `buildDiscordInboundJob(...)` 추출
- worker handoff 테스트 추가
- 남은 과제:
  - `DiscordInboundJob`를 plain data only로 만들기
  - live dependency를 worker-owned service로 옮기기

### Phase 2: in-memory worker queue

- `DiscordInboundWorkerQueue` 추가
- listener가 `processDiscordMessage(...)`를 직접 기다리지 않고 enqueue
- worker가 프로세스 내부 메모리 큐에서 실행

### Phase 3: process split

- delivery, typing, draft streaming을 worker-facing adapter 뒤로 이동
- live preflight context 대신 worker context reconstruction 사용

### Phase 4: command semantics

- queued work에서도 `stop`, `new`, `reset`이 올바르게 동작하게 보정

### Phase 5: observability

- queue depth / active worker count 노출
- enqueue/start/finish/timeout/cancel 이유 기록
- worker-owned failure를 로그에서 명확히 표시

### Phase 6: optional durability

- gateway restart 후 queued job을 유지할지 결정
- 필요하면 job descriptor와 delivery checkpoint를 영속화
- 아니면 in-memory 경계를 문서화

## 현재 주요 파일

- `src/discord/monitor/listeners.ts`
- `src/discord/monitor/message-handler.ts`
- `src/discord/monitor/message-handler.preflight.ts`
- `src/discord/monitor/message-handler.process.ts`
- `src/discord/monitor/status.ts`
- `src/discord/monitor/inbound-job.ts`
- `src/discord/monitor/inbound-worker.ts`

## 바로 다음 단계

1. live runtime dependency를 `DiscordInboundJob` 밖으로 이동
2. worker가 그 dependency를 직접 소유
3. queued job은 plain Discord data만 유지
4. worker 내부에서 실행 context를 재구성

즉 다음 항목은 job이 아니라 worker 쪽에 살아야 합니다.

- `client`
- `threadBindings`
- `guildHistories`
- `discordRestFetch`
- 기타 mutable runtime handle

## 수용 기준

완료 기준:

1. Discord listener timeout이 건강한 장기 실행 turn을 더 이상 중단하지 않음
2. listener lifetime과 agent-turn lifetime이 코드에서 분리됨
3. 기존 per-session ordering 유지
4. ACP-bound Discord channel도 같은 worker path에서 동작
5. `stop`이 listener-owned call stack이 아니라 worker-owned run을 겨냥
6. timeout 및 delivery failure가 silent drop이 아니라 명시적 worker outcome이 됨
