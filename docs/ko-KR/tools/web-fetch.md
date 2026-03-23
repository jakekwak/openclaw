---
summary: "web_fetch 도구 - 읽기 가능한 콘텐츠 추출이 포함된 HTTP fetch"
read_when:
  - URL을 가져와 읽기 가능한 콘텐츠를 추출하려 할 때
  - `web_fetch` 또는 Firecrawl fallback을 설정하려 할 때
  - `web_fetch`의 제한과 캐싱 동작을 이해하려 할 때
title: "Web Fetch"
sidebarTitle: "Web Fetch"
---

# Web Fetch

`web_fetch` 도구는 일반 HTTP GET을 수행하고 읽기 가능한 콘텐츠를 추출합니다 (HTML을 markdown 또는 text로 변환). JavaScript는 실행하지 않습니다.

JavaScript 비중이 큰 사이트나 로그인 보호 페이지는 [Web Browser](/ko-KR/tools/browser)를 사용하세요.

## 빠른 시작

`web_fetch`는 **기본 활성화**되어 있어 별도 설정이 필요 없습니다. 에이전트가 바로 호출할 수 있습니다:

```javascript
await web_fetch({ url: "https://example.com/article" });
```

## 도구 파라미터

| Parameter     | Type     | Description                              |
| ------------- | -------- | ---------------------------------------- |
| `url`         | `string` | URL to fetch (required, http/https only) |
| `extractMode` | `string` | `"markdown"` (default) or `"text"`       |
| `maxChars`    | `number` | Truncate output to this many chars       |

## 동작 방식

<Steps>
  <Step title="Fetch">
    Chrome 유사 User-Agent와 `Accept-Language` 헤더로 HTTP GET을 보냅니다. private/internal hostname은 차단하고 redirect도 다시 검사합니다.
  </Step>
  <Step title="Extract">
    HTML 응답에 대해 Readability(본문 추출)를 실행합니다.
  </Step>
  <Step title="Fallback (optional)">
    Readability가 실패하고 Firecrawl이 설정돼 있으면, bot-circumvention 모드로 Firecrawl API를 통해 다시 시도합니다.
  </Step>
  <Step title="Cache">
    같은 URL을 반복 fetch하는 비용을 줄이기 위해 결과를 기본 15분 동안 캐시합니다.
  </Step>
</Steps>

## Config

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true, // default: true
        maxChars: 50000, // max output chars
        maxCharsCap: 50000, // hard cap for maxChars param
        maxResponseBytes: 2000000, // max download size before truncation
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        readability: true, // use Readability extraction
        userAgent: "Mozilla/5.0 ...", // override User-Agent
      },
    },
  },
}
```

## Firecrawl fallback

Readability 추출이 실패하면 `web_fetch`는 bot-circumvention과 더 나은 추출을 위해 [Firecrawl](/ko-KR/tools/firecrawl)로 fallback할 수 있습니다:

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          enabled: true,
          apiKey: "fc-...", // FIRECRAWL_API_KEY가 있으면 선택 사항
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // cache duration (1 day)
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

`tools.web.fetch.firecrawl.apiKey`는 SecretRef 객체를 지원합니다.

<Note>
  Firecrawl이 활성화돼 있는데 SecretRef가 해석되지 않고 `FIRECRAWL_API_KEY` env fallback도 없으면, gateway startup은 즉시 실패합니다.
</Note>

## 제한과 안전장치

- `maxChars`는 `tools.web.fetch.maxCharsCap` 이하로 clamp됩니다
- 응답 본문은 파싱 전에 `maxResponseBytes`로 제한되며, 초과 시 경고와 함께 잘립니다
- private/internal hostname은 차단됩니다
- redirect는 검사되며 `maxRedirects` 제한을 따릅니다
- `web_fetch`는 best-effort입니다. 일부 사이트는 [Web Browser](/ko-KR/tools/browser)가 필요합니다

## 도구 프로필

tool profile이나 allowlist를 사용한다면 `web_fetch` 또는 `group:web`를 추가하세요:

```json5
{
  tools: {
    allow: ["web_fetch"],
    // 또는: allow: ["group:web"]  (web_fetch와 web_search를 모두 포함)
  },
}
```

## 관련 문서

- [Web Search](/ko-KR/tools/web) - 여러 provider로 웹 검색
- [Web Browser](/ko-KR/tools/browser) - JS-heavy 사이트용 전체 브라우저 자동화
- [Firecrawl](/ko-KR/tools/firecrawl) - Firecrawl 검색 및 스크래핑 도구
