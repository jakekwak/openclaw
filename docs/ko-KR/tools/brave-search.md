---
summary: "web_search용 Brave Search API 설정"
read_when:
  - web_search에 Brave Search를 사용하려 할 때
  - `BRAVE_API_KEY`나 요금제 정보가 필요할 때
title: "Brave Search"
---

# Brave Search API

OpenClaw는 Brave Search API를 `web_search` provider로 지원합니다.

## API 키 받기

1. [https://brave.com/search/api/](https://brave.com/search/api/)에서 Brave Search API 계정을 만듭니다.
2. 대시보드에서 **Search** 요금제를 선택하고 API 키를 생성합니다.
3. 키를 config에 저장하거나 Gateway 환경 변수 `BRAVE_API_KEY`로 설정합니다.

## 설정 예시

```json5
{
  plugins: {
    entries: {
      brave: {
        config: {
          webSearch: {
            apiKey: "BRAVE_API_KEY_HERE",
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "brave",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

Brave 전용 검색 설정은 이제 `plugins.entries.brave.config.webSearch.*` 아래에 둡니다. 레거시 `tools.web.search.apiKey`도 호환 shim을 통해 계속 읽히지만, 더 이상 정식 config 경로는 아닙니다.

## 도구 파라미터

| Parameter     | Description                                                         |
| ------------- | ------------------------------------------------------------------- |
| `query`       | Search query (required)                                             |
| `count`       | Number of results to return (1-10, default: 5)                      |
| `country`     | 2-letter ISO country code (e.g., "US", "DE")                        |
| `language`    | ISO 639-1 language code for search results (e.g., "en", "de", "fr") |
| `ui_lang`     | ISO language code for UI elements                                   |
| `freshness`   | Time filter: `day` (24h), `week`, `month`, or `year`                |
| `date_after`  | Only results published after this date (YYYY-MM-DD)                 |
| `date_before` | Only results published before this date (YYYY-MM-DD)                |

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
```

## 참고

- OpenClaw는 Brave **Search** 요금제를 사용합니다. 예전 구독(예: 월 2,000쿼리의 초기 Free 플랜)은 계속 유효하지만, LLM Context나 더 높은 rate limit 같은 최신 기능은 포함하지 않을 수 있습니다.
- Brave 각 요금제에는 매월 갱신되는 **\$5 무료 크레딧**이 포함됩니다. Search 플랜은 요청 1,000건당 \$5이므로, 이 크레딧으로 월 1,000쿼리를 커버할 수 있습니다. 예상치 못한 과금을 피하려면 Brave 대시보드에서 사용 한도를 설정하세요.
- Search 플랜에는 LLM Context 엔드포인트와 AI inference 사용 권한이 포함됩니다. 결과를 저장해 모델을 학습 또는 튜닝하려면 별도 저장 권한이 명시된 요금제가 필요합니다. 자세한 내용은 Brave [Terms of Service](https://api-dashboard.search.brave.com/terms-of-service)를 보세요.
- 결과는 기본적으로 15분 동안 캐시됩니다 (`cacheTtlMinutes`로 변경 가능).

## 관련 문서

- [Web Search overview](/ko-KR/tools/web) - 모든 provider와 자동 감지
- [Perplexity Search](/ko-KR/tools/perplexity-search) - 도메인 필터링이 있는 구조화된 결과
- [Exa Search](/ko-KR/tools/exa-search) - 콘텐츠 추출이 가능한 뉴럴 검색
