---
summary: "/btw로 보내는 일시적 사이드 질문"
read_when:
  - 현재 세션에 대한 짧은 사이드 질문을 하고 싶을 때
  - 클라이언트 간 BTW 동작을 구현하거나 디버그할 때
title: "BTW Side Questions"
---

# BTW Side Questions

`/btw`는 **현재 세션**에 대한 짧은 사이드 질문을 일반 대화 히스토리에 남기지 않고 물을 수 있게 해 줍니다.

## 무엇을 하는가

예를 들어:

```text
/btw what changed?
```

OpenClaw는:

1. 현재 세션 컨텍스트를 스냅샷하고,
2. 별도의 **tool-less** 모델 호출을 실행하고,
3. 사이드 질문에만 답하고,
4. 메인 실행은 그대로 두고,
5. BTW 질문과 답을 세션 히스토리에 쓰지 않으며,
6. 일반 assistant 메시지가 아닌 **live side result**로 응답합니다.

핵심 모델:

- 같은 세션 컨텍스트
- 별도의 일회성 side query
- tool 호출 없음
- 이후 컨텍스트 오염 없음
- transcript 영속 저장 없음

## 무엇을 하지 않는가

`/btw`는 다음을 하지 않습니다:

- 새로운 영속 세션 생성
- 미완료된 메인 작업 계속 수행
- tool 또는 agent tool loop 실행
- `chat.history`에 기록
- reload 이후 유지

의도적으로 **ephemeral**합니다.

## 전달 모델

BTW는 일반 assistant transcript 메시지로 전달되지 않습니다.

- 일반 assistant chat: `chat`
- BTW: `chat.side_result`

BTW는 `chat.history`로 replay되지 않으므로 reload 후에는 사라집니다.

## Surface 동작

### TUI

- 일반 assistant 답변과 시각적으로 구분됨
- `Enter` 또는 `Esc`로 닫을 수 있음
- reload 시 다시 나오지 않음

### 외부 채널

Telegram, WhatsApp, Discord 같은 채널에서는 BTW가 명확히 표시된 일회성 답변으로 전달됩니다. 그래도 normal session history가 아니라 side result입니다.

### Control UI / web

Gateway는 BTW를 `chat.side_result`로 올바르게 emit하지만, 현재 Control UI에는 이를 실시간 렌더링하는 전용 consumer가 아직 필요합니다.

## 언제 사용하나

- 현재 작업에 대한 짧은 확인 질문
- 긴 실행 도중의 사실 확인
- 이후 세션 컨텍스트에 남기고 싶지 않은 임시 답변

예시:

```text
/btw what file are we editing?
/btw what does this error mean?
/btw summarize the current task in one sentence
```

## 관련 문서

- [Slash commands](/ko-KR/tools/slash-commands)
- [Thinking Levels](/ko-KR/tools/thinking)
- [Session](/ko-KR/concepts/session)
