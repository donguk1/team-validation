# team-validation

Claude Code 멀티 에이전트 프로젝트 검증 & 설계 플러그인.

전문 에이전트가 병렬로 프로젝트를 검증하거나, 새 기능을 설계합니다. 외부 의존성 없음.

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

## 커맨드

### `/team-validation` — 프로젝트 검증

4~10개 에이전트가 병렬로 기존 코드를 읽기 전용 분석합니다. 유연한 범위 지정을 지원합니다.

```
/team-validation <프로젝트 경로>    # 프로젝트 전체
/team-validation staged             # git staged 변경만
/team-validation current            # git unstaged 변경만
/team-validation #42                # PR #42 변경
/team-validation src/auth           # 특정 디렉토리
/team-validation .                  # 현재 디렉토리 전체
```

실행하면:
1. 범위 결정 + 프로젝트 기술 스택을 자동 파악
2. 적합한 에이전트 4~10개를 선택하여 사용자에게 확인
3. 모든 에이전트를 병렬 실행 (읽기 전용 분석)
4. 검증 범위가 표시된 종합 리포트 생성

### `/team-design` — 기능 설계

3~6개 에이전트가 병렬로 새 기능의 설계 산출물을 만듭니다.

```
/team-design <기능 요구사항>
/team-design "유저 인증 시스템 추가"
/team-design "실시간 알림 기능 구현"
```

실행하면:
1. 기능 요구사항 파악 + 현재 프로젝트 분석
2. 적합한 에이전트 3~6개를 선택하여 사용자에게 확인
3. 모든 에이전트를 병렬 실행 (설계 산출물 생성)
4. 통합 설계 문서 생성
5. 사용자 선택: "문서만 유지" or "코드 구현 진행"

## 검증 에이전트 (validation-*)

프로젝트 특성에 따라 최소 4개 ~ 최대 10개 에이전트가 자동 선택됩니다.

| 에이전트 | 역할 |
|---------|------|
| `validation-architect` | 아키텍처/의존성/설계 패턴 검증 |
| `validation-backend` | API/인증/트랜잭션 검증 |
| `validation-code-quality` | SOLID/코드 스멜/네이밍 검증 |
| `validation-bug-hunter` | 버그/보안 취약점 탐지 |
| `validation-security` | OWASP Top 10 심층 감사/CVE 스캔 |
| `validation-frontend` | React/Next.js 패턴/접근성/SEO |
| `validation-game` | 게임 아키텍처/성능/밸런스 검증 |
| `validation-db-optimizer` | DB 쿼리/인덱스/스키마 검증 |
| `validation-python` | Python 프로젝트 설정 검증 (uv/ruff/pytest) |
| `validation-data` | 데이터 파이프라인/ML 검증 |
| `validation-product` | 기능 완성도/UX 검증 |
| `validation-dedup` | 함수 로직 중복 탐지/리팩토링 제안 |

### 프로젝트 타입별 기본 구성

| 프로젝트 타입 | 에이전트 | 수 |
|-------------|---------|---|
| Python 백엔드 (FastAPI/Django) | 필수4 + backend, dedup, db-optimizer, python | 8 |
| JS/TS 프론트엔드 (Next.js/React) | 필수4 + frontend, dedup, product | 7 |
| 게임 (Godot/Unity/Unreal) | 필수4 + game | 5 |
| 데이터/ML | 필수4 + dedup, db-optimizer, python, data | 8 |
| 풀스택/모노레포 | 필수4 + backend, frontend, dedup, db-optimizer, product, python | 10 |

> **필수 4개**: architect, code-quality, bug-hunter, security
>
> **code-quality 분할**: 대상 파일 30개 이상이면 code-quality 2개 투입

## 설계 에이전트 (design-*)

프로젝트 특성에 따라 최소 3개 ~ 최대 6개 에이전트가 자동 선택됩니다.

| 에이전트 | 역할 |
|---------|------|
| `design-architect` | 전체 아키텍처 설계, 레이어 구조, Mermaid 다이어그램 |
| `design-api` | API 엔드포인트 스펙, 요청/응답 스키마, 인증 흐름 |
| `design-data-model` | DB 스키마/ERD, 테이블 관계, 마이그레이션 계획 |
| `design-frontend` | UI 컴포넌트 구조, 페이지 흐름, 상태 관리 설계 |
| `design-task-breakdown` | 구현 태스크 분해, 의존성/순서, 예상 복잡도 |
| `design-test-strategy` | 테스트 전략, 테스트 케이스 목록, E2E 시나리오 |

### 프로젝트 타입별 기본 구성

| 프로젝트 타입 | 에이전트 | 수 |
|-------------|---------|---|
| Python 백엔드 (FastAPI/Django) | architect, api, data-model, task-breakdown | 4 |
| JS/TS 프론트엔드 (Next.js/React) | architect, frontend, task-breakdown | 3 |
| 게임 (Godot/Unity/Unreal) | architect, task-breakdown, test-strategy | 3 |
| 풀스택 | architect, api, data-model, frontend, task-breakdown, test-strategy | 6 |
| 데이터/ML | architect, data-model, task-breakdown | 3 |

## 검증 vs 설계 비교

| | `/team-validation` | `/team-design` |
|---|---|---|
| 목적 | 기존 코드 문제 찾기 | 새 기능 설계 |
| 입력 | 경로/staged/current/PR#/디렉토리 | 기능 요구사항 텍스트 |
| 동작 | 읽기 전용 분석 | 읽기 + 설계 문서 생성 |
| 출력 | 점수/이슈 리포트 (범위 표시 포함) | 설계 문서/다이어그램/태스크 |
| 후속 | 이슈 수정 | 문서만 or 코드 구현까지 선택 |

## 출력 예시

### 검증 (/team-validation)

실제 Python 데이터 프로젝트에서 테스트한 결과:

```
🔍 Team Validation Report: materialized_view_manager

검증 범위: 전체 프로젝트

| 에이전트 | 🔴 Critical | 🟡 Warning | 🟢 Suggestion | 점수 |
|---------|------------|-----------|--------------|------|
| architect | 3 | 11 | 6 | 6.5/10 |
| code-quality | 5 | 10 | 7 | 5.5/10 |
| bug-hunter | 4 | 8 | 6 | 7.0/10 |
| security | 2 | 5 | 3 | 8.0/10 |
| dedup | 3 | 4 | 2 | 4.0/10 |
| db-optimizer | 4 | 10 | 6 | 7.5/10 |
| python | 4 | 10 | 6 | 5.0/10 |

📊 Overall Score: 6.2/10
```

각 에이전트가 `파일:라인` 수준으로 구체적인 이슈를 보고하며, Critical/Warning/Suggestion으로 분류합니다.

### 설계 (/team-design)

```
📐 Team Design Report: 유저 인증 시스템

| # | 에이전트 | 역할 |
|---|---------|------|
| 1 | design-architect | 아키텍처 설계 |
| 2 | design-api | API 설계 |
| 3 | design-data-model | 데이터 모델 설계 |
| 4 | design-task-breakdown | 태스크 분해 |

1. 아키텍처 설계 — 레이어 구조, Mermaid 다이어그램
2. API 설계 — 엔드포인트, 스키마, 인증 흐름
3. 데이터 모델 — ERD, 마이그레이션 계획
4. 구현 태스크 — Phase별 태스크, 의존성
```

## 구조

```
team-validation/
├── .claude-plugin/
│   ├── plugin.json              # 플러그인 메타데이터
│   └── marketplace.json         # 마켓플레이스 등록 정보
├── commands/
│   ├── team-validation.md       # /team-validation 슬래시 커맨드
│   └── team-design.md           # /team-design 슬래시 커맨드
├── agents/
│   ├── validation-architect.md     # 검증: 아키텍처
│   ├── validation-backend.md       # 검증: 백엔드
│   ├── validation-code-quality.md  # 검증: 코드 품질
│   ├── validation-bug-hunter.md    # 검증: 버그 탐지
│   ├── validation-security.md      # 검증: 보안 심층 감사
│   ├── validation-frontend.md      # 검증: 프론트엔드
│   ├── validation-game.md          # 검증: 게임
│   ├── validation-db-optimizer.md  # 검증: DB 최적화
│   ├── validation-python.md        # 검증: Python 설정
│   ├── validation-data.md          # 검증: 데이터/ML
│   ├── validation-product.md       # 검증: 프로덕트
│   ├── validation-dedup.md         # 검증: 중복 탐지
│   ├── design-architect.md         # 설계: 아키텍처
│   ├── design-api.md               # 설계: API
│   ├── design-data-model.md        # 설계: 데이터 모델
│   ├── design-frontend.md          # 설계: 프론트엔드
│   ├── design-task-breakdown.md    # 설계: 태스크 분해
│   └── design-test-strategy.md     # 설계: 테스트 전략
└── README.md
```

## 권한 설정 (권장)

에이전트 실행 시 파일 읽기/검색마다 권한을 묻는 게 번거롭다면, 프로젝트의 `.claude/settings.json`에 다음을 추가하세요:

```json
{
  "permissions": {
    "allowedTools": [
      "Read",
      "Glob",
      "Grep",
      "Bash"
    ]
  }
}
```

이 설정은 **플러그인 설치 시 자동 적용되지 않으므로**, 검증 대상 프로젝트에서 직접 설정해야 합니다.

## 요구사항

- [Claude Code](https://claude.ai/code) CLI
- 외부 스킬/플러그인 불필요
