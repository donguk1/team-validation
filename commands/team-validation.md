# Team Validation - 멀티 에이전트 프로젝트 검증

인자: $ARGUMENTS

## 개요

프로젝트를 멀티 에이전트 팀으로 검증합니다.
`.claude/agents/validation-*.md`에 정의된 에이전트를 사용하므로 **외부 스킬/플러그인 의존성이 없습니다.**

## 사용법

```
/team-validation <프로젝트 경로>    # 프로젝트 전체
/team-validation staged             # git staged 변경만
/team-validation current            # git unstaged 변경만
/team-validation #42                # PR #42 변경
/team-validation src/auth           # 특정 디렉토리
/team-validation .                  # 현재 디렉토리 전체
```

## 사용 가능한 에이전트 (프로젝트 내장)

| 에이전트 | 파일 | 역할 |
|---------|------|------|
| validation-architect | `.claude/agents/validation-architect.md` | 아키텍처/설계 패턴 검증 |
| validation-backend | `.claude/agents/validation-backend.md` | API/인증/트랜잭션 검증 |
| validation-code-quality | `.claude/agents/validation-code-quality.md` | SOLID/코드 스멜/네이밍 검증 |
| validation-bug-hunter | `.claude/agents/validation-bug-hunter.md` | 버그/보안 취약점 탐지 |
| validation-security | `.claude/agents/validation-security.md` | OWASP Top 10 심층 감사/CVE 스캔 |
| validation-frontend | `.claude/agents/validation-frontend.md` | React/Next.js 패턴/접근성/SEO |
| validation-game | `.claude/agents/validation-game.md` | 게임 아키텍처/성능/밸런스 검증 |
| validation-db-optimizer | `.claude/agents/validation-db-optimizer.md` | DB 쿼리/인덱스/스키마 검증 |
| validation-python | `.claude/agents/validation-python.md` | Python 도구 설정 검증 |
| validation-data | `.claude/agents/validation-data.md` | 데이터 파이프라인/ML 검증 |
| validation-product | `.claude/agents/validation-product.md` | 기능 완성도/UX 검증 |
| validation-dedup | `.claude/agents/validation-dedup.md` | 함수 로직 중복 탐지/리팩토링 제안 |

## 실행 순서

### Phase 0: 범위 결정 + 프로젝트 파악

1. `$ARGUMENTS` 파싱:
   - **비어있음** → AskUserQuestion으로 아래 중 택 1 질문:
     - 프로젝트 전체 (현재 디렉토리)
     - staged 변경 사항 (git diff --cached)
     - unstaged 변경 사항 (git diff)
     - 특정 디렉토리/파일 (사용자 입력)
   - **`staged`** → `git diff --cached --name-only`로 대상 파일 목록 수집
   - **`current`** → `git diff --name-only`로 대상 파일 목록 수집
   - **`#숫자`** → `gh pr view 숫자 --json files -q '.files[].path'`로 PR 변경 파일 목록 수집
   - **경로** → 해당 경로가 디렉토리면 하위 전체, 파일이면 해당 파일
2. 프로젝트의 CLAUDE.md, pyproject.toml, package.json, project.godot 등을 읽어 기술 스택 파악
3. 프로젝트 타입 분류

### Phase 1: 에이전트 선택 (최소 4개, 최대 10개)

**필수 4개** (모든 프로젝트):
- architect, code-quality, bug-hunter, security

**code-quality 분할**: 대상 파일이 30개 이상이면 code-quality를 2개 투입 (디렉토리 기준으로 파일 반씩 분담). 분할 시에도 총 에이전트 수는 최대 10개를 넘지 않는다.

프로젝트 특성에 따라 추가 에이전트를 선택합니다:

- **Python 백엔드** (FastAPI/Django): 필수4 + backend, dedup, db-optimizer, python (8개)
- **JS/TS 프론트엔드** (Next.js/React): 필수4 + frontend, dedup, product (7개)
- **게임** (Godot/Unity/Unreal): 필수4 + game (5개)
- **데이터/ML**: 필수4 + dedup, db-optimizer, python, data (8개)
- **풀스택/모노레포**: 필수4 + backend, frontend, dedup, db-optimizer, product + python(Python 존재 시) (9~10개)

선택한 구성을 사용자에게 테이블로 보여주고 확인받을 것.

### Phase 2: 팀 생성 및 병렬 실행

1. TaskCreate로 에이전트별 검증 태스크 생성
2. Task tool로 각 에이전트를 **병렬** 실행
   - `subagent_type: "general-purpose"` 사용
   - 각 에이전트의 역할과 프롬프트를 Task의 prompt에 포함
   - 대상 프로젝트 경로를 명확히 전달
   - **검증 범위** 정보를 각 에이전트에게 명시적으로 전달:
     - 전체 프로젝트: "프로젝트 전체를 분석하세요."
     - staged/current/PR: "다음 파일들의 변경 사항만 분석하세요: {파일 목록}"
     - 특정 디렉토리: "다음 디렉토리 내 파일만 분석하세요: {경로}"
   - **읽기 전용 분석만 수행** (코드 수정 금지)

### Phase 3: 결과 종합 보고

모든 에이전트 완료 후 통합 리포트:

```
## 🔍 Team Validation Report: {프로젝트명}

### 검증 범위
{범위 설명 — 예: "staged changes — 12 files", "전체 프로젝트", "src/auth 디렉토리", "PR #42 — 8 files"}

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

## 주의사항

- 코드 수정 없이 **읽기 전용 분석**만 수행
- 민감한 정보(.env 등)는 분석에서 제외
- 프로젝트 경로가 비어있으면 AskUserQuestion으로 범위를 물어볼 것
- GDScript는 싱글스레드이므로 race condition 지적은 무효

## 실행 지시

위 Phase 0~3을 순서대로 실행하세요. 대상: **$ARGUMENTS**
