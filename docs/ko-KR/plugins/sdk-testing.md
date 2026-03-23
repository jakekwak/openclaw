---
title: "Plugin Testing"
sidebarTitle: "Testing"
summary: "OpenClaw plugin용 테스트 유틸리티와 패턴"
read_when:
  - plugin 테스트를 작성할 때
  - plugin SDK의 테스트 유틸리티가 필요할 때
  - 번들 plugin용 contract test를 이해하려 할 때
---

# Plugin Testing

OpenClaw plugin용 테스트 유틸리티, 패턴, lint enforcement를 정리한 레퍼런스입니다.

<Tip>
  실제 예제는 [Channel plugin tests](/ko-KR/plugins/sdk-channel-plugins#step-6-test)와 [Provider plugin tests](/ko-KR/plugins/sdk-provider-plugins#step-6-test)를 보세요.
</Tip>

## 테스트 유틸리티

**Import:** `openclaw/plugin-sdk/testing`

```typescript
import {
  installCommonResolveTargetErrorCases,
  shouldAckReaction,
  removeAckReactionAfterReply,
} from "openclaw/plugin-sdk/testing";
```

### 제공 항목

| Export                                 | Purpose                                        |
| -------------------------------------- | ---------------------------------------------- |
| `installCommonResolveTargetErrorCases` | target resolution 오류 처리 공통 테스트 케이스 |
| `shouldAckReaction`                    | 채널이 ack reaction을 추가해야 하는지 판단     |
| `removeAckReactionAfterReply`          | reply 전송 후 ack reaction 제거                |

테스트 파일에서 유용한 타입도 함께 re-export됩니다.

## 대표 테스트 패턴

### Channel plugin 단위 테스트

- config에서 account를 올바르게 해석하는지
- `inspectAccount`가 secret 값을 materialize하지 않는지
- 누락된 config를 적절히 보고하는지

### Provider plugin 단위 테스트

- dynamic model resolution이 기대한 모델 ID/provider/api를 반환하는지
- API 키가 있을 때 catalog가 정상 반환되는지

### Runtime mocking

`createPluginRuntimeStore`를 쓰는 코드라면 테스트에서 `PluginRuntime` mock을 주입해 검증하는 패턴을 권장합니다.

### Per-instance stub

prototype mutation보다 인스턴스별 stub을 권장합니다:

```typescript
const client = new MyChannelClient();
client.sendMessage = vi.fn().mockResolvedValue({ id: "msg-1" });
```

## Contract test

저장소 내부 번들 plugin은 contract test로 registration ownership을 검증합니다:

```bash
pnpm test -- src/plugins/contracts/
```

보통 다음을 확인합니다:

- 어떤 plugin이 어떤 provider를 등록하는지
- speech/media/image/web search ownership
- registration shape
- runtime contract 준수

## Lint enforcement

저장소 내부 plugin에는 다음 규칙이 강제됩니다:

1. monolithic root import 금지 (`openclaw/plugin-sdk`)
2. 직접 `src/` import 금지
3. 자기 plugin의 `plugin-sdk/<name>` self-import 금지

외부 plugin에는 강제되지 않지만 같은 패턴을 따르는 것이 좋습니다.

## 테스트 실행

```bash
pnpm test
pnpm test -- extensions/my-channel/
pnpm test -- extensions/my-channel/ -t "resolves account"
pnpm test:coverage
```

메모리 압박이 있으면:

```bash
OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test
```

## 관련 문서

- [SDK Overview](/ko-KR/plugins/sdk-overview)
- [Building Channel Plugins](/ko-KR/plugins/sdk-channel-plugins)
- [Building Provider Plugins](/ko-KR/plugins/sdk-provider-plugins)
