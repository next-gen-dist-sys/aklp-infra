# AKLP Kubernetes 배포 상세 가이드

k3s 클러스터에서 AKLP 서비스를 운영하기 위한 상세 가이드입니다.

> **Quick Start**: 처음 설치하는 경우 [루트 README의 Quick Start](../README.md#quick-start)를 참고하세요.

## 서비스 접근

### NodePort 정보

| 서비스     | 내부 포트 | NodePort | 접근 URL             |
| ---------- | --------- | -------- | -------------------- |
| aklp-agent | 8001      | 30001    | http://NODE_IP:30001 |
| aklp-note  | 8002      | 30002    | http://NODE_IP:30002 |
| aklp-task  | 8003      | 30003    | http://NODE_IP:30003 |
| aklp-file  | 8004      | 30004    | http://NODE_IP:30004 |

### Health Check

```bash
# NODE_IP를 마스터 또는 워커 노드 IP로 변경
curl http://NODE_IP:30001/health  # agent
curl http://NODE_IP:30002/health  # note
curl http://NODE_IP:30003/health  # task
curl http://NODE_IP:30004/health  # file
```

### Swagger UI

- Agent: http://NODE_IP:30001/docs
- Note: http://NODE_IP:30002/docs
- Task: http://NODE_IP:30003/docs
- File: http://NODE_IP:30004/docs

## 디렉토리 구조

```text
k8s/
├── kustomization.yaml    # 전체 리소스 통합
├── namespace.yaml        # aklp 네임스페이스
├── postgres/             # PostgreSQL StatefulSet
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── statefulset.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── agent/                # Agent 서비스
├── task/                 # Task 서비스
├── note/                 # Note 서비스
└── file/                 # File 서비스
```

## 운영

### 로그 확인

```bash
# 특정 Pod 로그
kubectl logs -n aklp deployment/aklp-agent

# 실시간 로그 (-f)
kubectl logs -n aklp deployment/aklp-task -f

# PostgreSQL 로그
kubectl logs -n aklp statefulset/postgres
```

### Pod 재시작

```bash
# 특정 서비스 재시작
kubectl rollout restart deployment/aklp-agent -n aklp

# 모든 서비스 재시작
kubectl rollout restart deployment -n aklp
```

### 이미지 업데이트

```bash
# 최신 이미지로 업데이트
kubectl set image deployment/aklp-agent aklp-agent=ghcr.io/next-gen-dist-sys/aklp-agent:latest -n aklp

# 특정 SHA로 롤백
kubectl set image deployment/aklp-agent aklp-agent=ghcr.io/next-gen-dist-sys/aklp-agent:sha-abc1234 -n aklp
```

### 스케일링

```bash
# replica 수 조정
kubectl scale deployment/aklp-agent --replicas=2 -n aklp
```

## 삭제

### 전체 삭제

```bash
kubectl delete -k k8s/
```

### 특정 서비스만 삭제

```bash
kubectl delete -k k8s/agent/
```

### 데이터 포함 완전 삭제

```bash
# 리소스 삭제
kubectl delete -k k8s/

# PVC 삭제 (PostgreSQL 데이터 삭제됨!)
kubectl delete pvc -n aklp --all

# 네임스페이스 삭제
kubectl delete ns aklp
```

## 트러블슈팅

### Pod이 Pending 상태

```bash
# 이벤트 확인
kubectl describe pod POD_NAME -n aklp

# PVC 상태 확인 (스토리지 문제일 수 있음)
kubectl get pvc -n aklp
```

### Pod이 CrashLoopBackOff

```bash
# 로그 확인
kubectl logs POD_NAME -n aklp --previous

# DB 연결 문제일 가능성 - PostgreSQL 상태 확인
kubectl get pods -n aklp -l app=postgres
```

### ImagePullBackOff

```bash
# 에러 상세 확인
kubectl describe pod POD_NAME -n aklp | tail -20

# 이미지 존재 확인
docker pull ghcr.io/next-gen-dist-sys/aklp-agent:latest
```

이미지는 ghcr.io에 public으로 공개되어 있어 별도 인증 없이 pull 가능합니다.
GitHub Actions가 아직 실행되지 않았다면 이미지가 없을 수 있습니다.

## GitHub Actions

각 서비스 레포지토리에 `.github/workflows/build.yaml`이 있습니다.

- **트리거**: `main` 브랜치 push
- **이미지 태그**: `latest`, `sha-xxxxxxx`
- **레지스트리**: ghcr.io/next-gen-dist-sys/

이미지 자동 빌드 후, 클러스터에서 최신 이미지를 적용하려면:

```bash
kubectl rollout restart deployment -n aklp
```