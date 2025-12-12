# AKLP Infrastructure

AI-powered Kubernetes Learning Platform의 인프라 설정 및 오케스트레이션 레포지토리입니다.

## 전체 아키텍처

```text
┌──────────────────────────────────────────────────────────────────────────┐
│                           User (CLI)                                      │
└────────────────────────────────┬─────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                        AI Agent (aklp-agent)                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │ • 학습 가이드 제공                                                    │ │
│  │ • YAML 파일 생성/검토                                                 │ │
│  │ • kubectl 명령 실행 및 결과 분석                                       │ │
│  │ • 학습 자료 및 노트 자동 생성                                          │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
└────────┬─────────────────────┬─────────────────────┬─────────────────────┘
         │                     │                     │
         ▼                     ▼                     ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│   aklp-note     │   │   aklp-task     │   │   aklp-file     │
│   (포트 8002)    │   │   (포트 8003)    │   │   (포트 8004)    │
│                 │   │                 │   │                 │
│ • 학습 노트      │   │ • 학습 과제      │   │ • YAML 파일     │
│ • 세션 요약      │   │ • Batch 관리    │   │ • 로그 저장      │
│ • 명령어 기록    │   │ • 진행 상태      │   │ • 문서 관리      │
└────────┬────────┘   └────────┬────────┘   └────────┬────────┘
         │                     │                     │
         └─────────────────────┴─────────────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   aklp-postgres     │
                    │    (포트 5432)       │
                    │                     │
                    │ • aklp_note DB      │
                    │ • aklp_task DB      │
                    │ • aklp_file DB      │
                    │ • aklp_agent DB     │
                    └─────────────────────┘
```

## 빠른 시작

### 1. 모든 서비스 레포지토리 클론

```bash
# 작업 디렉토리 생성
mkdir aklp && cd aklp

# 인프라 레포지토리
git clone https://github.com/next-gen-dist-sys/aklp-infra.git

# 서비스 레포지토리들 (같은 레벨에 클론)
git clone https://github.com/next-gen-dist-sys/aklp-postgres.git
git clone https://github.com/next-gen-dist-sys/aklp-note.git
git clone https://github.com/next-gen-dist-sys/aklp-task.git
git clone https://github.com/next-gen-dist-sys/aklp-file.git
git clone https://github.com/next-gen-dist-sys/aklp-agent.git
```

### 2. 디렉토리 구조 확인

```text
aklp/
├── aklp-infra/          # 이 레포지토리 (Docker Compose, K8s 매니페스트)
├── aklp-postgres/       # PostgreSQL 서비스
├── aklp-note/           # Note 서비스 (학습 노트 관리)
├── aklp-task/           # Task 서비스 (할 일 관리)
├── aklp-file/           # File 서비스 (파일 관리)
└── aklp-agent/          # Agent 서비스 (AI 오케스트레이션)
```

### 3. 서비스 실행

```bash
cd aklp-infra
docker compose up
```

또는 백그라운드 실행:

```bash
docker compose up -d
```

---

## 서비스 구성

| 서비스        | 포트 | 데이터베이스 | 용도                            |
| ------------- | ---- | ------------ | ------------------------------- |
| aklp-postgres | 5432 | -            | 모든 서비스의 공용 데이터베이스 |
| aklp-agent    | 8001 | aklp_agent   | AI 오케스트레이션               |
| aklp-note     | 8002 | aklp_note    | 학습 노트, 세션 요약            |
| aklp-task     | 8003 | aklp_task    | 학습 과제 및 Batch 관리         |
| aklp-file     | 8004 | aklp_file    | 파일 업로드/다운로드            |

### API 문서

- Agent Service: [Agent 서비스 스웨거](http://localhost:8001/docs)
- Note Service: [Note 서비스 스웨거](http://localhost:8002/docs)
- Task Service: [Task 서비스 스웨거](http://localhost:8003/docs)
- File Service: [File 서비스 스웨거](http://localhost:8004/docs)

---

## Agent/CLI 통합 가이드

### session_id 관리

모든 Backend 서비스는 `session_id`로 데이터를 그룹화합니다. Agent는 세션 시작 시 UUID를 생성하고 모든 API 호출에 동일한 값을 사용해야 합니다.

```python
import uuid
SESSION_ID = str(uuid.uuid4())  # 세션당 한 번만 생성
```

### 학습 흐름 예시

```text
1. 세션 시작
   └── Agent: session_id 생성

2. 학습 주제 선택 (예: "Kubernetes Pod 기초")
   └── Agent: POST /api/v1/batches (학습 과제 생성)

3. 학습 자료 제공
   └── Agent: POST /api/v1/files (YAML 파일 업로드)
   └── Agent: POST /api/v1/notes (개념 설명 노트 생성)

4. 실습 진행
   └── User: 파일 다운로드 → kubectl apply
   └── Agent: 명령어 기록 → POST /api/v1/notes

5. 과제 완료
   └── Agent: PUT /api/v1/tasks/{id} (status: completed)

6. 세션 종료
   └── Agent: POST /api/v1/notes (학습 요약 생성)
```

### 서비스별 사용 패턴

#### Task Service (aklp-task)

| 상황              | API                      | 메서드 |
| ----------------- | ------------------------ | ------ |
| 새 주제 학습 시작 | `/api/v1/batches`        | POST   |
| 추가 과제 생성    | `/api/v1/tasks`          | POST   |
| 과제 완료 처리    | `/api/v1/tasks/{id}`     | PUT    |
| 진행 상황 확인    | `/api/v1/batches/latest` | GET    |
| 여러 과제 완료    | `/api/v1/tasks/bulk`     | PUT    |

#### Note Service (aklp-note)

| 상황             | API                            | 메서드 |
| ---------------- | ------------------------------ | ------ |
| 학습 내용 기록   | `/api/v1/notes`                | POST   |
| 명령어 기록 저장 | `/api/v1/notes`                | POST   |
| 세션 요약 생성   | `/api/v1/notes`                | POST   |
| 노트 업데이트    | `/api/v1/notes/{id}`           | PUT    |
| 세션 노트 조회   | `/api/v1/notes?session_id=...` | GET    |

#### File Service (aklp-file)

| 상황              | API                            | 메서드 |
| ----------------- | ------------------------------ | ------ |
| YAML 파일 생성    | `/api/v1/files`                | POST   |
| kubectl 출력 저장 | `/api/v1/files`                | POST   |
| 파일 다운로드     | `/api/v1/files/{id}/download`  | GET    |
| 파일 교체         | `/api/v1/files/{id}`           | PUT    |
| 세션 파일 목록    | `/api/v1/files?session_id=...` | GET    |

### 공통 응답 형식

#### 목록 조회 응답

```json
{
  "items": [...],
  "total": 25,
  "page": 1,
  "limit": 10,
  "total_pages": 3,
  "has_next": true,
  "has_prev": false
}
```

#### 에러 응답

```json
{
  "detail": "Resource not found"
}
```

---

## 개발 가이드

### 환경 변수 설정

각 서비스의 `.env` 파일을 확인하고 필요에 따라 수정하세요:

```bash
# 각 서비스 디렉토리에서
cp .env.example .env
```

### 개별 서비스 재시작

```bash
docker compose restart aklp-note
docker compose restart aklp-task
docker compose restart aklp-file
```

### 로그 확인

```bash
# 전체 로그
docker compose logs -f

# 특정 서비스 로그
docker compose logs -f aklp-task
```

### 데이터베이스 초기화

```bash
# 모든 컨테이너와 볼륨 삭제
docker compose down -v

# 다시 시작
docker compose up
```

### Health Check

```bash
# 모든 서비스 상태 확인
curl http://localhost:8001/health  # agent
curl http://localhost:8002/health  # note
curl http://localhost:8003/health  # task
curl http://localhost:8004/health  # file
```

---

## Kubernetes 배포

k3s 클러스터에 배포하려면 **[k8s/README.md](./k8s/README.md)** 를 참고하세요.

```bash
# 전체 배포
kubectl apply -k k8s/

# 상태 확인
kubectl get all -n aklp
```

---

## 기술 스택

| 항목          | 기술                    |
| ------------- | ----------------------- |
| 컨테이너      | Docker & Docker Compose |
| 데이터베이스  | PostgreSQL 17           |
| 백엔드 언어   | Python 3.12             |
| 웹 프레임워크 | FastAPI                 |
| ORM           | SQLAlchemy 2.0 (async)  |
| 패키지 관리   | uv                      |

---

## 관련 레포지토리

| 레포지토리                                                          | 설명              | 담당   |
| ------------------------------------------------------------------- | ----------------- | ------ |
| [aklp-postgres](https://github.com/next-gen-dist-sys/aklp-postgres) | PostgreSQL 서비스 | 나형진 |
| [aklp-note](https://github.com/next-gen-dist-sys/aklp-note)         | Note 서비스       | 나형진 |
| [aklp-task](https://github.com/next-gen-dist-sys/aklp-task)         | Task 서비스       | 나형진 |
| [aklp-file](https://github.com/next-gen-dist-sys/aklp-file)         | File 서비스       | 나형진 |
| [aklp-agent](https://github.com/next-gen-dist-sys/aklp-agent)       | Agent 서비스      | 김수민 |
| [aklp-cli](https://github.com/next-gen-dist-sys/aklp-cli)           | CLI 애플리케이션  | 박범식 |

## 라이선스

MIT
