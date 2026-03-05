# Team Plan - 멀티 에이전트 프로젝트 기획

인자: $ARGUMENTS

## 개요

프로젝트 아이디어나 요구사항을 받아 멀티 에이전트 팀이 병렬로 기획 산출물을 만듭니다.
`.claude/agents/plan-*.md`에 정의된 에이전트를 사용하므로 **외부 스킬/플러그인 의존성이 없습니다.**

## 사용법

```
/team-plan <프로젝트 아이디어 또는 요구사항>
/team-plan "인터뷰 스케줄링 웹앱 — Google 로그인, 캘린더 연동"
/team-plan "사내 지식 관리 시스템"
```

## 사용 가능한 에이전트 (프로젝트 내장)

| 에이전트 | 파일 | 역할 |
|---------|------|------|
| plan-scope | `.claude/agents/plan-scope.md` | MVP 스코프 정의, MoSCoW 우선순위, 페이즈 로드맵 |
| plan-tech-stack | `.claude/agents/plan-tech-stack.md` | 기술 스택 추천, 대안 비교, Build vs Buy |
| plan-user-story | `.claude/agents/plan-user-story.md` | 사용자 스토리, 인수 조건, 여정 맵 |
| plan-devils-advocate | `.claude/agents/plan-devils-advocate.md` | 기획 반론, 빠진 요구사항, 스코프 리스크 |

## 실행 순서

### Phase 0: 요구사항 파악 + 기존 프로젝트 확인

1. `$ARGUMENTS`에서 프로젝트 아이디어/요구사항 텍스트를 추출
2. 기존 프로젝트가 있는지 확인:
   - 현재 작업 디렉토리에 CLAUDE.md, pyproject.toml, package.json 등이 있으면 기존 프로젝트로 판단
   - 기존 프로젝트가 있으면 기술 스택과 현재 기능을 파악 (에이전트에게 컨텍스트로 전달)
   - 기존 프로젝트가 없으면 순수 기획 모드 (새 프로젝트)
3. 요구사항이 비어있으면 사용자에게 물어볼 것

### Phase 1: 에이전트 확인 (항상 4개)

기획 에이전트는 항상 4개 모두 실행합니다:

| # | 에이전트 | 역할 |
|---|---------|------|
| 1 | plan-scope | MVP 스코프 정의, MoSCoW 우선순위, 페이즈 로드맵 |
| 2 | plan-tech-stack | 기술 스택 추천, 대안 비교, Build vs Buy |
| 3 | plan-user-story | 사용자 스토리, 인수 조건, 여정 맵 |
| 4 | plan-devils-advocate | 기획 반론, 빠진 요구사항, 스코프 리스크 |

선택한 구성을 사용자에게 테이블로 보여주고 확인받을 것.

### Phase 2a: 기획 에이전트 3개 병렬 실행

1. TaskCreate로 scope, tech-stack, user-story 3개 태스크 생성
2. Task tool로 3개 에이전트를 **병렬** 실행
   - `subagent_type: "team-validation:plan-{role}"` 사용 (예: `"team-validation:plan-scope"`)
   - 에이전트 프롬프트는 자동 로딩됨 — prompt에 에이전트 역할/책임을 포함하지 말 것
   - prompt에는 **컨텍스트만** 전달: 프로젝트 아이디어/요구사항 + 기존 프로젝트 경로(있는 경우) + 기술 스택(있는 경우)
   - **읽기 전용 분석 + 기획 문서 생성만 수행** (기존 코드 수정 금지)

### Phase 2b: 반론 에이전트 실행 (Phase 2a 완료 후)

1. Phase 2a의 3개 에이전트 결과를 수집
2. plan-devils-advocate 에이전트를 실행
   - `subagent_type: "team-validation:plan-devils-advocate"`
   - prompt에 **Phase 2a 결과 전체를 포함**: 프로젝트 요구사항 + scope 결과 + tech-stack 결과 + user-story 결과
   - devils-advocate는 이 3개 결과를 기반으로 반론·검증 수행

### Phase 3: 결과 종합 → 통합 기획서 생성

모든 에이전트 완료 후 통합 기획서:

```
## 📋 Team Plan Report: {프로젝트명}

### 팀 구성
| # | 에이전트 | 역할 |
|---|---------|------|

### 1. 스코프 정의 (plan-scope)
(scope 에이전트 결과 요약)

### 2. 기술 스택 추천 (plan-tech-stack)
(tech-stack 에이전트 결과 요약)

### 3. 사용자 스토리 (plan-user-story)
(user-story 에이전트 결과 요약)

### 4. 기획 반론 검토 (plan-devils-advocate)
(devils-advocate 에이전트 결과 요약)

### 종합 기획 요약
- MVP 범위: ...
- 핵심 기술 스택: ...
- 주요 사용자 시나리오: ...
- 주의사항: ...
```

에이전트 결과 간 충돌이 있으면 직접 조율하여 일관성 있는 문서로 통합할 것.

### Phase 4: 사용자 확인

통합 기획서를 보여준 뒤 사용자에게 확인:

> "기획서가 완성되었습니다. 어떻게 진행할까요?"
> 1. **기획서만 유지** — 기획서를 참고용으로 보존
> 2. **/team-design으로 설계 진행** — 기획서를 기반으로 상세 설계 시작

AskUserQuestion 도구를 사용하여 선택지를 제공할 것.

## 주의사항

- Phase 2에서는 **읽기 + 기획 문서 생성**만 수행 (기존 코드 수정 금지)
- 민감한 정보(.env 등)는 분석에서 제외
- 요구사항이 비어있으면 사용자에게 물어볼 것
- 에이전트 간 결과가 충돌하면 Phase 3에서 조율

## 실행 지시

위 Phase 0~4를 순서대로 실행하세요. 요구사항: **$ARGUMENTS**
