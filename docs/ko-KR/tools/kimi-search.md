---
summary: "Moonshot 웹 검색을 사용하는 Kimi 웹 검색"
read_when:
  - web_search에 Kimi를 사용하려 할 때
  - `KIMI_API_KEY` 또는 `MOONSHOT_API_KEY`가 필요할 때
title: "Kimi Search"
---

# Kimi Search

OpenClaw는 Moonshot 웹 검색을 사용해 Kimi를 `web_search` provider로 지원합니다. 이 방식은 citation이 포함된 AI 합성 답변을 생성합니다.

## API 키 받기

<Steps>
  <Step title="키 생성">
    [Moonshot AI](https://platform.moonshot.cn/)에서 API 키를 발급받습니다.
  </Step>
  <Step title="키 저장">
    Gateway 환경 변수에 `KIMI_API_KEY` 또는 `MOONSHOT_API_KEY`를 설정하거나, 다음으로 구성합니다:

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
      moonshot: {
        config: {
          webSearch: {
            apiKey: "sk-...", // KIMI_API_KEY 또는 MOONSHOT_API_KEY가 있으면 선택 사항
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "kimi",
      },
    },
  },
}
```

**환경 변수 대안:** Gateway 환경 변수에 `KIMI_API_KEY` 또는 `MOONSHOT_API_KEY`를 설정할 수 있습니다. 설치형 Gateway라면 `~/.openclaw/.env`에 두면 됩니다.

## 동작 방식

Kimi는 Moonshot 웹 검색을 사용해 인라인 citation이 포함된 답변을 합성합니다. 동작 방식은 Gemini나 Grok의 grounding 응답 접근과 비슷합니다.

## 지원 파라미터

Kimi search는 표준 `query`, `count` 파라미터를 지원합니다. provider 전용 필터는 현재 지원하지 않습니다.

## 관련 문서

- [Web Search overview](/ko-KR/tools/web) - 모든 provider와 자동 감지
- [Gemini Search](/ko-KR/tools/gemini-search) - Google grounding 기반 AI 합성 답변
- [Grok Search](/ko-KR/tools/grok-search) - xAI grounding 기반 AI 합성 답변
