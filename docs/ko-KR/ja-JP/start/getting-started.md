```markdown
---
read_when:
  - 처음으로 설치할 때
  - 작동하는 채팅으로 가는 가장 빠른 방법을 알고 싶을 때
summary: OpenClaw를 설치하고 몇 분 안에 첫 채팅을 실행하세요.
title: 시작하기
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# 시작하기

목표: 최소한의 설정으로 처음부터 작동하는 채팅을 구현하십시오.

<Info>
가장 빠른 채팅 방법: Control UI를 엽니다 (채널 설정 불필요). `openclaw dashboard`를 실행하여 브라우저에서 채팅하거나, <Tooltip headline="게이트웨이 호스트" tip="OpenClaw 게이트웨이 서비스를 실행하는 머신입니다.">게이트웨이 호스트</Tooltip>에서 `http://127.0.0.1:18789/`를 엽니다.
문서: [대시보드](/web/dashboard)와 [Control UI](/web/control-ui).
</Info>

## 전제 조건

- Node 22 이상

<Tip>
확실하지 않다면 `node --version`을 통해 Node 버전을 확인하세요.
</Tip>

## 빠른 설정 (CLI)

<Steps>
  <Step title="OpenClaw 설치 (권장)">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      </Tab>
    </Tabs>

    <Note>
    다른 설치 방법과 요구 사항: [설치](/install).
    </Note>

  </Step>
  <Step title="온보딩 마법사 실행">
    ```bash
    openclaw onboard --install-daemon
    ```

    마법사는 인증, 게이트웨이 설정 및 옵션 채널을 구성합니다.
    자세한 내용은 [온보딩 마법사](/start/wizard)를 참조하세요.

  </Step>
  <Step title="게이트웨이 확인">
    서비스를 설치했다면 이미 실행 중일 것입니다:

    ```bash
    openclaw gateway status
    ```

  </Step>
  <Step title="Control UI 열기">
    ```bash
    openclaw dashboard
    ```
  </Step>
</Steps>

<Check>
Control UI가 로드되면, 게이트웨이가 사용 가능한 상태입니다.
</Check>

## 옵션 확인 및 추가 기능

<AccordionGroup>
  <Accordion title="포어그라운드에서 게이트웨이 실행">
    빠른 테스트나 문제 해결에 유용합니다.

    ```bash
    openclaw gateway --port 18789
    ```

  </Accordion>
  <Accordion title="테스트 메시지 전송">
    구성된 채널이 필요합니다.

    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```

  </Accordion>
</AccordionGroup>

## 더 알아보기

<Columns>
  <Card title="온보딩 마법사 (상세)" href="/start/wizard">
    전체 CLI 마법사 참조와 고급 옵션.
  </Card>
  <Card title="macOS 앱 온보딩" href="/start/onboarding">
    macOS 앱의 첫 실행 흐름.
  </Card>
</Columns>

## 완료 후 상태

- 실행 중인 게이트웨이
- 구성된 인증
- Control UI 접근 또는 연결된 채널

## 다음 단계

- 다이렉트 메시지의 안전성과 승인: [페어링](/channels/pairing)
- 더 많은 채널 연결: [채널](/channels)
- 고급 워크플로우와 소스에서 빌드: [설정](/start/setup)
```