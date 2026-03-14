---
summary: "Windows (WSL2) 지원 + 동반 앱 상태"
read_when:
  - Windows에 OpenClaw 설치
  - Windows 동반 앱 상태 찾기
title: "Windows (WSL2)"
---

# Windows (WSL2)

Windows용 OpenClaw는 **WSL2를 통해** 사용을 권장합니다 (Ubuntu 권장). CLI + 게이트웨이는 Linux 내부에서 실행되며, 이는 실행 환경을 일관성 있게 유지하고 도구들을 훨씬 더 호환 가능하게 만듭니다 (Node/Bun/pnpm, Linux 바이너리, 스킬). Windows 네이티브는 더 까다로울 수 있습니다. WSL2는 완전한 Linux 경험을 제공합니다 — 설치를 위한 한 줄 명령어: `wsl --install`.

네이티브 Windows 동반 앱은 계획 중입니다.

## 설치 (WSL2)

- [시작하기](/start/getting-started) (WSL 내에서 사용)
- [설치 및 업데이트](/install/updating)
- 공식 WSL2 가이드 (Microsoft): [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## 네이티브 Windows 상태

네이티브 Windows CLI 흐름은 계속 개선되고 있지만, 권장 경로는 여전히 WSL2입니다.

현재 네이티브 Windows에서 잘 동작하는 항목:

- `install.ps1`를 통한 웹사이트 설치기
- `openclaw --version`, `openclaw doctor`, `openclaw plugins list --json` 같은 로컬 CLI 사용
- 다음과 같은 내장 local-agent/provider smoke 테스트:

```powershell
openclaw agent --local --agent main --thinking low -m "Reply with exactly WINDOWS-HATCH-OK."
```

현재 주의사항:

- `openclaw onboard --non-interactive`는 `--skip-health`를 넘기지 않으면 여전히 도달 가능한 로컬 gateway를 기대합니다
- `openclaw onboard --non-interactive --install-daemon`과 `openclaw gateway install`은 먼저 Windows Scheduled Tasks를 시도합니다
- Scheduled Task 생성이 거부되면 OpenClaw는 사용자별 Startup 폴더 로그인 항목으로 폴백하고 즉시 gateway를 시작합니다
- `schtasks` 자체가 멈추거나 응답하지 않으면, OpenClaw는 이제 무한 대기하지 않고 그 경로를 빠르게 중단한 뒤 폴백합니다
- Scheduled Tasks를 사용할 수 있으면 여전히 선호됩니다. supervisor 상태를 더 잘 제공하기 때문입니다

gateway 서비스 설치 없이 네이티브 CLI만 쓰고 싶다면 다음 중 하나를 사용하세요:

```powershell
openclaw onboard --non-interactive --skip-health
openclaw gateway run
```

네이티브 Windows에서 관리형 시작을 원한다면:

```powershell
openclaw gateway install
openclaw gateway status --json
```

Scheduled Task 생성이 막혀도, 폴백 서비스 모드는 현재 사용자 Startup 폴더를 통해 로그인 후 자동 시작됩니다.

## 게이트웨이

- [게이트웨이 실행 가이드](/gateway)
- [설정](/gateway/configuration)

## 게이트웨이 서비스 설치 (CLI)

WSL2 내에서:

```
openclaw onboard --install-daemon
```

또는:

```
openclaw gateway install
```

또는:

```
openclaw configure
```

**게이트웨이 서비스**를 선택하라는 메시지가 뜹니다.

수리/마이그레이션:

```
openclaw doctor
```

## 고급: WSL 서비스 LAN 노출 (portproxy)

WSL은 자체 가상 네트워크를 가지고 있습니다. 다른 머신이 **WSL 내부에서** 실행 중인 서비스 (SSH, 로컬 TTS 서버 또는 게이트웨이)에 접근해야 하는 경우, Windows 포트를 현재 WSL IP로 포워딩해야 합니다. WSL IP는 재시작 후 변경되므로 포워딩 규칙을 새로 고쳐야 할 수 있습니다.

예제 (관리자 권한으로 PowerShell):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Windows 방화벽을 통해 포트 허용 (한 번만):

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

WSL 재시작 후 portproxy 새로 고침:

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

주의사항:

- 다른 머신에서 SSH는 **Windows 호스트 IP**를 목표로 합니다 (예: `ssh user@windows-host -p 2222`).
- 원격 노드는 **접근 가능한** 게이트웨이 URL을 가리켜야 합니다 (`127.0.0.1`은 아님); 이를 확인하려면 `openclaw status --all`을 사용하세요.
- LAN 접근을 위해 `listenaddress=0.0.0.0`을 사용하세요; `127.0.0.1`은 로컬로만 유지합니다.
- 자동화를 원하면, 로그인 시 새로 고침 단계를 실행하도록 예약 작업을 등록하세요.

## 단계별 WSL2 설치

### 1) WSL2 + Ubuntu 설치

PowerShell (관리자 권한) 열기:

```powershell
wsl --install
# 또는 배포판을 명시적으로 선택:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Windows에서 요청할 경우 재부팅하십시오.

### 2) systemd 활성화 (게이트웨이 설치에 필요)

WSL 터미널에서:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

그런 다음 PowerShell에서:

```powershell
wsl --shutdown
```

Ubuntu를 다시 열고 확인:

```bash
systemctl --user status
```

### 3) OpenClaw 설치 (WSL 내부)

WSL 내에서 Linux 시작하기 흐름을 따르세요:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 첫 실행 시 UI 의존성을 자동으로 설치합니다
pnpm build
openclaw onboard
```

전체 가이드: [시작하기](/start/getting-started)

## Windows 동반 앱

현재 Windows 동반 앱이 없습니다. 기여를 원하신다면 기여를 통해 실현 가능성에 도움을 주실 수 있습니다.
