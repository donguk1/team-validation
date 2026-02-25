# team-validation

Claude Code 멀티 에이전트 프로젝트 검증 도구.

외부 스킬/플러그인 의존성 없이, 프로젝트에 포함된 에이전트 정의만으로 동작합니다.

## 설치

프로젝트 루트에 `.claude/` 디렉토리를 복사하면 됩니다:

```bash
# 방법 1: 직접 복사
cp -r .claude/commands/team-validation.md <your-project>/.claude/commands/
cp -r .claude/agents/validation-*.md <your-project>/.claude/agents/

# 방법 2: 서브모듈 (선택)
git submodule add https://github.com/donguk1/team-validation.git .team-validation
```

## 사용법

```
/team-validation <프로젝트 경로>
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

## 프로젝트 타입별 기본 구성

- **Python 백엔드** (FastAPI/Django): architect, backend, code-quality, bug-hunter, db-optimizer, python (6개)
- **JS/TS 프론트엔드** (Next.js/React): architect, code-quality, bug-hunter, product (4개)
- **데이터/ML**: architect, code-quality, db-optimizer, python, data (5개)
- **풀스택/모노레포**: 최대 8개 전부 활용

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
.claude/
├── commands/
│   └── team-validation.md          # /team-validation 슬래시 커맨드
└── agents/
    ├── validation-architect.md
    ├── validation-backend.md
    ├── validation-code-quality.md
    ├── validation-bug-hunter.md
    ├── validation-db-optimizer.md
    ├── validation-python.md
    ├── validation-data.md
    └── validation-product.md
```

## 요구사항

- [Claude Code](https://claude.ai/code) CLI
- 외부 스킬/플러그인 불필요
