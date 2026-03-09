---
summary: "웹 검색 + fetch 도구 (Brave, Gemini, Grok, Kimi, Perplexity 프로바이더)"
read_when:
  - `web_search` 또는 `web_fetch`를 활성화하려고 할 때
  - Brave 또는 Perplexity Search API 키 설정이 필요할 때
  - Gemini의 Google Search grounding을 쓰고 싶을 때
title: "Web Tools"
---

# Web tools

OpenClaw는 두 가지 경량 웹 도구를 제공합니다.

- `web_search`: Brave Search API, Gemini, Grok, Kimi, Perplexity Search API를 사용한 웹 검색
- `web_fetch`: HTTP fetch + 읽기 쉬운 본문 추출

이들은 **브라우저 자동화가 아닙니다**. JS가 많은 사이트나 로그인 흐름은 [브라우저 도구](/ko-KR/tools/browser)를 사용하세요.

## 작동 방식

- `web_search`는 구성한 프로바이더를 호출해 결과를 반환합니다.
- 결과는 쿼리별로 15분간 캐시됩니다.
- `web_fetch`는 일반 HTTP GET 후 HTML을 markdown/텍스트로 추출합니다.
- `web_fetch`는 기본 활성화입니다.

프로바이더별 세부사항:

- [Brave Search 설정](/ko-KR/brave-search)
- [Perplexity Search 설정](/ko-KR/perplexity)

## 검색 프로바이더 선택

| 프로바이더                | 결과 형태                    | 프로바이더 전용 필터                         | 참고                            | API 키                                      |
| ------------------------- | ---------------------------- | -------------------------------------------- | ------------------------------- | ------------------------------------------- |
| **Brave Search API**      | snippet이 포함된 구조화 결과 | `country`, `language`, `ui_lang`, 시간       | Brave `llm-context` 모드 지원   | `BRAVE_API_KEY`                             |
| **Gemini**                | 인용이 포함된 AI 합성 답변   | -                                            | Google Search grounding 사용    | `GEMINI_API_KEY`                            |
| **Grok**                  | 인용이 포함된 AI 합성 답변   | -                                            | xAI web-grounded 응답 사용      | `XAI_API_KEY`                               |
| **Kimi**                  | 인용이 포함된 AI 합성 답변   | -                                            | Moonshot 웹 검색 사용           | `KIMI_API_KEY` / `MOONSHOT_API_KEY`         |
| **Perplexity Search API** | snippet이 포함된 구조화 결과 | `country`, `language`, 시간, `domain_filter` | OpenRouter Sonar 호환 경로 지원 | `PERPLEXITY_API_KEY` / `OPENROUTER_API_KEY` |

### 자동 감지

`provider`를 명시하지 않으면 다음 순서로 자동 감지합니다.

1. Brave
2. Gemini
3. Grok
4. Kimi
5. Perplexity

키가 하나도 없으면 Brave로 폴백하며, 이때 키 설정 안내 오류를 반환합니다.

## 웹 검색 설정

`openclaw configure --section web`로 API 키 저장과 프로바이더 선택을 할 수 있습니다.

### Brave Search

1. [brave.com/search/api](https://brave.com/search/api/)에서 계정을 만듭니다.
2. 대시보드에서 **Search** 플랜을 선택하고 API 키를 만듭니다.
3. `openclaw configure --section web` 또는 `BRAVE_API_KEY` 환경 변수로 저장합니다.

### Perplexity Search

1. [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)에서 계정을 만듭니다.
2. API 키를 생성합니다.
3. `openclaw configure --section web` 또는 `PERPLEXITY_API_KEY`로 저장합니다.

레거시 Sonar/OpenRouter 호환이 필요하면 `OPENROUTER_API_KEY`를 사용하거나 `tools.web.search.perplexity` 아래에 `baseUrl`/`model`을 설정하세요.

### 키 저장 위치

- 구성 파일: `openclaw configure --section web`
- 환경 변수: `BRAVE_API_KEY`, `PERPLEXITY_API_KEY`, `OPENROUTER_API_KEY`, `GEMINI_API_KEY`, `XAI_API_KEY`, `KIMI_API_KEY`, `MOONSHOT_API_KEY`

게이트웨이 설치에서는 보통 `~/.openclaw/.env` 또는 서비스 환경을 사용합니다. [환경 변수](/ko-KR/help/faq#how-does-openclaw-load-environment-variables)를 참조하세요.

## 구성 예시

### Brave Search

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "brave",
        apiKey: "YOUR_BRAVE_API_KEY", // BRAVE_API_KEY가 있으면 생략 가능
      },
    },
  },
}
```

### Brave LLM Context 모드

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "brave",
        brave: {
          mode: "llm-context",
        },
      },
    },
  },
}
```

`llm-context`는 일반 Brave snippet 대신 grounding용 페이지 청크를 반환합니다.

### Perplexity Search

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...", // PERPLEXITY_API_KEY가 있으면 생략 가능
        },
      },
    },
  },
}
```

### Perplexity via OpenRouter / Sonar compatibility

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "<openrouter-api-key>",
          baseUrl: "https://openrouter.ai/api/v1",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## Gemini 사용

Gemini 모델은 Google Search grounding을 지원합니다. 자세한 내용은 [Google Gemini](/ko-KR/providers/openai) 대신 Gemini 관련 provider 문서를 참조하세요.

## web_search

설정된 프로바이더를 사용해 웹을 검색합니다.

### 요구 사항

- `tools.web.search.enabled`가 `false`가 아니어야 합니다.
- 프로바이더별 API 키:
  - Brave: `BRAVE_API_KEY` 또는 `tools.web.search.apiKey`
  - Gemini: `GEMINI_API_KEY` 또는 `tools.web.search.gemini.apiKey`
  - Grok: `XAI_API_KEY` 또는 `tools.web.search.grok.apiKey`
  - Kimi: `KIMI_API_KEY`, `MOONSHOT_API_KEY`, 또는 `tools.web.search.kimi.apiKey`
  - Perplexity: `PERPLEXITY_API_KEY`, `OPENROUTER_API_KEY`, 또는 `tools.web.search.perplexity.apiKey`

### 구성

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### 도구 매개변수

- `query` (필수)
- `count` (1-10)
- `country`
- `search_lang`
- `ui_lang`
- `freshness`

예:

```javascript
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de",
});

await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr",
});

await web_search({
  query: "TMBG interview",
  freshness: "pw",
});
```

## web_fetch

URL을 가져와 읽기 쉬운 콘텐츠를 추출합니다.

### 요구 사항

- `tools.web.fetch.enabled`가 `false`가 아니어야 합니다.
- 선택적 Firecrawl 폴백: `tools.web.fetch.firecrawl.apiKey` 또는 `FIRECRAWL_API_KEY`

### 구성

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000,
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### 도구 매개변수

- `url` (필수, http/https만)
- `extractMode` (`markdown` | `text`)
- `maxChars`

참고:

- `web_fetch`는 먼저 Readability를 쓰고, 필요하면 Firecrawl을 사용합니다.
- Chrome 유사 User-Agent와 `Accept-Language`를 기본 전송합니다.
- private/internal hostname은 차단하고 redirect를 재검사합니다.
- `maxChars`는 `tools.web.fetch.maxCharsCap`에 맞춰 clamp됩니다.
- 너무 큰 응답은 `tools.web.fetch.maxResponseBytes`에서 잘리고 경고를 남깁니다.
- 일부 사이트는 브라우저 도구가 필요할 수 있습니다.
- [Firecrawl](/ko-KR/tools/firecrawl)에서 키 설정을 확인하세요.
