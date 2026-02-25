# 플러그인 스케일링 계획

## 현황 (2026-02-25)

- **repo**: `donguk1/team-validation`
- **agents/**: 15개 (validation 9 + design 6) — flat 구조
- **commands/**: 2개 (`/team-validation`, `/team-design`)

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

## 현재 결정

- **지금은 분리하지 않음** — 15개는 prefix로 관리 가능
- 카테고리가 3개 이상 되거나 에이전트 총합이 25개 넘으면 분리 시작
