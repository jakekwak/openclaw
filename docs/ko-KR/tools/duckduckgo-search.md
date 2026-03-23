---
summary: "DuckDuckGo 웹 검색 - 키 없는 fallback provider (experimental, HTML 기반)"
read_when:
  - API 키가 필요 없는 웹 검색 provider를 원할 때
  - web_search에 DuckDuckGo를 사용하려 할 때
  - zero-config 검색 fallback이 필요할 때
title: "DuckDuckGo Search"
---

# DuckDuckGo Search

OpenClaw는 DuckDuckGo를 **키 없는** `web_search` provider로 지원합니다. API 키나 계정이 필요하지 않습니다.

<Warning>
  DuckDuckGo 통합은 **실험적이며 비공식적**입니다. DuckDuckGo의 비 JavaScript 검색 페이지에서 결과를 가져오며, 공식 API를 쓰는 방식이 아닙니다. bot-challenge 페이지나 HTML 변경으로 인해 가끔 깨질 수 있습니다.
</Warning>

## 설정

API 키는 필요 없고, provider만 DuckDuckGo로 지정하면 됩니다:

<Steps>
  <Step title="구성">
    ```bash
    openclaw configure --section web
    # provider로 "duckduckgo" 선택
    ```
  </Step>
</Steps>

## Config

```json5
{
  tools: {
    web: {
      search: {
        provider: "duckduckgo",
      },
    },
  },
}
```

리전과 SafeSearch를 plugin 수준에서 추가로 설정할 수도 있습니다:

```json5
{
  plugins: {
    entries: {
      duckduckgo: {
        config: {
          webSearch: {
            region: "us-en", // DuckDuckGo region code
            safeSearch: "moderate", // "strict", "moderate", or "off"
          },
        },
      },
    },
  },
}
```

## 도구 파라미터

| Parameter    | Description                                                |
| ------------ | ---------------------------------------------------------- |
| `query`      | Search query (required)                                    |
| `count`      | Results to return (1-10, default: 5)                       |
| `region`     | DuckDuckGo region code (e.g. `us-en`, `uk-en`, `de-de`)    |
| `safeSearch` | SafeSearch level: `strict`, `moderate` (default), or `off` |

리전과 SafeSearch는 plugin config에서도 설정할 수 있으며, 도구 파라미터가 쿼리별로 우선합니다.

## 참고

- **No API key** - 즉시 사용 가능, zero configuration
- **Experimental** - 비 JavaScript HTML 검색 페이지를 파싱하며, 공식 API나 SDK가 아닙니다
- **Bot-challenge risk** - 대량 또는 자동화된 사용 시 CAPTCHA나 차단이 발생할 수 있습니다
- **HTML parsing** - 결과는 페이지 구조에 의존하므로 예고 없이 바뀔 수 있습니다
- **Auto-detection order** - DuckDuckGo는 자동 감지 순서에서 마지막(order 100)이므로, API 키가 있는 provider가 있으면 그것이 우선됩니다
- **SafeSearch 기본값은 moderate**입니다

<Tip>
  운영 환경에서는 [Brave Search](/ko-KR/tools/brave-search)나 다른 API 기반 provider를 고려하는 편이 좋습니다.
</Tip>

## 관련 문서

- [Web Search overview](/ko-KR/tools/web) - 모든 provider와 자동 감지
- [Brave Search](/ko-KR/tools/brave-search) - 무료 플랜이 있는 구조화 검색
- [Exa Search](/ko-KR/tools/exa-search) - 콘텐츠 추출이 가능한 뉴럴 검색
