---
title: "Perplexity (Provider)"
summary: "Perplexity 웹 검색 provider 설정 (API 키, 검색 모드, 필터링)"
read_when:
  - Perplexity를 웹 검색 provider로 설정하려 할 때
  - Perplexity API 키나 OpenRouter 프록시 설정이 필요할 때
---

# Perplexity (Web Search Provider)

Perplexity 플러그인은 Perplexity Search API 또는 OpenRouter 경유 Perplexity Sonar를 통해 웹 검색 기능을 제공합니다.

<Note>
이 페이지는 Perplexity **provider** 설정을 설명합니다. 에이전트가 실제로 도구를 어떻게 사용하는지는 [Perplexity tool](/ko-KR/tools/perplexity-search)을 참조하세요.
</Note>

- Type: web search provider (모델 provider 아님)
- Auth: `PERPLEXITY_API_KEY` (직접) 또는 `OPENROUTER_API_KEY` (OpenRouter 경유)
- Config path: `plugins.entries.perplexity.config.webSearch.apiKey`

## 빠른 시작

1. API 키를 설정합니다:

```bash
openclaw configure --section web
```

또는 직접 설정할 수 있습니다:

```bash
openclaw config set plugins.entries.perplexity.config.webSearch.apiKey "pplx-xxxxxxxxxxxx"
```

2. 설정 후에는 에이전트가 웹 검색 시 자동으로 Perplexity를 사용합니다.

## 검색 모드

플러그인은 API 키 prefix를 기준으로 전송 방식을 자동 선택합니다:

| Key prefix | Transport                    | Features                                         |
| ---------- | ---------------------------- | ------------------------------------------------ |
| `pplx-`    | Native Perplexity Search API | Structured results, domain/language/date filters |
| `sk-or-`   | OpenRouter (Sonar)           | AI-synthesized answers with citations            |

## Native API 필터링

native Perplexity API(`pplx-` 키)를 사용할 때는 다음 검색 필터를 지원합니다:

- **Country**: 2자리 국가 코드
- **Language**: ISO 639-1 언어 코드
- **Date range**: day, week, month, year
- **Domain filters**: allowlist/denylist (최대 20개 도메인)
- **Content budget**: `max_tokens`, `max_tokens_per_page`

## 환경 변수 참고

Gateway가 daemon(launchd/systemd)으로 실행된다면, `PERPLEXITY_API_KEY`가 해당 프로세스에서 보이도록 해야 합니다. 예를 들어 `~/.openclaw/.env`나 `env.shellEnv`를 사용할 수 있습니다.
