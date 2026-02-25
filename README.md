# team-validation

Claude Code 멀티 에이전트 프로젝트 검증 플러그인.

4~8개 전문 에이전트가 병렬로 프로젝트를 검증합니다. 외부 의존성 없음.

## 설치

```bash
claude install donguk1/team-validation
```

## 사용법

```
/team-validation <프로젝트 경로>
/team-validation jiwon-sync-dio
/team-validation .
```

## 에이전트 구성

프로젝트 특성에 따라 최소 4개 ~ 최대 8개 에이전트가 자동 선택됩니다.

| 에이전트 | 역할 |
|---------|------|
| `validation-architect` | 아키텍처/설계 패턴 검증 |
| `validation-backend` | API/인증/트랜잭션 검증 |
| `validation-code-quality` | SOLID/코드 스멜/네이밍 검증 |
| `validation-bug-hunter` | 버그/보안 취약점 탐지 |
| `validation-db-optimizer` | DB 쿼리/인덱스/스키마 검증 |
| `validation-python` | Python 도구 설정 검증 |
| `validation-data` | 데이터 파이프라인/ML 검증 |
| `validation-product` | 기능 완성도/UX 검증 |
| `validation-dedup` | 함수 로직 중복 탐지/리팩토링 제안 |

## 프로젝트 타입별 기본 구성

- **Python 백엔드** (FastAPI/Django): architect, backend, code-quality, bug-hunter, dedup, db-optimizer, python (7개)
- **JS/TS 프론트엔드** (Next.js/React): architect, code-quality, bug-hunter, dedup, product (5개)
- **데이터/ML**: architect, code-quality, dedup, db-optimizer, python, data (6개)
- **풀스택/모노레포**: 최대 9개 전부 활용

## 출력 예시

```
🔍 Team Validation Report: my-project

| 에이전트 | 🔴 Critical | 🟡 Warning | 🟢 Suggestion | 점수 |
|---------|------------|-----------|--------------|------|
| architect | 0 | 2 | 3 | 8/10 |
| backend | 1 | 1 | 2 | 7/10 |
| ...     | ... | ... | ... | ... |

📊 Overall Score: 7.5/10
```

## 구조

```
team-validation/
├── .claude-plugin/
│   └── plugin.json                 # 플러그인 메타데이터
├── commands/
│   └── team-validation.md          # /team-validation 슬래시 커맨드
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
