---
name: address-reviews
description: Fetches GitHub PR review comments, classifies them by status and file, and enters plan mode to create an actionable plan addressing review feedback. Use when you need to process and respond to PR review comments.
allowed-tools: Bash(git branch:*) Bash(gh pr view:*) Bash(gh api:*) Bash(gh repo view:*) Read Grep Glob EnterPlanMode
---

## Context

- Current branch: !`git branch --show-current`
- PR info: !`gh pr view --json number,title,url,state,author 2>/dev/null || echo "none"`
- Repository: !`gh repo view --json nameWithOwner --jq '.nameWithOwner' 2>/dev/null`

## Your task

Collect, classify, and plan responses to GitHub PR review comments.

## Step 1: Detect PR

- Check the PR info from the context above
- If the user passed a PR number as an argument, use that instead
- If no PR is found, inform the user and **stop**

Extract `{owner}`, `{repo}`, and `{number}` from the PR info for subsequent API calls.

## Step 2: Collect review data

Fetch review feedback and commit history using `gh api`. Run all four calls **in parallel**.
Each REST call MUST include `--paginate` to retrieve all pages.

1. **Reviews** — top-level verdicts (APPROVED, CHANGES_REQUESTED, etc.)
   ```
   gh api --paginate repos/{owner}/{repo}/pulls/{number}/reviews \
     --jq '[.[] | {id, user: .user.login, state, body, submitted_at}]'
   ```
2. **Inline comments** — file/line-level code comments
   ```
   gh api --paginate repos/{owner}/{repo}/pulls/{number}/comments \
     --jq '[.[] | {id, user: .user.login, body, path, line, diff_hunk, created_at, in_reply_to_id}]'
   ```
3. **Conversation comments** — general PR discussion
   ```
   gh api --paginate repos/{owner}/{repo}/issues/{number}/comments \
     --jq '[.[] | {id, user: .user.login, body, created_at}]'
   ```
4. **Commits** — timestamps for filtering
   ```
   gh api --paginate repos/{owner}/{repo}/pulls/{number}/commits \
     --jq '[.[] | {sha: .sha, date: .commit.committer.date}]'
   ```

## Step 3: Filter by timestamp — "Needs action" vs "Likely addressed"

Determine the **latest commit timestamp** from the commits list. Classify each comment:
- **Needs action**: `created_at` (or `submitted_at`) is **after** the latest commit — not yet addressed
- **Likely addressed**: timestamp is **before** the latest commit — a subsequent commit likely handled it

Focus exclusively on "Needs action" comments going forward. Report "Likely addressed" as a count only.

## Step 4: Cross-reference resolution status

Fetch thread resolution via GraphQL:

```bash
gh api graphql -f query='
query($owner: String!, $repo: String!, $number: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $number) {
      reviewThreads(first: 100) {
        nodes {
          isResolved
          comments(first: 1) {
            nodes { databaseId }
          }
        }
      }
    }
  }
}' -f owner='{owner}' -f repo='{repo}' -F number='{number}'
```

A comment requires action only when **both** conditions are true:
- Timestamp says **"Needs action"** (Step 3)
- Thread is **unresolved** (Step 4)

Among the remaining actionable comments, classify each as:
- **Actionable feedback** — requires a code change → plan it
- **Question** — requires a reply only, no code change → note "reply only"
- **Praise / acknowledgment** — **skip entirely**, do not include in output

**Early exit**: If all "Needs action" comments are resolved, inform the user and **stop**.

Present a summary:
- Total comments (all) → filtered as noise → likely addressed → resolved → **actionable**
- File-by-file breakdown of actionable comments with brief excerpts
- Group by reviewer

## Step 5: Read source files and enter plan mode

Before entering plan mode, **read the source files** referenced by inline comments using the `Read` tool. The `diff_hunk` shows only the diff context — you need the current file content to plan accurate changes.

Then call `EnterPlanMode`.

In plan mode, create a plan that:
- Addresses each actionable comment, prioritized by file
- Specifies concrete changes (what to modify, where) referencing current source code
- Uses `file_path:line_number` format for all source references
- Groups related comments that can be addressed together
- Marks "question" items separately with suggested reply text
