---
summary: "web_search용 Perplexity Search API 및 Sonar/OpenRouter 호환성"
read_when:
  - 웹 검색에 Perplexity Search를 사용하려 할 때
  - `PERPLEXITY_API_KEY` 또는 `OPENROUTER_API_KEY` 설정이 필요할 때
title: "Perplexity Search"
---

# Perplexity Search API

OpenClaw는 Perplexity Search API를 `web_search` provider로 지원합니다. 이 경로는 `title`, `url`, `snippet` 필드가 있는 구조화된 결과를 반환합니다.

호환성을 위해 레거시 Perplexity Sonar/OpenRouter 설정도 계속 지원합니다. `OPENROUTER_API_KEY`를 사용하거나, `plugins.entries.perplexity.config.webSearch.apiKey`에 `sk-or-...` 키를 저장하거나, `plugins.entries.perplexity.config.webSearch.baseUrl` / `model`을 설정하면 provider는 chat-completions 경로로 전환되고 구조화된 Search API 결과 대신 citation이 포함된 AI 합성 답변을 반환합니다.

## Perplexity API 키 받기

1. [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)에서 Perplexity 계정을 만듭니다
2. 대시보드에서 API 키를 생성합니다
3. 키를 config에 저장하거나 Gateway 환경 변수 `PERPLEXITY_API_KEY`로 설정합니다

## OpenRouter 호환성

이미 OpenRouter로 Perplexity Sonar를 사용 중이었다면, `provider: "perplexity"`를 유지하고 Gateway 환경 변수에 `OPENROUTER_API_KEY`를 설정하거나 `plugins.entries.perplexity.config.webSearch.apiKey`에 `sk-or-...` 키를 저장하면 됩니다.

선택적 호환성 제어 항목:

- `plugins.entries.perplexity.config.webSearch.baseUrl`
- `plugins.entries.perplexity.config.webSearch.model`

## Config 예시

### Native Perplexity Search API

```json5
{
  plugins: {
    entries: {
      perplexity: {
        config: {
          webSearch: {
            apiKey: "pplx-...",
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "perplexity",
      },
    },
  },
}
```

### OpenRouter / Sonar 호환성

```json5
{
  plugins: {
    entries: {
      perplexity: {
        config: {
          webSearch: {
            apiKey: "<openrouter-api-key>",
            baseUrl: "https://openrouter.ai/api/v1",
            model: "perplexity/sonar-pro",
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "perplexity",
      },
    },
  },
}
```

## 키를 어디에 둘까

**config로 설정:** `openclaw configure --section web`를 실행하면 `~/.openclaw/openclaw.json`의 `plugins.entries.perplexity.config.webSearch.apiKey` 아래에 저장됩니다. 이 필드는 SecretRef 객체도 받을 수 있습니다.

**환경 변수로 설정:** Gateway 프로세스 환경 변수에 `PERPLEXITY_API_KEY` 또는 `OPENROUTER_API_KEY`를 설정합니다. 설치형 Gateway라면 `~/.openclaw/.env`나 서비스 환경 변수에 두면 됩니다. 자세한 내용은 [Env vars](/ko-KR/help/faq#env-vars-and-env-loading)를 보세요.

`provider: "perplexity"`가 설정돼 있는데 Perplexity 키 SecretRef가 해석되지 않고 env fallback도 없으면, startup/reload 시 즉시 실패합니다.

## 도구 파라미터

다음 파라미터는 native Perplexity Search API 경로에 적용됩니다.

| Parameter             | Description                                          |
| --------------------- | ---------------------------------------------------- |
| `query`               | Search query (required)                              |
| `count`               | Number of results to return (1-10, default: 5)       |
| `country`             | 2-letter ISO country code (e.g., "US", "DE")         |
| `language`            | ISO 639-1 language code (e.g., "en", "de", "fr")     |
| `freshness`           | Time filter: `day` (24h), `week`, `month`, or `year` |
| `date_after`          | Only results published after this date (YYYY-MM-DD)  |
| `date_before`         | Only results published before this date (YYYY-MM-DD) |
| `domain_filter`       | Domain allowlist/denylist array (max 20)             |
| `max_tokens`          | Total content budget (default: 25000, max: 1000000)  |
| `max_tokens_per_page` | Per-page token limit (default: 2048)                 |

레거시 Sonar/OpenRouter 호환 경로에서는 `query`, `freshness`만 지원합니다. `country`, `language`, `date_after`, `date_before`, `domain_filter`, `max_tokens`, `max_tokens_per_page` 같은 Search API 전용 필터를 넘기면 명시적인 오류를 반환합니다.

**예시:**

```javascript
// 국가/언어별 검색
await web_search({
  query: "renewable energy",
  country: "DE",
  language: "de",
});

// 최근 1주 결과
await web_search({
  query: "AI news",
  freshness: "week",
});

// 날짜 범위 검색
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});

// 도메인 허용 목록
await web_search({
  query: "climate research",
  domain_filter: ["nature.com", "science.org", ".edu"],
});

// 도메인 차단 목록
await web_search({
  query: "product reviews",
  domain_filter: ["-reddit.com", "-pinterest.com"],
});

// 더 많은 콘텐츠 추출
await web_search({
  query: "detailed AI research",
  max_tokens: 50000,
  max_tokens_per_page: 4096,
});
```

### 도메인 필터 규칙

- 필터당 최대 20개 도메인
- 동일 요청에서 allowlist와 denylist를 혼용할 수 없음
- denylist 항목은 `-` prefix 사용 (예: `["-reddit.com"]`)

## 참고

- Perplexity Search API는 구조화된 웹 검색 결과(`title`, `url`, `snippet`)를 반환합니다
- OpenRouter 또는 명시적 `plugins.entries.perplexity.config.webSearch.baseUrl` / `model` 설정은 호환성을 위해 Sonar chat completions 경로로 전환합니다
- 결과는 기본적으로 15분 동안 캐시됩니다 (`cacheTtlMinutes`로 변경 가능)

## 관련 문서

- [Web Search overview](/ko-KR/tools/web) - 모든 provider와 자동 감지
- [Perplexity Search API docs](https://docs.perplexity.ai/docs/search/quickstart) - 공식 문서
- [Brave Search](/ko-KR/tools/brave-search) - 국가/언어 필터가 있는 구조화 검색
- [Exa Search](/ko-KR/tools/exa-search) - 콘텐츠 추출이 가능한 뉴럴 검색
