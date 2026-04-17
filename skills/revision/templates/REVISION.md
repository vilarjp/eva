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

{{One paragraph. State the outcome of the review: clean pass, minor tune-up, or material rework. Cite highest-severity finding count. If single-doc mode, state which sibling was missing and why cross-doc alignment was skipped.}}

**Result:** {{clean pass | N findings · P0=x · P1=y · P2=z · P3=w · Suppressed=s}}

## Scope of Review

- **PRD reviewed:** `{{prd_path or "not present — single-doc mode"}}`
- **SPEC reviewed:** `{{spec_path or "not present — single-doc mode"}}`
- **Revision complexity:** {{LOW | MEDIUM | HIGH}} (inherited MAX from PRD=<tier>, SPEC=<tier>)
- **Lenses run:** {{cross-doc alignment | coherence | adversarial | scope-feasibility — list only those that ran; mark cross-doc as "skipped — single-doc mode" when applicable}}
- **Mode:** {{full cross-review | single-doc review}}

## Grounding Summary

{{3–8 bullets capturing what the review found in the source documents before running the rubric. This is the baseline reading so future readers can calibrate.}}

- {{PRD structure: goals count, non-goals count, user stories, options, recommended direction, open questions}}
- {{SPEC structure: mini-ADRs count, tracer-bullet phases, data-model touched yes/no, API surface additive/breaking, risks count}}
- {{Complexity tiers as declared in frontmatter}}
- {{Material facts the rubric will lean on}}

## Findings

Grouped by severity. Within a severity, grouped by lens. Every finding quotes evidence.

### P0 — Blockers

<!-- Findings that must be fixed before implementation. Contradicts user's explicit ask, zero SPEC coverage of a PRD goal, SPEC violates a PRD non-goal, security/data-integrity gap, untestable acceptance criterion. Omit this subsection if empty. -->

#### F-1 — {{Title}}

- **Lens:** {{cross-doc | coherence | adversarial | scope-feasibility}}
- **Type:** {{error | omission}}
- **Target:** {{PRD | SPEC | PRD+SPEC}}
- **Evidence:**
  - `{{PRD §Goals}}`: > "{{direct quote, 5–20 words}}"
  - `{{SPEC §D-2}}`: > "{{direct quote}}"
- **Why it matters:** {{One sentence on the risk if unfixed.}}
- **Proposed fix:** {{One to two sentences. Minimal. No new scope.}}

### P1 — Major

<!-- Key assumption unverified, mini-ADR missing alternatives, scope creep with real cost, risk table misses a real failure mode. Omit this subsection if empty. -->

#### F-2 — {{Title}}

- **Lens:** {{...}}
- **Type:** {{...}}
- **Target:** {{...}}
- **Evidence:**
  - `{{doc §section}}`: > "{{quote}}"
- **Why it matters:** {{...}}
- **Proposed fix:** {{...}}

### P2 — Minor

<!-- Ambiguous wording, terminology drift, broken refs, missing prior-art. Omit this subsection if empty. -->

#### F-3 — {{Title}}

- **Lens:** {{...}}
- **Type:** {{...}}
- **Target:** {{...}}
- **Evidence:**
  - `{{doc §section}}`: > "{{quote}}"
- **Why it matters:** {{...}}
- **Proposed fix:** {{...}}

### P3 — Nits

<!-- Typos, capitalization, missing optional fields. Omit this subsection if empty. -->

#### F-4 — {{Title}}

- **Lens:** {{...}}
- **Type:** {{...}}
- **Target:** {{...}}
- **Evidence:**
  - `{{doc §section}}`: > "{{quote}}"
- **Proposed fix:** {{...}}

## Suppressed Findings *(low confidence)*

<!-- Never silently drop a finding. Record here anything that felt real but couldn't be grounded in direct evidence. Future readers can re-evaluate. Omit this subsection if empty. -->

| # | Title | Lens | Why suppressed |
|---|-------|------|----------------|
| S-1 | {{title}} | {{lens}} | {{one-line reason: indirect evidence, requires domain knowledge not in the docs, etc.}} |

## Clean-pass Note *(only when there are zero findings)*

{{Remove this whole subsection when any finding is present. When zero findings: state the lenses that ran, confirm both docs were read in full, and confirm no change is warranted. This records that the review happened and PRD/SPEC are untouched.}}

## Self-review Checklist

- [x] Both documents read in full (or the one, in single-doc mode)
- [x] Grounding summary emitted and corrected before findings
- [x] Every finding quotes ≥1 piece of evidence with section reference
- [x] No finding introduces new scope or capability
- [x] No lens padded with synthetic findings
- [x] Adversarial lens ran (or clean-lens statement included)
- [x] Every finding has severity, type, target, and proposed fix
- [x] Suppressed findings listed, not silently dropped
- [x] Complexity tier matches MAX of PRD/SPEC frontmatter

## Patch Plan *(populated at Gate 2 approval)*

{{Remove this subsection if Gate 2 was declined (audit-only) or if clean pass. When patches are approved, record the exact bullets that were appended to PRD.md and SPEC.md so future readers can trace the revision back to the source fixes.}}

**PRD.md — ## Revision {{date}}:**

- [{{section}}] {{what changed}} — {{why, finding F-<id>, severity}}
- [{{section}}] {{what changed}} — {{why, finding F-<id>, severity}}

**SPEC.md — ## Revision {{date}}:**

- [{{section}}] {{what changed}} — {{why, finding F-<id>, severity}}
- [{{section}}] {{what changed}} — {{why, finding F-<id>, severity}}

## Adversarial Appendix

<!--
Standard / Deep tier: paste the `revision-adversarial-review` agent's full returned document verbatim below, `AF-N` IDs intact. The `F-N` findings above already merged the agent's material concerns into the main table; this appendix preserves the original cold-context read for auditability.

Lightweight tier (no dispatch) OR fallback when the agent failed: use the one-line-per-technique summary form instead. Omit techniques with no material concern; a single "no material concern across all five techniques" line is valid.
-->

### Dispatch mode — agent output

{{paste revision-adversarial-review agent's full markdown output here, preserving AF-N IDs and section structure}}

### Fallback mode — inline summary *(use when Lightweight tier or dispatch failed)*

- **Premise:** {{the PRD's framing, challenged}} → {{addressed in F-<id> | open question | no material concern}}
- **Unstated assumption:** {{the silent assumption}} → {{addressed in F-<id> | open question | no material concern}}
- **Decision stress:** {{D-<id> under load/partition/rollback}} → {{addressed in F-<id> | open question | no material concern}}
- **Simplification pressure:** {{abstraction that may not earn its keep}} → {{addressed in F-<id> | open question | no material concern}}
- **Alternative blindness:** {{credible option neither doc considered}} → {{addressed in F-<id> | open question | no material concern}}
