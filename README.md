# 🥋 SQL Dojo

로컬에서 돌리는 **PostgreSQL 학습 연습장**. 실제 PostgreSQL 에 쿼리를 실행하며 14개 챕터 · 96문제를 풀고, 즉시 자동 채점을 받습니다.

- 문서: [PRD](docs/PRD.md) · [문제 작성 가이드](docs/CONTENT_GUIDE.md)
- 스택: React 19 + Vite + TypeScript / Spring Boot 4.1 (Java 21) / PostgreSQL 16 (Docker)

## 실행 방법

필요한 것: Docker Desktop, Java 21, Node.js 20+

```bash
# 1) PostgreSQL 실행 (최초 1회 롤·DB 생성)
docker compose up -d

# 2) 백엔드 (http://localhost:8088) — 최초 실행 시 실습 DB 스키마와 데이터를 자동 생성
cd backend
./gradlew bootRun          # Windows PowerShell: .\gradlew.bat bootRun

# 3) 프론트엔드 (http://localhost:5173) — 새 터미널에서
cd frontend
npm install
npm run dev
```

브라우저에서 **http://localhost:5173** 을 엽니다.

| 포트 | 용도 |
|---|---|
| 5173 | 프론트엔드 (Vite, `/api` 는 백엔드로 프록시) |
| 8088 | 백엔드 API |
| 5433 | PostgreSQL (`psql -h localhost -p 5433 -U learner sqldojo_practice`, 비밀번호 `learner`) |

## 기능

- **학습**: 챕터별 개념 노트 + 문제 (조회 / 데이터 변경 / 구조 설계)
- **문제 풀이**: SQL 에디터 (PostgreSQL 문법 강조, 테이블·컬럼 자동완성)
  - `Ctrl+Enter` 실행 · `Ctrl+Shift+Enter` 제출 · `EXPLAIN` 실행 계획 보기
  - 채점 피드백: 컬럼 수 불일치, 빠진 행 / 기대하지 않은 행, 정렬 순서 차이
  - 단계별 힌트, 모범 답안, 제출 기록, 문제별 메모, 작성 중인 SQL 자동 저장
- **대시보드**: 전체·난이도별 진도, 이어서 풀기, 12주 활동 기록, 오답 노트
- **플레이그라운드**: 자유 실험, 예제 쿼리, 커밋 모드(변경을 실제로 저장), 실습 DB 초기화

## 안전장치 (샌드박스)

- 사용자 SQL 은 권한이 제한된 `learner` 롤로 **별도의 실습 DB** 에서 실행 (진도 DB 접근 불가, 실습 테이블 DROP/ALTER 불가)
- 모든 실행은 트랜잭션 안에서 수행 후 **자동 ROLLBACK** (플레이그라운드 커밋 모드 제외)
- `COMMIT`, `SET`, `COPY`, `DO` 등 트랜잭션·세션 제어 문장 차단
- 실행 시간 5초 제한, 결과 최대 1,000행 표시

## 프로젝트 구조

```
├─ docker-compose.yml          # postgres:16 (호스트 포트 5433)
├─ db/init/                    # 롤·데이터베이스 생성 (컨테이너 최초 기동 시)
├─ backend/
│  └─ src/main/
│     ├─ java/com/sqldojo/
│     │  ├─ sandbox/           # SqlGuard(문장 차단), SandboxExecutor(롤백 실행)
│     │  ├─ grading/           # Grader, ResultComparator(결과셋 비교)
│     │  ├─ content/           # YAML 챕터/문제 로더
│     │  ├─ practice/          # 실습 DB 초기화·스키마 조회
│     │  ├─ progress/          # 진도·제출 이력 저장
│     │  └─ api/               # REST 컨트롤러
│     └─ resources/
│        ├─ content/*.yaml     # 챕터·문제 (여기에 문제 추가)
│        ├─ practice/          # 실습 DB schema.sql, seed.sql, grants.sql
│        └─ db/migration/      # 진도 DB (Flyway)
└─ frontend/src/
   ├─ pages/                   # 대시보드, 커리큘럼, 챕터, 문제, 플레이그라운드
   └─ components/              # SqlEditor, ResultView, SchemaBrowser …
```

## 자주 쓰는 작업

| 하고 싶은 것 | 방법 |
|---|---|
| 실습 데이터를 처음 상태로 | 플레이그라운드 → "실습 DB 초기화" (또는 `POST /api/practice/reset`) |
| 진도 초기화 | 대시보드 하단 "진도 초기화" |
| 문제 추가·수정 | `backend/src/main/resources/content/*.yaml` 편집 후 백엔드 재시작 ([가이드](docs/CONTENT_GUIDE.md)) |
| DB 를 완전히 새로 | `docker compose down -v && docker compose up -d` 후 백엔드 재시작 |
| 백엔드 테스트 | `cd backend && ./gradlew test` |
