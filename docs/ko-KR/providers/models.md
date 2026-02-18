---
summary: "Model providers (LLMs) supported by OpenClaw"
read_when:
  - You want to choose a model provider
  - You want quick setup examples for LLM auth + model selection
title: "Model Provider Quickstart"
---

# Model Providers

OpenClaw can use many LLM providers. Pick one, authenticate, then set the default
model as `provider/model`.

## Highlight: Venice (Venice AI)

Venice is our recommended Venice AI setup for privacy-first inference with an option to use Opus for the hardest tasks.

- Default: `venice/llama-3.3-70b`
- Best overall: `venice/claude-opus-45` (Opus remains the strongest)

See [Venice AI](/ko-KR/providers/venice).

## Quick start (two steps)

1. Authenticate with the provider (usually via `openclaw onboard`).
2. Set the default model:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Supported providers (starter set)

- [OpenAI (API + Codex)](/ko-KR/providers/openai)
- [Anthropic (API + Claude Code CLI)](/ko-KR/providers/anthropic)
- [OpenRouter](/ko-KR/providers/openrouter)
- [Vercel AI Gateway](/ko-KR/providers/vercel-ai-gateway)
- [Cloudflare AI Gateway](/ko-KR/providers/cloudflare-ai-gateway)
- [Moonshot AI (Kimi + Kimi Coding)](/ko-KR/providers/moonshot)
- [Synthetic](/ko-KR/providers/synthetic)
- [OpenCode Zen](/ko-KR/providers/opencode)
- [Z.AI](/ko-KR/providers/zai)
- [GLM models](/ko-KR/providers/glm)
- [MiniMax](/ko-KR/providers/minimax)
- [Venice (Venice AI)](/ko-KR/providers/venice)
- [Amazon Bedrock](/ko-KR/providers/bedrock)
- [Qianfan](/ko-KR/providers/qianfan)

For the full provider catalog (xAI, Groq, Mistral, etc.) and advanced configuration,
see [Model providers](/ko-KR/concepts/model-providers).