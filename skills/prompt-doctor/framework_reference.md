# Prompt Doctor Framework Reference

Deeper diagnostic tools for cases where the SKILL.md workflow needs reinforcement: complex multi-section prompts, ambiguous severity, structural problems, or user-requested detailed analysis.

## Defect Taxonomy

Only report defects that are actually present. Use these categories to organize findings, not as a checklist to exhaust.

### Wording defects

- Ambiguous instruction: a directive that two reasonable readers would interpret differently
- Inconsistent terminology: alternating names for the same concept (e.g., "ticket" and "issue")
- Untestable quality target: "write good output" vs "output must include X, Y, Z"
- Generic role label: "helpful assistant" instead of a domain-specific practitioner identity

### Structural defects

- No section boundaries in a prompt over ~30 lines
- Related constraints scattered across unrelated sections
- Edge cases or exceptions placed before the core behavior they modify
- Contradictions between sections (check: does section A promise what section B forbids?)

### Compliance defects

- Negative-heavy instructions: more than half of directives say what NOT to do
- Rigid output constraints (exact counts, strict formats) with no scaffolding or example
- Format rules separated from their exceptions by unrelated content
- Critical constraints buried in the middle 60% of the prompt body

### Prompt-type risks

| Prompt type | Common failure mode |
|-------------|-------------------|
| RAG | Context and instruction blur -- model treats retrieved content as directives |
| Structured output | Schema rules unclear or separated from the output specification |
| Agent / tool-use | Missing fallback behavior when tools fail or return unexpected results |
| System prompt | No clear boundary between platform instructions and user-supplied data |
| Multi-turn chat | State assumptions from earlier turns not validated in later turns |

## Diagnostic Procedures

### Contradiction scan

When a prompt has multiple sections or rule lists:

1. List each directive that constrains output behavior.
2. For each pair, check: can both be satisfied simultaneously?
3. If not, flag the contradiction and note which directive should win based on the prompt's core intent.

### Salience audit

When a prompt is long (50+ lines) or dense:

1. Identify the 3 most important directives.
2. Check their position: are they in the top 20% or bottom 20% of the prompt?
3. If any critical directive is in the middle 60%, flag it -- LLM attention is weakest there.
4. Check for repetition: is the same rule stated multiple times as a substitute for positioning it well?

### Structure assessment

When rewrite scope might be Standard or Heavy:

1. Can each section's purpose be stated in one sentence? If not, the section is doing too much.
2. Does information flow top-down (identity -> task -> constraints -> format)? If not, the reader must jump around.
3. Are there more than 2 flat lists with 5+ items? If so, grouping or hierarchy is needed.
4. Remove any section. Does the prompt still make sense? If yes, the section may be filler.

## Analytical Lenses

Use these to explain WHY a change matters, not as mandatory categories.

| Lens | Explains changes involving |
|------|--------------------------|
| CogPsy | Attention allocation, ordering effects (primacy/recency), chunking, cognitive load |
| InfoDes | Section labeling, progressive disclosure, visual hierarchy, scanability |
| ReqEng | Testable acceptance criteria, constraint completeness, edge case coverage |
| InsDes | Example scaffolding, learning sequence, worked examples vs abstract rules |
| TechCom | Parallel phrasing, active voice, sentence-level precision, term consistency |
| Rhetoric | Persona credibility, tone calibration, audience-appropriate register |
| Pragma | Speech-act clarity (is this a command, suggestion, or observation?), redundancy, ambiguity |
| BehSci | Default bias, loss framing, anchoring effects in examples |

When citing a lens in the diagnosis, include one sentence explaining the specific mechanism. Example: "CogPsy (primacy effect): the most critical constraint appears at line 47 of 60, where attention is lowest."
