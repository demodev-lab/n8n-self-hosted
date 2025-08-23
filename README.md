# n8n Docker 환경

n8n 워크플로우 자동화 도구를 Docker 환경에서 실행하기 위한 설정입니다.

## 🚀 빠른 시작

### 1. 환경 설정

먼저 환경변수를 설정하세요:

```bash
cp .env.example .env
```

`.env` 파일을 편집하여 보안 설정을 변경하세요:

- `N8N_ENCRYPTION_KEY`: 강력한 암호화 키로 변경
- `POSTGRES_PASSWORD`: 안전한 PostgreSQL 비밀번호로 변경

### 2. 실행 방법

#### PostgreSQL 버전 (권장)

```bash
docker compose -f postgres-docker-compose.yml up -d
```

#### SQLite 버전 (간단한 버전)

```bash
docker compose -f sqlite-docker-compose.yml up -d
```

### 3. 접속

브라우저에서 `http://localhost:5678`로 접속하여 n8n을 사용하세요.

## 📁 프로젝트 구조

```
n8n/
├── README.md                     # 이 파일
├── .env.example                  # 환경변수 예제
├── .gitignore                    # Git 무시 파일 목록
├── postgres-docker-compose.yml  # PostgreSQL 기반 구성
└── sqlite-docker-compose.yml    # SQLite 기반 구성
```

## ⚙️ 구성 옵션

### PostgreSQL 버전 특징

- **데이터베이스**: PostgreSQL 16 Alpine
- **볼륨**: 데이터 지속성 보장
- **헬스체크**: 데이터베이스 및 애플리케이션 상태 모니터링
- **의존성**: PostgreSQL이 준비된 후 n8n 시작

### SQLite 버전 특징

- **데이터베이스**: 내장 SQLite (파일 기반)
- **간단함**: 별도 데이터베이스 서버 불필요
- **용도**: 개발/테스트 환경에 적합

### 환경변수

| 변수명               | 설명                      | 기본값     |
| -------------------- | ------------------------- | ---------- |
| `GENERIC_TIMEZONE`   | 시간대 설정               | Asia/Seoul |
| `N8N_ENCRYPTION_KEY` | 데이터 암호화 키          | 변경 필요  |
| `POSTGRES_USER`      | PostgreSQL 사용자명       | n8n        |
| `POSTGRES_PASSWORD`  | PostgreSQL 비밀번호       | 변경 필요  |
| `POSTGRES_DB`        | PostgreSQL 데이터베이스명 | n8n        |

## 🔧 관리 명령어

### 서비스 상태 확인

```bash
docker compose -f postgres-docker-compose.yml ps
```

### 로그 확인

```bash
docker compose -f postgres-docker-compose.yml logs -f
```

### 서비스 중지

```bash
docker compose -f postgres-docker-compose.yml down
```

### 데이터 완전 제거 (주의!)

```bash
docker compose -f postgres-docker-compose.yml down -v
```

## 🛡️ 보안 고려사항

1. **암호화 키**: `.env` 파일의 `N8N_ENCRYPTION_KEY`를 강력한 값으로 변경
2. **데이터베이스 비밀번호**: `POSTGRES_PASSWORD` 변경
3. **네트워크**: 외부 접근이 필요한 경우 포트 설정 검토
4. **백업**: 중요한 워크플로우는 정기적으로 백업

## 📊 모니터링

두 구성 모두 헬스체크가 설정되어 있어 서비스 상태를 모니터링할 수 있습니다:

- PostgreSQL: `pg_isready` 명령으로 데이터베이스 상태 확인
- n8n: `/healthz` 엔드포인트로 애플리케이션 상태 확인

## 🚨 문제해결

### 자주 발생하는 문제

1. **포트 충돌**: 5678 포트가 사용중인 경우 docker-compose 파일에서 포트 변경
2. **권한 문제**: Docker 볼륨 권한 확인
3. **데이터베이스 연결**: PostgreSQL 컨테이너가 완전히 시작되기까지 대기

### 로그 확인

```bash
# n8n 로그
docker compose -f postgres-docker-compose.yml logs n8n

# PostgreSQL 로그
docker compose -f postgres-docker-compose.yml logs postgres
```
