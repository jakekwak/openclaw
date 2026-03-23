---
summary: "Oracle Cloud Always Free ARM 티어에 OpenClaw 호스팅하기"
read_when:
  - Oracle Cloud에서 OpenClaw를 설정할 때
  - OpenClaw용 무료 VPS 호스팅을 찾을 때
  - 작은 서버에서 24/7 OpenClaw를 운영하고 싶을 때
title: "Oracle Cloud"
---

# Oracle Cloud

Oracle Cloud의 **Always Free** ARM 티어(최대 4 OCPU, 24 GB RAM, 200 GB 스토리지)에서 지속적으로 실행되는 OpenClaw Gateway를 무료로 운영합니다.

## 필요 조건

- Oracle Cloud 계정 ([signup](https://www.oracle.com/cloud/free/))
  가입 중 문제가 생기면 [커뮤니티 가입 가이드](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)를 참고하세요.
- Tailscale 계정 ([tailscale.com](https://tailscale.com)에서 무료)
- SSH 키 쌍
- 약 30분

## 설정

<Steps>
  <Step title="OCI 인스턴스 만들기">
    1. [Oracle Cloud Console](https://cloud.oracle.com/)에 로그인합니다.
    2. **Compute > Instances > Create Instance**로 이동합니다.
    3. 다음처럼 설정합니다:
       - **Name:** `openclaw`
       - **Image:** Ubuntu 24.04 (aarch64)
       - **Shape:** `VM.Standard.A1.Flex` (Ampere ARM)
       - **OCPUs:** 2 (최대 4)
       - **Memory:** 12 GB (최대 24 GB)
       - **Boot volume:** 50 GB (무료 최대 200 GB)
       - **SSH key:** 공개 키 추가
    4. **Create**를 클릭하고 공인 IP 주소를 기록합니다.

    <Tip>
    인스턴스 생성이 "Out of capacity"로 실패하면 다른 availability domain을 시도하거나 나중에 다시 시도하세요. 무료 티어 용량은 제한적입니다.
    </Tip>

  </Step>

  <Step title="접속 후 시스템 업데이트">
    ```bash
    ssh ubuntu@YOUR_PUBLIC_IP

    sudo apt update && sudo apt upgrade -y
    sudo apt install -y build-essential
    ```

    일부 종속성은 ARM에서 컴파일이 필요하므로 `build-essential`이 필요합니다.

  </Step>

  <Step title="사용자와 호스트명 설정">
    ```bash
    sudo hostnamectl set-hostname openclaw
    sudo passwd ubuntu
    sudo loginctl enable-linger ubuntu
    ```

    linger를 켜면 로그아웃 후에도 사용자 서비스가 계속 실행됩니다.

  </Step>

  <Step title="Tailscale 설치">
    ```bash
    curl -fsSL https://tailscale.com/install.sh | sh
    sudo tailscale up --ssh --hostname=openclaw
    ```

    이후부터는 Tailscale을 통해 `ssh ubuntu@openclaw`로 접속하면 됩니다.

  </Step>

  <Step title="OpenClaw 설치">
    ```bash
    curl -fsSL https://openclaw.ai/install.sh | bash
    source ~/.bashrc
    ```

    "How do you want to hatch your bot?"라는 질문이 나오면 **Do this later**를 선택하세요.

  </Step>

  <Step title="Gateway 설정">
    안전한 원격 접근을 위해 token auth와 Tailscale Serve를 사용합니다.

    ```bash
    openclaw config set gateway.bind loopback
    openclaw config set gateway.auth.mode token
    openclaw doctor --generate-gateway-token
    openclaw config set gateway.tailscale.mode serve
    openclaw config set gateway.trustedProxies '["127.0.0.1"]'

    systemctl --user restart openclaw-gateway
    ```

  </Step>

  <Step title="VCN 보안 잠그기">
    네트워크 경계에서는 Tailscale만 허용하고 나머지는 차단합니다.

    1. OCI Console에서 **Networking > Virtual Cloud Networks**로 이동합니다.
    2. 해당 VCN을 클릭한 뒤 **Security Lists > Default Security List**로 이동합니다.
    3. `0.0.0.0/0 UDP 41641`(Tailscale)을 제외한 모든 ingress 규칙을 제거합니다.
    4. 기본 egress 규칙(모든 outbound 허용)은 유지합니다.

    이렇게 하면 포트 22 SSH, HTTP, HTTPS를 포함한 나머지 트래픽이 네트워크 경계에서 차단됩니다. 이후 접속은 Tailscale만 사용합니다.

  </Step>

  <Step title="확인">
    ```bash
    openclaw --version
    systemctl --user status openclaw-gateway
    tailscale serve status
    curl http://localhost:18789
    ```

    tailnet에 연결된 아무 기기에서 다음 주소로 Control UI에 접속합니다:

    ```text
    https://openclaw.<tailnet-name>.ts.net/
    ```

    `<tailnet-name>`은 `tailscale status`에서 확인할 수 있는 tailnet 이름으로 바꾸세요.

  </Step>
</Steps>

## 대안: SSH 터널

Tailscale Serve가 동작하지 않으면 로컬 머신에서 SSH 터널을 사용하세요:

```bash
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

그다음 `http://localhost:18789`를 엽니다.

## 문제 해결

**인스턴스 생성 실패 ("Out of capacity")**: 무료 ARM 인스턴스는 경쟁이 큽니다. 다른 availability domain을 시도하거나 한가한 시간대에 다시 시도하세요.

**Tailscale 연결 실패**: `sudo tailscale up --ssh --hostname=openclaw --reset`으로 다시 인증하세요.

**Gateway가 시작되지 않음**: `openclaw doctor --non-interactive`를 실행하고 `journalctl --user -u openclaw-gateway -n 50`로 로그를 확인하세요.

**ARM 바이너리 문제**: 대부분의 npm 패키지는 ARM64에서 동작합니다. 네이티브 바이너리는 `linux-arm64` 또는 `aarch64` 릴리스를 찾고, `uname -m`으로 아키텍처를 확인하세요.

## 다음 단계

- [채널](/ko-KR/channels) - Telegram, WhatsApp, Discord 등 연결
- [Gateway configuration](/ko-KR/gateway/configuration) - 전체 설정 옵션
- [Updating](/ko-KR/install/updating) - OpenClaw 최신 상태 유지
