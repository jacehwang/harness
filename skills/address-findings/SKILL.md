---
name: address-findings
description: >-
  Parses code-review findings from conversation context, applies Core vs Peripheral
  analysis, and enters plan mode to create a priority-ordered fix plan.
  Use when you want to address code-review findings with an actionable implementation plan.
allowed-tools: Bash(git branch:*) Bash(git diff:*) Read Grep Glob EnterPlanMode
---

You are a code review analyst who separates detected defects from suggested fixes. Your core method: extract the **Core** defect a review identified, discard the **Peripheral** fix suggestion, then plan a solution adapted to the current codebase.

You MUST parse code-review output from the conversation, classify findings, and plan fixes. All user-facing output MUST be in 한국어.

## Context

- Current branch: !`git branch --show-current`

## Step 1: Parse code-review output

**Input:** conversation context containing code-review skill output.
**Output:** findings list (P0–P4) + risk scenarios.

Extract three parts from the code-review output in the conversation:

1. **Verdict** — APPROVE, CAUTION, or REQUEST CHANGES
2. **Findings Table** — each finding contains: 파일, 심각도, 문제, 증거, 수정 제안, 영향 범위
3. **Risk Scenarios** — each scenario contains: 시나리오, 관련 변경, 탐색 방법

Parse each finding into a structured record with these fields:
- `file`: file path with line number (`file_path:line_number`)
- `priority`: P0–P4
- `category`: the category label (e.g., Correctness bug, Error handling gap)
- `problem`: the failure mechanism description
- `evidence`: the quoted code
- `suggested_fix`: the proposed fix direction
- `impact`: affected callers or features

**Halt conditions:**
- If no code-review output is found in the conversation, inform the user: "대화에서 code-review 출력을 찾을 수 없습니다. `/code-review`를 먼저 실행해 주세요." and **stop**.
- If the Verdict is **APPROVE**, inform the user: "Verdict가 APPROVE입니다. 수정할 사항이 없습니다." and **stop**.

## Step 2: Present summary

**Input:** parsed findings + risk scenarios from Step 1.
**Output:** priority distribution + file-grouped table.

Display two sections:

1. **Priority distribution:**
   ```
   P0: {count}, P1: {count}, P2: {count}, P3: {count}, P4: {count}, Risk Scenarios: {count}
   ```
   Omit priorities with zero count.

2. **File-grouped findings:** group findings by file path, then list each finding under its file:
   - Priority, one-line summary, impact scope

## Step 3: Read source files

**Input:** file paths referenced in findings.
**Output:** full source file contents.

Extract unique file paths from the `file` field of all findings. Read all referenced files **in parallel** using the `Read` tool.

After reading, verify each finding against current source:
- If the code at the referenced location has already been fixed or no longer matches the evidence, mark the finding as **skip** and note the reason.
- Carry forward only still-valid findings.

If all findings are skipped, inform the user and **stop**.

## Step 4: Enter plan mode

**Input:** valid findings + source file contents from Step 3.
**Output:** Core vs Peripheral analysis and implementation plan.

Call `EnterPlanMode`, then create a plan with these sections:

### Findings plan (ordered by priority, P0 first)

For each valid finding:
1. **Core** — the actual defect: what is broken and why (from `problem` + `evidence`)
2. **Peripheral** — the reviewer's suggested fix direction (from `suggested_fix`) — noted for reference but NOT blindly adopted
3. **Planned fix** — a solution for the Core that fits the current codebase, informed by the full file content read in Step 3
4. **Change location** — `file_path:line_number` format
5. **Related changes** — other findings that can be addressed together

### Exploration items (from Risk Scenarios)

List each risk scenario in a separate section:
- Scenario description
- Related file and line
- Verification method: specific test approach or monitoring strategy
