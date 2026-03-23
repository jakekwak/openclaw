---
summary: "DigitalOcean Droplet에 OpenClaw 호스팅하기"
read_when:
  - DigitalOcean에서 OpenClaw를 설정할 때
  - OpenClaw용으로 단순한 유료 VPS를 찾을 때
title: "DigitalOcean"
---

# DigitalOcean

DigitalOcean Droplet에서 지속적으로 실행되는 OpenClaw Gateway를 운영합니다.

## 필요 조건

- DigitalOcean 계정 ([signup](https://cloud.digitalocean.com/registrations/new))
- SSH 키 쌍 (또는 비밀번호 인증 사용 의사)
- 약 20분

## 설정

<Steps>
  <Step title="Droplet 만들기">
    <Warning>
    깨끗한 베이스 이미지(Ubuntu 24.04 LTS)를 사용하세요. 시작 스크립트와 기본 방화벽 설정을 직접 검토하지 않았다면 서드파티 Marketplace 1-click 이미지는 피하는 편이 안전합니다.
    </Warning>

    1. [DigitalOcean](https://cloud.digitalocean.com/)에 로그인합니다.
    2. **Create > Droplets**를 클릭합니다.
    3. 다음처럼 선택합니다:
       - **Region:** 본인과 가장 가까운 리전
       - **Image:** Ubuntu 24.04 LTS
       - **Size:** Basic, Regular, 1 vCPU / 1 GB RAM / 25 GB SSD
       - **Authentication:** SSH key 권장, 또는 password
    4. **Create Droplet**를 클릭하고 IP 주소를 기록합니다.

  </Step>

  <Step title="접속 후 설치">
    ```bash
    ssh root@YOUR_DROPLET_IP

    apt update && apt upgrade -y

    # Node.js 24 설치
    curl -fsSL https://deb.nodesource.com/setup_24.x | bash -
    apt install -y nodejs

    # OpenClaw 설치
    curl -fsSL https://openclaw.ai/install.sh | bash
    openclaw --version
    ```

  </Step>

  <Step title="온보딩 실행">
    ```bash
    openclaw onboard --install-daemon
    ```

    이 마법사는 모델 인증, 채널 설정, gateway 토큰 생성, 데몬 설치(systemd)를 순서대로 안내합니다.

  </Step>

  <Step title="스왑 추가 (1 GB Droplet 권장)">
    ```bash
    fallocate -l 2G /swapfile
    chmod 600 /swapfile
    mkswap /swapfile
    swapon /swapfile
    echo '/swapfile none swap sw 0 0' >> /etc/fstab
    ```
  </Step>

  <Step title="Gateway 확인">
    ```bash
    openclaw status
    systemctl --user status openclaw-gateway.service
    journalctl --user -u openclaw-gateway.service -f
    ```
  </Step>

  <Step title="Control UI 접속">
    Gateway는 기본적으로 loopback에 바인드됩니다. 아래 방법 중 하나를 고르세요.

    **옵션 A: SSH 터널 (가장 단순함)**

    ```bash
    # 로컬 머신에서 실행
    ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP
    ```

    그런 다음 `http://localhost:18789`를 엽니다.

    **옵션 B: Tailscale Serve**

    ```bash
    curl -fsSL https://tailscale.com/install.sh | sh
    tailscale up
    openclaw config set gateway.tailscale.mode serve
    openclaw gateway restart
    ```

    그다음 tailnet에 연결된 아무 기기에서나 `https://<magicdns>/`를 엽니다.

    **옵션 C: Tailnet bind (Serve 없이)**

    ```bash
    openclaw config set gateway.bind tailnet
    openclaw gateway restart
    ```

    그다음 `http://<tailscale-ip>:18789`에 접속합니다 (토큰 필요).

  </Step>
</Steps>

## 문제 해결

**Gateway가 시작되지 않음**: `openclaw doctor --non-interactive`를 실행하고 `journalctl --user -u openclaw-gateway.service -n 50`로 로그를 확인하세요.

**포트가 이미 사용 중임**: `lsof -i :18789`로 프로세스를 찾은 뒤 중지하세요.

**메모리 부족**: `free -h`로 swap이 활성화됐는지 확인하세요. 계속 OOM이 난다면 로컬 모델 대신 API 기반 모델(Claude, GPT)을 쓰거나 2 GB Droplet로 올리세요.

## 다음 단계

- [채널](/ko-KR/channels) - Telegram, WhatsApp, Discord 등 연결
- [Gateway configuration](/ko-KR/gateway/configuration) - 전체 설정 옵션
- [Updating](/ko-KR/install/updating) - OpenClaw 최신 상태 유지
