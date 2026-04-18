---
name: execution
date: "{{DATE}}"
status: "{{in_progress | completed}}"
topic: "{{short topic — what was built or fixed or refactored}}"
mode: "{{FEATURE | BUG | FIX | REFACTOR | RAW}}"
source: "{{path to SPEC.md | REVISION.md | DIAGNOSIS.md | CODE-REVIEW.md | AUDIT.md, or 'raw prompt'}}"
scope: "{{LIGHTWEIGHT | STANDARD | DEEP}}"
branch: "{{branch name this execution ran on}}"
slices_total: {{integer}}
slices_completed: {{integer}}
tests_added: {{integer}}
commits: {{integer}}
---

# Execution: {{topic}}

## Summary

**What shipped:** {{one-paragraph plain-language account of what the code now does (or what stops breaking)}}

**Source of truth:** {{`docs/YYYY-MM-DD-<slug>/SPEC.md` at phase N | `docs/YYYY-MM-DD-<slug>/DIAGNOSIS.md` | `docs/YYYY-MM-DD-<slug>/CODE-REVIEW.md` | `docs/YYYY-MM-DD-<slug>/AUDIT.md` | raw prompt}}

**Integration gate:** {{GREEN (all passed) | GREEN with carve-out (see below)}}

## Micro-spec (RAW mode only)

_Include this section ONLY when `mode: RAW`. Otherwise delete._

- **Goal:** {{one sentence}}
- **Acceptance criteria:**
  - {{criterion 1}}
  - {{criterion 2}}
  - {{criterion 3}}
- **Modules likely touched:** {{short file list}}
- **Risks identified up front:** {{honest, short}}

## Findings Addressed (FIX mode only)

_Include this section ONLY when `mode: FIX`. Otherwise delete._

Selected from `{{path/to/CODE-REVIEW.md}}` at Phase 0.5 triage. Each finding maps to the slice that resolved it.

| Finding | Severity | Title | File:line | Slice | Commit | Test |
|---------|----------|-------|-----------|-------|--------|------|
| F-1 | P0 | {{title}} | {{file:line}} | S1 | `{{SHA}}` | {{test/path.ts:line}} |
| F-2 | P1 | {{title}} | {{file:line}} | S2 | `{{SHA}}` | {{test/path.ts:line}} |

## Findings Skipped (FIX mode only)

_Include this section ONLY when `mode: FIX`. Otherwise delete._

Findings surfaced by code review and intentionally NOT addressed in this execution. These remain open issues — they will reappear in the next `/code-review` run unless explicitly addressed later.

| Finding | Severity | Title | Reason |
|---------|----------|-------|--------|
| F-4 | P2 | {{title}} | {{user-supplied reason — default "deferred by user at triage"}} |
| F-6 | P3 | {{title}} | {{reason}} |

_If every shown finding was selected, write `None.`_

## Findings Addressed (REFACTOR mode only)

_Include this section ONLY when `mode: REFACTOR`. Otherwise delete._

Selected from `{{path/to/AUDIT.md}}` at Phase 0.5 triage. Each finding maps to the slice that resolved it. Behaviour-preservation is the contract — the `Test` column cites the characterization test that stayed GREEN through the refactor, or `verification` for mechanical slices.

| Finding | Severity | Smell | Title | File:line | Slice | Commit | Test |
|---------|----------|-------|-------|-----------|-------|--------|------|
| F-1 | P1 | {{Feature Envy | Shotgun Surgery | SCATTER | ...}} | {{title}} | {{file:line}} | S1 | `{{SHA}}` | {{test/path.ts:line | verification}} |
| F-2 | P2 | {{smell}} | {{title}} | {{file:line}} | S2 | `{{SHA}}` | {{test/path.ts:line}} |

## Findings Skipped (REFACTOR mode only)

_Include this section ONLY when `mode: REFACTOR`. Otherwise delete._

Findings surfaced by audit and intentionally NOT addressed in this execution. These remain open debt items — they will resurface in the next `/audit` re-run under `## Re-audit <DATE>` unless addressed later.

| Finding | Severity | Smell | Title | Reason |
|---------|----------|-------|-------|--------|
| F-4 | P2 | {{smell}} | {{title}} | {{user-supplied reason — default "deferred by user at triage"}} |
| F-6 | P3 | {{smell}} | {{title}} | {{reason}} |

_If every shown finding was selected, write `None.`_

## Bugs surfaced routed to /diagnosis (REFACTOR mode only)

_Include this section ONLY when `mode: REFACTOR` AND the AUDIT.md had a `## Bugs surfaced` section. Otherwise delete._

Real current bugs flagged by the audit were NOT processed by this REFACTOR execution — they require `/diagnosis`, not a refactor. Listed here so the user can chase them next.

| Audit ID | Shape | File:line | Recommended |
|----------|-------|-----------|-------------|
| B-1 | {{crash | data loss | silent failure | security hole}} | {{file:line}} | run `/diagnosis` |

## Slices

Vertical tracer-bullet slices executed in order. One row per slice, committed independently. `Mode` values: `full-tdd` (RED-first behaviour), `characterization-tdd` (REFACTOR — GREEN baseline pinned first, preserved after), `verification` (non-behaviour or mechanical REFACTOR).

| # | Intent | Mode | Test file | Commit SHA | Result |
|---|--------|------|-----------|------------|--------|
| 1 | {{slice 1 capability or fix or refactor}} | {{full-tdd | characterization-tdd | verification}} | {{path/to/test-file}} | {{7-char SHA}} | {{GREEN}} |
| 2 | {{slice 2}} | {{full-tdd | characterization-tdd | verification}} | {{path}} | {{SHA}} | {{GREEN}} |

### Slice 1 — {{intent}}

**Mode:** {{full-tdd | characterization-tdd | verification}}

**Intent:** {{what became possible / what stopped breaking / what smell was resolved}}

**RED proof (full-tdd mode only):**
```
$ {{exact test command}}
{{literal failing output with assertion details}}
```

**Characterization baseline (characterization-tdd mode only — REFACTOR):**
```
$ {{exact test command}}
{{literal PASSING output BEFORE the refactor — this is the behavioural baseline}}
```

**GREEN proof:**
```
$ {{exact test command}}
{{literal passing output — for characterization-tdd, same assertions still pass after the refactor}}
```

**Verification Mode output (verification mode only):**
```
$ {{build command — e.g. pnpm build}}
{{literal successful output}}
$ {{test command}}
{{literal passing output confirming no regressions}}
```

**Files touched:** {{comma-separated, explicit — not "git add -A"}}

**Commit:** `{{7-char SHA}}` — `{{conventional commit subject}}`

**Acceptance criteria satisfied (FEATURE / BUG / FIX / RAW):** {{cite SPEC phase N criteria, DIAGNOSIS root cause, or CODE-REVIEW F-N}}

**Finding resolved (REFACTOR only):** {{F-N, severity, smell — e.g. F-3 (P1, Shotgun Surgery)}}

_Repeat the above block per slice._

## Integration Gate

Ran after the final slice. All captured commands are fresh runs, not paraphrased.

**Full test suite:**
```
$ {{exact command}}
{{literal output — counts line plus summary}}
```
**Result:** `{{N passed / M failed / K skipped}}` using `{{runner}}`

**Lint:**
```
$ {{exact command}}
{{literal output}}
```
**Result:** {{OK | errors — listed}}

**Type-check:**
```
$ {{exact command}}
{{literal output}}
```
**Result:** {{OK | errors — listed}}

**Skipped tests audit:** {{"none" OR a short list of each skip with its legitimate reason — pre-existing skip, platform-gated, deferred by user approval}}

**Regression red-green proof (BUG mode and FIX mode, when practical):** {{"captured below" OR "not applicable — <reason>"}}

```
$ {{revert fix → re-run repro test → verify RED}}
{{literal RED output}}
$ {{restore fix → re-run repro test → verify GREEN}}
{{literal GREEN output}}
```

**Behaviour-preservation proof (REFACTOR mode):** {{"captured below — full suite green before Slice 1 and after the last slice" OR "not applicable — RAW"}}

```
$ {{full-suite command before Slice 1}}
{{literal GREEN output — baseline}}
$ {{full-suite command after the last slice}}
{{literal GREEN output — behaviour preserved}}
```

**Carve-outs (if any):** {{list any failure or skip that was explicitly accepted by the user with their approval quoted; otherwise write "none"}}

## Acceptance Criteria Trace

Map every source-of-truth requirement to the slice and test that satisfies it.

| Criterion (source) | Slice | Test | Status |
|--------------------|-------|------|--------|
| {{SPEC Phase 1 criterion 1 quote | DIAGNOSIS root cause | CODE-REVIEW F-N | AUDIT F-N}} | {{S1}} | {{test/path.ts:line}} | {{GREEN}} |
| {{criterion 2}} | {{S1}} | {{test/path.ts:line}} | {{GREEN}} |

## NOTICED BUT NOT TOUCHING

Out-of-scope improvements surfaced during execution and NOT acted on. Each entry is an invitation for a future task, not a hidden change.

- {{observation — file / location}} — out of scope because {{reason}}
- {{observation}} — out of scope because {{reason}}

_If nothing was noticed, write `None.`_

## Follow-ups

Concrete TODOs worth tracking after this lands. These are NOT part of the current change.

- {{follow-up 1 — enough context to pick up cold}}
- {{follow-up 2}}

_If nothing is pending, write `None.`_

## Concerns

Risks or low-confidence areas the reviewer should watch.

- {{risk — e.g. touches error path rarely exercised in production}}
- {{low-confidence area — e.g. happy-path test only; edge cases deferred to SPEC Phase 2}}
- {{post-deploy watch — e.g. monitor error rate on endpoint X}}

_If none, write `None.`_

## Commits

Chronological list of commits created by this execution. None were pushed.

| # | SHA | Subject |
|---|-----|---------|
| 1 | `{{7-char SHA}}` | {{conventional commit subject}} |
| 2 | `{{7-char SHA}}` | {{subject}} |

## Relationship to Source Artifact

_Include the appropriate subsection; delete the others._

**FEATURE mode:** Implemented SPEC at `{{path/to/SPEC.md}}`. Appended `## Implemented {{DATE}}` back-pointer to that file.

**BUG mode:** Applied fix for DIAGNOSIS at `{{path/to/DIAGNOSIS.md}}`. Appended `## Fixed {{DATE}}` back-pointer to that file.

**FIX mode:** Addressed selected findings in `{{path/to/CODE-REVIEW.md}}`. Appended `## Fixes applied {{DATE}}` back-pointer to that file with the addressed-vs-skipped breakdown and a link to this EXECUTION.md. Next step for the human: re-run `/code-review` to append a `## Re-review {{NEXT_DATE}}` section confirming the delta.

**REFACTOR mode:** Applied selected refactors from `{{path/to/AUDIT.md}}`. Appended `## Refactors applied {{DATE}}` back-pointer to that file with the addressed-vs-skipped breakdown (including the named smell per finding) and a link to this EXECUTION.md. Bugs surfaced by the audit (if any) route to `/diagnosis`, not here. Next step for the human: re-run `/audit` to append a `## Re-audit {{NEXT_DATE}}` section confirming prior findings cleared.

**RAW mode:** No upstream artifact. This EXECUTION.md is standalone.
