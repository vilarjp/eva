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

{{One paragraph. If a PRD exists, cite it here and summarize its recommended direction in one sentence. If standalone, state the problem and the user / system impact in technical terms.}}

## Goals (technical)

1. {{Verifiable goal — behavior you can write an acceptance test for}}
2. {{Verifiable goal}}
3. {{Verifiable goal}}

## Non-Goals (technical)

1. {{Real exclusion — something a reader might reasonably expect that is NOT covered}}
2. {{Real exclusion}}

## Constraints

{{Tech stack, language / runtime, deployment target, backwards-compatibility requirements, infra, regulatory, timeline.}}

## Assumptions

1. {{Assumption}} — verified by reading {{path}}: {{YES | NO}}
2. {{Assumption}} — verified: {{YES | NO}}
3. {{Assumption}} — verified: {{YES | NO}}

## Current Architecture

{{One paragraph describing the parts of the system this SPEC affects today, grounded in files read during the pre-scan.}}

- **{{Module / file}}** — {{one-line responsibility as it exists now}}
- **{{Module / file}}** — {{one-line responsibility}}
- **{{Module / file}}** — {{one-line responsibility}}

## Proposed Architecture

{{One paragraph describing the system after this work lands. Name modules by their responsibilities, not their implementations.}}

- **{{Module / file}}** — {{one-line responsibility in the proposed state}}
- **{{Module / file}}** — {{one-line responsibility}}
- **{{Module / file}}** *(new)* — {{one-line responsibility}}

## Key Decisions

### D-1: {{Short decision title}}

- **Context:** {{What pressure forced this decision — constraint, goal, or trade-off}}
- **Decision:** {{One or two sentences, definitive}}
- **Alternatives Considered:**
  - *{{Alternative A}}* — {{one-line reason for rejection}}
  - *{{Alternative B}}* — {{one-line reason for rejection}}
- **Consequences:** {{What becomes easier · what becomes harder · rollback cost}}

### D-2: {{Short decision title}}

- **Context:** {{...}}
- **Decision:** {{...}}
- **Alternatives Considered:**
  - *{{Alternative}}* — {{reason}}
- **Consequences:** {{...}}

<!-- Add D-3, D-4 as needed. Delete unused entries. One mini-ADR per real decision only. -->

## Data Model

{{Entities, fields, types, relationships. Call out invariants (uniqueness, foreign keys, cascade rules). For schema changes, state whether they are additive, require a migration, or require a rollout sequence.}}

```
{{inline schema sketch, TypeScript interface, SQL DDL, Protobuf, or whatever matches the stack}}
```

**Invariants:**

- {{e.g. `(user_id, task_id)` unique}}
- {{e.g. deleting a `User` cascades to their `Session`s}}

**Migration shape:** {{additive / requires migration — describe briefly / N/A}}

## API Contracts / Interfaces

### {{Endpoint or function name}}

- **Input:** {{required / optional fields with types}}
- **Output (success):** {{shape}}
- **Errors:** {{status codes and error shape — consistent with the rest of the codebase}}
- **Idempotency / retry:** {{safe to retry? any conditions?}}
- **Backwards-compat:** {{additive / breaking — with reason}}

### {{Another endpoint or function}}

- **Input:** {{...}}
- **Output:** {{...}}
- **Errors:** {{...}}
- **Idempotency / retry:** {{...}}
- **Backwards-compat:** {{...}}

## Module Boundaries

**Created:**

- `{{path/to/new-module}}` — {{responsibility}}

**Modified:**

- `{{path/to/existing-module}}` — {{what changes in its responsibility}}

**Absorbed / Removed:**

- `{{path}}` — {{why it is no longer needed or where it moved}}

## Tracer-bullet Phases

### Phase 1 — {{Capability name}}

- **Delivers:** {{one paragraph of observable behavior delivered by this slice}}
- **Acceptance criteria:**
  - {{Behavior-focused criterion}}
  - {{Behavior-focused criterion}}
  - {{Behavior-focused criterion}}
- **Depends on:** {{none | previous phase}}

### Phase 2 — {{Capability name}}

- **Delivers:** {{...}}
- **Acceptance criteria:**
  - {{...}}
- **Depends on:** {{Phase 1}}

<!-- 1-4 phases total. No file names, no function names. -->

## Test Strategy

- **Unit tests:** {{what to cover at the unit level — risky logic, branches, invariants}}
- **Integration tests:** {{what boundaries to cover — DB, external API, component↔component}}
- **End-to-end:** {{if any — which user-observable flows}}
- **Prior art:** {{cite 1-2 existing test files in the codebase that illustrate the pattern}}
- **Coverage focus:** {{any risky paths that need extra attention — concurrency, migrations, third-party failure modes}}

## Risks & Mitigations

| # | Risk | Impact | Mitigation |
|---|------|--------|------------|
| R-1 | {{Risk}} | {{LOW / MED / HIGH}} | {{Mitigation}} |
| R-2 | {{Risk}} | {{LOW / MED / HIGH}} | {{Mitigation}} |
| R-3 | {{Risk}} | {{LOW / MED / HIGH}} | {{Mitigation}} |

## Diagram *(optional — include when topic has structural or flow complexity; mandatory for Deep)*

```mermaid
{{Mermaid diagram: component, sequence, state, or flowchart that clarifies the architecture}}
```

## Not-Doing

- {{Technical scope item omitted}} — {{reason}}
- {{Technical scope item omitted}} — {{reason}}
- {{Technical scope item omitted}} — {{reason}}

## Open Questions

- [ ] {{Question}} — `now` *(blocks SPEC finalization)* or `during-build` *(acceptable to defer)*
- [ ] {{Question}} — `now | during-build`
- [ ] {{Question}} — `now | during-build`

## Appendix — Grounding Summary *(optional)*

{{3-10 bullets summarizing the codebase pre-scan: relevant files read, conventions observed, prior art cited. Include when it aids future readers.}}

## Appendix — Adversarial Red-team Pass *(optional — include when issues raised were material)*

- **Staff engineer:** {{one-line critique}} → {{Addressed in <section> | Open Question | N/A}}
- **Security reviewer:** {{one-line critique}} → {{Addressed in <section> | Open Question | N/A}}
- **Future maintainer:** {{one-line critique}} → {{Addressed in <section> | Open Question | N/A}}
