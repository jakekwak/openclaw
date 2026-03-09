---
summary: "reload, audit, configure, apply를 위한 `openclaw secrets` CLI 참조"
read_when:
  - 런타임에서 secret ref를 다시 해석할 때
  - 평문 잔여물과 미해결 ref를 감사할 때
  - SecretRef를 구성하고 일방향 scrub 변경을 적용할 때
title: "secrets"
---

# `openclaw secrets`

`openclaw secrets`는 SecretRef를 관리하고 활성 런타임 스냅샷을 건강하게 유지하는 데 사용합니다.

명령 역할:

- `reload`: secret ref를 다시 해석하고 전체 성공 시에만 런타임 스냅샷을 교체하는 게이트웨이 RPC (`secrets.reload`)입니다. 설정 파일을 쓰지 않습니다.
- `audit`: 구성/auth/생성된 모델 저장소와 레거시 잔여물을 읽기 전용으로 스캔하여 평문, 미해결 ref, 우선순위 drift를 찾습니다.
- `configure`: 프로바이더 설정, 대상 매핑, 사전검사를 위한 대화형 planner입니다 (TTY 필요).
- `apply`: 저장된 계획을 실행합니다 (`--dry-run`은 검증만 수행). 이후 지정된 평문 잔여물을 scrub합니다.

권장 운영 루프:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets audit --check
openclaw secrets reload
```

CI/게이트용 종료 코드 참고:

- `audit --check`는 발견 사항이 있으면 `1`을 반환합니다.
- 미해결 ref는 `2`를 반환합니다.

관련 문서:

- 비밀 관리 가이드: [Secrets Management](/ko-KR/gateway/secrets)
- 자격 증명 표면: [SecretRef Credential Surface](/ko-KR/reference/secretref-credential-surface)
- 보안 가이드: [Security](/ko-KR/gateway/security)

## 런타임 스냅샷 다시 불러오기

secret ref를 다시 해석하고 런타임 스냅샷을 원자적으로 교체합니다.

```bash
openclaw secrets reload
openclaw secrets reload --json
```

주의:

- 게이트웨이 RPC 메서드 `secrets.reload`를 사용합니다.
- 해석에 실패하면 게이트웨이는 마지막으로 정상 동작하던 스냅샷을 유지하고 오류를 반환합니다 (부분 활성화 없음).
- JSON 응답에는 `warningCount`가 포함됩니다.

## 감사

OpenClaw 상태를 다음 항목에 대해 스캔합니다:

- 평문 비밀 저장
- 미해결 ref
- 우선순위 drift (`auth-profiles.json` 자격 증명이 `openclaw.json` ref를 가리는 경우)
- 생성된 `agents/*/agent/models.json` 잔여물 (provider `apiKey` 값과 민감한 provider header)
- 레거시 잔여물 (legacy auth store 항목, OAuth reminder)

Header 잔여물 참고:

- 민감한 provider header 탐지는 이름 휴리스틱 기반입니다 (`authorization`, `x-api-key`, `token`, `secret`, `password`, `credential` 같은 공통 auth/credential header 이름과 조각).

```bash
openclaw secrets audit
openclaw secrets audit --check
openclaw secrets audit --json
```

종료 동작:

- `--check`는 발견 사항이 있으면 non-zero로 종료합니다.
- 미해결 ref는 더 높은 우선순위의 non-zero 종료 코드를 사용합니다.

보고서 주요 형식:

- `status`: `clean | findings | unresolved`
- `summary`: `plaintextCount`, `unresolvedRefCount`, `shadowedRefCount`, `legacyResidueCount`
- finding code:
  - `PLAINTEXT_FOUND`
  - `REF_UNRESOLVED`
  - `REF_SHADOWED`
  - `LEGACY_RESIDUE`
