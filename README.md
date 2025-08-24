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

#### SQLite 버전 (간단한 테스트용)

```bash
docker compose -f sqlite-docker-compose.yml up -d
```

#### Redis + PostgreSQL 버전 (고성능 프로덕션용)

```bash
docker compose -f redis-postgres-docker-compose.yml up -d
```

### 3. 접속

브라우저에서 `http://localhost:5678`로 접속하여 n8n을 사용하세요.

## 📁 프로젝트 구조

```
n8n/
├── README.md                            # 이 파일
├── .env.example                         # 환경변수 예제
├── .gitignore                           # Git 무시 파일 목록
├── postgres-docker-compose.yml         # PostgreSQL 기반 구성
├── sqlite-docker-compose.yml           # SQLite 기반 구성
└── redis-postgres-docker-compose.yml   # Redis + PostgreSQL 고성능 구성
```

## ⚙️ 구성 옵션

### 볼륨이 2개 필요한 이유

Docker Compose에서 `postgres_data`와 `n8n_data` 두 개의 볼륨을 분리하는 이유:

1. **데이터 분리**: 
   - `postgres_data`: PostgreSQL 데이터베이스 파일 (`/var/lib/postgresql/data`)
   - `n8n_data`: n8n 애플리케이션 데이터 (`/home/node/.n8n`) - 워크플로우, 자격증명, 설정

2. **독립적 관리**: 데이터베이스와 애플리케이션을 개별적으로 백업/복구 가능

3. **확장성**: PostgreSQL을 다른 서버로 이전하거나 독립적으로 스케일링 가능

4. **보안**: 각 볼륨에 다른 접근 권한과 백업 정책 적용

5. **유지보수**: 컴포넌트 업그레이드 시 데이터 손실 위험 최소화

### PostgreSQL 버전 특징 (권장)

- **데이터베이스**: PostgreSQL 16 Alpine
- **볼륨**: 데이터 지속성 보장
- **헬스체크**: 데이터베이스 및 애플리케이션 상태 모니터링
- **의존성**: PostgreSQL이 준비된 후 n8n 시작

#### PostgreSQL이 권장되는 이유

1. **성능**: 대용량 데이터와 동시 연결 처리에 최적화
2. **안정성**: 트랜잭션 안정성과 ACID 규정 준수
3. **확장성**: 워크플로우 수가 증가해도 성능 저하 없음
4. **동시성**: 여러 사용자의 동시 작업 지원
5. **백업**: pg_dump를 통한 안정적인 백업/복구
6. **프로덕션 준비**: 실제 운영 환경에서 검증된 안정성
7. **확장 가능**: 필요 시 Redis를 추가하여 큐 모드와 병렬 실행 지원

### SQLite 버전 특징

- **데이터베이스**: 내장 SQLite (파일 기반)
- **간단함**: 별도 데이터베이스 서버 불필요
- **용도**: 개발/테스트 환경에 적합
- **제한사항**: 동시 연결 제한, 대용량 데이터 처리 시 성능 저하

### Redis + PostgreSQL 버전 특징 (고성능 프로덕션용)

- **데이터베이스**: PostgreSQL 16 Alpine
- **캐시/큐**: Redis 7 Alpine
- **실행 모드**: 큐 모드 (`EXECUTIONS_MODE: 'queue'`)
- **성능**: 워크플로우 병렬 실행으로 처리량 대폭 향상
- **확장성**: 워커 노드 추가로 수평적 확장 가능
- **안정성**: 작업 큐를 통한 안정적인 워크플로우 실행
- **용도**: 대규모 프로덕션 환경, 고성능이 필요한 경우

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
