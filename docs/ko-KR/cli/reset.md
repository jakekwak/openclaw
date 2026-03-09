---
summary: "`openclaw reset`에 대한 CLI 참조 (로컬 상태/설정 초기화)"
read_when:
  - CLI를 설치한 상태에서 로컬 상태를 초기화하고 싶을 때
  - 삭제될 항목을 미리 보기 원할 때
title: "reset"
---

# `openclaw reset`

로컬 설정/상태 초기화 (CLI 설치는 유지).

```bash
openclaw backup create
openclaw reset
openclaw reset --dry-run
openclaw reset --scope config+creds+sessions --yes --non-interactive
```

로컬 상태를 지우기 전에 복원 가능한 스냅샷이 필요하다면 먼저 `openclaw backup create`를 실행하십시오.
