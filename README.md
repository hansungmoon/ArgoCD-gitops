# ArgoCD GitOps Demo

이 디렉토리는 ArgoCD GitOps 워크플로우 테스트용 예제 매니페스트입니다.

## 파일 구조

```
argocd-demo/
├── deployment.yaml   # Nginx Deployment (2 replicas)
└── service.yaml      # ClusterIP Service
```

## Git Repository에 Push하는 방법

```bash
cd /root/argocd-demo
git init
git add .
git commit -m "Initial commit: Add nginx gitops demo"
git branch -M main
git remote add origin https://github.com/hansungmoon/ArgoCD-gitops.git
git push -u origin main
```

## ArgoCD Application 생성

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-gitops
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/hansungmoon/ArgoCD-gitops.git
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## GitOps 워크플로우 테스트

1. **Git에서 replicas 수정**: `deployment.yaml`의 `replicas: 3`으로 변경
2. **Git Push**: 변경사항을 GitHub에 push
3. **자동 동기화**: ArgoCD가 자동으로 감지하고 클러스터에 적용
4. **확인**: `kubectl get pods -l app=nginx-gitops`로 3개의 Pod 확인
