# 플러그인 스케일링 계획

## 현황 (2026-03-05)

- **repo**: `donguk1/team-validation`
- **agents/**: 26개 (validation 15 + design 7 + plan 4) — flat 구조
- **commands/**: 3개 (`/team-plan`, `/team-design`, `/team-validation`)

## 문제

- Claude Code 플러그인의 `agents/` 서브디렉토리는 코드상 재귀 로딩되지만 **공식 미지원**
- `/plugin` 관리 UI의 에이전트 목록 함수가 비재귀적 → 서브디렉토리 에이전트가 UI에 안 보일 수 있음
- 에이전트 이름이 `plugin:subdir:name` 형태로 바뀜 (기존 참조 전부 수정 필요)
- 40~50개 이상 에이전트가 되면 flat 구조는 가독성 한계

## 스케일링 전략: 플러그인 분리 (C안 채택)

카테고리별로 **독립 플러그인(독립 repo)**으로 분리:

```
donguk1/team-validation    → /team-validation  (기존 코드 검증)
donguk1/team-design        → /team-design      (새 기능 설계)
donguk1/team-refactor      → /team-refactor    (리팩토링) [미래]
donguk1/team-migration     → /team-migration   (마이그레이션) [미래]
donguk1/team-review        → /team-review      (PR 리뷰) [미래]
```

### 장점
- 각 플러그인의 `agents/`가 10개 내외 → flat으로 충분
- 사용자가 필요한 것만 골라 설치 가능
- 독립 버전 관리, 독립 배포
- 플러그인 시스템 제약 안에서 자연스럽게 스케일링

### 분리 기준
- 카테고리별 에이전트가 **10개 이상**이 되거나
- 새 카테고리(refactor, migration 등)가 추가될 때

### 분리 시 작업 목록
1. 새 repo 생성 (`donguk1/team-{category}`)
2. `.claude-plugin/plugin.json` + `marketplace.json` 작성
3. 해당 카테고리의 agents + command 이동
4. 원본 repo에서 해당 파일 삭제
5. README 각각 업데이트
6. 마켓플레이스 등록

## 버전 관리

### 규칙

- **Semantic Versioning** 사용: `MAJOR.MINOR.PATCH`
- `plugin.json`의 `version` 필드가 **플러그인 캐시 업데이트의 기준** — 버전을 올리지 않으면 캐시가 갱신되지 않음

### 버전 올리는 기준

| 변경 유형 | 버전 | 예시 |
|----------|------|------|
| 에이전트/커맨드 추가·삭제 | **MINOR** | 1.0.0 → 1.1.0 |
| 에이전트 프롬프트 수정, 버그 수정 | **PATCH** | 1.1.0 → 1.1.1 |
| 호환성 깨지는 변경 (커맨드 이름 변경 등) | **MAJOR** | 1.1.1 → 2.0.0 |
| README, SCALING.md 등 문서만 수정 | 올리지 않음 | - |

### 릴리스 체크리스트

1. `plugin.json`의 `version` 업데이트
2. SCALING.md 버전 히스토리에 새 항목 추가 (plugin.json과 버전 동기화 확인)
3. git commit + push
4. `claude plugin marketplace update team-validation`
5. `claude plugin update team-validation@team-validation`
6. Claude Code 재시작

### 버전 히스토리

| 버전 | 날짜 | 변경 내용 |
|------|------|----------|
| 1.0.0 | 2026-02-25 | 초기 릴리스 — validation 에이전트 9개, `/team-validation` 커맨드 |
| 1.1.0 | 2026-02-25 | design 에이전트 6개, `/team-design` 커맨드 추가 |
| 1.1.1 | 2026-02-25 | marketplace.json description 업데이트, LICENSE 추가, 에이전트 description 형식 수정 |
| 1.2.0 | 2026-02-28 | security/frontend/game 에이전트 3개 추가, 유연한 범위 지정 (staged/current/PR#/디렉토리), 필수 에이전트 4개 도입, 프리셋 업데이트 (4~10개), 로컬 스킬 통합 |
| 1.2.1 | 2026-02-28 | 검증 리포트 기반 품질 개선 — gh pr 명령어 수정, 에이전트 언어/형식 통일, design 에이전트 READ-ONLY 명시, bug-hunter↔security 역할 경계 정리, README allowedTools에서 Bash 제거 |
| 1.2.2 | 2026-02-28 | 에이전트 tools 최소화 — 16개 Read/Glob/Grep만, python/security만 Bash 유지, Task 전면 제거 |
| 1.3.0 | 2026-03-01 | validation-testing, validation-devops 에이전트 추가, 프리셋 업데이트 (4~12개) |
| 1.3.1 | 2026-03-01 | 오케스트레이터 subagent_type 전환 — "general-purpose" → "team-validation:{agent}" 전용 타입, 토큰 절약 |
| 1.3.2 | 2026-03-01 | 셀프 검증 기반 품질 개선 — .env 금지 전체 에이전트 적용, security 영어 전환, Bash 화이트리스트 추가, bug-hunter↔security OWASP 역할 분리, frontend↔product a11y 역할 분리, 데이터/ML 프리셋 testing 추가, #숫자 파싱 검증, Analysis Process 6단계 통일, .gitignore 추가, 문서 동기화 (4~12개) |
| 1.3.3 | 2026-03-03 | 에이전트 프롬프트 High Impact 개선 — validation 14개 "문제+이유+수정 제안" 3요소 출력 필수화, 점수 산정 기준(Critical/Warning 건수 기반) 명시, design 5개 "설계 결정사항(결정/이유/대안)" 테이블 추가 |
| 1.3.4 | 2026-03-03 | 마스터 가이드 기반 대규모 개선 — 복잡도 기반 모델 분배 (architect/security/bug-hunter → opus), 20개 에이전트 프롬프트에 AI 코드 감사·슬롭스쿼팅·기술부채·보안 품질 게이트 등 마스터 가이드 14개 섹션 인사이트 통합 |
| 1.4.0 | 2026-03-03 | 반론 전용 에이전트 2개 추가 (validation-devils-advocate, design-devils-advocate), design 에이전트 전체 opus 전환, 필수 에이전트 4→5개, 프리셋 업데이트 (validation 5~13개, design 4~7개) |
| 1.5.0 | 2026-03-05 | plan 에이전트 4개 추가 (plan-scope, plan-tech-stack, plan-user-story, plan-devils-advocate), `/team-plan` 커맨드 추가, `plan → design → validation` 전체 흐름 완성 |

### 자동 업데이트

- `known_marketplaces.json`에 `"autoUpdate": true` 설정됨
- Claude Code 시작 시 + 10분 간격으로 백그라운드 체크
- 자동 업데이트가 동작하려면 **반드시 version을 올려야** 함

## 현재 결정

- **지금은 분리하지 않음** — 26개는 prefix로 관리 가능 (plan 4 + design 7 + validation 15)
- 카테고리가 4개 이상 되거나 에이전트 총합이 35개 넘으면 분리 시작
