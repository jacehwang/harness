# Prompt Doctor Framework Reference

Use this file only when the prompt needs deeper diagnosis than the main `SKILL.md` provides.

This file is a reference for analysis, not a required output template. Prefer concrete findings and minimal intervention over exhaustive taxonomy.

## Practical Diagnosis

Start with four questions:

1. What is the prompt trying to make the model do?
2. What must be preserved exactly?
3. What is most likely to fail in real use?
4. How much rewriting is actually justified?

If those four questions are enough, do not force a full framework pass.

## Defect Checklist

Only report defects that are actually present.

### Clarity defects

- ambiguous instruction wording
- inconsistent names for the same thing
- generic role framing that does not help behavior
- vague quality targets with no testable meaning
- critical constraints buried in the middle of dense prose

### Structure defects

- long prompt with no section boundaries
- related constraints scattered across unrelated sections
- edge cases before core behavior
- long flat lists with no grouping
- contradictions between sections

### Compliance defects

- too many negative instructions instead of positive actions
- exact counts or rigid limits with no output scaffolding
- format requirements separated from their exceptions
- important rules repeated instead of positioned clearly
- context or examples overwhelming the actual instructions

### Prompt-specific risks

- RAG prompts that blur context and instruction
- structured-output prompts with unclear schema rules
- agent prompts that lack fallback or refusal behavior when needed
- system prompts that lack clear boundaries between instructions and user data

## Rewrite Scope

Use rewrite scope to decide effort, not to maximize defect count.

- `None`: already fit for purpose; optional polish only
- `Light`: local wording, ordering, or format fixes
- `Standard`: multiple related fixes or section-level restructuring
- `Heavy`: full rewrite due to contradictions, structural collapse, or high operational risk

Borderline rule: choose the lower scope unless the prompt would likely fail in real use.

## High-Signal Heuristics

Prefer these heuristics over rigid doctrine:

- Put the most important instruction early.
- Keep related constraints together.
- Replace vague wording with testable wording.
- Convert unnecessary negatives into positive instructions.
- Preserve exact template syntax.
- Remove filler before adding safeguards.
- Add structure only when it improves compliance.

## 8-Field Reference

Use these as optional explanatory lenses, not mandatory output categories.

| Field | What it helps with |
|------|---------------------|
| CogPsy | attention, ordering, grouping, salience |
| InfoDes | sectioning, labels, progressive disclosure |
| ReqEng | explicit criteria, constraints, edge handling |
| InsDes | scaffolding, examples, learning alignment |
| TechCom | parallel phrasing, active voice, precision |
| Rhetoric | persona credibility, tone, situational fit |
| Pragma | speech-act fit, redundancy, ambiguity |
| BehSci | defaults, framing, loss emphasis when justified |

Use field names only when they help explain why a change matters.

## Detailed Diagnosis Triggers

Do a deeper pass when one or more are true:

- the prompt is macro-scale
- the user asks for a diagnostic report
- the prompt is high-risk or user-visible at scale
- the rewrite changes overall structure, not just wording
- preservation constraints are heavy enough that careless editing is risky
