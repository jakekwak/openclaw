---
summary: "`openclaw tui`에 대한 CLI 참조 (게이트웨이에 연결된 터미널 UI)"
read_when:
  - 게이트웨이를 위한 터미널 UI가 필요하다면 (원격 사용에 적합)
  - 스크립트에서 URL/토큰/세션을 전달하고 싶다면
title: "tui"
---

# `openclaw tui`

게이트웨이에 연결된 터미널 UI를 엽니다.

관련 문서:

- TUI 가이드: [TUI](/web/tui)

주의사항:

- `tui`는 가능한 경우 토큰/비밀번호 인증용 구성된 gateway auth SecretRef를 해석합니다 (`env`/`file`/`exec` provider).
- 구성된 에이전트 workspace 디렉터리 내부에서 실행하면, TUI는 세션 키 기본값으로 해당 에이전트를 자동 선택합니다 (`--session`이 명시적으로 `agent:<id>:...`가 아닌 경우).

## 예시

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
# 에이전트 workspace 내부에서 실행하면 해당 에이전트를 자동 추론
openclaw tui --session bugfix
```
