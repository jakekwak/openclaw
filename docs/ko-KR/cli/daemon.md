---
summary: "게이트웨이 서비스 관리용 레거시 별칭 `openclaw daemon` CLI 참조"
read_when:
  - 아직 스크립트에서 `openclaw daemon ...`을 사용할 때
  - 서비스 라이프사이클 명령(`install/start/stop/restart/status`)이 필요할 때
title: "daemon"
---

# `openclaw daemon`

게이트웨이 서비스 관리 명령의 레거시 별칭입니다.

`openclaw daemon ...`은 `openclaw gateway ...` 서비스 명령과 같은 서비스 제어 표면에 매핑됩니다.

## 사용법

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## 하위 명령어

- `status`: 서비스 설치 상태를 표시하고 게이트웨이 상태를 프로브
- `install`: 서비스 설치 (`launchd`/`systemd`/`schtasks`)
- `uninstall`: 서비스 제거
- `start`: 서비스 시작
- `stop`: 서비스 중지
- `restart`: 서비스 재시작

## 공통 옵션

- `status`: `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
- `install`: `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
- 라이프사이클(`uninstall|start|stop|restart`): `--json`

주의:

- `status`는 가능한 경우 프로브 인증을 위해 구성된 auth SecretRef를 해석합니다.
- Linux systemd 설치에서는 `status`의 token-drift 검사가 `Environment=`와 `EnvironmentFile=` 유닛 소스를 모두 포함합니다.
- 토큰 인증에 토큰이 필요하고 `gateway.auth.token`이 SecretRef-managed인 경우, `install`은 SecretRef가 해석 가능한지 검증하되 해석된 토큰을 서비스 환경 메타데이터에 저장하지 않습니다.
- 토큰 인증에 토큰이 필요한데 구성된 token SecretRef가 해석되지 않으면 install은 실패 닫힘(fail closed)합니다.
- `gateway.auth.token`과 `gateway.auth.password`가 모두 설정돼 있고 `gateway.auth.mode`가 비어 있으면, mode를 명시적으로 설정할 때까지 install이 차단됩니다.

## 권장

최신 문서와 예제는 [`openclaw gateway`](/ko-KR/cli/gateway)를 사용하십시오.
