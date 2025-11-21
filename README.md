# AKLP Infrastructure

AI-powered Kubernetes Learning Platform의 인프라 설정 및 오케스트레이션 레포지토리입니다.

## 📂 구조

이 레포지토리는 AKLP의 모든 마이크로서비스를 로컬에서 실행하기 위한 Docker Compose 설정을 포함합니다. 추후 k8s or k3s 관련 설정 파일도 포함될 예정입니다.

## 🚀 빠른 시작

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
git clone https://github.com/next-gen-dist-sys/aklp-agent.git
```

### 2. 디렉토리 구조 확인

클론 후 디렉토리 구조는 다음과 같아야 합니다:

```text
aklp/
├── aklp-infra/          # 이 레포지토리 (Docker Compose, K8s 매니페스트)
├── aklp-postgres/       # PostgreSQL 서비스
├── aklp-note/           # Note 서비스 (파일 저장 및 관리)
├── aklp-task/           # Task 서비스 (할 일 관리)
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

## 🔧 서비스 구성

### PostgreSQL (aklp-postgres)

- **포트**: 5432
- **데이터베이스**: `aklp_note`, `aklp_task`, `aklp_agent`
- **용도**: 모든 서비스의 공용 데이터베이스

### Note Service (aklp-note)

- **포트**: 8000
- **기능**: AI 대화 세션 로깅 및 노트 관리
- **API**: `http://localhost:8000/docs`

### Task Service (aklp-task)

- **포트**: 8001
- **기능**: 학습 과정에서 필요한 할 일 관리

### Agent Service (aklp-agent)

- **포트**: 8002 (예정)
- **기능**: AI 오케스트레이션

## 📋 개발 가이드

### 환경 변수 설정

각 서비스의 `.env` 파일을 확인하고 필요에 따라 수정하세요:

```bash
# 각 서비스 디렉토리에서
cp .env.example .env
```

### 개별 서비스 재시작

```bash
docker compose restart aklp-note
docker compose restart aklp-postgres
```

### 로그 확인

```bash
# 전체 로그
docker compose logs -f

# 특정 서비스 로그
docker compose logs -f aklp-note
```

### 데이터베이스 초기화

```bash
# 모든 컨테이너와 볼륨 삭제
docker compose down -v

# 다시 시작
docker compose up
```

## 🛠 기술 스택

- **Docker & Docker Compose**: 로컬 개발 환경
- **PostgreSQL 17**: 데이터베이스
- **Python 3.12**: 서비스 개발 언어
- **FastAPI**: 웹 프레임워크
- **uv**: Python 패키지 관리

## 📚 관련 레포지토리

- [aklp-postgres](https://github.com/next-gen-dist-sys/aklp-postgres) - PostgreSQL 서비스
- [aklp-note](https://github.com/next-gen-dist-sys/aklp-note) - Note 서비스
- [aklp-task](https://github.com/next-gen-dist-sys/aklp-task) - Task 서비스
- [aklp-agent](https://github.com/next-gen-dist-sys/aklp-agent) - Agent 서비스
- [aklp-cli](https://github.com/next-gen-dist-sys/aklp-cli) - CLI 애플리케이션

## 👥 팀

- **CLI**: 박범식
- **AI Agent**: 김수민
- **Note Service**: 나형진
- **Task Service**: 나형진

## 📄 라이선스

MIT License
