# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Claude Code plugin repository containing custom skills and subagents. Two installation paths to support different needs:

- **Skills only** — Skills CLI로 설치 (`bunx skills add jacehwang/harness`). Skills만 필요한 사용자에게 권장.
- **Skills + Subagents** — Claude Code plugin으로 설치. Subagent까지 사용하려는 사용자에게 권장.

### Installation

```bash
# Skills만 사용
bunx skills add jacehwang/harness

# Plugin으로 설치 (Skills + Subagents)
/plugin marketplace add jacehwang/harness
/plugin install jace@harness
```

### Development / Testing

```bash
claude --plugin-dir ./
```

## Purpose

This repository serves as a personal coding agent plugin designed for:
- Easy installation and reuse across multiple projects
- Shareable format that others can install and use directly
- Skills: main conversation context에서 실행 (Skills CLI: `/commit`, Plugin: `/jace:commit`)
- Subagents: isolated context에서 실행, plugin 설치 시에만 사용 가능

## Repository Structure

```
harness/
├── .claude-plugin/
│   ├── plugin.json            # Plugin manifest
│   └── marketplace.json       # Marketplace catalog
├── skills/
│   ├── address-reviews/
│   │   └── SKILL.md           # PR review comment handler skill
│   ├── commit/
│   │   └── SKILL.md           # Git commit skill
│   └── pr/
│       └── SKILL.md           # PR creation/update skill
├── agents/                    # Subagent definitions
│   └── <name>.md
├── CLAUDE.md
└── README.md
```

## Creating Skills

Skills are defined in `SKILL.md` files within the `skills/` directory. Each skill file uses YAML frontmatter followed by the skill prompt content.

### Required Frontmatter Fields

**name** (max 64 characters)
- Lowercase letters, numbers, and hyphens only
- No consecutive hyphens (`--`)
- Must match the directory name

**description** (max 1024 characters)
- Write in third person ("Creates...", "Extracts...")
- Include what it does and when to use it

**allowed-tools**
- Space-delimited list of permitted tools
- Pattern examples:
  - `Bash(git:*)` - Bash commands starting with `git`
  - `Bash(gh pr:*)` - GitHub CLI PR commands
  - `Read` - File reading
  - `Write` - File writing
  - `Grep` - Content search

### Example Frontmatter

```yaml
---
name: my-skill
description: Creates something useful. Use when the user needs to do X.
allowed-tools: Bash(git:*) Read Grep
---
```

### Extended Structure (Optional)

For complex skills, you can add supporting directories:

```
my-skill/
├── SKILL.md              # Required: Skill definition
├── scripts/              # Optional: Executable code
├── references/           # Optional: Additional documentation
└── assets/               # Optional: Templates, resources
```

### Dynamic Context Syntax

Use `` !`command` `` syntax to inject command output at runtime:

```markdown
Current status:
!`git status`

Recent commits:
!`git log --oneline -5`
```

Commands are executed when the skill loads, providing current state to the prompt.

## Creating Subagents

Subagents are defined as Markdown files in the `agents/` directory. They run in isolated context windows with their own system prompt and tool access.

### Subagent File Format

```markdown
---
name: my-agent
description: Does something specific. Use when the user needs X.
tools: Read, Grep, Glob
model: sonnet
---

System prompt content here.
```

### Available Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (lowercase, hyphens) |
| `description` | Yes | When Claude should delegate to this subagent |
| `tools` | No | Allowed tools (inherits all if omitted) |
| `model` | No | `sonnet`, `opus`, `haiku`, or `inherit` |
| `memory` | No | Persistent memory: `user`, `project`, or `local` |
| `permissionMode` | No | `default`, `acceptEdits`, `dontAsk`, `plan` |
| `maxTurns` | No | Maximum agentic turns |
| `hooks` | No | Lifecycle hooks scoped to this subagent |

## References

- [Agent Skills Specification](https://agentskills.io/specification)
- [Claude Code Plugins](https://code.claude.com/docs/en/plugins)
- [Claude Code Subagents](https://code.claude.com/docs/en/custom-subagents)
