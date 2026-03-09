---
summary: "`secrets apply` 계획 파일의 타깃 검증, 경로 매칭, `auth-profiles.json` 범위 계약"
read_when:
  - `openclaw secrets apply` 계획을 생성하거나 검토할 때
  - `Invalid plan target path` 오류를 디버깅할 때
  - 타깃 타입과 경로 검증 동작을 이해해야 할 때
title: "Secrets Apply Plan Contract"
---

# Secrets apply 계획 계약

이 페이지는 `openclaw secrets apply`가 강제하는 엄격한 계약을 정의합니다.

타깃이 이 규칙과 맞지 않으면, 구성 파일을 바꾸기 전에 apply가 실패합니다.

## 계획 파일 형태

`openclaw secrets apply --from <plan.json>`은 `targets` 배열을 기대합니다.

지원되는 자격 증명 경로:

- [SecretRef Credential Surface](/ko-KR/reference/secretref-credential-surface)

## 타깃 타입 동작

일반 규칙:

- `target.type`은 인식 가능한 값이어야 하며 정규화된 `target.path` 형태와 일치해야 합니다.

기존 계획과의 호환 별칭:

- `models.providers.apiKey`
- `skills.entries.apiKey`
- `channels.googlechat.serviceAccount`

## 경로 검증 규칙

각 타깃은 다음을 모두 통과해야 합니다.

- `type`은 인식 가능한 타깃 타입이어야 함
- `path`는 비어 있지 않은 dot path여야 함
- `pathSegments`가 있으면 `path`와 정확히 같은 정규화 결과여야 함
- 금지 세그먼트(`__proto__`, `prototype`, `constructor`)는 거부
- 정규화된 path는 등록된 target type 형태와 일치해야 함
- `providerId`/`accountId`가 있으면 path 안의 ID와 일치해야 함
- `auth-profiles.json` 타깃은 `agentId`가 필요함
- 새 `auth-profiles.json` 매핑 생성 시 `authProfileProvider`를 포함해야 함

## 실패 동작

타깃 검증이 실패하면 다음과 같은 오류와 함께 종료됩니다.

```text
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

잘못된 계획에 대해서는 아무 쓰기도 커밋되지 않습니다.

## 운영자 체크

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
```

적용이 실패하면 `openclaw secrets configure`로 계획을 다시 만들거나, 위에서 정의한 지원 경로에 맞게 target path를 수정하세요.

## 관련 문서

- [Secrets Management](/ko-KR/gateway/secrets)
- [CLI `secrets`](/ko-KR/cli/secrets)
- [SecretRef Credential Surface](/ko-KR/reference/secretref-credential-surface)
- [Configuration Reference](/ko-KR/gateway/configuration-reference)
