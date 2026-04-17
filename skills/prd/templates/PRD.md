---
doc_type: prd
date: "{{date}}"
status: "{{status}}"
topic: "{{topic}}"
slug: "{{slug}}"
complexity: "{{LOW | MEDIUM | HIGH}}"
author: "{{author}}"
---

# PRD: {{topic}}

## Problem Statement

{{One paragraph. Specific, not vague. Who is affected, what is painful or missing today, and why it matters now.}}

## Goals

1. {{Primary goal — must be measurable or verifiable}}
2. {{Secondary goal — must be measurable or verifiable}}
3. {{Tertiary goal — must be measurable or verifiable}}

## Non-Goals

1. {{What this effort explicitly will NOT address}}
2. {{Another genuine boundary — not a goal in disguise}}

## Constraints

{{Timeline, tech stack limitations, backwards-compatibility requirements, deployment constraints, regulatory or compliance boundaries.}}

## Assumptions

1. {{Assumption about the codebase — verified by reading code: YES / NO}}
2. {{Assumption about dependencies or tools — verified: YES / NO}}
3. {{Assumption about environment or infrastructure — verified: YES / NO}}

## Options Considered

### Option A — {{Name}}

- **Description:** {{2-3 sentences}}
- **Pros:**
  - {{Specific advantage}}
  - {{Specific advantage}}
- **Cons:**
  - {{Specific disadvantage}}
  - {{Specific disadvantage}}
- **Complexity:** {{XS | S | M | L | XL}}
- **Risk:** {{Low | Med | High}} — {{one-line reason}}

### Option B — {{Name}}

- **Description:** {{2-3 sentences}}
- **Pros:**
  - {{Specific advantage}}
  - {{Specific advantage}}
- **Cons:**
  - {{Specific disadvantage}}
  - {{Specific disadvantage}}
- **Complexity:** {{XS | S | M | L | XL}}
- **Risk:** {{Low | Med | High}} — {{one-line reason}}

### Option C — {{Name}} *(optional)*

- **Description:** {{2-3 sentences}}
- **Pros:**
  - {{Specific advantage}}
- **Cons:**
  - {{Specific disadvantage}}
- **Complexity:** {{XS | S | M | L | XL}}
- **Risk:** {{Low | Med | High}} — {{one-line reason}}

## Recommended Direction

- **Choice:** {{Option letter and name}}
- **Rationale:** {{Why this option wins — reference specific pros/cons, goals, and constraints}}
- **Key trade-off accepted:** {{The main downside we are consciously accepting, and why}}
- **Why alternatives were not chosen:** {{One short sentence per discarded option}}

## Complexity

**Tier:** {{LOW | MEDIUM | HIGH}}

{{One-sentence justification referencing file count, pattern reuse vs novelty, or cross-cutting impact.}}

## User Stories

1. As a {{actor}}, I want {{feature}}, so that {{benefit}}.
2. As a {{actor}}, I want {{feature}}, so that {{benefit}}.
3. As a {{actor}}, I want {{feature}}, so that {{benefit}}.

## Diagram *(optional — include when topic has spatial or systemic structure)*

```mermaid
{{Mermaid diagram: flowchart, sequence, state, or component diagram that clarifies the design}}
```

## Not-Doing

- {{Feature or scope item omitted}} — {{reason}}
- {{Feature or scope item omitted}} — {{reason}}
- {{Feature or scope item omitted}} — {{reason}}

## Open Questions

- [ ] {{Question}} — `now` *(blocks PRD finalization)* or `during-build` *(acceptable to defer)*
- [ ] {{Question}} — `now | during-build`
- [ ] {{Question}} — `now | during-build`

## Appendix — Grounding Summary *(optional)*

{{3-8 bullets summarizing the codebase pre-scan: relevant files, conventions, prior art. Use when it aids future readers.}}
