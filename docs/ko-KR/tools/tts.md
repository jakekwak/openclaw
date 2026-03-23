---
summary: "발신 답변용 text-to-speech (TTS)"
read_when:
  - 답변에 text-to-speech를 켜려 할 때
  - TTS provider나 제한을 설정하려 할 때
  - `/tts` 명령을 사용할 때
title: "Text-to-Speech"
---

# Text-to-speech (TTS)

OpenClaw는 ElevenLabs, Microsoft, OpenAI를 사용해 outbound reply를 오디오로 변환할 수 있습니다. OpenClaw가 오디오를 보낼 수 있는 채널이라면 어디서나 동작하며, Telegram에서는 둥근 voice-note bubble로 전달됩니다.

## 지원 서비스

- **ElevenLabs** (primary 또는 fallback)
- **Microsoft** (primary 또는 fallback)
- **OpenAI** (primary 또는 fallback, summary에도 사용)

### Microsoft speech 참고

번들 Microsoft speech provider는 현재 `node-edge-tts`를 통해 Microsoft Edge 온라인 neural TTS 서비스를 사용합니다. API 키는 필요 없지만, 공개 웹 서비스를 사용하는 best-effort 경로입니다. 엄격한 quota/SLA가 필요하다면 OpenAI나 ElevenLabs를 쓰는 편이 낫습니다.

## 선택적 키

OpenAI 또는 ElevenLabs를 쓰려면:

- `ELEVENLABS_API_KEY` 또는 `XI_API_KEY`
- `OPENAI_API_KEY`

Microsoft speech는 API 키가 필요 없습니다. 아무 키도 없으면 OpenClaw는 기본적으로 Microsoft를 사용합니다.

여러 provider가 설정돼 있으면 선택된 provider를 먼저 쓰고, 나머지는 fallback으로 사용합니다.

## 기본 활성화 여부

아닙니다. auto-TTS는 기본적으로 꺼져 있습니다. config의 `messages.tts.auto` 또는 세션별 `/tts always`로 켜세요.

## Config

TTS 설정은 `openclaw.json`의 `messages.tts` 아래에 있습니다.

### 최소 설정

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs",
    },
  },
}
```

### OpenAI primary + ElevenLabs fallback

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
      },
    },
  },
}
```

### Microsoft primary

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "microsoft",
      microsoft: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
      },
    },
  },
}
```

### 주요 필드

- `auto`: `off`, `always`, `inbound`, `tagged`
- `provider`: `"elevenlabs"`, `"microsoft"`, `"openai"`
- `summaryModel`: 긴 답변 auto-summary에 쓸 모델
- `maxTextLength`: TTS 입력 하드 제한
- `timeoutMs`: 요청 타임아웃
- `prefsPath`: 로컬 preference 저장 경로
- `microsoft.enabled`: Microsoft speech 사용 허용 여부
- `microsoft.voice`, `microsoft.lang`, `microsoft.outputFormat`

`provider`를 지정하지 않으면 OpenClaw는 가능한 경우 `openai`, 그다음 `elevenlabs`, 없으면 `microsoft`를 선호합니다.

## Model-driven override

기본적으로 모델은 한 번의 reply에 대해 TTS directive를 넣을 수 있습니다. `messages.tts.auto`가 `tagged`일 때는 이 directive가 있어야 오디오가 생성됩니다.

예:

```text
[[tts:voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

사용 가능한 directive key:

- `provider`
- `voice` / `voiceId`
- `model`
- `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
- `applyTextNormalization`
- `languageCode`
- `seed`

모든 model override를 끄려면:

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false,
      },
    },
  },
}
```

## Per-user preference

slash command는 `prefsPath`(기본 `~/.openclaw/settings/tts.json`)에 로컬 override를 기록합니다.

저장 필드:

- `enabled`
- `provider`
- `maxLength`
- `summarize`

이 값은 해당 호스트에서 `messages.tts.*`보다 우선합니다.

## 출력 포맷

- **Telegram**: Opus voice note
- **기타 채널**: MP3
- **Microsoft**: `microsoft.outputFormat` 사용

OpenAI/ElevenLabs 포맷은 고정이며, Telegram voice-note UX에는 Opus가 가장 적합합니다.

## Auto-TTS 동작

활성화되면 OpenClaw는:

- reply에 이미 media나 `MEDIA:` directive가 있으면 TTS를 건너뜀
- 너무 짧은 reply(< 10 chars)는 건너뜀
- 긴 reply는 summary가 켜져 있으면 요약 후 TTS
- 생성된 오디오를 reply에 붙임

reply가 `maxLength`를 넘고 summary가 꺼져 있으면 오디오는 만들지 않고 일반 텍스트만 전송합니다.

## Slash command

단일 명령은 `/tts`입니다.

```text
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

- `off|always|inbound|tagged`는 세션별 토글
- `limit`, `summary`는 로컬 preference에 저장
- `/tts audio`는 일회성 오디오 reply 생성

## Agent tool / Gateway RPC

`tts` tool은 텍스트를 음성으로 변환하고 오디오 attachment를 반환합니다.

Gateway method:

- `tts.status`
- `tts.enable`
- `tts.disable`
- `tts.convert`
- `tts.setProvider`
- `tts.providers`
