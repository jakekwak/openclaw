---
title: "PDF Tool"
summary: "네이티브 프로바이더 지원과 추출 폴백으로 하나 이상의 PDF를 분석"
read_when:
  - 에이전트에서 PDF를 분석하고 싶을 때
  - pdf 도구의 정확한 매개변수와 한계를 알아야 할 때
  - 네이티브 PDF 모드와 추출 폴백을 디버깅할 때
---

# PDF tool

`pdf`는 하나 이상의 PDF 문서를 분석해 텍스트를 반환합니다.

빠른 동작 요약:

- Anthropic 및 Google 프로바이더는 네이티브 PDF 모드를 사용합니다.
- 다른 프로바이더는 먼저 텍스트를 추출하고 필요하면 페이지 이미지를 추가하는 폴백 모드를 사용합니다.
- 단일 입력(`pdf`)과 다중 입력(`pdfs`)을 모두 지원하며, 한 번에 최대 10개 PDF를 다룹니다.

## 사용 가능 조건

다음 순서로 PDF 가능한 모델 구성을 해석할 수 있을 때만 도구가 등록됩니다.

1. `agents.defaults.pdfModel`
2. 없으면 `agents.defaults.imageModel`
3. 그래도 없으면 사용 가능한 인증을 기준으로 best-effort 기본값

## 입력 참조

- `pdf` (`string`): PDF 경로 또는 URL 하나
- `pdfs` (`string[]`): 여러 PDF 경로/URL, 총 10개까지
- `prompt` (`string`): 분석 프롬프트, 기본값 `Analyze this PDF document.`
- `pages` (`string`): `1-5`, `1,3,7-9` 같은 페이지 필터
- `model` (`string`): 선택적 모델 오버라이드 (`provider/model`)
- `maxBytesMb` (`number`): PDF별 크기 제한(MB)

참고:

- `pdf`와 `pdfs`는 합쳐서 중복 제거 후 로드합니다.
- PDF 입력이 없으면 오류가 발생합니다.
- `pages`는 1부터 시작하는 페이지 번호로 해석되며 중복 제거/정렬 후 최대 페이지 수로 clamp됩니다.
- `maxBytesMb` 기본값은 `agents.defaults.pdfMaxBytesMb` 또는 `10`입니다.

## 지원되는 PDF 참조

- 로컬 파일 경로 (`~` 확장 포함)
- `file://` URL
- `http://`, `https://` URL

## 실행 모드

### 네이티브 프로바이더 모드

`anthropic`, `google` 프로바이더에서 사용합니다.

- 원본 PDF 바이트를 그대로 프로바이더 API로 전송합니다.
- 이 모드에서는 `pages`를 지원하지 않습니다.

### 추출 폴백 모드

네이티브 지원이 없는 프로바이더에서 사용합니다.

흐름:

1. 선택된 페이지에서 텍스트를 추출
2. 추출 텍스트가 200자 미만이면 선택 페이지를 PNG로 렌더링
3. 추출 콘텐츠와 프롬프트를 선택 모델에 전송

추가 세부사항:

- 페이지 이미지 추출은 `4,000,000` 픽셀 예산을 사용합니다.
- 대상 모델이 이미지 입력을 지원하지 않고 추출 텍스트도 없으면 오류가 납니다.
- 폴백 모드에는 `pdfjs-dist`가 필요하며, 이미지 렌더링에는 `@napi-rs/canvas`가 필요합니다.

## 구성

```json5
{
  agents: {
    defaults: {
      pdfModel: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai/gpt-5-mini"],
      },
      pdfMaxBytesMb: 10,
      pdfMaxPages: 20,
    },
  },
}
```

자세한 필드는 [Configuration Reference](/ko-KR/gateway/configuration-reference)를 참조하세요.

## 출력 세부사항

도구는 `content[0].text`에 텍스트를, `details`에 구조화된 메타데이터를 반환합니다.

공통 `details` 필드:

- `model`: 해석된 모델 참조
- `native`: 네이티브 프로바이더 모드 여부
- `attempts`: 성공 전 실패한 폴백 시도 수

## 오류 동작

- PDF 입력 누락: `pdf required: provide a path or URL to a PDF document`
- 너무 많은 PDF: `details.error = "too_many_pdfs"`
- 지원되지 않는 참조 스킴: `details.error = "unsupported_pdf_reference"`
- 네이티브 모드에서 `pages` 사용: `pages is not supported with native PDF providers`
