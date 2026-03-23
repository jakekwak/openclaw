---
title: "Memory configuration reference"
summary: "OpenClaw memory search, embedding provider, QMD backend, hybrid search, multimodal memory 설정 레퍼런스"
read_when:
  - memory search provider나 embedding model을 설정하려 할 때
  - QMD backend를 설정하려 할 때
  - hybrid search, MMR, temporal decay를 튜닝하려 할 때
  - multimodal memory indexing을 켜려 할 때
---

# Memory configuration reference

이 문서는 OpenClaw memory search의 전체 설정 표면을 다룹니다. 개념적 개요는 [Memory](/ko-KR/concepts/memory)를 보세요.

## 기본값

- 기본 활성화
- memory 파일 변경을 감시함
- 설정 위치는 `agents.defaults.memorySearch`
- provider를 명시하지 않으면 OpenClaw가 `local -> openai -> gemini -> voyage -> mistral` 순으로 자동 선택
- `ollama`도 지원하지만 자동 선택되지는 않음

remote embedding은 embedding provider API 키가 필요합니다. OpenClaw는 auth profile, `models.providers.*.apiKey`, env var에서 키를 해석합니다.

## QMD backend (experimental)

`memory.backend = "qmd"`를 설정하면 내장 SQLite 인덱서 대신 [QMD](https://github.com/tobi/qmd)를 사용합니다.

핵심 포인트:

- 기본 비활성화
- `qmd` CLI를 별도 설치해야 함
- 로컬 Bun + `node-llama-cpp` 기반
- 상태는 `~/.openclaw/agents/<agentId>/qmd/` 아래에 저장
- macOS/Linux 우선 지원, Windows는 WSL2 권장

### 주요 설정 (`memory.qmd.*`)

- `command`
- `searchMode`
- `includeDefaultMemory`
- `paths[]`
- `sessions`
- `update`
- `limits`
- `scope`

QMD가 실패하거나 binary가 없으면 OpenClaw는 builtin SQLite provider로 fallback합니다.

## Additional memory paths

기본 workspace layout 밖의 Markdown을 인덱싱하려면 `extraPaths`를 사용합니다:

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

## Multimodal memory

Gemini embedding 2를 사용할 때 `extraPaths` 아래 image/audio 파일도 인덱싱할 수 있습니다:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-2-preview",
      extraPaths: ["assets/reference", "voice-notes"],
      multimodal: {
        enabled: true,
        modalities: ["image", "audio"],
        maxFileBytes: 10000000
      }
    }
  }
}
```

주의:

- 현재는 `gemini-embedding-2-preview`에서만 지원
- multimodal indexing은 `extraPaths`에서 발견된 파일에만 적용
- `memorySearch.fallback`은 `"none"`이어야 함

## 실무 팁

- memory search는 provider와 key 해석이 가장 흔한 실패 지점
- multimodal을 켤 때는 file size와 fallback 제약을 함께 확인
- QMD는 로컬 성능과 richer retrieval을 주지만 별도 sidecar/CLI 관리가 필요

## 관련 문서

- [Memory](/ko-KR/concepts/memory)
- [Gateway configuration](/ko-KR/gateway/configuration)
