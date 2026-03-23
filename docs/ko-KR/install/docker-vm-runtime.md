---
summary: "장기 실행 OpenClaw Gateway 호스트를 위한 공용 Docker VM 런타임 단계"
read_when:
  - Docker로 OpenClaw를 클라우드 VM에 배포할 때
  - 공통 바이너리 bake, 영속성, 업데이트 흐름이 필요할 때
title: "Docker VM 런타임"
---

# Docker VM 런타임

GCP, Hetzner 같은 VPS 기반 Docker 설치에서 공통으로 쓰는 런타임 단계입니다.

## 필요한 바이너리를 이미지에 bake 하기

실행 중인 컨테이너 안에 바이너리를 설치하는 방식은 함정입니다.
런타임에 설치한 것은 재시작 시 모두 사라집니다.

스킬이 필요로 하는 외부 바이너리는 모두 이미지 빌드 시점에 설치해야 합니다.

아래 예시는 대표적인 세 가지 바이너리만 보여줍니다:

- Gmail 접근용 `gog`
- Google Places용 `goplaces`
- WhatsApp용 `wacli`

이들은 예시일 뿐이며 완전한 목록이 아닙니다.
같은 패턴으로 필요한 바이너리를 더 추가할 수 있습니다.

나중에 추가 스킬이 더 많은 바이너리에 의존하게 되면 다음을 수행해야 합니다:

1. Dockerfile 업데이트
2. 이미지 재빌드
3. 컨테이너 재시작

**예시 Dockerfile**

```dockerfile
FROM node:24-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# 예시 바이너리 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# 예시 바이너리 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# 예시 바이너리 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# 같은 패턴으로 여기에 더 추가하세요

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

## 빌드 및 실행

```bash
docker compose build
docker compose up -d openclaw-gateway
```

`pnpm install --frozen-lockfile` 중 `Killed` 또는 `exit code 137`로 빌드가 실패하면 VM 메모리가 부족한 것입니다.
더 큰 머신 클래스로 변경한 뒤 다시 시도하세요.

바이너리 확인:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

예상 출력:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

Gateway 확인:

```bash
docker compose logs -f openclaw-gateway
```

예상 출력:

```
[gateway] listening on ws://0.0.0.0:18789
```

## 무엇이 어디에 유지되는가

OpenClaw는 Docker 안에서 실행되지만, Docker 자체가 진실의 원본은 아닙니다.
장기 상태는 재시작, 재빌드, 재부팅 이후에도 유지되어야 합니다.

| 구성 요소             | 위치                              | 영속성 방식            | 비고                        |
| --------------------- | --------------------------------- | ---------------------- | --------------------------- |
| Gateway 설정          | `/home/node/.openclaw/`           | 호스트 볼륨 마운트     | `openclaw.json`, 토큰 포함  |
| 모델 인증 프로필      | `/home/node/.openclaw/`           | 호스트 볼륨 마운트     | OAuth 토큰, API 키          |
| 스킬 설정             | `/home/node/.openclaw/skills/`    | 호스트 볼륨 마운트     | 스킬 수준 상태              |
| 에이전트 워크스페이스 | `/home/node/.openclaw/workspace/` | 호스트 볼륨 마운트     | 코드 및 에이전트 산출물     |
| WhatsApp 세션         | `/home/node/.openclaw/`           | 호스트 볼륨 마운트     | QR 로그인 유지              |
| Gmail 키링            | `/home/node/.openclaw/`           | 호스트 볼륨 + 비밀번호 | `GOG_KEYRING_PASSWORD` 필요 |
| 외부 바이너리         | `/usr/local/bin/`                 | Docker 이미지          | 빌드 시 bake 해야 함        |
| Node 런타임           | 컨테이너 파일시스템               | Docker 이미지          | 이미지 빌드 때마다 재생성   |
| OS 패키지             | 컨테이너 파일시스템               | Docker 이미지          | 런타임에 설치하지 마세요    |
| Docker 컨테이너       | 일시적                            | 재시작 가능            | 삭제해도 안전               |

## 업데이트

VM에서 OpenClaw를 업데이트하려면:

```bash
git pull
docker compose build
docker compose up -d
```
