---
name: plan-ticket
description: >-
  Explores the codebase and fills a Linear ticket with goals, non-goals,
  and an implementation plan so coding agents can work autonomously.
  Use when a ticket has only a title and needs a detailed specification.
allowed-tools: >-
  Read Grep Glob AskUserQuestion
  Bash(git log:*) Bash(git diff:*) Bash(git ls-files:*)
  mcp__plugin_linear_linear__get_issue
  mcp__plugin_linear_linear__save_issue
  mcp__plugin_linear_linear__list_projects
  mcp__plugin_linear_linear__get_project
  mcp__plugin_linear_linear__list_teams
  mcp__plugin_linear_linear__get_team
  mcp__plugin_linear_linear__list_issue_labels
  mcp__plugin_linear_linear__list_issue_statuses
  mcp__plugin_linear_linear__list_issues
  mcp__plugin_linear_linear__create_comment
---

You receive a Linear ticket identifier as an argument (e.g., `PROJ-123`). You MUST explore the codebase, clarify ambiguities with the user, and write a complete implementation specification into the ticket body so a coding agent can execute it autonomously. You also set priority, estimate, and project.

## Context

Use the following repository information to guide keyword extraction in Step 3 and scope assessment.

- Repository root: !`git rev-parse --show-toplevel`
- Current branch: !`git branch --show-current`
- Primary languages: !`git ls-files | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -5`
- Top-level structure: !`ls -d */ 2>/dev/null | head -20`

## Step 1: Fetch the ticket

**Input:** argument from user.
**Output:** extracted ticket fields + `existing_body` (if applicable).

1. If no argument is provided, tell the user the required syntax (`/plan-ticket PROJ-123`) and **stop**.
2. Call `mcp__plugin_linear_linear__get_issue` with the identifier. If the call fails, inform the user the identifier may be invalid and **stop**.
3. Extract: `id`, `title`, `description`, `team`, `project`, `labels`, `state`, `priority`, `estimate`.
4. If the ticket already has a non-trivial description (more than 2 lines of substantive content):
   - Save the existing description as `existing_body`.
   - Call `AskUserQuestion`: "이 티켓에 이미 본문이 있습니다. 어떻게 처리할까요?" with options:
     - **덮어쓰기** — 기존 본문을 참고하되 새로 작성
     - **기존 내용에 추가** — 기존 본문의 구조를 유지하고 비어 있는 섹션만 채움
     - **중단** — 작업을 중단
   - Both "덮어쓰기" and "추가" proceed to Step 2. Extract requirements, file paths, and design decisions from `existing_body` for use in Step 5.

## Step 2: Gather Linear context

**Input:** `team` from Step 1.
**Output:** team settings, project list, label list, detected estimate scale.

Call these **in parallel**, passing `teamId` from Step 1's `team` field:

1. `mcp__plugin_linear_linear__get_team` with `teamId` — team settings.
2. `mcp__plugin_linear_linear__list_projects` with `teamId` — candidate projects.
3. `mcp__plugin_linear_linear__list_issue_labels` with `teamId` — available labels.
4. `mcp__plugin_linear_linear__list_issues` with `teamId`, `limit: 50`, `orderBy: updatedAt` — for estimate scale inference.

If any individual call fails, proceed with the data from successful calls. Only **stop** if `get_team` fails (team context is required).

**Estimate scale detection:** Collect all non-null `estimate` values from fetched issues.
1. If fewer than 3 issues have estimates, default to Fibonacci and state this assumption to the user.
2. If any value is in {4} or the maximum value is 5, the scale is **Linear** (1-5).
3. If any value is in {8} or values skip 4 (e.g., 1, 2, 3, 5), the scale is **Fibonacci**.
4. If all values are in {1, 2, 3} (ambiguous overlap), check the team's estimation settings from `get_team`. If still unclear, default to Fibonacci and state this assumption.

## Step 3: Explore the codebase

**Input:** `title`, `existing_body` (if any), repository context.
**Output:** list of relevant file paths, scope assessment.

Parse the ticket title and existing description for domain keywords. Follow this search strategy in order:

1. **Glob** for file/directory names matching domain keywords (types, interfaces, routes, schemas, tests).
2. **Grep** for symbol references (function names, class names, constants) found in Step 1's title/description.
3. **Read** the top 3-5 most relevant files to understand data flow and dependencies.
4. **git log** — `git log --oneline -20 -- <relevant-paths>` for recent change patterns.

Produce a scope assessment:
- Files to modify (list paths)
- New files to create (list paths)
- Test files affected or to create
- Schema/migration changes needed (yes/no)

## Step 4: Clarify ambiguities

**Input:** scope assessment from Step 3.
**Output:** user answers (0-4 questions).

Identify ambiguities that would produce a wrong specification if assumed incorrectly. Call `AskUserQuestion` **once** with 1-4 questions, each with 2-4 concrete options derived from codebase findings.

**Skip this step** when the title is specific and codebase exploration reveals a single clear path.

## Step 5: Determine metadata

**Input:** team data and estimate scale from Step 2, scope assessment from Step 3, user answers from Step 4.
**Output:** priority value, estimate value, project ID.

### Priority

Assign based on impact:

| Value | Label | Criteria |
|-------|-------|----------|
| 1 | Urgent | Production bugs, service outages |
| 2 | High | Security issues, non-critical bugs |
| 3 | Normal | Features, improvements |
| 4 | Low | Tech debt, documentation |

Adjust based on user answers and blocking relationships.

### Estimate

Use the scale detected in Step 2. Size by files affected, complexity, and test scope. Cross-reference with sibling issue estimates in the same project.

**Fibonacci scale:**

| Estimate | Scope |
|----------|-------|
| 1 | Single file, trivial change |
| 2 | 2-3 files, minor test updates |
| 3 | 3-5 files, new tests required |
| 5 | Cross-cutting, schema changes |
| 8 | Major feature, new module |

**Linear scale:** 1 = trivial, 2 = small, 3 = medium, 4 = large, 5 = very large.

### Project

Keep existing project if set. Otherwise, match by keyword overlap with project names/descriptions and present top 1-2 candidates to the user via `AskUserQuestion`.

## Step 6: Write the ticket body

**Input:** scope assessment from Step 3, user answers from Step 4, metadata from Step 5, `existing_body` (if applicable).
**Output:** complete ticket body in 한국어.

Write the description using this template. Each rule is embedded at the relevant section:

```
## Goals

- [ ] (검증 가능한 목표 — "X 파일에서 Y 함수가 Z를 반환한다" 형태로 작성)
- [ ] (각 목표에 Step 3에서 발견한 파일 경로를 1개 이상 포함)

## Non-goals

- (범위 밖으로 명시적으로 제외하는 항목 — 최소 1개 필수)

## Background

(티켓의 맥락: 왜 이 작업이 필요한지, 관련 코드 영역, 현재 동작)

## Plan

1. (구체적 단계 — 변경할 파일 경로와 변경 내용 명시)
2. ...
n. 테스트 작성 및 검증 (이 단계는 반드시 마지막에 포함)

## Notes

- (관련 파일, 기존 패턴, 주의사항)
```

**`existing_body` 처리:** 사용자가 "덮어쓰기"를 선택한 경우 `existing_body`의 요구사항과 설계 결정을 새 본문에 반영한다. "추가"를 선택한 경우 `existing_body`의 기존 섹션 구조를 유지하고 비어 있는 섹션만 채운다.

## Step 7: Update Linear and confirm

**Input:** all outputs from Steps 1, 5, 6.

Call `mcp__plugin_linear_linear__save_issue` with:
- `id`: issue ID from Step 1
- `description`: body from Step 6
- `priority`: value from Step 5
- `estimate`: value from Step 5
- `project`: only if changed
- `state`: change to "Todo" if current state is "Backlog"

If the save fails, inform the user with the error and **stop**.

After saving, display a summary:

| Item | Value |
|------|-------|
| Ticket | identifier and title |
| Priority | value and label |
| Estimate | value and scale |
| Project | name (or unchanged) |
| Goals | count |
| Non-goals | count |
| Implementation steps | count |

Then call `mcp__plugin_linear_linear__create_comment` with a brief summary of the codebase analysis and planning performed.
