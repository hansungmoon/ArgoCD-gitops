# ArgoCD GitOps Demo

이 디렉토리는 ArgoCD GitOps 워크플로우 테스트용 예제입니다.

## 파일 구조

```
argocd-demo/
├── deployment.yaml            # Nginx Deployment (Plain Manifest)
├── service.yaml               # ClusterIP Service (Plain Manifest)
├── argocd-application.yaml    # ArgoCD Application CRD
└── myapp/                     # Custom Helm Chart
    ├── Chart.yaml             # Chart 메타데이터
    ├── values.yaml            # 기본 값
    ├── values-dev.yaml        # Dev 환경 값
    ├── values-staging.yaml    # Staging 환경 값
    ├── values-prod.yaml       # Production 환경 값
    └── templates/             # Kubernetes 템플릿
        ├── deployment.yaml
        ├── service.yaml
        └── NOTES.txt
```

## 1. Plain Manifest로 ArgoCD 사용

### ArgoCD Application 생성

```bash
kubectl apply -f argocd-application.yaml
```

### GitOps 워크플로우 테스트

1. **Git에서 replicas 수정**: `deployment.yaml`의 `replicas: 3`으로 변경
2. **Git Push**: 변경사항을 GitHub에 push
3. **자동 동기화**: ArgoCD가 자동으로 감지하고 클러스터에 적용
4. **확인**: `kubectl get pods -l app=nginx-gitops`로 3개의 Pod 확인

## 2. Helm Chart로 ArgoCD 사용

### Helm Chart 검증

```bash
# Chart 유효성 검사
helm lint myapp

# 템플릿 렌더링 테스트
helm template myapp myapp -f myapp/values-dev.yaml
```

### 환경별 배포

**Dev 환경 (1 replica, 최소 리소스):**
```yaml
source:
  path: myapp
  helm:
    valueFiles:
    - values-dev.yaml
```

**Staging 환경 (2 replicas, 중간 리소스):**
```yaml
source:
  path: myapp
  helm:
    valueFiles:
    - values-staging.yaml
```

**Production 환경 (3 replicas, 높은 리소스):**
```yaml
source:
  path: myapp
  helm:
    valueFiles:
    - values-prod.yaml
```

## Self-Heal 테스트

```bash
# 수동으로 replica 변경
kubectl scale deployment myapp --replicas=5

# ArgoCD가 자동으로 Git 상태로 복구하는지 확인
kubectl get pods -l app.kubernetes.io/name=myapp -w
```
