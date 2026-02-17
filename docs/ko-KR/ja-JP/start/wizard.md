```yaml
---
read_when:
  - 온보딩 마법사를 실행하거나 설정할 때
  - 새로운 머신을 설정할 때
sidebarTitle: 마법사 (CLI)
summary: CLI 온보딩 마법사: 게이트웨이, 워크스페이스, 채널, 스킬의 대화형 설정
title: 온보딩 마법사 (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# 온보딩 마법사 (CLI)

CLI 온보딩 마법사는 macOS, Linux, Windows (WSL2 경유)에서 OpenClaw를 설정할 때 권장되는 경로입니다. 로컬 게이트웨이 또는 원격 게이트웨이 연결에 추가하여 워크스페이스의 기본 설정, 채널, 스킬을 구성합니다.

```bash
openclaw onboard
```

<Info>
가장 빠르게 첫 채팅을 시작하는 방법: Control UI를 엽니다 (채널 설정은 필요하지 않음). `openclaw dashboard`를 실행하여 브라우저에서 채팅할 수 있습니다. 문서: [Dashboard](/web/dashboard).
</Info>

## 빠른 시작 vs 상세 설정

마법사는 **빠른 시작**(기본 설정)과 **상세 설정**(완전한 제어) 중 하나를 선택하여 시작합니다.

<Tabs>
  <Tab title="빠른 시작 (기본 설정)">
    - 루프백 상의 로컬 게이트웨이
    - 기존 워크스페이스 또는 기본 워크스페이스
    - 게이트웨이 포트 `18789`
    - 게이트웨이 인증 토큰은 자동 생성 (루프백 상에서도 생성)
    - Tailscale 공개는 오프
    - Telegram과 WhatsApp의 다이렉트 메시지는 기본으로 허용 목록 (전화번호 입력을 요청받을 수 있음)
  </Tab>
  <Tab title="상세 설정 (완전한 제어)">
    - 모드, 워크스페이스, 게이트웨이, 채널, 데몬, 스킬의 완전한 프롬프트 흐름을 표시
  </Tab>
</Tabs>

## CLI 온보딩의 자세한 내용

<Columns>
  <Card title="CLI 참조" href="/start/wizard-cli-reference">
    로컬 및 원격 흐름의 완전한 설명, 인증 및 모델 매트릭스, 설정 출력, 마법사 RPC, signal-cli의 동작.
  </Card>
  <Card title="자동화 및 스크립팅" href="/start/wizard-cli-automation">
    비대화형 온보딩의 레시피와 자동화된 `agents add`의 예.
  </Card>
</Columns>

## 자주 사용하는 추후 명령어

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json`은 비대화 모드를 의미하지 않습니다. 스크립트에서는 `--non-interactive`를 사용하세요.
</Note>

<Tip>
추천: 에이전트가 `web_search`를 사용할 수 있도록 Brave Search API 키를 설정하세요 (`web_fetch`는 키 없이 작동). 가장 쉬운 방법: `openclaw configure --section web`을 실행하여 `tools.web.search.apiKey`를 저장합니다. 문서: [Web 도구](/tools/web).
</Tip>

## 관련 문서

- CLI 명령어 참조: [`openclaw onboard`](/cli/onboard)
- macOS 앱 온보딩: [온보딩](/start/onboarding)
- 에이전트 첫 시작 절차: [에이전트 부트스트랩](/start/bootstrapping)
```