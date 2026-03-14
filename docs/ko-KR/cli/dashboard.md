---
summary: "`openclaw dashboard` 명령어에 대한 CLI 참조 (제어 UI 열기)"
read_when:
  - 현재 토큰으로 제어 UI를 열고 싶을 때
  - 브라우저를 실행하지 않고 URL만 출력하고 싶을 때
title: "대시보드"
---

# `openclaw dashboard`

현재 인증을 사용하여 제어 UI를 엽니다.

```bash
openclaw dashboard
openclaw dashboard --no-open
```

주의사항:

- `dashboard`는 가능한 경우 구성된 `gateway.auth.token` SecretRef를 해석합니다.
- 토큰이 SecretRef-managed인 경우(해석 가능 여부와 무관하게), `dashboard`는 외부 비밀이 터미널 출력, 클립보드 기록, 브라우저 실행 인자에 노출되지 않도록 토큰이 없는 URL만 출력/복사/오픈합니다.
- `gateway.auth.token`이 SecretRef-managed이지만 현재 명령 경로에서 해석되지 않으면, 잘못된 토큰 플레이스홀더를 넣지 않고 non-tokenized URL과 명시적인 remediation 안내를 출력합니다.
