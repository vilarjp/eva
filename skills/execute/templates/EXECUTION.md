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

**Shipped:** {{one paragraph — what the code now does (or what stops breaking).}}

**Source of truth:** `{{path}}` | **Integration gate:** {{GREEN | GREEN with carve-out}}

## Findings triage

_Include only in FIX (source=CODE-REVIEW.md) and REFACTOR (source=AUDIT.md) modes. Otherwise cut._

**Addressed:**

| ID  | Sev | Title / smell         | File:line     | Slice | Commit      | Test                  |
|-----|-----|-----------------------|---------------|-------|-------------|-----------------------|
| F-1 | P0  | {{title}} ({{smell}}) | `{{path:line}}` | S1    | `{{SHA}}`   | `{{test:line}}`       |

**Skipped (open — will resurface in next `/code-review` or `/audit`):**

| ID  | Sev | Title / smell | Reason                             |
|-----|-----|---------------|------------------------------------|
| F-4 | P2  | {{title}}     | {{default "deferred at triage"}}  |

**Bugs routed to `/diagnosis`** _(REFACTOR only — audit's bugs surfaced are NOT refactored):_

| Audit ID | Shape | File:line |
|----------|-------|-----------|
| B-1 | {{crash / data loss / silent / security}} | `{{path:line}}` |

## Slices

`full-tdd` = RED-first behaviour. `characterization-tdd` = REFACTOR with GREEN
baseline pinned before change. `verification` = non-behaviour / mechanical.

| # | Intent                                       | Mode                   | Test                 | Commit    | Acceptance / Finding |
|---|----------------------------------------------|------------------------|----------------------|-----------|----------------------|
| 1 | {{capability / fix / refactor}}              | full-tdd               | `{{test:line}}`      | `{{SHA}}` | {{SPEC P1 c1 / DIAGNOSIS root / F-N}} |
| 2 | {{...}}                                      | characterization-tdd   | `{{test:line}}`      | `{{SHA}}` | {{F-3 (P1, Shotgun Surgery)}} |

### Slice proofs

_One block per slice. RED proof mandatory on `full-tdd`; characterization baseline
mandatory on `characterization-tdd`; build+test output mandatory on `verification`._

**Slice 1 — {{intent}}** — mode: {{full-tdd | characterization-tdd | verification}}

```
$ {{RED or baseline command}}
{{literal output}}
$ {{GREEN command}}
{{literal passing output}}
```

**Files:** {{explicit comma-separated list — never "git add -A"}}.
**Commit:** `{{SHA}}` — `{{conventional subject}}`.

## Integration gate

All fresh runs, no paraphrasing. Exactly one code block each, ≤ 15 lines.

```
$ {{full test suite command}}
{{literal output — counts + summary}}
$ {{lint command}}
{{literal output}}
$ {{type-check command}}
{{literal output}}
```

**Result:** suite `{{N pass / M fail / K skip}}`; lint `{{OK / errors}}`; types `{{OK / errors}}`.
**Skipped tests:** {{"none" or short list with legitimate reason per skip}}.
**Carve-outs:** {{"none" or list with quoted user approval}}.

**Regression proof (BUG / FIX mode):**

```
$ {{revert → repro → RED}}
{{literal RED}}
$ {{restore → repro → GREEN}}
{{literal GREEN}}
```

**Behaviour-preservation (REFACTOR):**

```
$ {{full suite before Slice 1}}
{{literal GREEN — baseline}}
$ {{full suite after last slice}}
{{literal GREEN — preserved}}
```

## Acceptance criteria trace

| Criterion (source)                       | Slice | Test              | Status |
|------------------------------------------|-------|-------------------|--------|
| {{SPEC P1 c1 / DIAGNOSIS root / F-N}}    | S1    | `{{test:line}}`   | GREEN  |

## Follow-ups & concerns

- **Noticed but not touching:** {{out-of-scope observation — why deferred}} _(invitation, not hidden change)_
- **Follow-up:** {{concrete TODO worth tracking after this lands}}
- **Risk / watch:** {{what could regress; what to monitor post-deploy}}

## Commits

| # | SHA | Subject |
|---|-----|---------|
| 1 | `{{SHA}}` | {{conventional subject}} |

## Relationship to source artifact

Pick one; delete the rest.

- **FEATURE:** Implemented SPEC at `{{path}}`. Appended `## Implemented {{DATE}}`.
- **BUG:** Applied fix for DIAGNOSIS at `{{path}}`. Appended `## Fixed {{DATE}}`.
- **FIX:** Addressed findings in `{{path}}`. Appended `## Fixes applied {{DATE}}` with the addressed/skipped breakdown + link to this file. Next: re-run `/code-review` for `## Re-review`.
- **REFACTOR:** Applied refactors from `{{path}}`. Appended `## Refactors applied {{DATE}}` with the addressed/skipped breakdown (smell per finding) + link to this file. Bugs surfaced route to `/diagnosis`. Next: re-run `/audit` for `## Re-audit`.
- **RAW:** No upstream artifact. Standalone.

<!--
Optional sections — include ONLY when non-empty:

### Micro-spec (RAW mode only)

- **Goal:** {{one sentence}}
- **Acceptance:** {{criterion}}; {{criterion}}; {{criterion}}
- **Modules likely touched:** {{file list}}
- **Risks up front:** {{honest, short}}

Cut any section above if empty. See _shared/artifact-compactness.md.
-->
