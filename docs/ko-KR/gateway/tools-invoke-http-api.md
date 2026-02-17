---
summary: "게이트웨이 HTTP 엔드포인트를 통해 단일 도구를 직접 호출"
read_when:
  - 전체 에이전트 턴을 실행하지 않고 도구 호출
  - 도구 정책 집행이 필요한 자동화 구축
title: "Tools Invoke API"
---

# Tools Invoke (HTTP)

OpenClaw의 게이트웨이는 단일 도구를 직접 호출하기 위한 간단한 HTTP 엔드포인트를 제공합니다. 항상 활성화되어 있으며 게이트웨이 인증 및 도구 정책에 의해 제어됩니다.

- `POST /tools/invoke`
- 게이트웨이와 동일한 포트 (WS + HTTP 멀티플렉스): `http://<gateway-host>:<port>/tools/invoke`

기본 최대 페이로드 크기는 2 MB입니다.

## Authentication

게이트웨이 인증 구성을 사용합니다. 베어러 토큰을 보내십시오:

- `Authorization: Bearer <token>`

주의 사항:

- `gateway.auth.mode="token"`일 때, `gateway.auth.token` (또는 `OPENCLAW_GATEWAY_TOKEN`)을 사용하십시오.
- `gateway.auth.mode="password"`일 때, `gateway.auth.password` (또는 `OPENCLAW_GATEWAY_PASSWORD`)를 사용하십시오.
- 만약 `gateway.auth.rateLimit`가 설정되어 있고 너무 많은 인증 실패가 발생하면, 엔드포인트는 `429` (재시도 대기 시간 설정) 응답을 반환합니다.

## Request body

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Fields:

- `tool` (string, required): 호출할 도구 이름.
- `action` (string, optional): 도구 스키마가 `action`을 지원하며 args 페이로드가 이를 생략했을 때 args에 맵핑됩니다.
- `args` (object, optional): 도구별 인수.
- `sessionKey` (string, optional): 대상 세션 키. 생략되거나 `"main"`인 경우, 게이트웨이는 설정된 주 세션 키를 사용합니다 (`session.mainKey` 및 기본 에이전트, 또는 글로벌 범위에서 `global`을 준수).
- `dryRun` (boolean, optional): 향후 사용을 위해 예약됨; 현재는 무시됩니다.

## Policy + routing behavior

도구 가용성은 게이트웨이 에이전트에 의해 사용되는 동일한 정책 체인을 통해 필터링됩니다:

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- 그룹 정책 (세션 키가 그룹 또는 채널에 맵핑될 경우)
- 하위 에이전트 정책 (하위 에이전트 세션 키로 호출할 때)

정책에 의해 도구가 허용되지 않으면, 엔드포인트는 **404**를 반환합니다.

게이트웨이 HTTP는 또한 기본으로 하드 거부 목록을 적용합니다 (세션 정책이 도구를 허용하더라도):

- `sessions_spawn`
- `sessions_send`
- `gateway`
- `whatsapp_login`

이 거부 목록은 `gateway.tools`를 통해 사용자 지정할 수 있습니다:

```json5
{
  gateway: {
    tools: {
      // HTTP /tools/invoke에서 차단할 추가 도구
      deny: ["browser"],
      // 기본 거부 목록에서 도구 제거
      allow: ["gateway"],
    },
  },
}
```

그룹 정책이 컨텍스트를 해석할 수 있도록, 다음을 선택적으로 설정할 수 있습니다:

- `x-openclaw-message-channel: <channel>` (예: `slack`, `telegram`)
- `x-openclaw-account-id: <accountId>` (여러 계정이 존재할 경우)

## Responses

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }` (잘못된 요청 또는 도구 입력 오류)
- `401` → 인증되지 않음
- `429` → 인증 속도 제한됨 (`Retry-After` 설정됨)
- `404` → 도구 가용하지 않음 (찾을 수 없거나 허용 목록에 없음)
- `405` → 허용되지 않는 메소드
- `500` → `{ ok: false, error: { type, message } }` (예상치 못한 도구 실행 오류; 메시지 정화됨)

## Example

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
