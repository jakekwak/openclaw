---
title: OpenShell
summary: "OpenClaw agent용 managed sandbox backend로 OpenShell 사용하기"
read_when:
  - 로컬 Docker 대신 클라우드 managed sandbox를 쓰고 싶을 때
  - OpenShell plugin을 설정할 때
  - mirror와 remote workspace 모드 중에서 골라야 할 때
---

# OpenShell

OpenShell은 OpenClaw용 managed sandbox backend입니다. 로컬 Docker 대신 `openshell` CLI가 원격 sandbox lifecycle을 관리하고, OpenClaw는 SSH 기반 명령 실행을 사용합니다.

OpenShell plugin은 일반 [SSH backend](/ko-KR/gateway/sandboxing#ssh-backend)와 같은 SSH 전송 및 원격 파일시스템 bridge를 재사용하면서, OpenShell 전용 lifecycle과 선택적 `mirror` workspace 모드를 추가합니다.

## 필요 조건

- `openshell` CLI가 설치되어 있고 `PATH`에 있어야 함
- OpenShell 계정과 sandbox 접근 권한
- 호스트에서 실행 중인 OpenClaw Gateway

## 빠른 시작

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "all",
        backend: "openshell",
        scope: "session",
        workspaceAccess: "rw",
      },
    },
  },
  plugins: {
    entries: {
      openshell: {
        enabled: true,
        config: {
          from: "openclaw",
          mode: "remote",
        },
      },
    },
  },
}
```

그다음 Gateway를 재시작하고 확인합니다:

```bash
openclaw sandbox list
openclaw sandbox explain
```

## Workspace 모드

### `mirror`

로컬 workspace를 canonical로 유지합니다.

- `exec` 전 로컬을 remote로 sync
- `exec` 후 remote를 다시 로컬로 sync
- turn 사이에는 로컬 workspace가 source of truth

적합한 경우:

- OpenClaw 밖에서 로컬 파일을 직접 수정함
- Docker backend와 비슷한 동작을 원함

### `remote`

OpenShell workspace를 canonical로 유지합니다.

- sandbox 첫 생성 시 로컬에서 remote로 한 번만 seed
- 이후 `exec`, `read`, `write`, `edit`, `apply_patch`가 remote에 직접 작동
- remote 변경을 로컬로 다시 sync하지 않음

적합한 경우:

- 원격 중심의 장기 sandbox를 운영할 때
- turn당 sync 오버헤드를 줄이고 싶을 때

중요: 초기 seed 후 호스트 파일을 바꿔도 remote는 그 변경을 자동 반영하지 않습니다. 다시 seed하려면 `openclaw sandbox recreate`를 사용하세요.

### 모드 선택

|                          | `mirror`                   | `remote`                  |
| ------------------------ | -------------------------- | ------------------------- |
| **Canonical workspace**  | Local host                 | Remote OpenShell          |
| **Sync direction**       | Bidirectional (each exec)  | One-time seed             |
| **Per-turn overhead**    | Higher (upload + download) | Lower (direct remote ops) |
| **Local edits visible?** | Yes, on next exec          | No, until recreate        |

## 설정 레퍼런스

`plugins.entries.openshell.config` 아래에 둡니다:

| Key                  | Default       | Description                          |
| -------------------- | ------------- | ------------------------------------ |
| `mode`               | `"mirror"`    | Workspace sync mode                  |
| `command`            | `"openshell"` | `openshell` CLI 경로 또는 이름       |
| `from`               | `"openclaw"`  | 첫 sandbox 생성 시 source            |
| `gateway`            | —             | OpenShell gateway 이름               |
| `gatewayEndpoint`    | —             | OpenShell gateway endpoint URL       |
| `policy`             | —             | sandbox 생성용 policy ID             |
| `providers`          | `[]`          | sandbox 생성 시 붙일 provider 이름   |
| `gpu`                | `false`       | GPU 요청 여부                        |
| `autoProviders`      | `true`        | `--auto-providers` 전달 여부         |
| `remoteWorkspaceDir` | `"/sandbox"`  | sandbox 내부 writable workspace 경로 |
| `timeoutSeconds`     | `120`         | `openshell` CLI 작업 타임아웃        |

공통 sandbox 설정은 `agents.defaults.sandbox` 아래에 둡니다.

## Lifecycle 관리

```bash
openclaw sandbox list
openclaw sandbox explain
openclaw sandbox recreate --all
```

`remote` 모드에서는 recreate가 특히 중요합니다. canonical remote workspace를 지우고 다음 사용 시 로컬에서 다시 seed합니다.
