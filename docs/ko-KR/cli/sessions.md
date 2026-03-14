---
summary: "`openclaw sessions` CLI 참조 (저장된 세션 목록 + 사용법)"
read_when:
  - 저장된 세션을 나열하고 최근 활동을 보고 싶을 때
title: "세션"
---

# `openclaw sessions`

저장된 대화 세션을 나열합니다.

```bash
openclaw sessions
openclaw sessions --active 120
openclaw sessions --json
```

범위 선택:

- `--agent <id>`: 단일 에이전트 저장소
- `--all-agents`: 구성된 모든 에이전트 저장소 집계
- `--store <path>`: 명시적 저장소 경로 (`--agent`, `--all-agents`와 함께 사용 불가)

`openclaw sessions --all-agents`는 구성된 에이전트 저장소를 읽습니다. Gateway와 ACP 세션 검색은 더 넓은 범위를 검사하며, 기본 `agents/` 루트 또는 템플릿이 적용된 `session.store` 루트 아래에서 디스크 전용 저장소도 포함합니다. 이렇게 검색된 저장소는 에이전트 루트 내부의 일반 `sessions.json` 파일이어야 하며, 심볼릭 링크와 루트 밖 경로는 건너뜁니다.

JSON 예시:

`openclaw sessions --all-agents --json`:

```json
[
  {
    "agentId": "main",
    "sessionKey": "agent:main:main",
    "updatedAt": "2026-02-16T12:34:56.000Z",
    "messageCount": 42
  }
]
```

정리 예시:

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:direct:123"
openclaw sessions cleanup --json
```
