# CLAUDE.md

You are working in a Claude Code plugin repository (`jacehwang/harness`) containing custom skills and subagents.

## Critical Rules

**FAILURE TO FOLLOW THESE RULES PRODUCES BROKEN SKILLS/SUBAGENTS.**

1. Skill directory name MUST exactly match the `name` field in its SKILL.md frontmatter.
2. Every SKILL.md MUST contain valid YAML frontmatter with all three required fields: `name`, `description`, `allowed-tools`.
3. Every subagent file MUST contain valid YAML frontmatter with both required fields: `name`, `description`.
4. You MUST NOT modify files under `.claude-plugin/`.
5. If you detect a frontmatter violation (missing field, name/directory mismatch, invalid characters), fix it before proceeding and inform the user.

## Language Rules

- Skill prompt content: written in the language the user uses.
- PR body: 한국어.
- PR titles and commit messages: English only.

## Repository Map

```
harness/
├── .claude-plugin/          # Plugin manifest + marketplace catalog
├── skills/                  # Each subdirectory = one skill
│   ├── address-reviews/
│   │   └── SKILL.md
│   ├── commit/
│   │   └── SKILL.md
│   └── pr/
│       └── SKILL.md
├── agents/                  # Each .md file = one subagent
│   └── prompt-doctor.md
├── CLAUDE.md                # This file
└── README.md                # User-facing documentation (한국어)
```

## Architecture

- **Skills** run in the main conversation context. Users invoke them via `/commit` (Skills CLI) or `/jace:commit` (plugin).
- **Subagents** run in isolated context windows with their own system prompt and tools. Available only via plugin installation.

## Creating Skills

You define a skill by creating `skills/<name>/SKILL.md`. The file uses YAML frontmatter followed by prompt content.

### Required Frontmatter Fields

| Field | Format | Constraints |
|-------|--------|-------------|
| `name` | String, max 64 chars | Lowercase letters, numbers, hyphens only. No consecutive hyphens (`--`). MUST match directory name. |
| `description` | String, max 1024 chars | Third person ("Creates...", "Extracts..."). State what it does and when to use it. |
| `allowed-tools` | Space-delimited list | See tool pattern examples below. |

### Tool Pattern Examples

| Pattern | Permits |
|---------|---------|
| `Bash(git:*)` | Bash commands starting with `git` |
| `Bash(gh pr:*)` | GitHub CLI PR commands |
| `Read` | File reading |
| `Write` | File writing |
| `Grep` | Content search |

### Frontmatter Example

```yaml
---
name: my-skill
description: Creates something useful. Use when the user needs to do X.
allowed-tools: Bash(git:*) Read Grep
---
```

### Extended Directory Structure (Optional)

For complex skills, you MAY add supporting directories alongside SKILL.md:

```
my-skill/
├── SKILL.md              # Required
├── scripts/              # Optional: executable code
├── references/           # Optional: additional documentation
└── assets/               # Optional: templates, resources
```

### Dynamic Context Syntax

Use `` !`command` `` to inject shell command output when the skill loads:

```markdown
Current status:
!`git status`

Recent commits:
!`git log --oneline -5`
```

These commands execute at load time, not at query time.

## Creating Subagents

You define a subagent by creating `agents/<name>.md`. The file uses YAML frontmatter followed by the system prompt.

### Frontmatter Fields

| Field | Required | Format / Values |
|-------|----------|-----------------|
| `name` | Yes | Lowercase letters, numbers, hyphens only. Unique identifier. |
| `description` | Yes | When Claude should delegate to this subagent. |
| `tools` | No | Comma-separated tool list. Inherits all if omitted. |
| `model` | No | `sonnet`, `opus`, `haiku`, or `inherit` |
| `memory` | No | `user`, `project`, or `local` |
| `permissionMode` | No | `default`, `acceptEdits`, `dontAsk`, `plan` |
| `maxTurns` | No | Integer. Maximum agentic turns. |
| `hooks` | No | Lifecycle hooks scoped to this subagent. |

### Subagent Example

```markdown
---
name: my-agent
description: Does something specific. Use when the user needs X.
tools: Read, Grep, Glob
model: sonnet
---

System prompt content here.
```

## Development and Testing

To test this plugin locally:

```bash
claude --plugin-dir ./
```

This loads the plugin from the current directory, making all skills and subagents available for testing.

**Before committing, verify:** every SKILL.md has valid frontmatter, every directory name matches its `name` field, and `.claude-plugin/` is unmodified.
