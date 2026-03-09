---
summary: "ACP 바인딩 대화용 장기 명령 권한 부여 모델 제안"
read_when:
  - Telegram/Discord의 ACP 바인딩 채널/토픽에서 네이티브 명령어 인증 동작을 설계할 때
title: "ACP 바인딩 명령 권한 부여 (제안)"
---

# ACP 바인딩 명령 권한 부여 (제안)

상태: 제안됨, **아직 구현되지 않음**.

이 문서는 ACP 바인딩 대화에서 네이티브 명령어에 대한 장기 권한 부여 모델을 설명합니다. 실험용 제안 문서이며, 현재 운영 동작을 대체하지 않습니다.

구현된 현재 동작은 다음 소스와 테스트를 참조하십시오.

- `src/telegram/bot-native-commands.ts`
- `src/discord/monitor/native-command.ts`
- `src/auto-reply/reply/commands-core.ts`

## 문제

현재는 `/new`, `/reset` 같은 명령어별 검사 로직이 있고, allowlist가 비어 있더라도 ACP 바인딩 채널/토픽 안에서는 동작해야 합니다.

이 방식은 당장의 UX 문제는 해결하지만, 명령어 이름별 예외 처리는 확장성이 없습니다.

## 장기적인 형태

명령 권한 부여를 ad-hoc 핸들러 로직에서 명령 메타데이터 + 공용 정책 평가기로 옮깁니다.

### 1) 명령 정의에 auth policy 메타데이터 추가

각 명령 정의는 auth policy를 선언해야 합니다. 예시 형태:

```ts
type CommandAuthPolicy =
  | { mode: "owner_or_allowlist" } // 기본값, 현재의 엄격한 동작
  | { mode: "bound_acp_or_owner_or_allowlist" } // 명시적으로 바인딩된 ACP 대화에서는 허용
  | { mode: "owner_only" };
```

`/new`와 `/reset`은 `bound_acp_or_owner_or_allowlist`를 사용합니다.
대부분의 다른 명령어는 계속 `owner_or_allowlist`를 유지합니다.

### 2) 채널 간 공용 evaluator 사용

다음 요소를 기준으로 명령 auth를 평가하는 helper 하나를 도입합니다.

- 명령 policy 메타데이터
- 발신자 권한 상태
- 해석된 대화 binding 상태

Telegram과 Discord의 네이티브 핸들러 모두 같은 helper를 호출해 동작 드리프트를 막아야 합니다.

### 3) 바인딩 매치를 우회 경계로 사용

정책이 bound ACP bypass를 허용하더라도, 현재 대화에 대해 설정된 binding match가 실제로 해석된 경우에만 권한을 부여합니다. 단지 현재 session key가 ACP처럼 보인다는 이유만으로는 허용하지 않습니다.

이렇게 하면 경계가 명시적으로 유지되고, 실수로 권한 범위가 넓어지는 일을 최소화할 수 있습니다.

## 왜 더 나은가

- 앞으로 명령어가 늘어나도 명령 이름별 조건문을 더 붙일 필요가 없습니다.
- 채널 간 동작을 일관되게 유지할 수 있습니다.
- 명시적 binding match를 요구하므로 현재 보안 모델을 유지합니다.
- allowlist는 보편적 필수가 아니라 선택적 강화 수단으로 남습니다.

## 롤아웃 계획 (향후)

1. 명령 레지스트리 타입과 명령 데이터에 command auth policy 필드를 추가
2. 공용 evaluator를 구현하고 Telegram + Discord 네이티브 핸들러를 마이그레이션
3. `/new`, `/reset`을 메타데이터 기반 정책으로 전환
4. 정책 모드별, 채널 표면별 테스트 추가

## 비목표

- 이 제안은 ACP 세션 라이프사이클 동작을 변경하지 않습니다.
- 모든 ACP 바인딩 명령어에 allowlist를 강제하지 않습니다.
- 기존 route binding 의미론을 바꾸지 않습니다.

## 참고

이 제안은 의도적으로 additive하며, 기존 실험 문서를 삭제하거나 대체하지 않습니다.
