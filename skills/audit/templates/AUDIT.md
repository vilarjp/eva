---
name: audit
date: "{{DATE}}"
status: "{{draft | approved}}"
topic: "{{short topic — what was audited}}"
target: "{{path | glob | directory}}"
focus: "{{omit if full audit, else: short focus phrase}}"
scope: "{{LIGHTWEIGHT | STANDARD | DEEP}}"
target_stats: "{{files: N, LOC: approx}}"
stack: "{{language + framework, e.g. 'TypeScript / React / Vitest'}}"
---

# Audit: {{topic}}

**Target:** `{{target}}` — {{N files, approx LOC}}.
**Focus:** {{"full audit" | phrase}}.
**Scope tier:** {{LIGHTWEIGHT | STANDARD | DEEP}}.
**Stack:** {{language + framework}}.

## TL;DR

**Posture:** {{one sentence — "clean surface, P2/P3 polish only" | "material debt, N P0/P1 worth addressing" | "structural problems — refactor track recommended"}}

- Blocker debt (P0):   **{{count}}**
- Major debt (P1):     **{{count}}**
- Minor debt (P2):     **{{count}}**
- Nits (P3):           **{{count}}**
- Bugs surfaced:       **{{count}}**  _(→ `/diagnosis`)_
- Architecture observations: **{{count}}**

{{one-line verdict — "Route bugs to /diagnosis first; then /execute REFACTOR on P0 + P1." | "Clean surface — P3 nits only; defer or drop." | "Architecture reshape needed before individual findings are worth addressing."}}

## Bugs surfaced

_Real current bugs encountered during the audit. These are NOT debt — they need a diagnosis, not a refactor._

_(Omit this section if none. Otherwise, for each bug:)_

### B-1 — {{title}}

- **File:** [`{{path}}:{{line}}`]({{path}}:{{line}})
- **Shape:** {{crash | data loss | silent failure | security hole}}
- **Evidence:** `{{verbatim quote}}`
- **Recommended:** run `/diagnosis` — this is a live defect, not debt.

## Architecture observations

_Cross-cutting shapes bigger than any single finding. Not individually handed off — they inform which findings to prioritise._

_(Omit if the audit is Lightweight or the target is small enough that no cross-cutting shape emerged.)_

### A-1 — {{observation title — e.g. "Shotgun Surgery across billing/"}}

- **Shape:** {{one sentence naming the structural concern — e.g. "adding a currency forces edits across 11 files"}}
- **Proof (individual findings that corroborate):** F-{{N}}, F-{{M}}, F-{{K}}
- **Cost:** {{the concrete expected pain — "blocks the multi-currency roadmap", "each incident of <X> fans out to ~6 classes"}}
- **Refactor shape:** {{one sentence — e.g. "extract `Money` as a value object; migrate callers one at a time"}}
- **Severity hint:** {{P0 | P1 | P2}} _(the architecture observation's priority; individual findings may be a tier lower)_

### A-2 — {{...}}

## Blocker debt (P0)

_Active incident risk. A user-visible failure mode is one pager away, or a critical invariant is implicit._

### F-1 — {{title}}

- **Severity:** P0
- **Smell:** {{category name — e.g. "SWALLOW" | "SCATTER" | "Feature Envy" | "Primitive Obsession"}}
- **File:** [`{{path}}:{{line}}`]({{path}}:{{line}})
- **Evidence:**
  - `{{verbatim 5-30-word quote from the file}}` — {{path:line}}
  - `{{optional second quote if the smell spans two sites}}`
- **Cost:** {{one sentence — name the incident shape: "if the upstream returns a partial response, the null reaches `render()` and crashes the page"}}
- **Suggested refactor shape:** {{one sentence — NOT code. E.g. "parse `response.data` through `UserSchema` at the boundary; downstream callers stop null-guarding."}}
- **Handoff:** `F-1 | P0 | {{path}}:{{line}} | {{smell}} | {{intent}}`

### F-2 — {{...}}

_(If no P0 findings: state "No blocker debt. Audit proceeds to Major." Do not list placeholder entries.)_

## Major debt (P1)

_Likely to cause an incident in a quarter, OR blocks a known-likely change._

### F-3 — {{title}}

[... same shape as P0 ...]

## Minor debt (P2)

_Makes maintenance harder. No active risk._

### F-N — {{title}}

[... same shape ...]

## Nits (P3)

_Small cleanup. Defer or drop if scope is tight._

- **F-M** [`{{path}}:{{line}}`]({{path}}:{{line}}) — {{title}}. Smell: {{category}}. Shape: {{one sentence}}.

_(For P3, the single-line form is acceptable to reduce noise.)_

## Positives

_What is already well-shaped — calibration anchors for future audits and for the REFACTOR pass, so the good parts are not regressed._

- {{file or module}}: {{one-line observation — "`Money` value object used consistently across billing; new callers inherit validation for free"}}
- {{...}}

## Suppressed candidates

_Candidates considered and dropped, with reason. Keeps the audit honest and prevents re-surfacing on re-audit._

| Candidate | Smell | Why dropped |
|-----------|-------|-------------|
| `{{path:line}}` | {{category}} | {{"deliberate per ADR-3" | "shared-kernel constraint" | "exemplar in repo, not a smell"}} |

_(Omit if empty.)_

## Target inventory

_Files read during the audit. Useful for re-audit and for confirming the right slice was evaluated._

| File | Role | Lines | Smells found |
|------|------|-------|--------------|
| `{{path/to/a.ts}}` | {{production | test | config}} | {{approx LOC}} | F-1, F-3 |
| `{{path/to/b.ts}}` | {{...}} | {{...}} | — |

_Files in target surface that were NOT read:_ {{"none" or list with reason — e.g. "generated/, excluded per `.gitignore`"}}

## Handoff — for `/execute` REFACTOR mode

_A flat list of every individual finding, machine-readable. Consumed directly by `/execute` at Phase 0.5 findings triage._

| ID | Severity | File:line | Smell | Intent |
|----|----------|-----------|-------|--------|
| F-1 | P0 | `{{path:line}}` | {{smell}} | {{one-line refactor shape}} |
| F-2 | P0 | `{{path:line}}` | {{smell}} | {{...}} |
| F-3 | P1 | `{{path:line}}` | {{smell}} | {{...}} |

_(Architecture observations are human-read; they are not in this table. Bugs surfaced go to `/diagnosis`, not `/execute`.)_

## Next steps

1. {{e.g., "Route B-1 through `/diagnosis` — it's a live crash, not debt."}}
2. {{e.g., "`/execute` REFACTOR on this folder — address F-1, F-2 (P0), F-3, F-4 (P1). Defer P2/P3."}}
3. {{e.g., "Review architecture observation A-1 with the team before picking refactor order — the Shotgun Surgery shape may motivate a bigger extraction."}}
4. {{e.g., "Re-run `/audit` after the REFACTOR pass — appended as `## Re-audit <DATE>` — to confirm the selected findings cleared and no new smells surfaced."}}

## Re-audit

_This section is appended on each re-audit. Original audit content above is preserved._

<!-- Re-audit append target: new ## Re-audit <DATE> sections are added below this comment on subsequent runs.

Each re-audit block follows the shape documented in the `audit` SKILL.md Phase 0.1:

## Re-audit {{DATE}}

**Scope:** {{TIER}}. **Target:** {{path | unchanged}}. **Findings:** P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}}.

### Verification of prior findings

(Include only when prior `## Refactors applied <DATE>` sections existed in adjacent EXECUTION.md. Status values:
 verified | still present | vanished | orphaned — prior claim, no current evidence.)

| Prior finding | Status | Notes |
|---------------|--------|-------|
| F-1 (addressed in EXECUTION at `abc1234`) | verified | No current finding at `src/checkout/coupon.ts:42` |
| F-2 (addressed at `def5678`) | still present | Current finding F-N quotes same smell |
| F-4 (deferred) | still present (expected) | — |

### What changed since last audit

- {{one-line delta}}

### New findings

[... severity-grouped findings in the same shape as the main artifact ...]
-->
