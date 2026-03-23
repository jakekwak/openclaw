---
summary: "xAI 웹 grounding 응답을 사용하는 Grok 웹 검색"
read_when:
  - web_search에 Grok를 사용하려 할 때
  - 웹 검색용 `XAI_API_KEY`가 필요할 때
title: "Grok Search"
---

# Grok Search

OpenClaw는 xAI의 웹 grounding 응답을 사용해 Grok를 `web_search` provider로 지원합니다. 이 방식은 실시간 검색 결과를 근거로 인용이 포함된 AI 합성 답변을 생성합니다.

## API 키 받기

<Steps>
  <Step title="키 생성">
    [xAI](https://console.x.ai/)에서 API 키를 발급받습니다.
  </Step>
  <Step title="키 저장">
    Gateway 환경 변수에 `XAI_API_KEY`를 설정하거나, 다음으로 구성합니다:

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
      xai: {
        config: {
          webSearch: {
            apiKey: "xai-...", // XAI_API_KEY가 있으면 선택 사항
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "grok",
      },
    },
  },
}
```

**환경 변수 대안:** Gateway 환경 변수에 `XAI_API_KEY`를 설정할 수 있습니다. 설치형 Gateway라면 `~/.openclaw/.env`에 두면 됩니다.

## 동작 방식

Grok는 xAI 웹 grounding 응답을 사용해 인라인 citation이 포함된 답변을 합성합니다. 전반적인 방식은 Gemini의 Google Search grounding과 유사합니다.

## 지원 파라미터

Grok search는 표준 `query`, `count` 파라미터를 지원합니다. provider 전용 필터는 현재 지원하지 않습니다.

## 관련 문서

- [Web Search overview](/ko-KR/tools/web) - 모든 provider와 자동 감지
- [Gemini Search](/ko-KR/tools/gemini-search) - Google grounding 기반 AI 합성 답변
