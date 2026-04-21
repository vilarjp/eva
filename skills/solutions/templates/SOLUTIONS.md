---
name: solutions
date: "{{DATE}}"
status: "{{draft | approved}}"
topic: "{{short topic — what the pipeline was about}}"
scope: "{{LIGHTWEIGHT | STANDARD | DEEP}}"
pipeline: {{[feature] | [bug] | [feature, bug]}}
artifacts_synthesised: [{{PRD.md, SPEC.md, REVISION.md, DIAGNOSIS.md, EXECUTION.md, CODE-REVIEW.md — actual list}}]
post_release_bug_fix_appended: {{false | true}}
supersedes: "{{omit unless this is SOLUTIONS-2.md, else: SOLUTIONS.md}}"
---

# Solutions: {{topic}}

**Pipeline:** {{feature | bug | mixed}} · **Scope:** {{LIGHTWEIGHT | STANDARD | DEEP}} · **Folder:** `docs/{{DATE}}-{{slug}}/`
**Artifacts synthesised:** {{comma-separated list}}

## Summary

{{2-3 sentences. Feature → what shipped + intent. Bug → what broke + fix shape.
Mixed → feature first, then bug fix.}}

## Root cause

_Bug / mixed pipelines only. Otherwise cut._

{{1-3 sentences naming the failure as an **invariant**, not a file:line.
Example: "`applyCoupon` reads `cart.items` synchronously before the cart-load
promise resolves — the invariant `coupons require a non-empty cart` is checked
downstream rather than at the entry point, so an empty array silently derives a
zero total and 500s the totals endpoint."}}

## Approach

{{2-4 sentences. What was done + why this path over alternatives. Pull from
SPEC mini-ADRs that proved their weight and EXECUTION's per-slice intent. For
mixed pipelines: feature shape first, then bug fix as a second paragraph.}}

## Key decisions

Compact mini-ADRs — only decisions that matter for future readers. One row per
decision. Alternatives column is mandatory except on Lightweight.

| Decision (behaviour / invariant)                    | Why (rationale)                     | Trade-off accepted           | Alternatives considered                    |
|-----------------------------------------------------|-------------------------------------|------------------------------|--------------------------------------------|
| {{e.g. idempotent coupon apply via dedup key}}      | {{what it optimises for}}           | {{honest downside}}          | {{(a) server lock — rejected: ...; (b) at-least-once — rejected: ...}} |

If no decisions emerged beyond SPEC mini-ADRs, state "See SPEC.md." Don't pad.

## Gotchas

Behaviour invariants — phrased to survive a file rename. Pull from EXECUTION
NOTICED-BUT-NOT-TOUCHING / concerns, DIAGNOSIS concerns, CODE-REVIEW residual
risks.

- {{e.g. "The `cart_items` cache must be invalidated before the write commits — invalidating after leaves a stale-read window."}}
- {{e.g. "`applyCoupon` assumes `cart.items` hydrated; callers during loading must gate on `cart.status === 'ready'`."}}

If none surfaced, cut the section.

## What didn't work

Map of the territory for the next session.

| Attempted approach          | Why it failed                         | Source                                |
|-----------------------------|---------------------------------------|---------------------------------------|
| {{approach}}                | {{evidence / outcome}}                | {{EXECUTION slice N / DIAGNOSIS H-N / CODE-REVIEW F-N}} |

If none, cut the section.

## Prevention

_Bug / mixed pipelines only. Otherwise cut._

- **Class of bug:** {{one-line label — e.g. "null propagation past a validation boundary"}}
- **Defense-in-depth:** {{entry / business / persistence / output — whichever DIAGNOSIS recommended}}
- **Regression test:** `{{test name from DIAGNOSIS — e.g. "applyCoupon throws when cart is still loading (GH#1234)"}}` plus any CODE-REVIEW testing gaps worth closing.

## References

- [PRD.md](./PRD.md) · [SPEC.md](./SPEC.md) · [REVISION.md](./REVISION.md) · [DIAGNOSIS.md](./DIAGNOSIS.md) · [EXECUTION.md](./EXECUTION.md) · [CODE-REVIEW.md](./CODE-REVIEW.md)

Cut any link whose artifact is absent from this folder.

<!--
Optional sections — include ONLY when non-empty:

### Diagram

_Mandatory on Deep, optional on Standard, cut on Lightweight._

```mermaid
{{sequenceDiagram / stateDiagram-v2 / flowchart — the durable mechanism}}
```

### Relationship to original spec (mixed pipelines only)

{{One paragraph on how the bug related to the original feature — what the SPEC
covered and what it missed.}}

**Forward-reference:** `## Post-Release Bug Fix {{DATE}}` in [SPEC.md](./SPEC.md).

Cut any section above if empty. See _shared/artifact-compactness.md.

Re-run append target: new `## Post-Release Bug Fix <DATE>` or `## Re-run <DATE>`
sections are added below on subsequent runs.
-->
