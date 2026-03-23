---
summary: "Exa AI 검색 - 콘텐츠 추출이 포함된 neural/keyword 검색"
read_when:
  - web_search에 Exa를 사용하려 할 때
  - `EXA_API_KEY`가 필요할 때
  - neural 검색이나 콘텐츠 추출이 필요할 때
title: "Exa Search"
---

# Exa Search

OpenClaw는 [Exa AI](https://exa.ai/)를 `web_search` provider로 지원합니다. Exa는 neural, keyword, hybrid 검색 모드와 함께 내장 콘텐츠 추출(highlights, text, summaries)을 제공합니다.

## API 키 받기

<Steps>
  <Step title="계정 생성">
    [exa.ai](https://exa.ai/)에서 가입한 뒤 대시보드에서 API 키를 생성합니다.
  </Step>
  <Step title="키 저장">
    Gateway 환경 변수에 `EXA_API_KEY`를 설정하거나, 다음으로 구성합니다:

    ```bash
    openclaw configure --section web
    ```

  </Step>
</Steps>

## Config

```json5
{
  plugins: {
    entries: {
      exa: {
        config: {
          webSearch: {
            apiKey: "exa-...", // EXA_API_KEY가 있으면 선택 사항
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "exa",
      },
    },
  },
}
```

**환경 변수 대안:** Gateway 환경 변수에 `EXA_API_KEY`를 설정할 수 있습니다. 설치형 Gateway라면 `~/.openclaw/.env`에 두면 됩니다.

## 도구 파라미터

| Parameter     | Description                                                                   |
| ------------- | ----------------------------------------------------------------------------- |
| `query`       | Search query (required)                                                       |
| `count`       | Results to return (1-100)                                                     |
| `type`        | Search mode: `auto`, `neural`, `fast`, `deep`, `deep-reasoning`, or `instant` |
| `freshness`   | Time filter: `day`, `week`, `month`, or `year`                                |
| `date_after`  | Results after this date (YYYY-MM-DD)                                          |
| `date_before` | Results before this date (YYYY-MM-DD)                                         |
| `contents`    | Content extraction options (see below)                                        |

### 콘텐츠 추출

Exa는 검색 결과와 함께 추출된 콘텐츠를 반환할 수 있습니다. 활성화하려면 `contents` 객체를 넘기세요:

```javascript
await web_search({
  query: "transformer architecture explained",
  type: "neural",
  contents: {
    text: true, // full page text
    highlights: { numSentences: 3 }, // key sentences
    summary: true, // AI summary
  },
});
```

| Contents option | Type                                                                  | Description            |
| --------------- | --------------------------------------------------------------------- | ---------------------- |
| `text`          | `boolean \| { maxCharacters }`                                        | Extract full page text |
| `highlights`    | `boolean \| { maxCharacters, query, numSentences, highlightsPerUrl }` | Extract key sentences  |
| `summary`       | `boolean \| { query }`                                                | AI-generated summary   |

### 검색 모드

| Mode             | Description                     |
| ---------------- | ------------------------------- |
| `auto`           | Exa가 최적 모드를 선택 (기본값) |
| `neural`         | 의미 기반 semantic 검색         |
| `fast`           | 빠른 keyword 검색               |
| `deep`           | 더 철저한 deep search           |
| `deep-reasoning` | 추론이 포함된 deep search       |
| `instant`        | 가장 빠른 결과                  |

## 참고

- `contents`를 지정하지 않으면 Exa는 기본적으로 `{ highlights: true }`를 사용해 핵심 문장 발췌를 포함합니다
- 가능한 경우 Exa API 응답의 `highlightScores`, `summary` 필드를 그대로 유지합니다
- 결과 설명은 highlights를 우선 사용하고, 없으면 summary, 그다음 full text를 사용합니다
- `freshness`와 `date_after`/`date_before`는 함께 사용할 수 없습니다
- 한 쿼리당 최대 100개의 결과를 반환할 수 있습니다 (Exa 검색 타입 제한에 따름)
- 기본적으로 15분 캐시됩니다 (`cacheTtlMinutes`로 변경 가능)
- Exa는 구조화된 JSON 응답을 제공하는 공식 API 통합입니다

## 관련 문서

- [Web Search overview](/ko-KR/tools/web) - 모든 provider와 자동 감지
- [Brave Search](/ko-KR/tools/brave-search) - 국가/언어 필터가 있는 구조화 검색
- [Perplexity Search](/ko-KR/tools/perplexity-search) - 도메인 필터가 있는 구조화 검색
