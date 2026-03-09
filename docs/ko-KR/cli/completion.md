---
summary: "쉘 completion 스크립트 생성/설치용 `openclaw completion` CLI 참조"
read_when:
  - zsh/bash/fish/PowerShell 자동 완성을 쓰고 싶을 때
  - 자동 완성 스크립트를 OpenClaw 상태 디렉터리에 캐시해야 할 때
title: "completion"
---

# `openclaw completion`

쉘 자동 완성 스크립트를 생성하고, 필요하면 쉘 프로필에 설치합니다.

## 사용법

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## 옵션

- `-s, --shell <shell>`: 대상 쉘 (`zsh`, `bash`, `powershell`, `fish`; 기본값 `zsh`)
- `-i, --install`: 쉘 프로필에 source 줄을 추가해 completion을 설치
- `--write-state`: stdout에 출력하지 않고 `$OPENCLAW_STATE_DIR/completions`에 completion 스크립트를 기록
- `-y, --yes`: 설치 확인 프롬프트 건너뛰기

## 참고

- `--install`은 쉘 프로필에 작은 `OpenClaw Completion` 블록을 쓰고 캐시된 스크립트를 가리키게 합니다.
- `--install`이나 `--write-state`가 없으면 스크립트를 stdout으로 출력합니다.
- completion 생성 시 중첩 서브커맨드가 포함되도록 명령 트리를 미리 로드합니다.
