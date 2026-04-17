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

**Pipeline:** {{feature | bug | mixed (feature + post-release bug fix)}}
**Scope:** {{LIGHTWEIGHT | STANDARD | DEEP}}
**Folder:** `docs/{{DATE}}-{{slug}}/`
**Artifacts synthesised:** {{comma-separated list}}

## Summary

{{2-3 sentences. For a feature pipeline, name what shipped and its core intent. For a bug pipeline, name what was broken, who was affected, and the shape of the fix. For a mixed pipeline, cover both in order: feature first, bug-fix second.}}

## Root Cause

_Omit this section unless the pipeline includes a DIAGNOSIS. For mixed pipelines, cover the bug's root cause here; the feature's original design is captured in Approach and Key Decisions._

{{1-3 sentences describing the mechanism of failure as an invariant. Name the boundary that was violated and why the violation propagated. DO NOT reference file:line as the substance — only as illustration inside a prose clause if unavoidable. Example:

"The `applyCoupon` handler reads `cart.items` synchronously before the cart-load promise resolves. When the user clicks Apply while the cart is still hydrating, the handler receives an empty array, derives a zero total, and returns a 500 from the downstream totals endpoint because the invariant `coupons require a non-empty cart` is checked downstream rather than at the entry point."}}

## Approach

{{2-4 sentences on what was done and why this path over the alternatives. Pull from SPEC mini-ADRs that proved their weight and EXECUTION's per-slice intent.

For feature pipelines: name the shape of the solution, the alternative it rejected, and the trade-off accepted.

For bug pipelines: name the fix (root-cause vs symptom vs defense-in-depth), whether a reproduction test was added, and whether the fix was single-point or layered.

For mixed pipelines: cover the feature shape first, then the bug fix as a second paragraph.}}

## Key Decisions

_Compact mini-ADR format. Only capture decisions that matter for future readers — planning decisions that proved their weight under implementation, unexpected decisions that emerged mid-execute, revision patches that shifted direction, and code-review drift that was deliberately accepted rather than fixed._

### Decision: {{one-line name — e.g. "Idempotent coupon apply via request dedup key"}}

- **What:** {{the decision, phrased as a behaviour or invariant. Example: "every coupon-apply request carries a client-generated dedup key; the server rejects duplicates with a 409 instead of double-applying"}}
- **Why:** {{rationale — what it optimises for. Example: "users double-click Apply under slow network; without dedup, the coupon stacks and the user sees a negative total"}}
- **Trade-off accepted:** {{honest downside. Example: "the client must manage a UUID per attempt; a 409 is returned on legitimate retries of the same attempt, which the UI must swallow silently"}}
- **Alternatives considered:** {{1-2 genuine alternatives, one line each. Omit for Lightweight scope. Example: "(a) server-side lock keyed on `user_id + coupon_code` — rejected: does not cover cross-session retries; (b) at-least-once semantics with reversal job — rejected: adds eventual-consistency window that confuses the UI"}}

### Decision: {{next}}

{{… same shape …}}

_(If no decisions emerged beyond what SPEC mini-ADRs already captured, state "No post-plan decisions surfaced during this pipeline; see SPEC.md for planning-time mini-ADRs." Do not list placeholder decisions.)_

## Gotchas

_Each entry is a behaviour invariant. Phrase so it would survive a file rename. Pull from EXECUTION's NOTICED-BUT-NOT-TOUCHING and Concerns, DIAGNOSIS concerns, CODE-REVIEW residual risks and testing gaps._

- {{Example: "The `cart_items` cache must be invalidated before the write commits; invalidating after leaves a window where concurrent readers see stale values."}}
- {{Example: "`applyCoupon` assumes `cart.items` is hydrated; callers that run during the loading state must gate on `cart.status === 'ready'` before dispatching."}}
- {{Example: "The coupon totals endpoint returns 500 on empty carts rather than a semantic error; until that endpoint is revised, upstream handlers must ensure the cart is non-empty before calling it."}}

_(If no gotchas surfaced, state "No gotchas surfaced during this pipeline. This section is intentionally empty." Do not pad with platitudes.)_

## What Didn't Work

_A map of the territory for the next session. Pull from EXECUTION 3-strike Step-Back events, DIAGNOSIS REJECTED hypotheses, CODE-REVIEW suppressed findings._

- **{{Attempted approach}}** — {{one-line summary}}. Why it failed: {{evidence / outcome}}. Source: {{EXECUTION slice N | DIAGNOSIS hypothesis N | CODE-REVIEW F-N}}.
- **{{Another dead end}}** — {{…}}
- **{{Another dead end}}** — {{…}}

_(If none, state "No material dead ends surfaced during this pipeline." Do not invent entries.)_

## Prevention

_Omit this section unless the pipeline includes a DIAGNOSIS._

- **Class of bug:** {{one-line label. Example: "null propagation past a validation boundary"}}
- **Where to add defense-in-depth:** {{entry point / business layer / persistence / output — whichever DIAGNOSIS recommended or CODE-REVIEW testing gaps highlighted. Example: "entry-point validation in the handler so the state invariant `cart.status === 'ready'` is enforced before the totals call, plus a schema-level NOT NULL on `cart.items` in the database"}}
- **Regression protection:** {{the reproduction test from DIAGNOSIS is the primary guard. Reference it by test name, not file path. Add any CODE-REVIEW testing gaps worth closing. Example: "`applyCoupon throws when cart is still loading (GH#1234)` guards the entry-point fix. CODE-REVIEW flagged missing coverage for the legacy retry path — add `applyCoupon dedups repeat attempts within 2s`."}}

## Diagram

_Required for Deep scope. Optional for Standard when structure warrants (sequence, state, or decision flow). Omit for Lightweight. The diagram depicts the durable mechanism — the invariant being protected or the flow being modelled — not the file layout._

```mermaid
{{mermaid block — sequenceDiagram for cross-component flow, stateDiagram-v2 for state machines, flowchart for decision trees}}
```

## References

- [{{PRD.md}}](./PRD.md) — {{YYYY-MM-DD}}, {{status}}
- [{{SPEC.md}}](./SPEC.md) — {{YYYY-MM-DD}}, {{status}}
- [{{REVISION.md}}](./REVISION.md) — {{YYYY-MM-DD}}, {{status}}  <!-- omit if absent -->
- [{{DIAGNOSIS.md}}](./DIAGNOSIS.md) — {{YYYY-MM-DD}}, {{status}}  <!-- omit if absent -->
- [{{EXECUTION.md}}](./EXECUTION.md) — {{YYYY-MM-DD}}, {{status}}  <!-- omit if absent -->
- [{{CODE-REVIEW.md}}](./CODE-REVIEW.md) — {{YYYY-MM-DD}}, {{status}}  <!-- omit if absent -->

## Relationship to Original Spec

_Include ONLY for mixed pipelines where a DIAGNOSIS was appended to a feature folder. Otherwise delete this section._

**How the bug related to the original feature:**

{{One paragraph. Example: "The original SPEC specified the coupon handler's happy path but did not capture the cart-loading edge case — the Test Strategy section classified it as 'covered by unit tests' but the integration-level race was below the coverage bar. The fix adds entry-point validation and defense-in-depth at the totals endpoint, which the SPEC's Risks section had flagged as 'consider layered validation' but the tracer-bullet phases did not schedule."}}

**Forward-reference in the source SPEC:** `## Post-Release Bug Fix {{DATE}}` section in [SPEC.md](./SPEC.md) (added by `/diagnosis` or `/execute` when the bug was handled).

<!-- Re-run append target: new ## Post-Release Bug Fix <DATE> or ## Re-run <DATE> sections are added below this comment on subsequent runs. Prior content above is preserved as history. -->
