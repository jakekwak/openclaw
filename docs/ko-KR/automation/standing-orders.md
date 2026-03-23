---
summary: "자율 에이전트 프로그램을 위한 상시 운영 권한 정의"
read_when:
  - 작업마다 별도 프롬프트 없이 실행되는 자율 에이전트 워크플로를 설정할 때
  - 에이전트가 독립적으로 할 수 있는 일과 사람 승인이 필요한 일을 구분할 때
  - 경계와 에스컬레이션 규칙이 분명한 멀티 프로그램 에이전트를 설계할 때
title: "상시 지시"
---

# 상시 지시

상시 지시는 에이전트에게 정의된 프로그램에 대한 **상시 운영 권한**을 부여합니다. 매번 개별 작업 지시를 내리는 대신, 명확한 범위, 트리거, 에스컬레이션 규칙이 있는 프로그램을 정의하고 에이전트는 그 경계 안에서 자율적으로 실행합니다.

이것은 매주 금요일마다 비서에게 "주간 보고서 보내"라고 지시하는 것과, 상시 권한을 부여하는 것의 차이입니다: "주간 보고서는 네 책임이다. 매주 금요일에 취합하고 발송하고, 뭔가 이상해 보일 때만 에스컬레이션해라."

## 왜 상시 지시인가

**상시 지시가 없으면:**

- 모든 작업마다 에이전트에 프롬프트를 보내야 합니다
- 에이전트는 요청 사이에 유휴 상태로 머뭅니다
- 반복 업무가 잊히거나 지연됩니다
- 사용자가 병목이 됩니다

**상시 지시가 있으면:**

- 에이전트가 정의된 경계 안에서 자율적으로 실행합니다
- 반복 업무가 프롬프트 없이 일정대로 처리됩니다
- 예외와 승인에만 개입하면 됩니다
- 에이전트가 유휴 시간을 생산적으로 활용합니다

## 동작 방식

상시 지시는 [에이전트 워크스페이스](/ko-KR/concepts/agent-workspace) 파일에 정의합니다. 권장 방식은 모든 세션에 자동 주입되는 `AGENTS.md`에 직접 포함하는 것입니다. 이렇게 하면 에이전트가 항상 해당 지시를 컨텍스트에 포함한 채 시작합니다. 구성이 더 커지면 `standing-orders.md` 같은 전용 파일에 두고 `AGENTS.md`에서 참조할 수도 있습니다.

각 프로그램은 다음을 지정합니다:

1. **범위**: 에이전트가 수행하도록 승인된 작업
2. **트리거**: 언제 실행할지 (스케줄, 이벤트, 조건)
3. **승인 게이트**: 실행 전에 사람 승인이 필요한 항목
4. **에스컬레이션 규칙**: 언제 멈추고 도움을 요청할지

에이전트는 매 세션마다 워크스페이스 bootstrap 파일을 통해 이 지시를 읽어 들이며 ([Agent Workspace](/ko-KR/concepts/agent-workspace)에서 자동 주입 파일 전체 목록 확인), 시간 기반 실행은 [크론 작업](/ko-KR/automation/cron-jobs)과 결합해 적용됩니다.

<Tip>
상시 지시는 `AGENTS.md`에 넣어 두는 것이 좋습니다. 그래야 모든 세션에서 확실히 로드됩니다. 워크스페이스 bootstrap은 `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`, `MEMORY.md`를 자동 주입하지만, 하위 디렉터리의 임의 파일은 자동으로 넣지 않습니다.
</Tip>

## 상시 지시의 구조

```markdown
## Program: Weekly Status Report

**Authority:** Compile data, generate report, deliver to stakeholders
**Trigger:** Every Friday at 4 PM (enforced via cron job)
**Approval gate:** None for standard reports. Flag anomalies for human review.
**Escalation:** If data source is unavailable or metrics look unusual (>2σ from norm)

### Execution Steps

1. Pull metrics from configured sources
2. Compare to prior week and targets
3. Generate report in Reports/weekly/YYYY-MM-DD.md
4. Deliver summary via configured channel
5. Log completion to Agent/Logs/

### What NOT to Do

- Do not send reports to external parties
- Do not modify source data
- Do not skip delivery if metrics look bad — report accurately
```

## 상시 지시 + 크론 작업

상시 지시는 에이전트가 **무엇을** 해도 되는지 정의합니다. [크론 작업](/ko-KR/automation/cron-jobs)은 **언제** 실행되는지를 정의합니다. 둘은 함께 동작합니다:

```text
Standing Order: "You own the daily inbox triage"
    ↓
Cron Job (8 AM daily): "Execute inbox triage per standing orders"
    ↓
Agent: Reads standing orders → executes steps → reports results
```

크론 작업 프롬프트는 상시 지시를 복제하기보다 참조만 해야 합니다:

```bash
openclaw cron add \
  --name daily-inbox-triage \
  --cron "0 8 * * 1-5" \
  --tz America/New_York \
  --timeout-seconds 300 \
  --announce \
  --channel bluebubbles \
  --to "+1XXXXXXXXXX" \
  --message "Execute daily inbox triage per standing orders. Check mail for new alerts. Parse, categorize, and persist each item. Report summary to owner. Escalate unknowns."
```

## 예시

### 예시 1: 콘텐츠 및 소셜 미디어 (주간 사이클)

```markdown
## Program: Content & Social Media

**Authority:** Draft content, schedule posts, compile engagement reports
**Approval gate:** All posts require owner review for first 30 days, then standing approval
**Trigger:** Weekly cycle (Monday review → mid-week drafts → Friday brief)

### Weekly Cycle

- **Monday:** Review platform metrics and audience engagement
- **Tuesday–Thursday:** Draft social posts, create blog content
- **Friday:** Compile weekly marketing brief → deliver to owner

### Content Rules

- Voice must match the brand (see SOUL.md or brand voice guide)
- Never identify as AI in public-facing content
- Include metrics when available
- Focus on value to audience, not self-promotion
```

### 예시 2: 재무 운영 (이벤트 트리거)

```markdown
## Program: Financial Processing

**Authority:** Process transaction data, generate reports, send summaries
**Approval gate:** None for analysis. Recommendations require owner approval.
**Trigger:** New data file detected OR scheduled monthly cycle

### When New Data Arrives

1. Detect new file in designated input directory
2. Parse and categorize all transactions
3. Compare against budget targets
4. Flag: unusual items, threshold breaches, new recurring charges
5. Generate report in designated output directory
6. Deliver summary to owner via configured channel

### Escalation Rules

- Single item > $500: immediate alert
- Category > budget by 20%: flag in report
- Unrecognizable transaction: ask owner for categorization
- Failed processing after 2 retries: report failure, do not guess
```

### 예시 3: 모니터링 및 알림 (연속 실행)

```markdown
## Program: System Monitoring

**Authority:** Check system health, restart services, send alerts
**Approval gate:** Restart services automatically. Escalate if restart fails twice.
**Trigger:** Every heartbeat cycle

### Checks

- Service health endpoints responding
- Disk space above threshold
- Pending tasks not stale (>24 hours)
- Delivery channels operational

### Response Matrix

| Condition        | Action                   | Escalate?                |
| ---------------- | ------------------------ | ------------------------ |
| Service down     | Restart automatically    | Only if restart fails 2x |
| Disk space < 10% | Alert owner              | Yes                      |
| Stale task > 24h | Remind owner             | No                       |
| Channel offline  | Log and retry next cycle | If offline > 2 hours     |
```

## Execute-Verify-Report 패턴

상시 지시는 엄격한 실행 규율과 결합될 때 가장 잘 작동합니다. 상시 지시 안의 모든 작업은 다음 루프를 따라야 합니다:

1. **Execute**: 실제 작업을 수행합니다 (지시를 확인만 하지 않습니다)
2. **Verify**: 결과가 올바른지 확인합니다 (파일 존재, 메시지 전달, 데이터 파싱 등)
3. **Report**: 무엇을 했고 무엇을 검증했는지 소유자에게 보고합니다

```markdown
### Execution Rules

- Every task follows Execute-Verify-Report. No exceptions.
- "I'll do that" is not execution. Do it, then report.
- "Done" without verification is not acceptable. Prove it.
- If execution fails: retry once with adjusted approach.
- If still fails: report failure with diagnosis. Never silently fail.
- Never retry indefinitely — 3 attempts max, then escalate.
```

이 패턴은 가장 흔한 에이전트 실패 방식, 즉 작업을 끝내지 않았는데도 수행했다고만 응답하는 문제를 막아 줍니다.

## 멀티 프로그램 아키텍처

여러 관심사를 다루는 에이전트라면, 상시 지시를 경계가 분명한 별도 프로그램으로 나누어 구성하세요:

```markdown
# Standing Orders

## Program 1: [Domain A] (Weekly)

...

## Program 2: [Domain B] (Monthly + On-Demand)

...

## Program 3: [Domain C] (As-Needed)

...

## Escalation Rules (All Programs)

- [Common escalation criteria]
- [Approval gates that apply across programs]
```

이렇게 하면 하나의 거대한 지시 파일이 되는 것을 피하고, 각 프로그램의 책임과 승인 기준을 더 명확하게 유지할 수 있습니다.
