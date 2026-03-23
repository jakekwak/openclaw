---
summary: "항상 켜진 self-hosting을 위해 Raspberry Pi에 OpenClaw 호스팅하기"
read_when:
  - Raspberry Pi에 OpenClaw를 설정할 때
  - ARM 기기에서 OpenClaw를 운영할 때
  - 저렴한 always-on 개인 AI를 구축하려 할 때
title: "Raspberry Pi"
---

# Raspberry Pi

Raspberry Pi에서 지속적으로 실행되는 always-on OpenClaw Gateway를 운영합니다. Pi는 Gateway 역할만 하고 모델은 API를 통해 클라우드에서 실행되므로, 보급형 Pi로도 충분히 감당할 수 있습니다.

## 필요 조건

- 2 GB 이상 RAM의 Raspberry Pi 4 또는 5 (권장 4 GB)
- microSD 카드(16 GB+) 또는 USB SSD(성능 권장)
- 정품 Pi 전원 어댑터
- 네트워크 연결(Ethernet 또는 WiFi)
- 64-bit Raspberry Pi OS
  필수입니다. 32-bit는 사용하지 마세요.
- 약 30분

## 설정

<Steps>
  <Step title="OS 굽기">
    headless 서버 용도라면 데스크톱 없이 **Raspberry Pi OS Lite (64-bit)**를 사용하세요.

    1. [Raspberry Pi Imager](https://www.raspberrypi.com/software/)를 내려받습니다.
    2. OS는 **Raspberry Pi OS Lite (64-bit)**를 선택합니다.
    3. 설정 대화상자에서 미리 구성합니다:
       - Hostname: `gateway-host`
       - SSH 활성화
       - 사용자 이름과 비밀번호 설정
       - Ethernet을 쓰지 않으면 WiFi 설정
    4. SD 카드나 USB 드라이브에 굽고, 삽입 후 Pi를 부팅합니다.

  </Step>

  <Step title="SSH로 접속">
    ```bash
    ssh user@gateway-host
    ```
  </Step>

  <Step title="시스템 업데이트">
    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install -y git curl build-essential

    # 시간대 설정 (cron과 reminder에 중요)
    sudo timedatectl set-timezone America/Chicago
    ```

  </Step>

  <Step title="Node.js 24 설치">
    ```bash
    curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
    sudo apt install -y nodejs
    node --version
    ```
  </Step>

  <Step title="스왑 추가 (2 GB 이하에서는 중요)">
    ```bash
    sudo fallocate -l 2G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

    # 저메모리 기기에서 swappiness 낮추기
    echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
    sudo sysctl -p
    ```

  </Step>

  <Step title="OpenClaw 설치">
    ```bash
    curl -fsSL https://openclaw.ai/install.sh | bash
    ```
  </Step>

  <Step title="온보딩 실행">
    ```bash
    openclaw onboard --install-daemon
    ```

    wizard 안내에 따라 진행하세요. headless 기기에서는 OAuth보다 API 키 사용이 더 낫고, 시작 채널로는 Telegram이 가장 간단합니다.

  </Step>

  <Step title="확인">
    ```bash
    openclaw status
    sudo systemctl status openclaw
    journalctl -u openclaw -f
    ```
  </Step>

  <Step title="Control UI 접속">
    먼저 Pi에서 dashboard URL을 확인합니다:

    ```bash
    ssh user@gateway-host 'openclaw dashboard --no-open'
    ```

    그다음 다른 터미널에서 SSH 터널을 엽니다:

    ```bash
    ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
    ```

    출력된 URL을 로컬 브라우저에서 여세요. 항상 켜진 원격 접근이 필요하면 [Tailscale integration](/ko-KR/gateway/tailscale)을 보세요.

  </Step>
</Steps>

## 성능 팁

**USB SSD 사용**: SD 카드는 느리고 마모도 빠릅니다. USB SSD를 쓰면 성능이 크게 좋아집니다. 자세한 내용은 [Pi USB boot guide](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot)를 보세요.

**module compile cache 활성화**: 저전력 Pi에서 반복적인 CLI 호출 속도를 높일 수 있습니다.

```bash
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF' # pragma: allowlist secret
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

**메모리 사용량 줄이기**: headless 환경이라면 GPU 메모리를 줄이고 불필요한 서비스를 끄세요.

```bash
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt
sudo systemctl disable bluetooth
```

## 문제 해결

**메모리 부족**: `free -h`로 swap이 활성화됐는지 확인하세요. 사용하지 않는 서비스(`sudo systemctl disable cups bluetooth avahi-daemon`)를 끄고 API 기반 모델만 사용하세요.

**느린 성능**: SD 카드 대신 USB SSD를 사용하세요. `vcgencmd get_throttled`로 CPU throttling 여부를 확인합니다 (`0x0`이 이상적).

**서비스가 시작되지 않음**: `journalctl -u openclaw --no-pager -n 100`으로 로그를 확인하고 `openclaw doctor --non-interactive`를 실행하세요.

**ARM 바이너리 문제**: 스킬이 "exec format error"로 실패하면 해당 바이너리에 ARM64 빌드가 있는지 확인하세요. `uname -m`은 `aarch64`를 보여야 합니다.

**WiFi 끊김**: WiFi power management를 끄세요: `sudo iwconfig wlan0 power off`.

## 다음 단계

- [채널](/ko-KR/channels) - Telegram, WhatsApp, Discord 등 연결
- [Gateway configuration](/ko-KR/gateway/configuration) - 전체 설정 옵션
- [Updating](/ko-KR/install/updating) - OpenClaw 최신 상태 유지
