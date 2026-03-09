---
summary: "로컬 OpenClaw 상태용 백업 아카이브를 만드는 `openclaw backup` CLI 참조"
read_when:
  - 로컬 OpenClaw 상태를 위한 정식 백업 아카이브가 필요할 때
  - reset 또는 uninstall 전에 포함될 경로를 미리 확인하고 싶을 때
title: "backup"
---

# `openclaw backup`

OpenClaw 상태, 설정, 자격 증명, 세션, 그리고 선택적으로 워크스페이스를 포함하는 로컬 백업 아카이브를 생성합니다.

```bash
openclaw backup create
openclaw backup create --output ~/Backups
openclaw backup create --dry-run --json
openclaw backup create --verify
openclaw backup create --no-include-workspace
openclaw backup create --only-config
openclaw backup verify ./2026-03-09T00-00-00.000Z-openclaw-backup.tar.gz
```

## 참고

- 아카이브에는 해석된 소스 경로와 아카이브 레이아웃이 담긴 `manifest.json` 파일이 포함됩니다.
- 기본 출력은 현재 작업 디렉터리에 생성되는 타임스탬프 기반 `.tar.gz` 아카이브입니다.
- 현재 작업 디렉터리가 백업 대상 소스 트리 내부에 있으면, OpenClaw는 기본 아카이브 위치로 홈 디렉터리로 폴백합니다.
- 기존 아카이브 파일은 절대 덮어쓰지 않습니다.
- 소스 상태/워크스페이스 트리 내부의 출력 경로는 자기 자신을 포함하는 일을 막기 위해 거부됩니다.
- `openclaw backup verify <archive>`는 아카이브에 루트 manifest가 정확히 하나만 있는지 검증하고, 경로 순회형 archive path를 거부하며, manifest에 선언된 모든 payload가 tarball 안에 존재하는지 확인합니다.
- `openclaw backup create --verify`는 아카이브를 쓴 직후 이 검증을 바로 실행합니다.
- `openclaw backup create --only-config`는 현재 활성 JSON 설정 파일만 백업합니다.

## 무엇이 백업되나요

`openclaw backup create`는 로컬 OpenClaw 설치를 기준으로 백업 소스를 계획합니다.

- OpenClaw의 로컬 상태 해석기가 반환하는 상태 디렉터리, 보통 `~/.openclaw`
- 현재 활성 설정 파일 경로
- OAuth / 자격 증명 디렉터리
- `--no-include-workspace`를 주지 않은 경우, 현재 설정에서 발견된 워크스페이스 디렉터리

`--only-config`를 사용하면 OpenClaw는 상태, 자격 증명, 워크스페이스 탐색을 건너뛰고 현재 활성 설정 파일 경로만 아카이브합니다.

OpenClaw는 아카이브를 만들기 전에 경로를 canonicalize합니다. 설정, 자격 증명, 또는 워크스페이스가 이미 상태 디렉터리 안에 있으면 별도의 최상위 백업 소스로 중복 포함하지 않습니다. 존재하지 않는 경로는 건너뜁니다.

아카이브 payload에는 해당 소스 트리의 파일 내용이 저장되고, 포함된 `manifest.json`에는 각 자산의 해석된 절대 소스 경로와 실제 사용된 아카이브 레이아웃이 기록됩니다.

## 잘못된 설정 파일일 때의 동작

`openclaw backup`은 복구 상황에서도 동작할 수 있도록 일반 설정 사전검사를 의도적으로 우회합니다. 다만 워크스페이스 탐색은 유효한 설정에 의존하므로, 설정 파일이 존재하지만 잘못되어 있고 워크스페이스 백업이 여전히 활성화돼 있으면 `openclaw backup create`는 즉시 실패합니다.

그 상황에서도 부분 백업이 필요하면 다음처럼 다시 실행하십시오.

```bash
openclaw backup create --no-include-workspace
```

그러면 상태, 설정, 자격 증명은 백업 범위에 유지하면서 워크스페이스 탐색만 완전히 건너뜁니다.

설정 파일 자체의 사본만 필요하다면, `--only-config`도 사용할 수 있습니다. 이 모드는 워크스페이스 탐색을 위해 설정 파싱에 의존하지 않기 때문에 설정이 손상돼 있어도 동작합니다.

## 크기와 성능

OpenClaw는 기본 제공되는 최대 백업 크기나 파일별 크기 제한을 두지 않습니다.

실제 한계는 로컬 머신과 대상 파일시스템에서 결정됩니다.

- 임시 아카이브 작성과 최종 아카이브 저장을 위한 가용 공간
- 큰 워크스페이스 트리를 순회하고 `.tar.gz`로 압축하는 데 걸리는 시간
- `openclaw backup create --verify` 또는 `openclaw backup verify` 사용 시 아카이브를 다시 검사하는 시간
- 대상 경로의 파일시스템 동작. OpenClaw는 덮어쓰기 없는 하드링크 publish 단계를 우선 시도하고, 하드링크를 지원하지 않으면 배타적 복사로 폴백합니다

보통 아카이브 크기를 가장 크게 만드는 요소는 대형 워크스페이스입니다. 더 작거나 빠른 백업이 필요하면 `--no-include-workspace`를 사용하십시오.

가장 작은 아카이브가 필요하면 `--only-config`를 사용하면 됩니다.
