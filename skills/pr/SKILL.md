---
name: pr
description: Creates or updates a GitHub pull request for the current branch. Use when ready to submit code changes for review.
allowed-tools: Bash(git status:*) Bash(git branch:*) Bash(git log:*) Bash(git diff:*) Bash(git push:*) Bash(gh pr create:*) Bash(gh pr view:*) Bash(gh pr edit:*)
---

## Context

- Current branch: !`git branch --show-current`
- Uncommitted changes: !`git status --short`
- Remote tracking status: !`git status -sb | head -1`
- Existing PR: !`gh pr view --json number,title,url 2>/dev/null || echo "none"`
- Commits since main: !`git log main..HEAD --oneline`
- Full commit details: !`git log main..HEAD --format="- %h: %s"`
- Changes summary: !`git diff main...HEAD --stat | tail -5`

## Task

You MUST create a new PR or update an existing PR for the current branch.

## Pre-checks

1. **Uncommitted changes**: If there are uncommitted changes, warn the user and ask if they want to commit first.
2. **Push status**: If the branch is not pushed or behind remote, push it with `git push -u origin <branch>`.

## If NO existing PR: Create new PR

### Title

- **English only, 40 chars max** (excluding prefix).
- Use short verbs: Add, Fix, Update, Remove, Refactor.
- **Ticket prefix**: If the branch name contains a ticket identifier (pattern: `letters-numbers`, e.g., `TASK-123`, `proj-42`), extract it, uppercase it, and prepend as `[TASK-123]` prefix.

Examples:
- Branch `TASK-123-add-feature` → `[TASK-123] Add feature management`
- Branch `feature/JIRA-456-fix-bug` → `[JIRA-456] Fix auth bug`
- Branch `proj-42-fix-layout` → `[PROJ-42] Fix layout issue`
- Branch `add-new-feature` → `Update Claude Code config` (no prefix)

### Command

```bash
gh pr create --title "[TASK-123] Description" --assignee @me --body "$(cat <<'EOF'
## 요약
<이 PR이 하는 일을 간략히 설명>

## 변경사항
<커밋 해시와 함께 목록으로 작성>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

## If PR already exists: Update PR

You MUST preserve the existing title — update the body only.

```bash
gh pr edit --body "$(cat <<'EOF'
## 요약
<이 PR이 하는 일을 간략히 설명 — 기존 요약 유지 또는 확장>

## 변경사항
<모든 커밋 해시와 함께 목록으로 작성 — 전체 목록>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

When updating, preserve the intent of the original summary while incorporating new changes.

## Body Rules

- **한국어로 작성**. Title은 영어, Body는 한국어.
- Analyze all commits in the branch to understand the full scope. Include every commit hash in the body for traceability.
