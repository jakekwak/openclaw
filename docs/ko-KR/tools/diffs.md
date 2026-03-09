---
title: "Diffs"
summary: "에이전트를 위한 읽기 전용 diff 뷰어 및 파일 렌더러 (선택적 플러그인 도구)"
description: "선택적 Diffs 플러그인을 사용하면 before/after 텍스트나 unified patch를 게이트웨이 호스팅 diff 뷰, 파일(PNG/PDF), 또는 둘 다로 렌더링할 수 있습니다."
read_when:
  - 에이전트가 코드 또는 마크다운 변경을 diff로 보여줘야 할 때
  - 캔버스용 viewer URL 또는 렌더된 diff 파일이 필요할 때
  - 보안 기본값을 유지한 임시 diff artifact가 필요할 때
---

# Diffs

`diffs`는 짧은 내장 시스템 가이드와 companion skill을 함께 제공하는 선택적 플러그인 도구입니다. 변경 내용을 읽기 전용 diff artifact로 만들어 에이전트가 보여줄 수 있게 합니다.

입력 형식:

- `before` + `after` 텍스트
- unified `patch`

출력 형식:

- canvas 표시용 게이트웨이 viewer URL
- 메시지 전송용 렌더된 파일 경로 (PNG 또는 PDF)
- 한 번의 호출에서 둘 다

## 빠른 시작

1. 플러그인을 활성화합니다.
2. canvas 중심 흐름에는 `mode: "view"`를 사용합니다.
3. 채팅 파일 전송 흐름에는 `mode: "file"`을 사용합니다.
4. 둘 다 필요하면 `mode: "both"`를 사용합니다.

## 플러그인 활성화

```json5
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
      },
    },
  },
}
```

## 내장 시스템 가이드 비활성화

도구는 켜 두고 내장 시스템 프롬프트 가이드만 끄고 싶다면 `plugins.entries.diffs.hooks.allowPromptInjection=false`를 설정합니다.

```json5
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        hooks: {
          allowPromptInjection: false,
        },
      },
    },
  },
}
```

## 전형적인 에이전트 흐름

1. 에이전트가 `diffs`를 호출합니다.
2. `details` 필드를 읽습니다.
3. 다음 중 하나를 수행합니다:
   - `details.viewerUrl`을 `canvas present`로 엽니다
   - `details.filePath`를 `message`의 `path`/`filePath`로 보냅니다
   - 둘 다 수행합니다

## 입력 예시

Before / after:

```json
{
  "before": "# Hello\n\nOne",
  "after": "# Hello\n\nTwo",
  "path": "docs/example.md",
  "mode": "view"
}
```

Patch:

```json
{
  "patch": "diff --git a/src/example.ts b/src/example.ts\n--- a/src/example.ts\n+++ b/src/example.ts\n@@ -1 +1 @@\n-const x = 1;\n+const x = 2;\n",
  "mode": "both"
}
```

## 입력 필드

- `before` (`string`): 원본 텍스트. `patch`가 없을 때 `after`와 함께 필요
- `after` (`string`): 변경 후 텍스트. `patch`가 없을 때 `before`와 함께 필요
- `patch` (`string`): unified diff. `before`/`after`와 동시에 사용할 수 없음
- `path` (`string`): before/after 모드의 표시 파일명
- `lang` (`string`): 언어 힌트
- `title` (`string`): viewer 제목
- `mode` (`"view" | "file" | "both"`): 출력 모드
- `theme` (`"light" | "dark"`): viewer 테마
- `layout` (`"unified" | "split"`): diff 레이아웃
- `expandUnchanged` (`boolean`): 가능한 경우 변경 없는 구간 펼치기
- `fileFormat` (`"png" | "pdf"`): 렌더 파일 형식
- `fileQuality` (`"standard" | "hq" | "print"`): 렌더 품질
- `fileScale` (`number`): device scale (`1`-`4`)
- `fileMaxWidth` (`number`): 최대 렌더 폭 (`640`-`2400`)
- `ttlSeconds` (`number`): viewer artifact TTL. 기본 1800, 최대 21600
- `baseUrl` (`string`): viewer URL origin 재정의. `http` 또는 `https`만 허용

## 출력 `details`

viewer가 생성되는 모드의 공통 필드:

- `artifactId`
- `viewerUrl`
- `viewerPath`
- `title`
- `expiresAt`
- `inputKind`
- `fileCount`
- `mode`

파일이 렌더될 때의 필드:

- `filePath`
- `path` (`filePath`와 동일; message tool 호환용)
- `fileBytes`
- `fileFormat`
- `fileQuality`
- `fileScale`
- `fileMaxWidth`

## 보안 모델

- viewer는 기본적으로 loopback 전용입니다.
- 토큰화된 viewer path와 엄격한 ID/token 검증을 사용합니다.
- `security.allowRemoteViewer=false`가 기본값입니다.
- `mode: "file"` 및 `mode: "both"`는 Chromium 호환 브라우저가 필요합니다.
- 외부 네트워크 요청은 차단되고, 로컬 viewer asset만 허용됩니다.

## viewer URL 동작

- viewer route: `/plugins/diffs/view/{artifactId}/{token}`
- asset 경로:
  - `/plugins/diffs/assets/viewer.js`
  - `/plugins/diffs/assets/viewer-runtime.js`
- `baseUrl`이 없으면 기본값은 loopback `127.0.0.1`
- `gateway.bind=custom`이고 `gateway.customBindHost`가 설정돼 있으면 그 호스트를 사용

## 운영 지침

- 로컬 interactive review에는 `mode: "view"`를 우선합니다.
- 아웃바운드 채널 첨부 파일에는 `mode: "file"`을 우선합니다.
- 배포에 꼭 필요하지 않다면 `allowRemoteViewer`는 끄십시오.
- 민감한 diff에는 짧은 `ttlSeconds`를 명시하십시오.
- 채널이 이미지를 과도하게 압축한다면(예: Telegram, WhatsApp) `fileFormat: "pdf"`를 선호하십시오.

Diff 렌더링 엔진:

- [Diffs](https://diffs.com)

## 관련 문서

- [도구 개요](/ko-KR/tools)
- [플러그인](/ko-KR/tools/plugin)
- [브라우저](/ko-KR/tools/browser)
