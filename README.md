# team-validation

Claude Code 멀티 에이전트 프로젝트 검증 플러그인.

4~9개 전문 에이전트가 병렬로 프로젝트를 검증합니다. 외부 스킬/플러그인 의존성 없음.

## 설치

```bash
# 1. 마켓플레이스 등록 (최초 1회)
claude plugin marketplace add donguk1/team-validation

# 2. 플러그인 설치
claude plugin install team-validation
```

## 삭제

```bash
claude plugin uninstall team-validation
claude plugin marketplace remove team-validation
```

## 사용법

```
/team-validation <프로젝트 경로>
/team-validation my-backend-project
/team-validation .
```

실행하면:
1. 프로젝트 기술 스택을 자동 파악
2. 적합한 에이전트 4~9개를 선택하여 사용자에게 확인
3. 모든 에이전트를 병렬 실행 (읽기 전용 분석)
4. 종합 리포트 생성

## 에이전트 구성

프로젝트 특성에 따라 최소 4개 ~ 최대 9개 에이전트가 자동 선택됩니다.

| 에이전트 | 역할 |
|---------|------|
| `validation-architect` | 아키텍처/의존성/설계 패턴 검증 |
| `validation-backend` | API/인증/트랜잭션 검증 |
| `validation-code-quality` | SOLID/코드 스멜/네이밍 검증 |
| `validation-bug-hunter` | 버그/보안 취약점 탐지 |
| `validation-db-optimizer` | DB 쿼리/인덱스/스키마 검증 |
| `validation-python` | Python 프로젝트 설정 검증 (uv/ruff/pytest) |
| `validation-data` | 데이터 파이프라인/ML 검증 |
| `validation-product` | 기능 완성도/UX 검증 |
| `validation-dedup` | 함수 로직 중복 탐지/리팩토링 제안 |

## 프로젝트 타입별 기본 구성

| 프로젝트 타입 | 에이전트 | 수 |
|-------------|---------|---|
| Python 백엔드 (FastAPI/Django) | architect, backend, code-quality, bug-hunter, dedup, db-optimizer, python | 7 |
| JS/TS 프론트엔드 (Next.js/React) | architect, code-quality, bug-hunter, dedup, product | 5 |
| 데이터/ML | architect, code-quality, dedup, db-optimizer, python, data | 6 |
| 풀스택/모노레포 | 최대 9개 전부 활용 | 9 |

## 출력 예시

실제 Python 데이터 프로젝트에서 테스트한 결과:

```
🔍 Team Validation Report: materialized_view_manager

| 에이전트 | 🔴 Critical | 🟡 Warning | 🟢 Suggestion | 점수 |
|---------|------------|-----------|--------------|------|
| architect | 3 | 11 | 6 | 6.5/10 |
| code-quality | 5 | 10 | 7 | 5.5/10 |
| bug-hunter | 4 | 8 | 6 | 7.0/10 |
| dedup | 3 | 4 | 2 | 4.0/10 |
| db-optimizer | 4 | 10 | 6 | 7.5/10 |
| python | 4 | 10 | 6 | 5.0/10 |

📊 Overall Score: 5.9/10
```

각 에이전트가 `파일:라인` 수준으로 구체적인 이슈를 보고하며, Critical/Warning/Suggestion으로 분류합니다.

## 구조

```
team-validation/
├── .claude-plugin/
│   ├── plugin.json              # 플러그인 메타데이터
│   └── marketplace.json         # 마켓플레이스 등록 정보
├── commands/
│   └── team-validation.md       # /team-validation 슬래시 커맨드
├── agents/
│   ├── validation-architect.md
│   ├── validation-backend.md
│   ├── validation-code-quality.md
│   ├── validation-bug-hunter.md
│   ├── validation-db-optimizer.md
│   ├── validation-python.md
│   ├── validation-data.md
│   ├── validation-product.md
│   └── validation-dedup.md
└── README.md
```

## 요구사항

- [Claude Code](https://claude.ai/code) CLI
- 외부 스킬/플러그인 불필요
