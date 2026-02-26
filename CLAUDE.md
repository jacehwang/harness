# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Claude Code skills repository containing custom skills that extend Claude Code's capabilities. Skills are installed via:

```bash
bunx skills add jacehwang/skills
```

## Purpose

This repository serves as a personal coding agent skills collection designed for:
- Easy installation and reuse across multiple projects
- Shareable format that others can install and use directly

## Repository Structure

```
skills/
├── skills/
│   ├── address-reviews/
│   │   └── SKILL.md          # PR review comment handler skill
│   ├── commit/
│   │   └── SKILL.md          # Git commit skill
│   └── pr/
│       └── SKILL.md          # PR creation/update skill
├── CLAUDE.md
└── README.md
```

Each skill lives in its own directory with a `SKILL.md` file defining the skill.

## Creating Skills

Skills are defined in `SKILL.md` files within subdirectories. Each skill file uses YAML frontmatter followed by the skill prompt content.

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

## References

- [Agent Skills Specification](https://agentskills.io/specification)
- [Claude Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
