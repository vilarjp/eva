---
name: code-review
date: "{{DATE}}"
status: "{{draft | approved}}"
topic: "{{short topic — what was reviewed}}"
scope: "{{LIGHTWEIGHT | STANDARD | DEEP}}"
diff_source: "{{uncommitted | branch-vs-<base> | head-1}}"
diff_stats: "{{files: N, +LOC / -LOC}}"
reviewers_run: "{{code-quality, convention, test[, plan-alignment][, security][, performance][, adversarial]}}"
plan_reference: "{{omit unless a plan existed, else: docs/YYYY-MM-DD-<slug>/SPEC.md}}"
spec_directory_reuse: {{false | true}}
original_spec_dir: "{{omit unless spec_directory_reuse is true, else: docs/YYYY-MM-DD-<slug>/}}"
---

# Code Review: {{topic}}

**Diff source:** `{{diff_source}}` — {{files count}} files, +{{added}} / -{{removed}} LOC.
**Scope tier:** {{LIGHTWEIGHT | STANDARD | DEEP}}.
**Reviewers:** {{list}}.
**Plan reference:** {{path or "none — review based on diff shape alone"}}.

## TL;DR

**Posture:** {{one sentence — green light with notes | yellow — N P0/P1 to resolve | red — blockers must be addressed}}

- Blockers (P0): **{{count}}**
- Major (P1): **{{count}}**
- Minor (P2): **{{count}}**
- Nits (P3): **{{count}}**
- Suppressed (low confidence): **{{count}}**
- Pre-existing (outside the diff): **{{count}}**

{{one-line verdict — "Fix P0s, triage P1s, merge." or "Stop — P0 at <file:line> must be addressed first." or "Clean pass — green to merge."}}

## Stage results

| Stage | Reviewer | Status | Findings | Notes |
|-------|----------|--------|----------|-------|
| 1 | plan-alignment | {{ran | skipped — <reason> | clean}} | P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}} | {{one-line}} |
| 2 | code-quality | {{ran | clean}} | P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}} | {{one-line}} |
| 2 | convention | {{ran | clean}} | P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}} | {{one-line}} |
| 2 | test | {{ran | clean}} | P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}} | {{one-line}} |
| 2 | security | {{ran (auto-triggered by <pattern>) | skipped — no trigger | clean}} | P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}} | {{one-line}} |
| 2 | performance | {{ran (auto-triggered by <pattern>) | skipped — no trigger | clean}} | P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}} | {{one-line}} |
| 3 | adversarial | {{ran | skipped — not Deep scope}} | P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}} | {{one-line}} |

## Blockers (P0)

_These must be resolved before merge._

### F-1 — {{title}}
- **Severity:** P0
- **Confidence:** {{0.00-1.00}}
- **File:** [`{{path}}:{{line}}`]({{path}}:{{line}})
- **Seen by:** {{reviewer(s) — e.g. "code-quality, security (agreement boost applied)"}}
- **Evidence:**
  - `{{verbatim quote from diff or file}}` — {{path:line}}
  - `{{optional second quote}}`
- **Why it matters:** {{one sentence — concrete risk}}
- **Suggested fix:** {{one or two sentences — minimal}}
- **Autofix class:** {{safe_auto | gated_auto | manual | advisory}}
- **Verification needed:** {{omit if false, else: "Yes — <what to verify>"}}

### F-2 — {{title}}
[... same shape ...]

_(If no P0 findings: state "No blockers. Review proceeds to Major." Do not list placeholder entries.)_

## Major (P1)

_Fix before merge or explicitly accept the residual risk._

### F-3 — {{title}}
[... same shape as P0 ...]

## Minor (P2)

_Worth fixing; not ship-blocking._

### F-N — {{title}}
[... same shape ...]

## Nits (P3)

_Preference; author may decline._

- **F-M** [`{{path}}:{{line}}`]({{path}}:{{line}}) — {{title}}. Fix: {{one sentence}}.

_(For P3, the single-line form is acceptable to reduce noise. Keep the severity-grouped ordering.)_

## Positives

_What the reviewers specifically liked — so it is not regressed on iteration._

- {{reviewer}}: {{one-line observation}}
- {{reviewer}}: {{one-line observation}}

## Residual risks

_Concerns a reviewer surfaced but could not fully confirm; worth the author's attention._

- {{one-line risk, reviewer attribution}}
- {{...}}

## Testing gaps

_Scenarios the tests under review do not cover, worth adding even if not formally P1._

- {{one-line — what is untested and where}}
- {{...}}

## Suppressed findings (low confidence)

_Findings below the 0.60 confidence threshold. Surfaced for visibility; not top-bar._

| ID | Severity | Confidence | Title | File:line | Reviewer |
|----|----------|-----------|-------|-----------|----------|
| S-1 | {{P2}} | 0.55 | {{title}} | `{{path:line}}` | {{reviewer}} |
| S-2 | {{P3}} | 0.45 | {{title}} | `{{path:line}}` | {{reviewer}} |

_(Omit this section if there are no suppressed findings.)_

## Pre-existing findings (outside this diff)

_Problems the diff happens near but did not introduce. Triage for a follow-up; author is not responsible for them in this review._

| ID | Severity | Title | File:line | Reviewer |
|----|----------|-------|-----------|----------|
| PE-1 | {{P1}} | {{title}} | `{{path:line}}` | {{reviewer}} |

_(Omit this section if there are no pre-existing findings.)_

## Plan alignment summary

_Only when a plan document is present (PRD/SPEC/DIAGNOSIS/REVISION/EXECUTION)._

- **Plan path:** `{{docs/YYYY-MM-DD-<slug>/SPEC.md}}`
- **Acceptance criteria satisfied:** {{N of M}}
- **Acceptance criteria unmet:** {{list — with finding IDs if flagged}}
- **Non-goals respected:** {{yes | see F-N}}
- **Mini-ADRs cross-checked:** {{N of M, all consistent | see F-N for drift}}
- **Silent decision changes:** {{none | see F-N}}

_(Omit this section if no plan document exists.)_

## Diff inventory

_Audit of what was reviewed. Useful for re-review and for confirming the right slice was evaluated._

| File | +LOC | -LOC | Reviewed by |
|------|------|------|-------------|
| `{{path/to/a.ts}}` | 42 | 12 | code-quality, convention, test |
| `{{path/to/b.ts}}` | 15 | 3 | code-quality, convention, security |
| `{{path/to/a.test.ts}}` | 60 | 0 | test |

_Files in diff that no reviewer opened:_ {{"none" or list — a gap signal if any}}

## Next steps

1. {{e.g., "Address F-1 and F-2 (P0s) — both in src/checkout/coupon.ts."}}
2. {{e.g., "Decide on F-3 (P1): accept the residual risk in the PR description, or fix now."}}
3. {{e.g., "Consider P2/P3 findings in a follow-up if out of scope for this PR."}}
4. {{e.g., "Optional: re-run `/code-review` after fixes to confirm the delta is clean."}}

## Re-review

_This section is appended on each re-review. Original review content above is preserved. On re-review, prior `## Fixes applied <DATE>` sections (if any, appended by `/execute` in FIX mode) are verified against the current diff — see the Verification of prior fixes subsection in each re-review block below._

<!-- Re-review append target: new ## Re-review <DATE> sections are added below this comment on subsequent runs.

Each re-review block follows the shape documented in the `code-review` SKILL.md Phase 9:

## Re-review {{DATE}}

**Scope:** {{TIER}}. **Reviewers:** {{list}}. **Findings:** P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}}.

### Verification of prior fixes

(Include only when prior `## Fixes applied` sections existed. Status values:
 verified | mismatch — still present | still present (expected) | vanished | orphaned.)

| Prior claim | F-N (current) | Severity | Status | Notes |
|-------------|---------------|----------|--------|-------|
| F-1 (addressed at `abc1234`) | — | P0 | verified | No current finding at `src/checkout/coupon.ts:42` |
| F-2 (addressed at `def5678`) | F-3 | P0 | **mismatch — still present** | Current finding quotes same line |
| F-4 (skipped) | F-7 | P2 | still present (expected) | — |
| F-6 (skipped) | — | P3 | vanished | Line removed in refactor |

### What changed since last review

- {{one-line delta — e.g. "F-1 (P0) fixed in commit abc123; verified absent."}}

### New findings

[... severity-grouped findings in the same shape as the main artifact ...]
-->

