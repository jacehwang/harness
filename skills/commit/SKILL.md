---
name: commit
description: Creates a git commit with proper message formatting. Use when committing staged changes with a descriptive commit message.
allowed-tools: Bash(git add:*) Bash(git status:*) Bash(git commit:*)
---

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Untracked files: !`git ls-files --others --exclude-standard`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`

## Your task

Based on the above changes, create a single git commit.

## Guidelines

### 1. Check for untracked files
- Review the list of untracked files above
- If there are untracked files that are relevant to the current changes, include them in the commit
- Common files to include: new source files, configuration files, documentation
- Files to typically exclude: build artifacts, temporary files, .env files
- **ALWAYS include**: `.claude/` directory files (project Claude Code settings)
- When in doubt, ask yourself: "Is this file part of the logical change I'm committing?"
- When using `git add` with file paths containing special characters (brackets, spaces, etc.), always quote the path: `git add "path/with[brackets]/file.ts"`

### 2. Commit message format

**Subject line: 50 chars max.** Be concise. Use short verbs (Add, Fix, Update, Remove, Refactor) + terse noun phrases.

✅ **CORRECT examples** (all under 50 chars):
- "Add user auth module" (20)
- "Fix login form validation" (25)
- "Update schema for user roles" (28)
- "Refactor database connection pool" (34)
- "Remove deprecated payment endpoint" (35)

**Too long → shorten:**
- "Add guidance for quoting special character paths" (49) → "Add special char quoting guidance" (32)
- "Implement investment proposal management feature" (48) → "Add proposal management" (22)
- "Reorganize skills into subdirectory and improve metadata" (56) → "Reorganize skills directory" (26)

❌ **WRONG examples (NEVER DO THIS):**
- "[TICKET-228] Add user authentication module" ← WRONG: Has branch prefix
- "[TICKET-60] Fix validation errors" ← WRONG: Has branch prefix
- "TICKET-228: Update schema" ← WRONG: Has branch reference
- "feat: Add new feature" ← WRONG: Has any kind of prefix

**Rules:**
- **50 chars max** for the subject line — no exceptions
- Describe *what* changed, not *why* (put "why" in the commit body if needed)
- Drop filler words (the, a, for the, in the, etc.)
- NEVER include branch names, ticket numbers, or any prefixes

**Self-check before committing:**

1. "Is the subject line over 50 characters?"
   - If YES → Shorten: drop adjectives, compress noun phrases, use a shorter verb
   - If NO → Continue

2. "Does it include [BRANCH-NAME] or ticket number?"
   - If YES → STOP and remove it immediately
   - If NO → Proceed with commit

### 3. Commit body (optional)

When the subject line alone is not enough, add a commit body:
- Leave one blank line after the subject
- Use the body to explain *why* the change was made
- Wrap body lines at 72 characters
