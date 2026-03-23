---
summary: "Google Search grounding을 사용하는 Gemini 웹 검색"
read_when:
  - web_search에 Gemini를 사용하려 할 때
  - `GEMINI_API_KEY`가 필요할 때
  - Google Search grounding을 사용하려 할 때
title: "Gemini Search"
---

# Gemini Search

OpenClaw는 [Google Search grounding](https://ai.google.dev/gemini-api/docs/grounding)이 내장된 Gemini 모델을 지원합니다. 이 방식은 실시간 Google Search 결과를 근거로 인용이 포함된 AI 합성 답변을 반환합니다.

## API 키 받기

<Steps>
  <Step title="키 생성">
    [Google AI Studio](https://aistudio.google.com/apikey)에서 API 키를 생성합니다.
  </Step>
  <Step title="키 저장">
    Gateway 환경 변수에 `GEMINI_API_KEY`를 설정하거나, 다음으로 구성합니다:

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
      google: {
        config: {
          webSearch: {
            apiKey: "AIza...", // GEMINI_API_KEY가 있으면 선택 사항
            model: "gemini-2.5-flash", // 기본값
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "gemini",
      },
    },
  },
}
```

**환경 변수 대안:** Gateway 환경 변수에 `GEMINI_API_KEY`를 설정할 수 있습니다. 설치형 Gateway라면 `~/.openclaw/.env`에 두면 됩니다.

## 동작 방식

기존 검색 provider처럼 링크와 snippet 목록을 반환하는 대신, Gemini는 Google Search grounding을 사용해 인라인 인용이 붙은 AI 합성 답변을 생성합니다. 결과에는 합성 답변과 함께 실제 출처 URL이 포함됩니다.

- Gemini grounding에서 나온 citation URL은 Google redirect URL에서 직접 URL로 자동 해석됩니다.
- redirect 해석은 최종 citation URL을 반환하기 전에 SSRF guard 경로(HEAD + redirect 확인 + http/https 검증)를 사용합니다.
- strict SSRF 기본값을 사용하므로 private/internal 대상으로 가는 redirect는 차단됩니다.

## 지원 파라미터

Gemini search는 표준 `query`, `count` 파라미터를 지원합니다. `country`, `language`, `freshness`, `domain_filter` 같은 provider 전용 필터는 지원하지 않습니다.

## 모델 선택

기본 모델은 빠르고 비용 효율적인 `gemini-2.5-flash`입니다. grounding을 지원하는 Gemini 모델이라면 `plugins.entries.google.config.webSearch.model`을 통해 사용할 수 있습니다.

## 관련 문서

- [Web Search overview](/ko-KR/tools/web) - 모든 provider와 자동 감지
- [Brave Search](/ko-KR/tools/brave-search) - snippet 기반 구조화 결과
- [Perplexity Search](/ko-KR/tools/perplexity-search) - 구조화 결과 + 콘텐츠 추출
