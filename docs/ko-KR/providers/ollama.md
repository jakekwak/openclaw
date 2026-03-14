---
summary: "OpenClaw를 Ollama (클라우드 및 로컬 모델)로 실행하기"
read_when:
  - OpenClaw를 Ollama를 통한 클라우드 또는 로컬 모델로 실행하려는 경우
  - Ollama 설치 및 설정 가이드를 필요로 하는 경우
title: "Ollama"
---

# Ollama

Ollama는 로컬 LLM 런타임으로, 오픈 소스 모델을 손쉽게 머신에서 실행할 수 있도록 해줍니다. OpenClaw는 Ollama의 네이티브 API (`/api/chat`)와 통합되어 스트리밍 및 도구 호출을 지원하며, `OLLAMA_API_KEY` (또는 인증 프로파일)로 opt-in하고 명시적인 `models.providers.ollama` 항목을 정의하지 않을 경우 로컬 Ollama 모델을 자동으로 검색할 수 있습니다.

## 빠른 시작

### 온보딩 마법사 (권장)

Ollama를 가장 빠르게 설정하는 방법은 온보딩 마법사를 사용하는 것입니다:

```bash
openclaw onboard
```

프로바이더 목록에서 **Ollama**를 선택하세요. 마법사는 다음을 수행합니다:

1. Ollama 인스턴스가 도달 가능한 base URL을 묻습니다 (기본값 `http://127.0.0.1:11434`).
2. **Cloud + Local**(클라우드 + 로컬) 또는 **Local**(로컬만) 중에서 선택하게 합니다.
3. **Cloud + Local**을 선택했고 `ollama.com`에 로그인되어 있지 않다면 브라우저 로그인 흐름을 엽니다.
4. 사용 가능한 모델을 검색하고 기본값을 제안합니다.
5. 선택한 모델이 로컬에 없으면 자동으로 pull 합니다.

비대화형 모드도 지원됩니다:

```bash
openclaw onboard --non-interactive \
  --auth-choice ollama \
  --accept-risk
```

사용자 지정 base URL이나 모델을 지정할 수도 있습니다:

```bash
openclaw onboard --non-interactive \
  --auth-choice ollama \
  --custom-base-url "http://ollama-host:11434" \
  --custom-model-id "qwen3.5:27b" \
  --accept-risk
```

### 수동 설정

1. Ollama 설치: [https://ollama.com/download](https://ollama.com/download)

2. 로컬 추론을 원하면 로컬 모델을 pull 하세요:

```bash
ollama pull glm-4.7-flash
# 또는
ollama pull gpt-oss:20b
# 또는
ollama pull llama3.3
```

3. 클라우드 모델도 쓰고 싶다면 로그인하세요:

```bash
ollama signin
```

4. 온보딩을 실행하고 `Ollama`를 선택하세요:

```bash
openclaw onboard
```

- `Local`: 로컬 모델만
- `Cloud + Local`: 로컬 모델 + 클라우드 모델
- `kimi-k2.5:cloud`, `minimax-m2.5:cloud`, `glm-5:cloud` 같은 클라우드 모델은 로컬 `ollama pull`이 필요하지 않습니다

OpenClaw가 현재 제안하는 기본값:

- 로컬 기본값: `glm-4.7-flash`
- 클라우드 기본값: `kimi-k2.5:cloud`, `minimax-m2.5:cloud`, `glm-5:cloud`

5. 수동 설정을 선호한다면 OpenClaw에서 Ollama를 직접 활성화할 수 있습니다 (아무 값이나 가능; Ollama는 실제 키를 요구하지 않음):

```bash
# 환경 변수 설정
export OLLAMA_API_KEY="ollama-local"

# 또는 설정 파일에서 구성하기
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

6. 모델을 확인하거나 전환합니다:

```bash
openclaw models list
openclaw models set ollama/glm-4.7-flash
```

7. 또는 config에서 기본값을 설정합니다:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/glm-4.7-flash" },
    },
  },
}
```

## 모델 검색 (암시적 프로바이더)

`OLLAMA_API_KEY` (또는 인증 프로파일)를 설정하고 `models.providers.ollama`를 정의하지 않으면 OpenClaw는 로컬 Ollama 인스턴스에서 모델을 검색합니다: `http://127.0.0.1:11434`

- `/api/tags` 쿼리
- 가능할 때 `contextWindow`를 읽기 위해 최선 노력의 `/api/show` 조회 수행
- 모델 이름 휴리스틱(`r1`, `reasoning`, `think`)으로 `reasoning` 표시
- OpenClaw가 사용하는 기본 Ollama max-token cap으로 `maxTokens` 설정
- 모든 비용을 `0`으로 설정

이를 통해 수동 모델 항목 없이도 로컬 Ollama 인스턴스와 정렬된 카탈로그를 유지할 수 있습니다.

사용 가능한 모델을 보려면 다음 명령어를 사용하세요:

```bash
ollama list
openclaw models list
```

새 모델을 추가하려면 단순히 Ollama로 모델을 가져오세요:

```bash
ollama pull mistral
```

새 모델은 자동으로 발견되어 사용 가능합니다.

`models.providers.ollama`를 명시적으로 설정하면 자동 발견이 건너뛰어지며 모델을 수동으로 정의해야 합니다 (아래 참조).

## 설정

### 기본 설정 (암시적 검색)

Ollama를 활성화하는 가장 간단한 방법은 환경 변수를 통한 것입니다:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### 명시적 설정 (수동 모델)

명시적 구성을 사용해야 하는 경우:

- Ollama가 다른 호스트/포트에서 실행 중일 때.
- 특정 컨텍스트 윈도우 또는 모델 목록을 강제하려는 경우.
- 도구 지원을 보고하지 않는 모델을 포함하려는 경우.

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

`OLLAMA_API_KEY`가 설정된 경우, 프로바이더 항목에서 `apiKey`를 생략할 수 있으며 OpenClaw는 가용성 확인을 위해 이를 채웁니다.

### 사용자 정의 기본 URL (명시적 구성)

Ollama가 다른 호스트나 포트에서 실행 중인 경우 (명시적 구성은 자동 발견을 비활성화하므로 모델을 수동으로 정의):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434",
      },
    },
  },
}
```

### 모델 선택

설정이 완료되면 모든 Ollama 모델을 사용할 수 있습니다:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## 고급

### Reasoning 모델

OpenClaw는 Ollama가 `/api/show`에서 `thinking`을 보고할 때 모델을 reasoning 가능한 것으로 표시합니다:

```bash
ollama pull deepseek-r1:32b
```

### 모델 비용

Ollama는 무료로 로컬에서 실행되므로 모든 모델 비용은 $0으로 설정됩니다.

### 스트리밍 설정

OpenClaw의 Ollama 통합은 기본적으로 **네이티브 Ollama API** (`/api/chat`)를 사용하며, 스트리밍 및 도구 호출을 동시에 완전히 지원합니다. 특별한 설정은 필요하지 않습니다.

#### 레거시 OpenAI 호환 모드

프록시 뒤에서 OpenAI 형식만 지원하는 경우와 같이 OpenAI 호환 엔드포인트를 사용해야 한다면, `api: "openai-completions"`을 명시적으로 설정하세요:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

참고: OpenAI 호환 엔드포인트는 스트리밍 + 도구 호출을 동시에 지원하지 않을 수 있습니다. 모델 설정에서 `params: { streaming: false }`로 스트리밍을 비활성화해야 할 수 있습니다.

### 컨텍스트 윈도우

자동으로 발견된 모델의 경우, OpenClaw는 가능하면 Ollama가 보고한 컨텍스트 윈도우를 사용하고, 그렇지 않을 경우 기본값인 `8192`를 사용합니다. 명시적 프로바이더 구성에서 `contextWindow`와 `maxTokens`를 재정의할 수 있습니다.

## 문제 해결

### Ollama가 감지되지 않음

Ollama가 실행 중인지 확인하고 `OLLAMA_API_KEY` (또는 인증 프로파일)를 설정했는지, 명시적 `models.providers.ollama` 항목을 정의하지 않았는지 확인하세요:

```bash
ollama serve
```

API가 접근 가능한지 확인하세요:

```bash
curl http://localhost:11434/api/tags
```

### 사용 가능한 모델이 없음

OpenClaw는 도구 지원을 보고하는 모델만 자동으로 발견합니다. 모델이 목록에 없으면, 다음 두 가지 중 하나를 수행하세요:

- 도구 지원 모델을 가져오거나,
- `models.providers.ollama`에 모델을 명시적으로 정의하세요.

모델을 추가하려면:

```bash
ollama list  # 설치된 항목 확인
ollama pull gpt-oss:20b  # 도구 지원 모델 가져오기
ollama pull llama3.3     # 또는 다른 모델
```

### 연결이 거부됨

Ollama가 올바른 포트에서 실행 중인지 확인하세요:

```bash
# Ollama가 실행 중인지 확인
ps aux | grep ollama

# 또는 Ollama 다시 시작
ollama serve
```

## 관련 문서

- [Model Providers](/ko-KR/concepts/model-providers) - 모든 프로바이더 개요
- [Model Selection](/ko-KR/concepts/models) - 모델 선택 방법
- [Configuration](/ko-KR/gateway/configuration) - 전체 구성 참조
