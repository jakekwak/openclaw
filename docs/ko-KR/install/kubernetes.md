---
summary: "Kustomize로 Kubernetes 클러스터에 OpenClaw Gateway 배포하기"
read_when:
  - OpenClaw를 Kubernetes 클러스터에서 실행하고 싶을 때
  - Kubernetes 환경에서 OpenClaw를 테스트하고 싶을 때
title: "Kubernetes"
---

# Kubernetes에서 OpenClaw

Kubernetes에서 OpenClaw를 실행하기 위한 최소 시작점입니다. 프로덕션 준비용 배포는 아니며, 핵심 리소스만 다루고 환경에 맞게 조정하는 것을 전제로 합니다.

## 왜 Helm이 아닌가

OpenClaw는 몇 개의 설정 파일을 가진 단일 컨테이너입니다. 흥미로운 커스터마이징은 인프라 템플릿보다 에이전트 내용(markdown 파일, 스킬, 설정 오버라이드)에 있습니다. Kustomize는 Helm 차트의 오버헤드 없이 오버레이를 처리할 수 있습니다. 배포가 더 복잡해지면 이 매니페스트 위에 Helm 차트를 얹을 수 있습니다.

## 필요 조건

- 동작 중인 Kubernetes 클러스터 (AKS, EKS, GKE, k3s, kind, OpenShift 등)
- 클러스터에 연결된 `kubectl`
- 최소 하나 이상의 모델 프로바이더 API 키

## 빠른 시작

```bash
# 사용할 프로바이더로 바꾸세요: ANTHROPIC, GEMINI, OPENAI, 또는 OPENROUTER
export <PROVIDER>_API_KEY="..."
./scripts/k8s/deploy.sh

kubectl port-forward svc/openclaw 18789:18789 -n openclaw
open http://localhost:18789
```

Gateway 토큰을 가져와 Control UI에 붙여넣습니다:

```bash
kubectl get secret openclaw-secrets -n openclaw -o jsonpath='{.data.OPENCLAW_GATEWAY_TOKEN}' | base64 -d
```

로컬 디버깅용으로는 `./scripts/k8s/deploy.sh --show-token`이 배포 후 토큰을 출력합니다.

## Kind로 로컬 테스트

클러스터가 없다면 [Kind](https://kind.sigs.k8s.io/)로 로컬 클러스터를 만들 수 있습니다:

```bash
./scripts/k8s/create-kind.sh           # docker 또는 podman 자동 감지
./scripts/k8s/create-kind.sh --delete  # 정리
```

그다음 평소처럼 `./scripts/k8s/deploy.sh`로 배포합니다.

## 단계별 진행

### 1) 배포

**옵션 A** — 환경 변수에 API 키 설정 (한 단계):

```bash
# 사용할 프로바이더로 바꾸세요: ANTHROPIC, GEMINI, OPENAI, 또는 OPENROUTER
export <PROVIDER>_API_KEY="..."
./scripts/k8s/deploy.sh
```

이 스크립트는 API 키와 자동 생성된 Gateway 토큰이 들어 있는 Kubernetes Secret을 만든 뒤 배포를 수행합니다. Secret이 이미 있으면 현재 Gateway 토큰과, 변경하지 않는 기존 프로바이더 키는 유지합니다.

**옵션 B** — Secret을 별도로 생성:

```bash
export <PROVIDER>_API_KEY="..."
./scripts/k8s/deploy.sh --create-secret
./scripts/k8s/deploy.sh
```

로컬 테스트용으로 토큰을 stdout에 보고 싶다면 어느 명령이든 `--show-token`을 추가하세요.

### 2) Gateway 접근

```bash
kubectl port-forward svc/openclaw 18789:18789 -n openclaw
open http://localhost:18789
```

## 배포되는 항목

```
Namespace: openclaw (OPENCLAW_NAMESPACE로 변경 가능)
├── Deployment/openclaw        # 단일 pod, init container + gateway
├── Service/openclaw           # 18789 포트의 ClusterIP
├── PersistentVolumeClaim      # 에이전트 상태와 설정용 10Gi
├── ConfigMap/openclaw-config  # openclaw.json + AGENTS.md
└── Secret/openclaw-secrets    # Gateway token + API keys
```

## 커스터마이징

### 에이전트 지침

`scripts/k8s/manifests/configmap.yaml`의 `AGENTS.md`를 수정한 뒤 재배포하세요:

```bash
./scripts/k8s/deploy.sh
```

### Gateway 설정

`scripts/k8s/manifests/configmap.yaml`의 `openclaw.json`을 수정하세요. 전체 참조는 [Gateway configuration](/gateway/configuration)를 보세요.

### 프로바이더 추가

추가 키를 export한 뒤 다시 실행합니다:

```bash
export ANTHROPIC_API_KEY="..."
export OPENAI_API_KEY="..."
./scripts/k8s/deploy.sh --create-secret
./scripts/k8s/deploy.sh
```

기존 프로바이더 키는 덮어쓰지 않는 한 Secret에 남아 있습니다.

또는 Secret을 직접 patch할 수 있습니다:

```bash
kubectl patch secret openclaw-secrets -n openclaw \
  -p '{"stringData":{"<PROVIDER>_API_KEY":"..."}}'
kubectl rollout restart deployment/openclaw -n openclaw
```

### 커스텀 네임스페이스

```bash
OPENCLAW_NAMESPACE=my-namespace ./scripts/k8s/deploy.sh
```

### 커스텀 이미지

`scripts/k8s/manifests/deployment.yaml`의 `image` 필드를 수정합니다:

```yaml
image: ghcr.io/openclaw/openclaw:2026.3.1
```

### port-forward를 넘어서 노출하기

기본 매니페스트는 pod 내부에서 gateway를 loopback에 바인드합니다. 이는 `kubectl port-forward`에는 맞지만, pod IP에 직접 접근해야 하는 Kubernetes `Service`나 Ingress 경로에는 맞지 않습니다.

Ingress나 로드 밸런서를 통해 gateway를 노출하려면:

- `scripts/k8s/manifests/configmap.yaml`에서 gateway bind를 `loopback`에서 배포 모델에 맞는 non-loopback 바인드로 변경
- Gateway 인증을 유지하고 적절한 TLS 종료 엔드포인트 사용
- 지원되는 웹 보안 모델에 맞게 Control UI 원격 접근 구성 (예: HTTPS/Tailscale Serve, 필요 시 명시적 allowed origins)

## 재배포

```bash
./scripts/k8s/deploy.sh
```

이 명령은 모든 매니페스트를 적용하고 pod를 재시작해 설정 또는 Secret 변경을 반영합니다.

## 정리

```bash
./scripts/k8s/deploy.sh --delete
```

이 명령은 네임스페이스와 그 안의 모든 리소스(PVC 포함)를 삭제합니다.

## 아키텍처 메모

- 기본적으로 gateway는 pod 내부 loopback에 바인드되므로, 포함된 설정은 `kubectl port-forward`용입니다
- 클러스터 범위 리소스는 없고, 모든 것은 단일 네임스페이스에 있습니다
- 보안: `readOnlyRootFilesystem`, `drop: ALL` capabilities, non-root user (UID 1000)
- 기본 설정은 더 안전한 로컬 접근 경로를 유지합니다: loopback bind + `kubectl port-forward`를 통한 `http://127.0.0.1:18789`
- localhost를 넘어서 접근한다면 지원되는 원격 모델을 사용하세요: HTTPS/Tailscale + 적절한 gateway bind와 Control UI origin 설정
- Secret은 임시 디렉터리에서 생성되어 클러스터에 직접 적용되며, 저장소 체크아웃에는 기록되지 않습니다

## 파일 구조

```
scripts/k8s/
├── deploy.sh                   # namespace + secret 생성, kustomize로 배포
├── create-kind.sh              # 로컬 Kind 클러스터 (docker/podman 자동 감지)
└── manifests/
    ├── kustomization.yaml      # Kustomize base
    ├── configmap.yaml          # openclaw.json + AGENTS.md
    ├── deployment.yaml         # 보안 강화가 적용된 Pod spec
    ├── pvc.yaml                # 10Gi 영구 스토리지
    └── service.yaml            # 18789의 ClusterIP
```
