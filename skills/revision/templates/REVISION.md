---
doc_type: revision
date: "{{date}}"
status: "{{status}}"
topic: "{{topic}}"
slug: "{{slug}}"
complexity: "{{LOW | MEDIUM | HIGH}}"
mode: "{{full | single-doc}}"
prd_path: "{{docs/<date>-<slug>/PRD.md or null}}"
spec_path: "{{docs/<date>-<slug>/SPEC.md or null}}"
author: "{{author}}"
---

# REVISION: {{topic}}

## Summary

{{One paragraph. State the outcome: clean pass / minor tune-up / material
rework. Cite highest-severity count. If single-doc mode, say which sibling was
missing.}}

**Result:** {{clean pass | P0={{x}} P1={{y}} P2={{z}} P3={{w}} Suppressed={{s}}}}

## Scope

| Field                  | Value                                                                   |
|------------------------|-------------------------------------------------------------------------|
| PRD reviewed           | `{{prd_path}}` or `not present — single-doc mode`                       |
| SPEC reviewed          | `{{spec_path}}` or `not present — single-doc mode`                      |
| Complexity             | {{LOW / MEDIUM / HIGH}} (MAX of PRD + SPEC frontmatter)                  |
| Lenses run             | {{cross-doc / coherence / adversarial / scope-feasibility — list those that ran}} |
| Mode                   | {{full cross-review / single-doc review}}                                |

## Findings

Grouped by severity. One row per finding. Every row quotes evidence.

### P0 — Blockers

_Must be fixed before implementation. Omit heading if empty._

| ID | Lens | Type (error/omission) | Target | Evidence | Why it matters | Fix |
|----|------|-----------------------|--------|----------|----------------|-----|
| F-1 | {{cross-doc}} | {{omission}} | {{PRD / SPEC / both}} | `{{doc §section}}`: "{{quote}}"; `{{doc §section}}`: "{{quote}}" | {{one sentence — risk if unfixed}} | {{one or two sentences — minimal, no new scope}} |

### P1 — Major

| ID | Lens | Type | Target | Evidence | Why it matters | Fix |
|----|------|------|--------|----------|----------------|-----|
| F-2 | {{...}} | {{...}} | {{...}} | `{{doc §section}}`: "{{quote}}" | {{...}} | {{...}} |

### P2 — Minor

| ID | Lens | Target | Evidence | Fix |
|----|------|--------|----------|-----|
| F-3 | {{...}} | {{...}} | `{{doc §section}}`: "{{quote}}" | {{...}} |

### P3 — Nits

- **F-4** {{lens}} · {{PRD/SPEC}} · `{{§section}}` — {{title}}. Fix: {{one line}}.

## Adversarial lens

_Standard/Deep dispatch the `revision-adversarial-review` agent; the agent's
material concerns are already merged into the tables above. This row is the
audit trail of what else was probed. Lightweight fills the row inline._

| Technique                | Concern probed                                    | Outcome                                    |
|--------------------------|---------------------------------------------------|--------------------------------------------|
| Premise                  | {{PRD's framing, challenged}}                     | {{addressed in F-N / open Q / no concern}} |
| Unstated assumption      | {{the silent assumption}}                         | {{addressed in F-N / open Q / no concern}} |
| Decision stress          | {{D-N under load / partition / rollback}}         | {{addressed in F-N / open Q / no concern}} |
| Simplification pressure  | {{abstraction that may not earn its keep}}        | {{addressed in F-N / open Q / no concern}} |
| Alternative blindness    | {{credible option neither doc considered}}        | {{addressed in F-N / open Q / no concern}} |

## Clean-pass note

_Include ONLY when zero findings across all lenses. Otherwise cut._

{{One line — lenses ran, both docs read in full, PRD + SPEC untouched.}}

<!--
Optional sections — include ONLY when non-empty:

### Suppressed findings (low confidence)

| ID | Title | Lens | Why suppressed |
|----|-------|------|----------------|
| S-1 | {{title}} | {{lens}} | {{indirect evidence / needs domain knowledge not in docs}} |

### Patch plan — populated at Gate 2 approval

**PRD.md — ## Revision {{date}}:**
- [{{section}}] {{what changed}} — F-{{id}} ({{severity}})

**SPEC.md — ## Revision {{date}}:**
- [{{section}}] {{what changed}} — F-{{id}} ({{severity}})

### Adversarial appendix (Standard/Deep dispatch output)

{{paste revision-adversarial-review agent's full markdown here, AF-N IDs intact}}

Cut any section above if empty. See _shared/artifact-compactness.md.
-->
