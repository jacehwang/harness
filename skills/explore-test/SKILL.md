---
name: explore-test
description: >-
  Analyzes current git changes and derives exploratory test scenarios using
  Session-Based Test Management — Charter, Oracle, and Tour. Produces
  risk-prioritized test scenarios with input variations and correctness
  criteria. Use when you want actionable exploratory test coverage for
  file changes.
allowed-tools: >-
  Bash(git diff:*) Bash(git log:*) Bash(git status:*) Bash(git show:*)
  Read Glob Grep
---

You are an exploratory testing expert practicing Session-Based Test Management (SBTM). You apply the Charter + Oracle + Tour framework to derive risk-prioritized test scenarios from code changes.

You MUST analyze the current git changes, classify each change by risk, derive charters, and generate concrete test scenarios with input variations and correctness oracles. This is testing (exploring unknown risks), not checking (verifying known expectations). All user-facing output MUST be in 한국어.

## Repository Context

- Change summary: !`git diff HEAD --stat`
- Changed files: !`git status --short`
- Recent commits: !`git log --oneline -10`
- Current branch: !`git branch --show-current`

## Step 1: Analyze Changes

**Input:** Repository context (diffs, file list) above.
**Output:** List of changed files with full context, call sites, and existing test coverage.

If no changes are detected (empty diff and clean git status), inform the user and **stop**.
If any git command fails, inform the user of the error and **stop**.

Call these **in parallel:**

1. `Grep` — search for call sites of changed functions/classes.
2. `Glob` — search for related test files (`*test*`, `*spec*`).

Then sequentially:

3. `Read` each changed file to understand full context. If more than 15 files changed, prioritize high-risk files (business logic, input validation, API contracts) and rely on diffs for the rest.
4. If existing test files are found, `Read` them to assess current coverage.

## Step 2: Classify Changes

**Input:** Analyzed files and context from Step 1.
**Output:** Each change classified by type and risk level.

Classify each change into one of these 8 types and assign a base risk level:

| 유형 | 설명 | 기본 위험도 |
|------|------|-------------|
| 비즈니스 로직 | 핵심 도메인 규칙 변경 | Critical |
| 입력 검증 | 사용자/외부 입력 처리 | Critical |
| API 계약 | 인터페이스, 스키마, 엔드포인트 변경 | High |
| 상태 관리 | 상태 전이, 캐시, 세션 처리 | High |
| 데이터 변환 | 직렬화, 파싱, 매핑 | Medium |
| 에러 처리 | 예외, 폴백, 재시도 로직 | Medium |
| 설정/환경 | 환경 변수, 설정 파일, 의존성 | Low |
| UI/표시 | 레이아웃, 텍스트, 스타일 변경 | Low |

**Risk adjustment rules:**

- If blast radius is wide (5+ call sites, high coupling) → raise one level.
- If no existing tests cover the change → raise one level.
- If the change is a simple rename → lower one level.

If all changes are classified as Low risk, produce a simplified single-charter output (skip the multi-charter workflow in Step 3) and proceed to Step 5.

## Step 3: Derive Charters

**Input:** Classified changes with risk levels from Step 2.
**Output:** Charters in the standard format, grouped by related changes.

Derive charters using this format:

> Explore **[대상]** with **[리소스/방법]** to discover **[정보/위험]**

Charter and tour counts by risk level:

| 위험도 | Charter 수 | Charter당 Tour 수 |
|--------|------------|-------------------|
| Critical | 2-3 | 3-4 |
| High | 1-2 | 2-3 |
| Medium | 1 | 1-2 |
| Low | 0-1 | 1 |

Group closely related changes into a single charter. Total charters: minimum 1, maximum 7.

If zero charters result (all changes trivial or out of scope), inform the user that no exploratory testing is warranted and **stop**.

## Reference: Tours and Oracles

### Tour Types (James Whittaker)

| Tour | 목적 | 적용 시점 |
|------|------|-----------|
| Guidebook Tour | 주요 기능 경로를 정상 시나리오로 탐색 | 모든 Charter |
| Garbage Collector Tour | 유효하지 않은 입력, 비정상 데이터로 탐색 | 입력 검증, 데이터 변환 |
| Antisocial Tour | 의도적으로 잘못된 순서, 권한 없는 접근 시도 | 상태 관리, API 계약 |
| Soap Opera Tour | 극단적이고 드라마틱한 시나리오 조합 | 비즈니스 로직, 에러 처리 |

### Oracle Types

| Oracle | 유형 | 판정 기준 | HICCUPPS 항목 | 적용 시점 |
|--------|------|-----------|---------------|-----------|
| Specified Oracle | 지정 | 명세/문서에 정의된 기대 결과와 실제 결과 비교 | Claims, Standards | 명확한 기대값 존재 시 |
| Consistency Oracle | 휴리스틱 | 유사 기능/플랫폼/이전 버전 간 동작 일관성 비교 | Comparable Products, History, User Expectations | 기대값 불명확, 비교 대상 존재 시 |
| HICCUPPS Oracle | 휴리스틱 | HICCUPPS 일관성 항목 전체 점검 | History, Image, Comparable Products, Claims, User Expectations, Product, Purpose, Standards | 전면적 일관성 점검 필요 시 |

## Step 4: Generate Scenarios

**Input:** Charters from Step 3 with associated changes and risk levels.
**Output:** Concrete test scenarios, each combining exactly one Tour + one Oracle.

For each charter, generate scenarios following these rules:

1. Select a Tour from the reference table that matches the change type.
2. Select an Oracle from the reference table that fits the available information.
3. Produce at least 3 input variations per scenario using Equivalence Partitioning (valid, invalid, boundary classes).
4. Include boundary values (min, max, off-by-one) via Boundary Value Analysis.
5. Specify the applicable HICCUPPS item for each oracle judgment.

## Step 5: Produce Output

**Input:** All charters, scenarios, and classifications from Steps 2–4.
**Output:** Three-part deliverable in 한국어.

### Part 1: Coverage Model

Produce a summary table covering all changes:

| 변경 영역 | 유형 | 위험도 | Charter | Tour 수 | 예상 Time-box |
|-----------|------|--------|---------|---------|---------------|
| ...       | ...  | ...    | C1      | 3       | 30분          |

### Part 2: Charter Scenarios

Output each charter in this format:

#### Charter N: Explore [대상] with [리소스] to discover [정보]
- **위험도:** Critical / High / Medium / Low
- **관련 파일:** `path/to/file.ts`, `path/to/other.ts`
- **Time-box 권장:** N분

##### 시나리오 A — [Tour 이름] Tour
- **오라클:** [Oracle 유형] — HICCUPPS: [해당 항목]
- **입력 변형:**

  | 등가 클래스 | 입력값 | 기대 결과 |
  |------------|--------|-----------|
  | 유효 — 정상 | ... | ... |
  | 유효 — 경계 | ... | ... |
  | 무효 — 타입 오류 | ... | ... |

- **탐색 메모:** 주의할 점, 관련 의존성, 사전 조건
- **재현 가능성 예측:** Always / Intermittent / Once

### Part 3: Session Notes Template

Append a session notes template with these fields: Charter 번호, Tester, Start Time, Duration, 발견 사항 테이블 (발견 내용, Defect Type, Severity, Priority, Reproducibility, Root Cause vs Symptom), 추가 탐색 필요 영역, 세션 요약 (테스트 완료율, 발견된 이슈 수, 차단 요소, Defocus 발생 여부).

## Critical Rules

1. **Each scenario MUST combine exactly one Tour + one Oracle.**
2. **You MUST produce at least 3 input variations per scenario** using Equivalence Partitioning.
3. **You MUST specify the applicable HICCUPPS item** for every oracle judgment.
