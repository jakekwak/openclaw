---
summary: "이전 Matrix plugin을 현재 구현으로 제자리 업그레이드하는 방법과, 암호화 상태 복구 한계 및 수동 복구 절차"
read_when:
  - 기존 Matrix 설치를 업그레이드할 때
  - 암호화된 Matrix history와 device state를 마이그레이션할 때
title: "Matrix migration"
---

# Matrix migration

이 문서는 이전 공개 `matrix` plugin에서 현재 구현으로 업그레이드하는 과정을 설명합니다.

대부분의 사용자는 제자리 업그레이드가 됩니다:

- plugin 이름은 계속 `@openclaw/matrix`
- channel 이름도 계속 `matrix`
- config는 계속 `channels.matrix`
- cached credential은 `~/.openclaw/credentials/matrix/`
- runtime state는 `~/.openclaw/matrix/`

즉, config key를 바꾸거나 plugin을 새 이름으로 다시 설치할 필요가 없습니다.

## 자동 마이그레이션이 하는 일

Gateway startup 또는 `openclaw doctor --fix` 실행 시 OpenClaw는 기존 Matrix state를 자동 복구하려고 시도합니다.

자동으로 포함되는 항목:

- `~/Backups/openclaw-migrations/` 아래 pre-migration snapshot 생성 또는 재사용
- 기존 Matrix credential 재사용
- `channels.matrix` config 유지
- flat sync/crypto store를 account-scoped 위치로 이동
- 이전 rust crypto store에서 room-key backup decryption key 추출 시도
- token 변경 후에도 같은 account/homeserver/user에 대한 기존 storage root 재사용
- 다음 Matrix startup에서 backup된 room key 복원

## 자동으로 못 하는 일

이전 Matrix plugin은 room-key backup을 자동 생성하지 않았습니다. 따라서 다음은 자동 복구가 안 될 수 있습니다:

- 한 번도 backup되지 않은 local-only room key
- `homeserver`, `userId`, `accessToken`이 아직 없어 target account를 확정할 수 없는 경우
- 여러 Matrix account가 있는데 `channels.matrix.defaultAccount`가 없는 경우
- 표준 package가 아닌 custom plugin path 설치
- backup은 있지만 recovery key를 로컬에 남기지 않았던 경우

즉, 예전 설치에서 local-only encrypted history만 있었다면 업그레이드 후에도 일부 오래된 암호화 메시지는 읽지 못할 수 있습니다.

## 권장 업그레이드 흐름

1. OpenClaw와 Matrix plugin을 일반 방식으로 업데이트
2. `openclaw doctor --fix` 실행
3. Gateway 시작 또는 재시작
4. 현재 verification/backup 상태 확인:

```bash
openclaw matrix verify status
openclaw matrix verify backup status
```

5. recovery key가 필요하다고 나오면:

```bash
openclaw matrix verify backup restore --recovery-key "<your-recovery-key>"
```

6. device가 아직 unverified라면:

```bash
openclaw matrix verify device "<your-recovery-key>"
```

7. 복구 불가능한 오래된 history를 포기하고 새 baseline으로 갈 경우:

```bash
openclaw matrix verify backup reset --yes
```

8. 아직 server-side key backup이 없다면:

```bash
openclaw matrix verify bootstrap
```

## 자주 보는 메시지와 의미

### 상태/경고 메시지

- `Legacy Matrix state detected ... but the target could not be resolved yet`
  현재 account/device root를 아직 결정할 수 없다는 뜻입니다.
- `... multiple Matrix accounts are configured and channels.matrix.defaultAccount is not set`
  shared legacy store를 어느 account로 넣어야 할지 OpenClaw가 추측하지 않겠다는 뜻입니다.
- `Matrix migration warnings are present, but no on-disk Matrix mutation is actionable yet`
  old state는 감지했지만 아직 identity/credential 데이터가 부족합니다.
- `Matrix is installed from a custom path`
  path install이므로 mainline update가 표준 Matrix package로 자동 교체하지 않습니다.

### 암호화 상태 복구 메시지

- `matrix: restored X/Y room key(s) ...`
  backup된 room key가 성공적으로 복원됐습니다.
- `legacy local-only room key(s) were never backed up ...`
  이전 로컬 저장소에만 있던 key라 자동 복구가 안 됩니다.
- `... no local backup decryption key was found`
  backup은 있지만 recovery key를 자동 복구하지 못했습니다.
- `matrix: failed restoring room keys ...`
  복원 시도를 했지만 Matrix가 오류를 반환했습니다.

### 수동 복구 메시지

- `Backup key is not loaded on this device`
- `Store a recovery key with 'openclaw matrix verify device <key>'`
- `Backup key mismatch on this device`
- `Matrix recovery key is required`
- `Invalid Matrix recovery key`

이런 경우에는 올바른 recovery key를 다시 넣고 `verify device` 또는 `verify backup restore`를 재실행하면 됩니다.

## 아직 encrypted history가 안 돌아오면

다음 순서로 확인하세요:

```bash
openclaw matrix verify status --verbose
openclaw matrix verify backup status --verbose
openclaw matrix verify backup restore --recovery-key "<your-recovery-key>" --verbose
```

backup restore는 성공했는데도 오래된 room 일부가 비어 있다면, 그 key는 이전 plugin 시절에 서버로 backup되지 않았을 가능성이 큽니다.

## 앞으로의 메시지만 새로 시작하려면

복구 불가능한 오래된 encrypted history를 포기하고 앞으로의 backup baseline만 새로 만들려면:

```bash
openclaw matrix verify backup reset --yes
openclaw matrix verify backup status --verbose
openclaw matrix verify status
```

## 관련 문서

- [Matrix](/ko-KR/channels/matrix)
- [Doctor](/ko-KR/gateway/doctor)
- [Migrating](/ko-KR/install/migrating)
- [Plugins](/ko-KR/tools/plugin)
