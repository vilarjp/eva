---
doc_type: spec
date: "{{date}}"
status: "{{status}}"
topic: "{{topic}}"
slug: "{{slug}}"
complexity: "{{LOW | MEDIUM | HIGH}}"
origin: "{{path/to/PRD.md or 'standalone'}}"
author: "{{author}}"
---

# SPEC: {{topic}}

## Context

{{One paragraph. If a PRD exists, cite it and summarise its recommended
direction in one sentence. Otherwise state the problem + technical impact.}}

## Goals & Non-Goals

- **Goal** — {{behaviour you can write an acceptance test for}}
- **Goal** — {{behaviour you can write an acceptance test for}}
- **Non-goal** — {{real exclusion a reader might reasonably expect}}

## Constraints & Assumptions

{{Stack, runtime, compat, infra, timeline in one line. Assumptions as
`claim — verified by <path>: YES/NO`. Unverified ones move to Open Questions.}}

## Architecture

**Now:** {{one paragraph — what this SPEC touches today, grounded in the
pre-scan.}}

**Proposed:** {{one paragraph — the shape after this lands. Name modules by
responsibility, not implementation.}}

| Module / file                  | Role                                      | State     |
|--------------------------------|-------------------------------------------|-----------|
| `{{path}}`                     | {{one-line responsibility}}               | existing  |
| `{{path}}`                     | {{one-line responsibility}}               | modified  |
| `{{path}}`                     | {{one-line responsibility}}               | new       |

## Key decisions

| ID  | Decision                             | Alternatives considered              | Consequence                            |
|-----|--------------------------------------|--------------------------------------|----------------------------------------|
| D-1 | {{one-sentence decision}}            | {{alt}} — {{reason rejected}}        | {{what becomes easier / harder}}       |
| D-2 | {{one-sentence decision}}            | {{alt}} — {{reason rejected}}        | {{what becomes easier / harder}}       |

Every real decision gets a row. Every row names at least one genuine
alternative. No padding.

## Data model

{{One paragraph naming entities + relationships. Schema trivia lives in the
code — call out only the invariants that matter:}}

- {{e.g. `(user_id, task_id)` unique}}
- {{e.g. status transitions are one-way except `submitted → cancelled`}}

**Migration shape:** {{additive / requires migration — describe briefly / N/A}}

## API contracts

| Endpoint / function     | Input (semantic)     | Output (semantic)     | Errors / idempotency / compat |
|-------------------------|----------------------|-----------------------|-------------------------------|
| `{{name}}`              | {{fields}}           | {{shape}}             | {{key error modes, retry, additive/breaking}} |
| `{{name}}`              | {{fields}}           | {{shape}}             | {{...}}                       |

## Phases (tracer-bullets)

| # | Capability             | Delivers (observable behaviour)                  | Acceptance criteria (≤ 3)                           |
|---|------------------------|--------------------------------------------------|------------------------------------------------------|
| 1 | {{name}}               | {{what the user / system can now do}}            | {{criterion}}; {{criterion}}                         |
| 2 | {{name}}               | {{...}}                                          | {{...}}                                              |

Capabilities, not file names. 1-4 phases total.

## Test strategy

- **Unit:** {{risky logic, invariants}}
- **Integration:** {{boundaries — DB, external API, component↔component}}
- **Prior art:** {{1-2 existing test files that illustrate the pattern}}

## Risks

| # | Risk                                 | Impact     | Mitigation                              |
|---|--------------------------------------|------------|-----------------------------------------|
| R-1 | {{risk}}                           | {{L/M/H}}  | {{mitigation}}                          |

## Not-doing

- {{omitted scope item}} — {{reason}}
- {{omitted scope item}} — {{reason}}

## Open questions

- [ ] {{question}} — `now` *(blocks SPEC)* or `during-build`

<!--
Optional sections — include ONLY when they add signal:
- Diagram (mermaid) — mandatory on Deep, offered on Standard when structure is
  spatial or flow is non-linear
- Appendix — Grounding Summary — 3-10 bullets from the pre-scan
- Appendix — Adversarial Red-team — one line per role, only if material

Cut any section above if empty. See _shared/artifact-compactness.md and
spec/references/anti-bloat.md for the rewrite patterns.
-->
