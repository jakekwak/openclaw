---
summary: "CLI 레퍼런스 `openclaw memory` (status/index/search)"
read_when:
  - 의미론적 메모리를 인덱싱하거나 검색하고 싶은 경우
  - 메모리 가용성 또는 인덱싱을 디버깅하는 경우
title: "memory"
---

# `openclaw memory`

의미론적 메모리 인덱싱 및 검색을 관리합니다. 활성 메모리 플러그인에 의해 제공됩니다(기본값: `memory-core`; `plugins.slots.memory = "none"`을 설정하여 비활성화).

연관:

- 메모리 개념: [Memory](/ko-KR/concepts/memory)
- 플러그인: [Plugins](/ko-KR/tools/plugin)

## Examples

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory search --query "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## Options

Common:

- `--agent <id>`: 단일 에이전트에 범위를 지정합니다 (기본값: 모든 구성된 에이전트).
- `--verbose`: 프로브 및 인덱싱 중에 자세한 로그를 출력합니다.

`memory search`:

- 쿼리 입력: 위치 인수 `[query]` 또는 `--query <text>` 중 하나로 전달합니다.
- 둘 다 제공된 경우 `--query`가 우선합니다.
- 둘 다 제공되지 않으면 명령어가 오류와 함께 종료됩니다.

Notes:

- `memory status --deep`은 벡터 및 임베딩 가용성을 프로브합니다.
- `memory status --deep --index`는 저장소가 깨끗하지 않을 경우 재인덱스를 실행합니다.
- `memory index --verbose`는 각 단계의 세부 정보를 출력합니다 (프로바이더, 모델, 소스, 배치 활동).
- `memory status`는 `memorySearch.extraPaths`를 통해 구성된 추가 경로를 포함합니다.
- 활성 메모리 원격 API 키 필드가 SecretRef로 구성된 경우, 명령은 활성 게이트웨이 스냅샷에서 값을 해석합니다. 게이트웨이에 연결할 수 없으면 즉시 실패합니다.
- 게이트웨이 버전 불일치 참고: 이 경로는 `secrets.resolve`를 지원하는 게이트웨이가 필요합니다. 오래된 게이트웨이는 unknown-method 오류를 반환합니다.
