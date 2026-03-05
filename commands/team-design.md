# Team Design - 멀티 에이전트 기능 설계

인자: $ARGUMENTS

## 개요

새 기능 요구사항을 받아 멀티 에이전트 팀이 병렬로 설계 산출물을 만듭니다.
`.claude/agents/design-*.md`에 정의된 에이전트를 사용하므로 **외부 스킬/플러그인 의존성이 없습니다.**

## 사용법

```
/team-design <기능 요구사항>
/team-design "유저 인증 시스템 추가"
/team-design "실시간 알림 기능 구현"
/team-design "결제 시스템 연동" --project jiwon-search-backend
```

## 사용 가능한 에이전트 (프로젝트 내장)

| 에이전트 | 파일 | 역할 |
|---------|------|------|
| design-architect | `.claude/agents/design-architect.md` | 전체 아키텍처 설계, 레이어 구조, Mermaid 다이어그램 |
| design-api | `.claude/agents/design-api.md` | API 엔드포인트 스펙, 요청/응답 스키마, 인증 흐름 |
| design-data-model | `.claude/agents/design-data-model.md` | DB 스키마/ERD, 테이블 관계, 마이그레이션 계획 |
| design-frontend | `.claude/agents/design-frontend.md` | UI 컴포넌트 구조, 페이지 흐름, 상태 관리 설계 |
| design-task-breakdown | `.claude/agents/design-task-breakdown.md` | 구현 태스크 분해, 의존성/순서, 예상 복잡도 |
| design-test-strategy | `.claude/agents/design-test-strategy.md` | 테스트 전략, 테스트 케이스 목록, E2E 시나리오 |
| design-devils-advocate | `.claude/agents/design-devils-advocate.md` | 설계 결정 반론, 빠진 고려사항, 대안 제시 |

## 실행 순서

### Phase 0: 기능 요구사항 파악 + 현재 프로젝트 분석

1. `$ARGUMENTS`에서 기능 요구사항 텍스트를 추출
2. 기존 `/team-plan` 결과(기획서)가 대화 컨텍스트에 있으면 해당 기획서를 컨텍스트로 활용 — 스코프, 기술 스택 추천, 사용자 스토리를 설계에 반영
3. `--project` 옵션이 있으면 해당 프로젝트, 없으면 현재 작업 디렉토리 사용
4. 프로젝트의 CLAUDE.md, pyproject.toml, package.json 등을 읽어 기술 스택 파악
4. 프로젝트 타입 분류:
   - **Python 백엔드** (FastAPI/Django): pyproject.toml 또는 requirements.txt에 fastapi/django 존재
   - **JS/TS 프론트엔드** (Next.js/React): package.json에 next/react 존재
   - **풀스택**: 백엔드 + 프론트엔드 모두 존재
   - **데이터/ML**: pandas, pycaret, sklearn 등 존재
   - **게임**: project.godot, *.csproj (Unity Assets/ 존재), *.uproject 존재
5. 기능 요구사항이 비어있으면 사용자에게 물어볼 것

### Phase 1: 에이전트 선택 (최소 4개, 최대 7개)

프로젝트 특성에 따라 적합한 에이전트를 선택합니다:

- **Python 백엔드** (FastAPI/Django): architect, api, data-model, task-breakdown, devils-advocate (5개)
- **JS/TS 프론트엔드** (Next.js/React): architect, frontend, task-breakdown, devils-advocate (4개)
- **게임** (Godot/Unity/Unreal): architect, task-breakdown, test-strategy, devils-advocate (4개)
- **풀스택**: architect, api, data-model, frontend, task-breakdown, test-strategy, devils-advocate (7개)
- **데이터/ML**: architect, data-model, task-breakdown, devils-advocate (4개)

선택한 구성을 사용자에게 테이블로 보여주고 확인받을 것.

### Phase 2: 팀 생성 및 병렬 실행

1. TaskCreate로 에이전트별 설계 태스크 생성
2. Task tool로 각 에이전트를 **병렬** 실행
   - `subagent_type: "team-validation:design-{role}"` 사용 (예: `"team-validation:design-architect"`)
   - 에이전트 프롬프트는 자동 로딩됨 — prompt에 에이전트 역할/책임을 포함하지 말 것
   - prompt에는 **컨텍스트만** 전달: 기능 요구사항 + 대상 프로젝트 경로 + 기술 스택

### Phase 3: 결과 종합 → 통합 설계 문서 생성

모든 에이전트 완료 후 통합 리포트:

```
## 📐 Team Design Report: {기능명}

### 팀 구성
| # | 에이전트 | 역할 |
|---|---------|------|

### 1. 아키텍처 설계 (design-architect)
(architect 에이전트 결과 요약)

### 2. API 설계 (design-api)
(api 에이전트 결과 요약 — 선택된 경우)

### 3. 데이터 모델 (design-data-model)
(data-model 에이전트 결과 요약 — 선택된 경우)

### 4. 프론트엔드 설계 (design-frontend)
(frontend 에이전트 결과 요약 — 선택된 경우)

### 5. 구현 태스크 (design-task-breakdown)
(task-breakdown 에이전트 결과)

### 6. 테스트 전략 (design-test-strategy)
(test-strategy 에이전트 결과 — 선택된 경우)

### 7. 설계 반론 검토 (design-devils-advocate)
(devils-advocate 에이전트 결과)
```

에이전트 결과 간 충돌이 있으면 직접 조율하여 일관성 있는 문서로 통합할 것.

### Phase 4: 사용자 확인

통합 설계 문서를 보여준 뒤 사용자에게 확인:

> "설계 문서가 완성되었습니다. 어떻게 진행할까요?"
> 1. **문서만 유지** — 설계 문서를 참고용으로 보존
> 2. **코드 구현 진행** — 설계를 기반으로 실제 코드 구현 시작

AskUserQuestion 도구를 사용하여 선택지를 제공할 것.

### Phase 5: (선택) 코드 구현 진행

사용자가 "코드 구현 진행"을 선택한 경우:

1. design-task-breakdown 에이전트의 태스크 목록을 기반으로 구현 시작
2. 오케스트레이터가 직접 코드를 작성 (design 에이전트가 아닌 메인 세션에서 수행)
3. Phase별로 순서대로 구현
4. 각 태스크 완료 시 사용자에게 진행 상황 공유

## 주의사항

- Phase 2에서는 **읽기 + 설계 문서 생성**만 수행 (기존 코드 수정 금지)
- 민감한 정보(.env 등)는 분석에서 제외
- 기능 요구사항이 비어있으면 사용자에게 물어볼 것
- DB 스키마 변경은 설계 단계에서는 제안만 하고, 실행은 사용자 확인 후 Phase 5에서
- 에이전트 간 결과가 충돌하면 Phase 3에서 조율

## 실행 지시

위 Phase 0~5를 순서대로 실행하세요. 기능 요구사항: **$ARGUMENTS**
