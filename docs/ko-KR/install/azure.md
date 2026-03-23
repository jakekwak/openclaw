---
summary: "내구성 있는 상태를 갖춘 Azure Linux VM에서 OpenClaw Gateway를 24/7 실행하기"
read_when:
  - Network Security Group 강화와 함께 Azure에서 OpenClaw를 24/7 운영하려 할 때
  - 자체 Azure Linux VM에서 프로덕션급 상시 OpenClaw Gateway를 원할 때
  - Azure Bastion SSH로 안전하게 관리하려 할 때
title: "Azure"
---

# Azure Linux VM에서 OpenClaw

이 가이드는 Azure CLI로 Azure Linux VM을 만들고, Network Security Group(NSG) 강화를 적용하고, SSH 접근용 Azure Bastion을 구성한 뒤, OpenClaw를 설치하는 과정을 다룹니다.

## 하게 될 일

- Azure CLI로 Azure 네트워크(VNet, subnet, NSG)와 컴퓨트 리소스를 생성
- VM SSH가 Azure Bastion에서만 허용되도록 Network Security Group 규칙 적용
- Azure Bastion으로 SSH 접근 사용 (VM에 public IP 없음)
- 설치 스크립트로 OpenClaw 설치
- Gateway 동작 확인

## 필요 조건

- 컴퓨트/네트워크 리소스를 만들 권한이 있는 Azure subscription
- Azure CLI 설치
  필요하면 [Azure CLI install steps](https://learn.microsoft.com/cli/azure/install-azure-cli)를 보세요.
- SSH 키 쌍
  필요하면 아래 단계에서 생성 방법도 안내합니다.
- 약 20~30분

## 배포 구성

<Steps>
  <Step title="Azure CLI 로그인">
    ```bash
    az login
    az extension add -n ssh
    ```

    Azure Bastion 네이티브 SSH 터널링에는 `ssh` extension이 필요합니다.

  </Step>

  <Step title="필수 resource provider 등록 (1회)">
    ```bash
    az provider register --namespace Microsoft.Compute
    az provider register --namespace Microsoft.Network
    ```

    두 provider 모두 `Registered`가 될 때까지 기다린 뒤 확인합니다.

    ```bash
    az provider show --namespace Microsoft.Compute --query registrationState -o tsv
    az provider show --namespace Microsoft.Network --query registrationState -o tsv
    ```

  </Step>

  <Step title="배포 변수 설정">
    ```bash
    RG="rg-openclaw"
    LOCATION="westus2"
    VNET_NAME="vnet-openclaw"
    VNET_PREFIX="10.40.0.0/16"
    VM_SUBNET_NAME="snet-openclaw-vm"
    VM_SUBNET_PREFIX="10.40.2.0/24"
    BASTION_SUBNET_PREFIX="10.40.1.0/26"
    NSG_NAME="nsg-openclaw-vm"
    VM_NAME="vm-openclaw"
    ADMIN_USERNAME="openclaw"
    BASTION_NAME="bas-openclaw"
    BASTION_PIP_NAME="pip-openclaw-bastion"
    ```

    이름과 CIDR 범위는 환경에 맞게 조정하세요. Bastion subnet은 최소 `/26`이어야 합니다.

  </Step>

  <Step title="SSH 키 선택">
    이미 공개 키가 있다면 그대로 사용합니다:

    ```bash
    SSH_PUB_KEY="$(cat ~/.ssh/id_ed25519.pub)"
    ```

    아직 없다면 새로 생성합니다:

    ```bash
    ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519 -C "you@example.com"
    SSH_PUB_KEY="$(cat ~/.ssh/id_ed25519.pub)"
    ```

  </Step>

  <Step title="VM 크기와 OS 디스크 크기 선택">
    ```bash
    VM_SIZE="Standard_B2as_v2"
    OS_DISK_SIZE_GB=64
    ```

    subscription과 region에서 사용 가능한 VM 크기와 OS 디스크 크기를 고르세요:

    - 가벼운 사용이라면 작게 시작하고 필요 시 확장
    - 자동화/채널/모델·도구 사용량이 많다면 vCPU/RAM/디스크를 더 크게 선택
    - region이나 quota 때문에 unavailable이면 가장 가까운 SKU를 선택

    대상 region에서 가능한 VM 크기 목록:

    ```bash
    az vm list-skus --location "${LOCATION}" --resource-type virtualMachines -o table
    ```

    현재 vCPU/디스크 사용량과 quota 확인:

    ```bash
    az vm list-usage --location "${LOCATION}" -o table
    ```

  </Step>
</Steps>

## Azure 리소스 배포

<Steps>
  <Step title="resource group 생성">
    ```bash
    az group create -n "${RG}" -l "${LOCATION}"
    ```
  </Step>

  <Step title="Network Security Group 생성">
    NSG를 만들고 Bastion subnet에서만 VM으로 SSH가 들어오도록 규칙을 추가합니다.

    ```bash
    az network nsg create \
      -g "${RG}" -n "${NSG_NAME}" -l "${LOCATION}"

    # Bastion subnet에서만 SSH 허용
    az network nsg rule create \
      -g "${RG}" --nsg-name "${NSG_NAME}" \
      -n AllowSshFromBastionSubnet --priority 100 \
      --access Allow --direction Inbound --protocol Tcp \
      --source-address-prefixes "${BASTION_SUBNET_PREFIX}" \
      --destination-port-ranges 22

    # 공용 인터넷에서의 SSH 차단
    az network nsg rule create \
      -g "${RG}" --nsg-name "${NSG_NAME}" \
      -n DenyInternetSsh --priority 110 \
      --access Deny --direction Inbound --protocol Tcp \
      --source-address-prefixes Internet \
      --destination-port-ranges 22

    # 다른 VNet 소스의 SSH 차단
    az network nsg rule create \
      -g "${RG}" --nsg-name "${NSG_NAME}" \
      -n DenyVnetSsh --priority 120 \
      --access Deny --direction Inbound --protocol Tcp \
      --source-address-prefixes VirtualNetwork \
      --destination-port-ranges 22
    ```

    규칙은 priority 순서(숫자가 낮을수록 우선)로 평가됩니다. Bastion 트래픽은 100에서 허용되고, 그 외 SSH는 110과 120에서 차단됩니다.

  </Step>

  <Step title="가상 네트워크와 subnet 생성">
    VM subnet(NSG 연결 포함)이 있는 VNet을 만들고, 그다음 Bastion subnet을 추가합니다.

    ```bash
    az network vnet create \
      -g "${RG}" -n "${VNET_NAME}" -l "${LOCATION}" \
      --address-prefixes "${VNET_PREFIX}" \
      --subnet-name "${VM_SUBNET_NAME}" \
      --subnet-prefixes "${VM_SUBNET_PREFIX}"

    # VM subnet에 NSG 연결
    az network vnet subnet update \
      -g "${RG}" --vnet-name "${VNET_NAME}" \
      -n "${VM_SUBNET_NAME}" --nsg "${NSG_NAME}"

    # AzureBastionSubnet — Azure에서 이름이 고정 요구됨
    az network vnet subnet create \
      -g "${RG}" --vnet-name "${VNET_NAME}" \
      -n AzureBastionSubnet \
      --address-prefixes "${BASTION_SUBNET_PREFIX}"
    ```

  </Step>

  <Step title="VM 생성">
    VM에는 public IP가 없습니다. SSH 접근은 Azure Bastion을 통해서만 가능합니다.

    ```bash
    az vm create \
      -g "${RG}" -n "${VM_NAME}" -l "${LOCATION}" \
      --image "Canonical:ubuntu-24_04-lts:server:latest" \
      --size "${VM_SIZE}" \
      --os-disk-size-gb "${OS_DISK_SIZE_GB}" \
      --storage-sku StandardSSD_LRS \
      --admin-username "${ADMIN_USERNAME}" \
      --ssh-key-values "${SSH_PUB_KEY}" \
      --vnet-name "${VNET_NAME}" \
      --subnet "${VM_SUBNET_NAME}" \
      --public-ip-address "" \
      --nsg ""
    ```

    `--public-ip-address ""`는 public IP를 붙이지 않게 하고, `--nsg ""`는 NIC 레벨 NSG 생성을 건너뜁니다. 보안은 subnet 레벨 NSG가 담당합니다.

    **재현성:** 위 명령은 Ubuntu 이미지에 `latest`를 사용합니다. 특정 버전으로 고정하려면 먼저 가능한 버전을 조회한 뒤 `latest`를 교체하세요:

    ```bash
    az vm image list \
      --publisher Canonical --offer ubuntu-24_04-lts \
      --sku server --all -o table
    ```

  </Step>

  <Step title="Azure Bastion 생성">
    Azure Bastion은 public IP를 노출하지 않고 VM에 관리형 SSH 접근을 제공합니다. CLI 기반 `az network bastion ssh`에는 tunneling이 가능한 Standard SKU가 필요합니다.

    ```bash
    az network public-ip create \
      -g "${RG}" -n "${BASTION_PIP_NAME}" -l "${LOCATION}" \
      --sku Standard --allocation-method Static

    az network bastion create \
      -g "${RG}" -n "${BASTION_NAME}" -l "${LOCATION}" \
      --vnet-name "${VNET_NAME}" \
      --public-ip-address "${BASTION_PIP_NAME}" \
      --sku Standard --enable-tunneling true
    ```

    Bastion provisioning은 보통 5~10분 정도 걸리지만, 지역에 따라 15~30분까지 걸릴 수 있습니다.

  </Step>
</Steps>

## OpenClaw 설치

<Steps>
  <Step title="Azure Bastion으로 VM SSH 접속">
    ```bash
    VM_ID="$(az vm show -g "${RG}" -n "${VM_NAME}" --query id -o tsv)"

    az network bastion ssh \
      --name "${BASTION_NAME}" \
      --resource-group "${RG}" \
      --target-resource-id "${VM_ID}" \
      --auth-type ssh-key \
      --username "${ADMIN_USERNAME}" \
      --ssh-key ~/.ssh/id_ed25519
    ```

  </Step>

  <Step title="OpenClaw 설치 (VM 셸 내부)">
    ```bash
    curl -fsSL https://openclaw.ai/install.sh -o /tmp/install.sh
    bash /tmp/install.sh
    rm -f /tmp/install.sh
    ```

    설치 스크립트는 필요시 Node LTS와 의존성을 설치하고, OpenClaw를 설치한 뒤 onboarding wizard를 시작합니다. 자세한 내용은 [Install](/ko-KR/install) 문서를 보세요.

  </Step>

  <Step title="Gateway 확인">
    onboarding이 끝난 뒤:

    ```bash
    openclaw gateway status
    ```

    많은 기업 Azure 팀은 이미 GitHub Copilot 라이선스를 가지고 있습니다. 해당되는 경우 OpenClaw onboarding wizard에서 GitHub Copilot provider를 선택하는 것을 권장합니다. 자세한 내용은 [GitHub Copilot provider](/ko-KR/providers/github-copilot)를 보세요.

  </Step>
</Steps>

## 비용 고려 사항

Azure Bastion Standard SKU는 대략 **월 \$140**, VM(Standard_B2as_v2)은 대략 **월 \$55** 정도입니다.

비용을 줄이려면:

- **사용하지 않을 때 VM 할당 해제**
  컴퓨트 과금은 멈추지만 디스크 비용은 남습니다. 할당 해제 중에는 OpenClaw Gateway에 접근할 수 없으므로, 다시 필요할 때 시작해야 합니다:

  ```bash
  az vm deallocate -g "${RG}" -n "${VM_NAME}"
  az vm start -g "${RG}" -n "${VM_NAME}"   # 나중에 다시 시작
  ```

- **필요 없을 때 Bastion 삭제**
  SSH 접근이 필요할 때만 다시 생성할 수 있습니다. Bastion이 가장 큰 비용 항목이며 생성도 몇 분이면 됩니다.
- **Basic Bastion SKU 사용**
  대략 **월 \$38** 수준이며, Portal 기반 SSH만 필요하고 CLI 터널링(`az network bastion ssh`)이 필요 없다면 적합합니다.

## 정리

이 가이드가 만든 리소스를 모두 삭제하려면:

```bash
az group delete -n "${RG}" --yes --no-wait
```

이 명령은 resource group과 그 안의 모든 리소스(VM, VNet, NSG, Bastion, public IP)를 함께 제거합니다.

## 다음 단계

- [채널](/ko-KR/channels) - 메시징 채널 설정
- [Nodes](/ko-KR/nodes) - 로컬 기기를 node로 페어링
- [Gateway configuration](/ko-KR/gateway/configuration) - Gateway 설정
- [OpenClaw on Azure with GitHub Copilot](https://github.com/johnsonshi/openclaw-azure-github-copilot) - Azure 배포 참고 자료
