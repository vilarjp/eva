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

## Problem

{{One paragraph. Who is affected, what is painful today, why it matters now.
Specific — not "users are frustrated".}}

## Goals & Non-Goals

- **Goal** — {{measurable or verifiable}}
- **Goal** — {{measurable or verifiable}}
- **Non-goal** — {{genuine exclusion a reader might otherwise expect}}

## Constraints & Assumptions

{{Timeline, stack, compat, regulatory constraints in one line. Then assumptions
as `claim — verified by <path>: YES/NO`. Unverified items move to Open
Questions.}}

## Options

| # | Option            | Trade-off                                   | Complexity | Risk     |
|---|-------------------|---------------------------------------------|------------|----------|
| A | {{name}}          | {{one-line pro-vs-con}}                     | {{XS-XL}}  | {{L/M/H}} |
| B | {{name}}          | {{one-line pro-vs-con}}                     | {{XS-XL}}  | {{L/M/H}} |
| C | {{name}} *(opt)*  | {{one-line pro-vs-con}}                     | {{XS-XL}}  | {{L/M/H}} |

Options must be genuinely distinct trade-off axes — not flavours of the same
approach.

## Recommendation

**{{Option letter}}.** {{Two sentences: why this wins, and the key trade-off
consciously accepted.}}

## User stories

- As a {{actor}}, I want {{feature}}, so that {{benefit}}.
- As a {{actor}}, I want {{feature}}, so that {{benefit}}.

## Not-doing

- {{omitted item}} — {{reason}}
- {{omitted item}} — {{reason}}

## Open questions

- [ ] {{question}} — `now` *(blocks PRD)* or `during-build`
- [ ] {{question}} — `now | during-build`

<!--
Optional sections — include ONLY when they add signal:
- Diagram (mermaid) — when the topic has spatial / systemic structure
- Appendix — Grounding Summary — 3-8 bullets from the pre-scan

Cut any section above if empty. See _shared/artifact-compactness.md.
-->
