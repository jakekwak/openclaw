---
summary: "CLI reference for `openclaw devices` (device pairing + token rotation/revocation)"
read_when:
  - You are approving device pairing requests
  - You need to rotate or revoke device tokens
title: "devices"
---

# `openclaw devices`

Manage device pairing requests and device-scoped tokens.

## Commands

### `openclaw devices list`

List pending pairing requests and paired devices.

```
openclaw devices list
openclaw devices list --json
```

### `openclaw devices remove <deviceId>`

페어링된 디바이스 항목 하나를 제거합니다.

```
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### `openclaw devices clear --yes [--pending]`

페어링된 디바이스를 일괄 제거합니다.

```
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### `openclaw devices approve [requestId] [--latest]`

대기 중인 디바이스 페어링 요청을 승인합니다. `requestId`가 생략되면 OpenClaw는
가장 최근의 대기 요청을 자동으로 승인합니다.

```
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### `openclaw devices reject <requestId>`

Reject a pending device pairing request.

```
openclaw devices reject <requestId>
```

### `openclaw devices rotate --device <id> --role <role> [--scope <scope...>]`

Rotate a device token for a specific role (optionally updating scopes).

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### `openclaw devices revoke --device <id> --role <role>`

Revoke a device token for a specific role.

```
openclaw devices revoke --device <deviceId> --role node
```

## Common options

- `--url <url>`: Gateway WebSocket URL (defaults to `gateway.remote.url` when configured).
- `--token <token>`: Gateway token (if required).
- `--password <password>`: Gateway password (password auth).
- `--timeout <ms>`: RPC timeout.
- `--json`: JSON 출력 (스크립팅에 권장).

참고: `--url`을 설정할 때, CLI는 구성이나 환경 변수 인증을 대체하지 않습니다.
`--token` 또는 `--password`를 명시적으로 전달하세요. 명시적 인증이 누락되면 오류가 발생합니다.

## Notes

- 토큰 로테이션은 새 토큰을 반환합니다 (민감한 정보). 비밀처럼 다루세요.
- 이러한 명령어는 `operator.pairing` (또는 `operator.admin`) 범위가 필요합니다.
- `devices clear`는 의도적으로 `--yes` 플래그가 필요합니다.
- 로컬 루프백에서 페어링 범위를 사용할 수 없는 경우 (명시적 `--url`도 없는 경우), list/approve는 로컬 페어링 폴백을 사용할 수 있습니다.

## 토큰 불일치 복구 체크리스트

Control UI나 다른 클라이언트가 계속 `AUTH_TOKEN_MISMATCH` 또는 `AUTH_DEVICE_TOKEN_MISMATCH`로 실패할 때 사용하세요.

1. 현재 gateway token source를 확인합니다:

```bash
openclaw config get gateway.auth.token
```

2. 페어링된 디바이스 목록을 보고 영향받은 device id를 확인합니다:

```bash
openclaw devices list
```

3. 영향받은 디바이스의 operator token을 회전합니다:

```bash
openclaw devices rotate --device <deviceId> --role operator
```

4. 회전만으로 충분하지 않으면 오래된 페어링을 제거하고 다시 승인합니다:

```bash
openclaw devices remove <deviceId>
openclaw devices list
openclaw devices approve <requestId>
```

5. 현재 공유 token/password로 클라이언트 연결을 다시 시도합니다.

관련 문서:

- [대시보드 인증 문제 해결](/web/dashboard#if-you-see-unauthorized-1008)
- [Gateway 문제 해결](/gateway/troubleshooting#dashboard-control-ui-connectivity)
