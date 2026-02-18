---
summary: "게이트웨이를 찾기 위한 노드 검색 및 전송 프로토콜 (Bonjour, Tailscale, SSH)"
read_when:
  - Bonjour 디바이스 검색/광고 구현 또는 변경
  - 원격 연결 모드 조정 (직접 vs SSH)
  - 원격 노드를 위한 노드 검색 + 페어링 설계
title: "디바이스 검색 및 전송 프로토콜"
---

# Discovery & transports

OpenClaw는 표면적으로는 유사해 보이는 두 가지 뚜렷한 문제를 가지고 있습니다:

1. **운영자 원격 제어**: 다른 곳에서 실행 중인 게이트웨이를 제어하는 macOS 메뉴 바 앱.
2. **노드 페어링**: iOS/Android (및 향후 노드들) 게이트웨이를 찾아 안전하게 페어링하기.

디자인 목표는 모든 네트워크 검색/광고를 **Node Gateway** (`openclaw gateway`)에 유지하고 클라이언트(mac 앱, iOS)를 소비자로 유지하는 것입니다.

## 용어

- **Gateway**: 상태(세션, 페어링, 노드 레지스트리)를 소유하고 채널을 실행하는 단일 장기 실행 게이트웨이 프로세스. 대부분의 설정에서는 하나의 호스트에 하나를 사용하며, 격리된 다중 게이트웨이 설정도 가능합니다.
- **Gateway WS (제어 플레인)**: 기본적으로 `127.0.0.1:18789`의 WebSocket 엔드포인트; LAN/tailnet을 통해 `gateway.bind`로 연결할 수 있습니다.
- **직접 WS 전송**: LAN/tailnet을 지향하는 Gateway WS 엔드포인트 (SSH 없음).
- **SSH 전송 (대체)**: `127.0.0.1:18789`를 SSH로 포워딩하여 원격 제어.
- **레거시 TCP 브리지 (사용 중단/제거됨)**: 오래된 노드 전송 (참고 [Bridge protocol](/ko-KR/gateway/bridge-protocol)); 더 이상 디바이스 검색에 광고되지 않음.

프로토콜 세부 사항:

- [Gateway protocol](/ko-KR/gateway/protocol)
- [Bridge protocol (legacy)](/ko-KR/gateway/bridge-protocol)

## 왜 '직접'과 SSH를 모두 유지하는가

- **직접 WS**는 동일 네트워크 및 tailnet 내에서 최고의 UX:
  - Bonjour를 통한 LAN에서의 자동 검색
  - 게이트웨이가 소유하는 페어링 토큰 + ACLs
  - 셸 접근 필요 없음; 프로토콜 표면을 단단하고 감사 가능하게 유지 가능
- **SSH**는 보편적인 대안으로 남아 있음:
  - SSH 접근이 가능한 어디서나 작동 (심지어 관련 없는 네트워크에서도)
  - 멀티캐스트/mDNS 문제를 극복할 수 있음
  - SSH를 제외한 새로운 인바운드 포트 필요 없음

## 디바이스 검색 입력 (클라이언트가 게이트웨이가 어디 있는지 배우는 방법)

### 1) Bonjour / mDNS (LAN 전용)

Bonjour는 최선을 다하지만 네트워크를 넘지 않습니다. 같은 LAN 편의를 위해서만 사용됩니다.

목표 방향:

- **게이트웨이**는 Bonjour를 통해 WS 엔드포인트를 광고합니다.
- 클라이언트는 브라우징하고 "게이트웨이를 선택하십시오" 목록을 보여주고 선택한 엔드포인트를 저장합니다.

문제 해결 및 비콘 세부 사항: [Bonjour](/ko-KR/gateway/bonjour).

#### 서비스 비콘 세부 사항

- 서비스 유형:
  - `_openclaw-gw._tcp` (게이트웨이 전송 비콘)
- TXT 키 (비밀 아님):
  - `role=gateway`
  - `lanHost=<호스트명>.local`
  - `sshPort=22` (광고된 경우)
  - `gatewayPort=18789` (Gateway WS + HTTP)
  - `gatewayTls=1` (TLS가 활성화된 경우에만)
  - `gatewayTlsSha256=<sha256>` (TLS가 활성화되고 지문이 사용 가능한 경우에만)
  - `canvasPort=<포트>` (캔버스 호스트 포트; 현재 캔버스 호스트가 활성화된 경우 `gatewayPort`와 동일)
  - `cliPath=<경로>` (옵션; 실행 가능한 `openclaw` 진입점 또는 바이너리의 절대 경로)
  - `tailnetDns=<magicdns>` (옵션 힌트; Tailscale 사용 시 자동 감지됨)

보안 노트:

- Bonjour/mDNS TXT 레코드는 인증되지 않습니다. 클라이언트는 TXT 값을 UX 힌트로만 처리해야 합니다.
- 라우팅 (호스트/포트)은 TXT로 제공된 `lanHost`, `tailnetDns`, 또는 `gatewayPort`보다 **해결된 서비스 엔드포인트** (SRV + A/AAAA)를 선호해야 합니다.
- TLS 고정은 이전에 저장된 핀을 광고된 `gatewayTlsSha256`가 덮어쓰지 않도록 해야 합니다.
- iOS/Android 노드는 검색 기반 직접 연결을 **TLS 전용**으로 처리하고 첫 번째 핀을 저장하기 전에 명시적인 "이 지문 신뢰" 확인이 필요합니다 (외부 검증).

비활성화/재정의:

- `OPENCLAW_DISABLE_BONJOUR=1`은 광고를 비활성화합니다.
- `~/.openclaw/openclaw.json`의 `gateway.bind`는 Gateway 바인드 모드를 제어합니다.
- `OPENCLAW_SSH_PORT`는 TXT에서 광고된 SSH 포트를 재설정합니다 (기본값 22).
- `OPENCLAW_TAILNET_DNS`는 `tailnetDns` 힌트를 게시합니다 (MagicDNS).
- `OPENCLAW_CLI_PATH`는 광고된 CLI 경로를 재설정합니다.

### 2) Tailnet (크로스 네트워크)

런던/비엔나 스타일 설정에서는 Bonjour가 도움이 되지 않습니다. 권장 "직접" 대상은:

- Tailscale MagicDNS 이름(선호) 또는 안정적인 tailnet IP입니다.

게이트웨이가 Tailscale 아래에서 실행 중임을 감지할 수 있다면, 클라이언트에 대한 옵션 힌트로 `tailnetDns`를 게시합니다 (광역 비콘 포함).

### 3) 수동 / SSH 대상

직접 경로가 없거나 (또는 직접 경로가 비활성화된 경우), 클라이언트는 로컬 루프백 게이트웨이 포트를 SSH로 전달하여 항상 연결할 수 있습니다.

[원격 접근](/ko-KR/gateway/remote)을 참조하세요.

## 전송 선택 (클라이언트 정책)

권장 클라이언트 행동:

1. 페어링된 직접 엔드포인트가 구성되고 도달 가능하면 사용합니다.
2. 그렇지 않으면 Bonjour가 LAN에서 게이트웨이를 찾으면 "이 게이트웨이 사용" 선택을 제공하고 직접 엔드포인트로 저장하세요.
3. 그렇지 않으면 tailnet DNS/IP가 구성되어 있으면 직접 시도하세요.
4. 그렇지 않으면 SSH로 대체합니다.

## 페어링 + 인증 (직접 전송)

게이트웨이는 노드/클라이언트 허가의 진실 원천입니다.

- 페어링 요청은 게이트웨이에서 생성/승인/거부됩니다 (참조 [Gateway pairing](/ko-KR/gateway/pairing)).
- 게이트웨이는 다음을 시행합니다:
  - 인증 (토큰 / 키페어)
  - 범위/ACLs (게이트웨이는 모든 메서드에 대해 원시 프록시가 아닙니다)
  - 속도 제한

## 구성 요소별 책임

- **Gateway**: 검색 비콘을 광고하고, 페어링 결정을 소유하며, WS 엔드포인트를 호스팅합니다.
- **macOS 앱**: 게이트웨이를 선택하도록 도와주고, 페어링 프롬프트를 보여주며, SSH만 대체로 사용합니다.
- **iOS/Android 노드**: 편의로 Bonjour를 검색하고, 페어링된 Gateway WS에 연결합니다.