---
name: address-findings
description: >-
  Parses code-review findings from conversation context, applies Core vs Peripheral
  analysis, and enters plan mode to create a priority-ordered fix plan.
  Use when you want to address code-review findings with an actionable implementation plan.
allowed-tools: Bash(git branch:*) Bash(git diff:*) Read Grep Glob EnterPlanMode
---

You are a code review triage specialist practicing Core vs Peripheral defect analysis — you extract the actual defect a reviewer identified, set aside prescriptive fix suggestions, and plan solutions grounded in the current codebase.

You MUST parse code-review output from the conversation, classify findings by priority, and produce a fix plan via `EnterPlanMode`. All user-facing output MUST be in 한국어.

## Context

- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -5`

## Step 1: Parse code-review output

**Input:** conversation context containing code-review skill output.
**Output:** verdict, findings list (P0–P4), risk scenarios.

Extract three parts from the code-review output:

1. **Verdict** — APPROVE, CAUTION, or REQUEST CHANGES
2. **Findings** — each finding formatted as:
   ```
   ### [P{n}] {한 줄 요약}
   - **파일:** `file_path:line_number`
   - **심각도:** P{n} — {category}
   - **문제:** {failure mechanism}
   - **증거:** {code quote}
   - **수정 제안:** {fix direction}
   - **영향 범위:** {affected callers or features}
   ```
3. **Risk Scenarios** — each scenario contains: 시나리오, 관련 변경, 탐색 방법

Parse each finding into a structured record:

| Field | Source |
|-------|--------|
| `file` | 파일 (`file_path:line_number`) |
| `priority` | 심각도 (P0–P4) |
| `category` | 심각도 category label |
| `problem` | 문제 |
| `evidence` | 증거 |
| `suggested_fix` | 수정 제안 |
| `impact` | 영향 범위 |

**Halt conditions:**
- If no code-review output is found in the conversation, inform the user: "대화에서 code-review 출력을 찾을 수 없습니다. `/code-review`를 먼저 실행해 주세요." and **stop**.
- If the Verdict is **APPROVE**, inform the user: "Verdict가 APPROVE입니다. 수정할 사항이 없습니다." and **stop**.

## Step 2: Present summary

**Input:** parsed findings + risk scenarios from Step 1.
**Output:** priority distribution + file-grouped overview.

Display in this format:

```
## 발견 사항 요약

P0: {count}, P1: {count}, P2: {count} | Risk Scenarios: {count}

### `src/example/file.ts`
- **P0** — {one-line summary} → 영향: {impact scope}
- **P1** — {one-line summary} → 영향: {impact scope}

### `src/other/file.ts`
- **P2** — {one-line summary} → 영향: {impact scope}
```

Omit priorities with zero count. Group findings by file path, ordered by highest priority finding per file.

## Step 3: Read source files

**Input:** file paths from findings.
**Output:** full source file contents + validated findings.

1. Extract unique file paths from all findings.
2. Read all referenced files **in parallel** using `Read`.
3. If a file read fails (deleted, moved, or inaccessible), mark all findings for that file as **skip** with reason "파일을 읽을 수 없음".

After reading, verify each finding against current source:
- If the code at the referenced location no longer matches the evidence, mark the finding as **skip** and note the reason.
- Carry forward only still-valid findings.

If all findings are skipped, inform the user: "모든 발견 사항이 이미 수정되었거나 해당 코드가 변경되었습니다." and **stop**.

## Step 4: Plan fixes

**Input:** valid findings + source file contents from Step 3.
**Output:** Core vs Peripheral analysis and implementation plan in plan mode.

Call `EnterPlanMode`, then create a plan with these sections:

### Findings plan (ordered by priority, P0 first)

For each valid finding:

1. **Core** — the actual defect: what is broken, the failure mechanism, and the triggering condition (from `problem` + `evidence`)
2. **Peripheral** — the reviewer's suggested fix direction (from `suggested_fix`) — noted for reference; evaluate independently against the current codebase
3. **Planned fix** — a concrete solution that:
   - Addresses the Core defect directly
   - Uses existing patterns and utilities found in the source files
   - Specifies the exact code location and change type (add, modify, remove)
4. **Change location** — `file_path:line_number`
5. **Related changes** — other findings addressable in the same edit

### Exploration items (from Risk Scenarios)

For each risk scenario:
- Scenario description
- Related file and line
- Verification method: specific test approach or monitoring strategy

### Self-check

Before delivering the plan, verify:
1. Every Core identifies a concrete failure mechanism, not a vague concern.
2. Every Planned fix specifies an exact change location and change type.
3. Findings are ordered by priority (P0 first).
4. No Planned fix blindly copies the Peripheral suggestion.
