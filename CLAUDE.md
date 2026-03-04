# CLAUDE.md

You are working in a Claude Code plugin repository (`jacehwang/harness`) containing custom skills and subagents.

## Critical Rules

**FAILURE TO FOLLOW THESE RULES PRODUCES BROKEN SKILLS/SUBAGENTS.**

1. Skill directory name MUST exactly match the `name` field in its SKILL.md frontmatter.
2. Every SKILL.md MUST contain valid YAML frontmatter with all three required fields: `name`, `description`, `allowed-tools`.
3. Every subagent file MUST contain valid YAML frontmatter with both required fields: `name`, `description`.
4. You MUST NOT modify files under `.claude-plugin/`.
5. If you detect a frontmatter violation (missing field, name/directory mismatch, invalid characters), fix it before proceeding and inform the user.

## Language Strategy

### Prompt Authoring

- **Instruction language**: All directives, conditions, and logic in skill and subagent prompts MUST be written in English. English instructions yield higher LLM compliance rates.
- **Output language**: Specify the output language explicitly within the prompt body. Default: 한국어. Example: `All user-facing output MUST be in 한국어.`
- Inline examples and templates within prompts follow the output language.

### Git Operations

- Commit messages and PR titles: English only.
- PR body: 한국어.

### Migration Note

Existing skills with Korean instruction text (`commit`, `pr`, `explore-test`, `address-reviews`) will be converted to English instructions on their next modification. Do not rewrite them solely for language migration.

## Architecture

- **Skills** run in the main conversation context. Users invoke them via `/commit` (Skills CLI) or `/jace:commit` (plugin).
- **Subagents** run in isolated context windows with their own system prompt and tools. Available only via plugin installation. Because subagents have no access to the parent conversation, their prompts must be self-contained.

## Repository Map

```
harness/
├── .claude-plugin/          # Plugin manifest + marketplace catalog — DO NOT MODIFY
├── skills/                  # Each subdirectory = one skill
│   ├── address-reviews/
│   │   └── SKILL.md
│   ├── commit/
│   │   └── SKILL.md
│   ├── explore-test/
│   │   └── SKILL.md
│   ├── plan-ticket/
│   │   └── SKILL.md
│   └── pr/
│       └── SKILL.md
├── agents/                  # Each .md file = one subagent
│   └── prompt-doctor.md
├── CLAUDE.md                # This file
└── README.md                # User-facing documentation (한국어)
```

## Creating Skills

You define a skill by creating `skills/<name>/SKILL.md`. The file uses YAML frontmatter followed by prompt content.

### Packaging

#### Required Frontmatter Fields

| Field | Format | Constraints |
|-------|--------|-------------|
| `name` | String, max 64 chars | Lowercase letters, numbers, hyphens only. No consecutive hyphens (`--`). MUST match directory name. |
| `description` | String, max 1024 chars | See Description Field Guide below. |
| `allowed-tools` | Space-delimited list | See tool pattern examples below. |

#### Tool Pattern Examples

| Pattern | Permits |
|---------|---------|
| `Bash(git:*)` | Bash commands starting with `git` |
| `Bash(gh pr:*)` | GitHub CLI PR commands |
| `Read` | File reading |
| `Write` | File writing |
| `Grep` | Content search |

#### Frontmatter Example

```yaml
---
name: my-skill
description: >-
  Creates something useful from repository context.
  Use when the user needs to do X.
allowed-tools: Bash(git:*) Read Grep
---
```

#### Extended Directory Structure (Optional)

For complex skills, you MAY add supporting directories alongside SKILL.md:

```
my-skill/
├── SKILL.md              # Required
├── scripts/              # Optional: executable code
├── references/           # Optional: additional documentation
└── assets/               # Optional: templates, resources
```

#### Dynamic Context Syntax

Use `` !`command` `` to inject shell command output when the skill loads:

```markdown
Current status:
!`git status`

Recent commits:
!`git log --oneline -5`
```

These commands execute at load time, not at query time.

#### Description Field Guide

Follow a two-sentence formula. This applies to both skills and subagents.

1. **Sentence 1 — What it does**: Start with a third-person action verb ("Creates...", "Explores...", "Fetches..."). State the primary action and output.
2. **Sentence 2 — When to use it**: Start with "Use when..." and describe the trigger condition from the user's perspective.

Good: `Creates a git commit with proper message formatting. Use when committing staged changes with a descriptive commit message.`

Bad: `A tool for commits.` (no action verb, no trigger condition)

### Prompt Body

This section defines how to write the prompt content that follows the frontmatter. The prompt body is the core of a skill — it determines what the agent actually does.

#### Scaffold Template

Structure every skill prompt using this skeleton. Omit sections that are not applicable, but preserve the order.

```
Role Definition          — One sentence: "You are a [role] that [action]." Use exact domain jargon (methodology names, technical terms, acronyms).
Core Directive           — 1–3 sentences: primary task + critical constraints + output language.
Repository Context       — Dynamic context (git status, branch, etc.) via !`command`.
Step N: [Title]          — For each step:
  Input:                   what this step receives.
  Output:                  what this step produces.
  Instructions:            numbered actions.
  Error handling:          "If [failure], [behavior] and stop/skip/retry."
```

**Canonical example**: `skills/plan-ticket/SKILL.md` — refer to this file for a complete implementation of the scaffold.

#### Writing Standards

| Level | # | Standard |
|-------|---|----------|
| MUST | 1 | Start with a role definition sentence: "You are a [role] that [action]." The role label MUST use exact practitioner vocabulary — named methodologies, framework names, technical acronyms — not generic labels ("helper", "assistant", "agent"). Example: "exploratory testing expert practicing Session-Based Test Management (SBTM)" not "testing helper". |
| MUST | 2 | Place the core directive (primary task + key constraints) within the first 5 lines after frontmatter. |
| MUST | 3 | Write all instructions in English (see Language Strategy). |
| MUST | 4 | Specify the output language explicitly: `All user-facing output MUST be in 한국어.` |
| MUST | 5 | Use imperative second-person: "You MUST...", not "The agent should..." |
| MUST | 6 | Define an explicit halt condition for every failure path: "If [X fails], inform the user and **stop**." |
| MUST | 7 | Use consistent terminology — one term per concept throughout the prompt. Do not alternate between synonyms (e.g., pick "ticket" or "issue", not both). |
| SHOULD | 8 | Mark each step's Input and Output to create a clear data flow between steps. |
| SHOULD | 9 | Identify parallel execution opportunities explicitly: "Call these **in parallel**:". |
| SHOULD | 10 | Include a concrete example for any non-obvious output format. |
| SHOULD | 11 | Place critical constraints in the top or bottom 20% of the prompt body — the middle receives weakest LLM attention. |
| MAY | 12 | Use bold on up to 3 key rules for emphasis. More dilutes the effect. |

## Creating Subagents

You define a subagent by creating `agents/<name>.md`. The file uses YAML frontmatter followed by the system prompt.

### Packaging

#### Frontmatter Fields

| Field | Required | Format / Values |
|-------|----------|-----------------|
| `name` | Yes | Lowercase letters, numbers, hyphens only. Unique identifier. |
| `description` | Yes | Follow the Description Field Guide (see Creating Skills → Packaging). |
| `tools` | No | Comma-separated tool list. Inherits all if omitted. |
| `model` | No | `sonnet`, `opus`, `haiku`, or `inherit` |
| `memory` | No | `user`, `project`, or `local` |
| `permissionMode` | No | `default`, `acceptEdits`, `dontAsk`, `plan` |
| `maxTurns` | No | Integer. Maximum agentic turns. |
| `hooks` | No | Lifecycle hooks scoped to this subagent. |

#### Subagent Example

```markdown
---
name: my-agent
description: Does something specific. Use when the user needs X.
tools: Read, Grep, Glob
model: sonnet
---

System prompt content here.
```

### System Prompt Writing Guide

Subagents run in isolated context windows — they cannot see the parent conversation. Their system prompt must be fully self-contained.

#### Structure

Order these sections in the system prompt. Omit sections that are not applicable.

```
Role + Core Directive    — Identity and primary task in 2–3 sentences.
Rules                    — Numbered, imperative constraints. Group by category if > 5 rules.
Input Handling           — Table of input types → actions. Cover edge cases (missing input, ambiguous input).
Workflow                 — Numbered steps with clear routing logic (if/then branching).
[Domain Sections]        — Add domain-specific sections as needed (e.g., Diagnose, Transform, Validate).
Output Format            — Exact structure of the deliverable. Include a template or example.
Revision Protocol        — How to handle follow-up requests and iteration.
```

#### Subagent-Specific Standards

- Define which tools the subagent should use and when. Explicit tool usage policy prevents unnecessary tool calls.
- Include self-validation steps: "Before delivering, verify: [checklist]."
- For complex workflows, define severity or routing tiers that control how much work to do based on input characteristics.

**Canonical example**: `agents/prompt-doctor.md` — refer to this file for a complete subagent system prompt.

## Quality Checklist

Run this checklist before committing any new or modified skill/subagent.

### Packaging (all artifacts)

- [ ] YAML frontmatter is valid and contains all required fields.
- [ ] `name` matches the directory name (skills) or filename (subagents).
- [ ] `description` follows the two-sentence formula (action verb + trigger condition).
- [ ] `.claude-plugin/` is unmodified.

### Prompt Quality (skills)

- [ ] Starts with a role definition sentence with domain-specific vocabulary (not generic labels).
- [ ] Core directive appears within the first 5 lines.
- [ ] All instructions are in English; output language is explicitly specified.
- [ ] Uses imperative second-person throughout ("You MUST...").
- [ ] Every failure path has an explicit halt or skip condition.
- [ ] Terminology is consistent — no synonym alternation for the same concept.
- [ ] Multi-step workflows label Input/Output for each step.
- [ ] Critical constraints are in the top or bottom 20% of the prompt body.
- [ ] Non-obvious output formats include a concrete example or template.
- [ ] Parallel execution opportunities are explicitly marked.

### Subagent-Specific (when applicable)

- [ ] System prompt is self-contained (no dependency on parent conversation context).
- [ ] Tool usage policy is explicitly defined.
- [ ] Includes a self-validation step before final output.
- [ ] Revision protocol handles follow-up requests.

## Development and Testing

To test this plugin locally:

```bash
claude --plugin-dir ./
```

This loads the plugin from the current directory, making all skills and subagents available for testing.

**Before committing**, run through the Quality Checklist above. Verify every SKILL.md has valid frontmatter, every directory name matches its `name` field, and `.claude-plugin/` is unmodified.
