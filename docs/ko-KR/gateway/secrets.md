---
summary: "SecretRef 계약, 런타임 스냅샷 동작, 안전한 단방향 스크러빙을 포함한 시크릿 관리"
read_when:
  - 프로바이더 자격 증명과 `auth-profiles.json` ref에 SecretRef를 설정할 때
  - 프로덕션에서 secrets reload, audit, configure, apply를 안전하게 운용할 때
  - 시작 시 fail-fast, 비활성 surface 필터링, last-known-good 동작을 이해할 때
title: "Secrets Management"
---

# Secrets management

OpenClaw는 지원되는 자격 증명을 구성 파일의 평문으로 저장하지 않아도 되도록 가산적 SecretRef를 지원합니다.

평문 자격 증명도 계속 동작합니다. SecretRef는 자격 증명별 opt-in입니다.

## 목표와 런타임 모델

시크릿은 메모리 내 런타임 스냅샷으로 해석됩니다.

- 해석은 요청 경로에서 지연 수행되지 않고 activation 시 eager하게 수행됩니다.
- 실제로 활성인 SecretRef를 해석할 수 없으면 시작이 즉시 실패합니다.
- reload는 원자적 교체를 사용합니다. 전체 성공이거나 마지막 정상 스냅샷을 유지합니다.
- 런타임 요청은 활성 메모리 스냅샷만 읽습니다.
- outbound delivery 경로도 이 활성 스냅샷만 읽습니다 (예: Discord reply/thread delivery, Telegram action send). 전송마다 SecretRef를 다시 해석하지 않습니다.

## 활성 surface 필터링

SecretRef는 실제로 활성인 surface에서만 검증됩니다.

- 활성 surface: 해석 불가 ref는 시작/reload를 막습니다.
- 비활성 surface: 해석 불가 ref는 시작/reload를 막지 않습니다.
- 비활성 ref는 `SECRETS_REF_IGNORED_INACTIVE_SURFACE` 진단만 남깁니다.

예:

- 비활성화된 채널/계정
- 어떤 활성 계정도 상속하지 않는 상위 채널 자격 증명
- 비활성 도구/기능 surface
- `tools.web.search.provider`로 선택되지 않은 web search 프로바이더 키

## Gateway 인증 surface 진단

`gateway.auth.password`, `gateway.remote.token`, `gateway.remote.password`에 SecretRef가 설정되면, 시작과 reload 시 해당 surface 상태를 명시적으로 기록합니다.

- `active`: 실제 인증 surface에 포함되며 반드시 해석돼야 함
- `inactive`: 다른 인증 surface가 우선하거나 remote auth가 비활성이라 이번 런타임에서 무시됨

로그 코드는 `SECRETS_GATEWAY_AUTH_SURFACE`입니다.

## 온보딩 preflight

대화형 온보딩에서 SecretRef 저장을 선택하면 저장 전에 preflight 검증을 수행합니다.

- env ref: 환경 변수 이름과 비어 있지 않은 값을 확인
- provider ref(`file`, `exec`): provider 선택, `id`, 해석된 값 타입 확인

실패하면 오류를 보여주고 다시 시도할 수 있습니다.

## SecretRef 계약

모든 곳에서 동일한 객체 형태를 사용합니다.

```json5
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

### `source: "env"`

```json5
{ source: "env", provider: "default", id: "OPENAI_API_KEY" }
```

- `provider`: `^[a-z][a-z0-9_-]{0,63}$`
- `id`: `^[A-Z][A-Z0-9_]{0,127}$`

### `source: "file"`

```json5
{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }
```

- `provider`: `^[a-z][a-z0-9_-]{0,63}$`
- `id`: 절대 JSON pointer (`/...`)

### `source: "exec"`

```json5
{ source: "exec", provider: "vault", id: "providers/openai/apiKey" }
```

- `provider`: `^[a-z][a-z0-9_-]{0,63}$`
- `id`: `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$`

## Provider 구성

`secrets.providers` 아래에 provider를 정의합니다.

```json5
{
  secrets: {
    providers: {
      default: { source: "env" },
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json",
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        args: ["--profile", "prod"],
        passEnv: ["PATH", "VAULT_ADDR"],
        jsonOnly: true,
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
  },
}
```

### Env provider

- 선택적 allowlist 지원
- 값이 없거나 비어 있으면 해석 실패

### File provider

- `path`에서 로컬 파일 읽기
- `mode: "json"`은 JSON 객체에서 pointer로 해석
- `mode: "singleValue"`는 `id: "value"`를 기대하고 파일 전체 내용을 반환

### Exec provider

- 셸 없이 절대 경로 바이너리를 실행
- 기본적으로 일반 파일만 허용하며 심볼릭 링크는 불가
- `allowSymlinkCommand: true`로 Homebrew shim 같은 경로 허용 가능
- timeout, output 바이트 제한, env allowlist, trusted dirs를 지원

stdin 요청 예:

```json
{ "protocolVersion": 1, "provider": "vault", "ids": ["providers/openai/apiKey"] }
```

stdout 응답 예:

```json
{ "protocolVersion": 1, "values": { "providers/openai/apiKey": "sk-..." } }
```

## Exec 통합 예시

다음 CLI와 쉽게 연동할 수 있습니다.

- 1Password CLI
- HashiCorp Vault CLI
- `sops`

## 지원되는 자격 증명 surface

표준적으로 지원/비지원되는 자격 증명 목록은 다음 문서에 정리되어 있습니다.

- [SecretRef Credential Surface](/ko-KR/reference/secretref-credential-surface)

회전형 자격 증명이나 OAuth refresh material처럼 런타임에서 생성되는 값은 의도적으로 제외됩니다.

## 요구 동작과 우선순위

- ref가 없는 필드는 그대로 둡니다.
- ref가 있는 필드는 활성 surface에서 activation 시 필수입니다.
- 평문과 ref가 둘 다 있으면, 지원되는 우선순위 경로에서는 ref가 우선합니다.

경고와 audit 시그널:

- `SECRETS_REF_OVERRIDES_PLAINTEXT`
- `REF_SHADOWED`

Google Chat 호환 동작:

- `serviceAccountRef`가 평문 `serviceAccount`보다 우선합니다.

## Activation 트리거

시크릿 activation은 다음에서 실행됩니다.

- 시작 시
- config reload hot-apply 경로
- config reload restart-check 경로
- 수동 `secrets.reload`

계약:

- 성공 시 스냅샷을 원자적으로 교체
- 시작 실패 시 게이트웨이 시작 중단
- 런타임 reload 실패 시 마지막 정상 스냅샷 유지

## 저하 및 복구 시그널

reload 시 activation이 실패하면 OpenClaw는 degraded secrets 상태로 들어갑니다.

- `SECRETS_RELOADER_DEGRADED`
- `SECRETS_RELOADER_RECOVERED`

동작:

- degraded: 마지막 정상 스냅샷 유지
- recovered: 다음 성공 activation 이후 한 번만 발생

## Command-path 해석

원격 메모리 경로나 `openclaw qr --remote`처럼 opt-in한 명령 경로는 게이트웨이 스냅샷 RPC를 통해 지원되는 SecretRef를 해석할 수 있습니다.

- 게이트웨이가 실행 중이면 활성 스냅샷을 읽습니다.
- 필요한 SecretRef가 있지만 게이트웨이가 없으면 즉시 실패하고 진단을 제공합니다.
- 백엔드 시크릿 회전 후 스냅샷 갱신은 `openclaw secrets reload`로 처리합니다.

관련 RPC 메서드:

- `secrets.resolve`

## Audit 및 configure 워크플로

기본 운영 흐름:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

### `secrets audit`

다음 항목을 찾습니다.

- 평문 값(`openclaw.json`, `auth-profiles.json`, `.env`)
- 해석되지 않는 ref
- 우선순위 shadowing
- 레거시 잔여물(`auth.json`, OAuth reminder)

### `secrets configure`

대화형 도우미가 다음을 수행합니다.

- `secrets.providers`를 먼저 설정
- `openclaw.json`과 `auth-profiles.json`의 지원 대상 필드 선택
- SecretRef 상세값(`source`, `provider`, `id`) 수집
- preflight 해석 실행
- 즉시 적용 가능

유용한 모드:

- `openclaw secrets configure --providers-only`
- `openclaw secrets configure --skip-provider-setup`
- `openclaw secrets configure --agent <id>`

### `secrets apply`

저장된 계획 적용:

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
```

엄격한 path 계약은 다음 문서를 참조하세요.

- [Secrets Apply Plan Contract](/ko-KR/gateway/secrets-plan-contract)

## 단방향 안전 정책

OpenClaw는 과거 평문 시크릿 값을 담는 rollback 백업을 의도적으로 작성하지 않습니다.

- write 모드 전에 preflight가 성공해야 함
- 커밋 전에 런타임 activation이 검증됨
- 적용은 원자적 파일 교체와 best-effort 복구를 사용

## 레거시 인증 호환 참고

정적 자격 증명의 경우 런타임은 더 이상 평문 레거시 auth 저장소에 의존하지 않습니다.

- 런타임 자격 증명 소스는 해석된 메모리 스냅샷
- 레거시 정적 `api_key` 항목은 발견 시 스크럽
- OAuth 관련 호환성은 별도 처리

## 관련 문서

- [CLI `secrets`](/ko-KR/cli/secrets)
- [Secrets Apply Plan Contract](/ko-KR/gateway/secrets-plan-contract)
- [SecretRef Credential Surface](/ko-KR/reference/secretref-credential-surface)
- [Authentication](/ko-KR/gateway/authentication)
- [Security](/ko-KR/gateway/security)
- [Environment Variables](/ko-KR/help/environment)
