---
summary: "`openclaw plugins` CLI 참조 (목록, 설치, 제거, 활성화/비활성화, doctor)"
read_when:
  - 인프로세스 게이트웨이 플러그인을 설치하거나 관리하고 싶을 때
  - 플러그인 로드 실패를 디버그하고 싶을 때
title: "plugins"
---

# `openclaw plugins`

게이트웨이 플러그인/확장을 관리합니다 (인프로세스 로드).

관련 항목:

- 플러그인 시스템: [Plugins](/ko-KR/tools/plugin)
- 플러그인 manifest + schema: [Plugin manifest](/ko-KR/plugins/manifest)
- 보안 강화: [Security](/ko-KR/gateway/security)

## 명령어

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

번들 플러그인은 OpenClaw와 함께 제공되지만 기본적으로 비활성화되어 있습니다. 활성화하려면 `plugins enable`을 사용하세요.

모든 플러그인은 인라인 JSON Schema(`configSchema`, 비어 있어도 됨)를 포함한 `openclaw.plugin.json` 파일을 제공해야 합니다. manifest 또는 schema가 없거나 유효하지 않으면 플러그인이 로드되지 않고 구성 검증도 실패합니다.

### 설치

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

보안 참고: 플러그인 설치는 코드를 실행하는 것처럼 취급하세요. 가능하면 고정된 버전을 사용하세요.

Npm 스펙은 **레지스트리 전용**입니다 (패키지 이름 + 선택적 **정확한 버전** 또는 **dist-tag**). Git/URL/file 스펙과 semver 범위는 거부됩니다. 의존성 설치는 안전을 위해 `--ignore-scripts`로 실행됩니다.

bare 스펙과 `@latest`는 stable 트랙에 머뭅니다. npm이 이들 중 하나를 prerelease로 해결하면, OpenClaw는 중단하고 `@beta`/`@rc` 같은 prerelease 태그나 `@1.2.3-beta.4` 같은 정확한 prerelease 버전으로 명시적으로 opt-in 하라고 요청합니다.

Install 스펙이 번들 플러그인 ID(예: `diffs`)와 동일하면 OpenClaw는 번들 플러그인을 직접 설치합니다.
같은 이름의 npm 패키지를 설치하려면 명시적인 스코프 스펙(예: `@scope/diffs`)을 사용하세요.

지원 아카이브 형식: `.zip`, `.tgz`, `.tar.gz`, `.tar`.

로컬 디렉터리를 복사하지 않으려면 `--link`를 사용하세요 (`plugins.load.paths`에 추가됨):

```bash
openclaw plugins install -l ./my-plugin
```

npm 설치 시 `--pin`을 사용하면 해결된 정확한 스펙(`name@version`)이
`plugins.installs`에 저장되며, 기본 동작은 고정되지 않습니다.

### 제거

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall`는 `plugins.entries`, `plugins.installs`, 플러그인 허용 목록 및 관련된 `plugins.load.paths` 항목에서 플러그인 기록을 제거합니다. 활성 메모리 플러그인의 경우, 메모리 슬롯은 `memory-core`로 재설정됩니다.

기본적으로, uninstall은 활성 상태 디렉토리 확장 루트(`$OPENCLAW_STATE_DIR/extensions/<id>`) 아래의 플러그인 설치 디렉토리도 제거합니다. 파일을 유지하려면 `--keep-files`를 사용하십시오.

`--keep-config`는 `--keep-files`의 폐기된 별칭으로 지원됩니다.

### 업데이트

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

업데이트는 npm에서 설치한 플러그인에만 적용됩니다 (`plugins.installs`에 추적됨).

저장된 무결성 해시가 존재하고 가져온 아티팩트 해시가 변경되면,
OpenClaw는 경고를 출력하고 진행하기 전에 확인을 요청합니다. CI/비대화형 실행에서는
전역 `--yes` 플래그를 사용하여 프롬프트를 건너뛰세요.
