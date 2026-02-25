# Team Validation - 멀티 에이전트 프로젝트 검증

인자: $ARGUMENTS

## 개요

프로젝트를 멀티 에이전트 팀으로 검증합니다.
`.claude/agents/validation-*.md`에 정의된 에이전트를 사용하므로 **외부 스킬/플러그인 의존성이 없습니다.**

## 사용법

```
/team-validation <프로젝트 경로>
/team-validation jiwon-sync-dio
/team-validation jiwon-search-backend
/team-validation .
```

## 사용 가능한 에이전트 (프로젝트 내장)

| 에이전트 | 파일 | 역할 |
|---------|------|------|
| validation-architect | `.claude/agents/validation-architect.md` | 아키텍처/설계 패턴 검증 |
| validation-backend | `.claude/agents/validation-backend.md` | API/인증/트랜잭션 검증 |
| validation-code-quality | `.claude/agents/validation-code-quality.md` | SOLID/코드 스멜/네이밍 검증 |
| validation-bug-hunter | `.claude/agents/validation-bug-hunter.md` | 버그/보안 취약점 탐지 |
| validation-db-optimizer | `.claude/agents/validation-db-optimizer.md` | DB 쿼리/인덱스/스키마 검증 |
| validation-python | `.claude/agents/validation-python.md` | Python 도구 설정 검증 |
| validation-data | `.claude/agents/validation-data.md` | 데이터 파이프라인/ML 검증 |
| validation-product | `.claude/agents/validation-product.md` | 기능 완성도/UX 검증 |
| validation-dedup | `.claude/agents/validation-dedup.md` | 함수 로직 중복 탐지/리팩토링 제안 |

## 실행 순서

### Phase 0: 프로젝트 파악

1. `$ARGUMENTS`로 전달된 프로젝트 경로 확인
2. 프로젝트의 CLAUDE.md, pyproject.toml, package.json 등을 읽어 기술 스택 파악
3. 프로젝트 타입 분류

### Phase 1: 에이전트 선택 (최소 4개, 최대 9개)

프로젝트 특성에 따라 위 에이전트 중 적합한 것을 선택합니다:

- **Python 백엔드** (FastAPI/Django): architect, backend, code-quality, bug-hunter, dedup, db-optimizer, python (7개)
- **JS/TS 프론트엔드** (Next.js/React): architect, code-quality, bug-hunter, dedup, product (5개)
- **데이터/ML 프로젝트**: architect, code-quality, dedup, db-optimizer, python, data (6개)
- **풀스택/모노레포**: 최대 9개 전부 활용

선택한 구성을 사용자에게 테이블로 보여주고 확인받을 것.

### Phase 2: 팀 생성 및 병렬 실행

1. TeamCreate로 `validation-{프로젝트명}` 팀 생성
2. TaskCreate로 에이전트별 검증 태스크 생성
3. Task tool로 각 에이전트를 **병렬** 실행
   - `subagent_type: "general-purpose"` 사용
   - 각 에이전트의 역할과 프롬프트를 Task의 prompt에 포함
   - 대상 프로젝트 경로를 명확히 전달
   - **읽기 전용 분석만 수행** (코드 수정 금지)

### Phase 3: 결과 종합 보고

모든 에이전트 완료 후 통합 리포트:

```
## 🔍 Team Validation Report: {프로젝트명}

### 팀 구성
| # | 에이전트 | 역할 |
|---|---------|------|

### 검증 요약
| 에이전트 | 🔴 Critical | 🟡 Warning | 🟢 Suggestion | 점수 |
|---------|------------|-----------|--------------|------|

### 🔴 Critical Issues (즉시 수정 필요)
- [에이전트명] 파일:라인 - 설명

### 🟡 Warnings (개선 권장)
- [에이전트명] 파일:라인 - 설명

### 🟢 Suggestions (선택적 개선)
- [에이전트명] 파일:라인 - 설명

### 📊 Overall Score: X/10
```

### Phase 4: 팀 정리

- 모든 에이전트에 shutdown 요청
- TeamDelete로 리소스 정리

## 주의사항

- 코드 수정 없이 **읽기 전용 분석**만 수행
- 민감한 정보(.env 등)는 분석에서 제외
- 프로젝트 경로가 비어있으면 사용자에게 물어볼 것

## 실행 지시

위 Phase 0~4를 순서대로 실행하세요. 대상 프로젝트: **$ARGUMENTS**
