# Codex Directive Writing Guide

Reference for writing effective directives targeting Codex (AGENTS.md / codex.md).

## 1. AGENTS.md Discovery Model

Codex auto-discovers and merges AGENTS.md files from repo root to cwd. Each file is injected as a separate user-role message prefixed with `# AGENTS.md instructions for <directory>`. Subdirectory files override parent-level instructions on conflict. Place project-wide directives in root AGENTS.md; scope-specific directives in subdirectory AGENTS.md files.

## 2. Autonomy Model

Codex operates as an autonomous senior engineer. Directives MUST be **outcome-oriented** — state the desired result, not the procedure.

- Good: "Ensure all API handlers validate input with zod schemas before processing."
- Bad: "First open the handler file, then find the function, then add a zod schema."

## 3. Format Preferences

- Use natural language with high-level markdown headings.
- Prefer declarative rules over step-by-step procedures.
- State tool preferences as a hierarchy when needed (e.g., "Prefer rg over grep").

## 4. Built-in Behaviors (Do NOT Repeat)

Codex already does these by default — repeating them wastes token budget and dilutes attention:

- Searches for existing patterns before writing new code
- Prioritizes correctness and reliability
- Uses its built-in planning tool to decompose tasks
- Reads multiple files in parallel

## 5. Conflict Avoidance

These patterns conflict with Codex internals — never use them in directives:

- **No step-by-step procedures:** Conflicts with Codex's built-in planning tool.
- **No file-by-file exploration instructions:** Conflicts with batch read and parallel tool calls.
- **No "think step by step" or chain-of-thought prompts:** Codex handles reasoning internally.
