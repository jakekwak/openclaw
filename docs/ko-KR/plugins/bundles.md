---
summary: "Codex, Claude, Cursor 번들을 OpenClaw 플러그인으로 설치하고 사용하는 방법"
read_when:
  - Codex, Claude, Cursor 호환 번들을 설치하려 할 때
  - OpenClaw가 번들 콘텐츠를 네이티브 기능으로 어떻게 매핑하는지 이해하려 할 때
  - 번들 감지나 누락된 capability를 디버그할 때
title: "Plugin Bundles"
---

# Plugin Bundles

OpenClaw는 세 가지 외부 생태계에서 플러그인을 설치할 수 있습니다: **Codex**, **Claude**, **Cursor**. 이런 포맷은 **bundles**라고 부르며, OpenClaw는 이 콘텐츠/메타데이터 묶음을 skills, hooks, MCP tools 같은 네이티브 기능으로 매핑합니다.

<Info>
  bundle은 네이티브 OpenClaw plugin과 동일하지 않습니다. 네이티브 plugin은 프로세스 내부에서 실행되며 어떤 capability든 등록할 수 있지만, bundle은 선택적으로 기능이 매핑되는 콘텐츠 팩이며 신뢰 경계도 더 좁습니다.
</Info>

## 왜 bundle이 필요한가

유용한 플러그인 중에는 Codex, Claude, Cursor 포맷으로 배포되는 것이 많습니다. OpenClaw는 작성자가 이를 네이티브 plugin으로 다시 쓰지 않아도 되도록, 지원되는 bundle 포맷을 감지해서 네이티브 기능 집합으로 매핑합니다. 그래서 Claude command pack이나 Codex skill bundle을 설치해 바로 사용할 수 있습니다.

## bundle 설치

<Steps>
  <Step title="디렉터리, 아카이브, 마켓플레이스에서 설치">
    ```bash
    # 로컬 디렉터리
    openclaw plugins install ./my-bundle

    # 아카이브
    openclaw plugins install ./my-bundle.tgz

    # Claude marketplace
    openclaw plugins marketplace list <marketplace-name>
    openclaw plugins install <plugin-name>@<marketplace-name>
    ```

  </Step>

  <Step title="감지 확인">
    ```bash
    openclaw plugins list
    openclaw plugins inspect <id>
    ```

    bundle은 `Format: bundle`로 표시되고, subtype은 `codex`, `claude`, `cursor` 중 하나입니다.

  </Step>

  <Step title="재시작 후 사용">
    ```bash
    openclaw gateway restart
    ```

    매핑된 기능(skills, hooks, MCP tools)은 다음 세션부터 사용할 수 있습니다.

  </Step>
</Steps>

## OpenClaw가 bundle에서 매핑하는 것

모든 bundle 기능이 지금 OpenClaw에서 실행되는 것은 아닙니다. 아래는 현재 동작하는 항목과 감지만 되는 항목입니다.

### 현재 지원

| Feature       | How it maps                                                                                          | Applies to     |
| ------------- | ---------------------------------------------------------------------------------------------------- | -------------- |
| Skill content | Bundle skill roots load as normal OpenClaw skills                                                    | All formats    |
| Commands      | `commands/` and `.cursor/commands/` treated as skill roots                                           | Claude, Cursor |
| Hook packs    | OpenClaw-style `HOOK.md` + `handler.ts` layouts                                                      | Codex          |
| MCP tools     | Bundle MCP config merged into embedded Pi settings; supported stdio servers launched as subprocesses | All formats    |
| Settings      | Claude `settings.json` imported as embedded Pi defaults                                              | Claude         |

### 감지되지만 실행되지 않음

다음 항목은 감지되고 diagnostics에는 표시되지만, OpenClaw가 실행하지는 않습니다:

- Claude `agents`, `hooks.json` automation, `lspServers`, `outputStyles`
- Cursor `.cursor/agents`, `.cursor/hooks.json`, `.cursor/rules`
- capability 보고 범위를 넘어서는 Codex inline/app metadata

## Bundle 포맷

<AccordionGroup>
  <Accordion title="Codex bundles">
    Marker: `.codex-plugin/plugin.json`

    선택적 콘텐츠: `skills/`, `hooks/`, `.mcp.json`, `.app.json`

    Codex bundle은 skill root와 OpenClaw식 hook-pack 디렉터리(`HOOK.md` + `handler.ts`)를 사용할 때 OpenClaw와 가장 잘 맞습니다.

  </Accordion>

  <Accordion title="Claude bundles">
    감지 방식은 두 가지입니다:

    - **Manifest-based:** `.claude-plugin/plugin.json`
    - **Manifestless:** 기본 Claude 레이아웃 (`skills/`, `commands/`, `agents/`, `hooks/`, `.mcp.json`, `settings.json`)

    Claude 전용 동작:

    - `commands/`는 skill 콘텐츠로 취급
    - `settings.json`은 embedded Pi 설정으로 가져옴 (shell override key는 sanitize됨)
    - `.mcp.json`은 지원되는 stdio 도구를 embedded Pi에 노출
    - `hooks/hooks.json`은 감지만 되고 실행되지는 않음
    - manifest 안의 custom component path는 additive하게 동작하며 기본값을 대체하지 않음

  </Accordion>

  <Accordion title="Cursor bundles">
    Marker: `.cursor-plugin/plugin.json`

    선택적 콘텐츠: `skills/`, `.cursor/commands/`, `.cursor/agents/`, `.cursor/rules/`, `.cursor/hooks.json`, `.mcp.json`

    - `.cursor/commands/`는 skill 콘텐츠로 취급
    - `.cursor/rules/`, `.cursor/agents/`, `.cursor/hooks.json`은 detect-only

  </Accordion>
</AccordionGroup>

## 감지 우선순위

OpenClaw는 먼저 네이티브 plugin 포맷을 확인합니다:

1. `openclaw.plugin.json` 또는 `openclaw.extensions`가 유효한 `package.json` - **native plugin**
2. bundle marker (`.codex-plugin/`, `.claude-plugin/`, 기본 Claude/Cursor 레이아웃) - **bundle**

디렉터리에 둘 다 있으면 OpenClaw는 네이티브 경로를 사용합니다. 이렇게 해야 dual-format 패키지가 bundle로 일부만 설치되는 일을 막을 수 있습니다.

## 보안

bundle은 네이티브 plugin보다 신뢰 경계가 더 좁습니다:

- OpenClaw는 임의의 bundle 런타임 모듈을 프로세스 내부에서 로드하지 않습니다
- skill과 hook-pack 경로는 plugin root 내부에 있어야 합니다 (boundary check)
- settings 파일도 같은 boundary check로 읽습니다
- 지원되는 stdio MCP 서버만 subprocess로 실행될 수 있습니다

기본적으로는 더 안전하지만, 노출되는 기능 범위 안에서는 서드파티 bundle도 신뢰된 콘텐츠로 취급해야 합니다.

## 문제 해결

<AccordionGroup>
  <Accordion title="bundle은 감지되는데 capability가 실행되지 않음">
    `openclaw plugins inspect <id>`를 실행하세요. capability가 목록에 있지만 not wired로 표시된다면 설치 문제라기보다 현재 제품 한계입니다.
  </Accordion>

  <Accordion title="Claude command 파일이 보이지 않음">
    bundle이 활성화되어 있는지 확인하고, markdown 파일이 감지되는 `commands/` 또는 `skills/` root 안에 있는지 확인하세요.
  </Accordion>

  <Accordion title="Claude settings가 적용되지 않음">
    `settings.json`의 embedded Pi 설정만 지원됩니다. OpenClaw는 bundle settings를 raw config patch처럼 적용하지 않습니다.
  </Accordion>

  <Accordion title="Claude hooks가 실행되지 않음">
    `hooks/hooks.json`은 detect-only입니다. 실제로 실행되는 hook이 필요하다면 OpenClaw hook-pack 레이아웃이나 네이티브 plugin을 사용하세요.
  </Accordion>
</AccordionGroup>

## 관련 문서

- [Install and Configure Plugins](/ko-KR/tools/plugin)
- [Building Plugins](/ko-KR/plugins/building-plugins) - 네이티브 plugin 작성
- [Plugin Manifest](/ko-KR/plugins/manifest) - 네이티브 manifest 스키마
