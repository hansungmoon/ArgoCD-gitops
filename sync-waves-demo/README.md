# Sync Waves 데모

ArgoCD Sync Waves를 사용한 **순차적 배포** 실습

## Sync Wave란?

리소스 배포 순서를 제어하는 기능입니다.

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "0"  # 낮은 번호부터 배포
```

## 배포 순서

```
Wave 0: Database (postgres)
  ↓
Wave 1: DB Migration Job (PreSync Hook)
  ↓
Wave 2: Application (webapp)
```

## Hook 종류

- **PreSync**: 동기화 전 실행 (DB 마이그레이션)
- **Sync**: 일반 리소스 배포
- **PostSync**: 동기화 후 실행 (테스트, 알림)
- **SyncFail**: 동기화 실패 시 실행

## 배포 방법

```bash
# ArgoCD Application 생성
kubectl apply -f sync-waves-app.yaml

# 배포 순서 확인
kubectl get pods -n sync-waves -w
```

## 예상 동작

1. Postgres Pod 시작 (Wave 0)
2. Postgres 준비 완료 대기
3. DB Migration Job 실행 (Wave 1, PreSync Hook)
4. 테이블 생성 및 데이터 삽입
5. WebApp Pod 시작 (Wave 2)
6. 모든 리소스 Healthy 상태
