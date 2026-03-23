---
summary: "Tavily 검색 및 추출 도구"
read_when:
  - Tavily 기반 웹 검색을 원할 때
  - Tavily API 키가 필요할 때
  - Tavily를 `web_search` provider로 쓰려 할 때
  - URL에서 콘텐츠를 추출하려 할 때
title: "Tavily"
---

# Tavily

OpenClaw는 **Tavily**를 두 가지 방식으로 사용할 수 있습니다:

- `web_search` provider로 사용
- 명시적 플러그인 도구 `tavily_search`, `tavily_extract`로 사용

Tavily는 AI 애플리케이션용 검색 API로, LLM 소비에 최적화된 구조화된 결과를 반환합니다. 검색 깊이, 주제 필터링, 도메인 필터, AI 생성 요약, URL 콘텐츠 추출(JavaScript 렌더링 페이지 포함)을 지원합니다.

## API 키 받기

1. [tavily.com](https://tavily.com/)에서 Tavily 계정을 만듭니다.
2. 대시보드에서 API 키를 생성합니다.
3. config에 저장하거나 Gateway 환경 변수 `TAVILY_API_KEY`로 설정합니다.

## Tavily 검색 구성

```json5
{
  plugins: {
    entries: {
      tavily: {
        enabled: true,
        config: {
          webSearch: {
            apiKey: "tvly-...", // TAVILY_API_KEY가 있으면 선택 사항
            baseUrl: "https://api.tavily.com",
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "tavily",
      },
    },
  },
}
```

참고:

- onboarding이나 `openclaw configure --section web`에서 Tavily를 고르면 번들 Tavily 플러그인이 자동 활성화됩니다.
- Tavily 설정은 `plugins.entries.tavily.config.webSearch.*` 아래에 둡니다.
- Tavily를 사용하는 `web_search`는 `query`, `count`(최대 20개 결과)를 지원합니다.
- `search_depth`, `topic`, `include_answer`, 도메인 필터 같은 Tavily 전용 제어가 필요하면 `tavily_search`를 사용하세요.

## Tavily 플러그인 도구

### `tavily_search`

일반 `web_search`보다 Tavily 전용 제어가 필요할 때 사용합니다.

| Parameter         | Description                                                           |
| ----------------- | --------------------------------------------------------------------- |
| `query`           | Search query string (keep under 400 characters)                       |
| `search_depth`    | `basic` (default, balanced) or `advanced` (highest relevance, slower) |
| `topic`           | `general` (default), `news` (real-time updates), or `finance`         |
| `max_results`     | Number of results, 1-20 (default: 5)                                  |
| `include_answer`  | Include an AI-generated answer summary (default: false)               |
| `time_range`      | Filter by recency: `day`, `week`, `month`, or `year`                  |
| `include_domains` | Array of domains to restrict results to                               |
| `exclude_domains` | Array of domains to exclude from results                              |

**검색 깊이:**

| Depth      | Speed  | Relevance | Best for                            |
| ---------- | ------ | --------- | ----------------------------------- |
| `basic`    | Faster | High      | General-purpose queries (default)   |
| `advanced` | Slower | Highest   | Precision, specific facts, research |

### `tavily_extract`

하나 이상의 URL에서 정제된 콘텐츠를 추출할 때 사용합니다. JavaScript 렌더링 페이지를 처리할 수 있고, 특정 쿼리에 맞춰 청크를 재정렬하는 targeted extraction도 지원합니다.

| Parameter           | Description                                                |
| ------------------- | ---------------------------------------------------------- |
| `urls`              | Array of URLs to extract (1-20 per request)                |
| `query`             | Rerank extracted chunks by relevance to this query         |
| `extract_depth`     | `basic` (default, fast) or `advanced` (for JS-heavy pages) |
| `chunks_per_source` | Chunks per URL, 1-5 (requires `query`)                     |
| `include_images`    | Include image URLs in results (default: false)             |

**추출 깊이:**

| Depth      | When to use                               |
| ---------- | ----------------------------------------- |
| `basic`    | 단순한 페이지 - 우선 시도                 |
| `advanced` | JS-heavy SPA, 동적 콘텐츠, 표 구조 페이지 |

팁:

- 요청당 최대 20개 URL만 허용됩니다. 더 많으면 여러 호출로 나누세요.
- 전체 페이지 대신 관련 부분만 원한다면 `query` + `chunks_per_source`를 사용하세요.
- 먼저 `basic`을 시도하고, 콘텐츠가 비거나 불완전하면 `advanced`로 넘어가세요.

## 어떤 도구를 써야 하나

| Need                                 | Tool             |
| ------------------------------------ | ---------------- |
| Quick web search, no special options | `web_search`     |
| Search with depth, topic, AI answers | `tavily_search`  |
| Extract content from specific URLs   | `tavily_extract` |

## 관련 문서

- [Web Search overview](/ko-KR/tools/web) - 모든 provider와 자동 감지
- [Firecrawl](/ko-KR/tools/firecrawl) - 검색 + 스크래핑 + 콘텐츠 추출
- [Exa Search](/ko-KR/tools/exa-search) - 콘텐츠 추출이 가능한 뉴럴 검색
