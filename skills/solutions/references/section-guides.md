# Section Guides — What to Pull from Which Artifact

Every section of SOLUTIONS.md pulls from specific places in the upstream artifacts. This is the map. Load this reference at Phase 2 if a section feels thin or if you're not sure where to look.

The universal rule behind every mapping: **phrase the output as a durable invariant**. A verbatim quote from EXECUTION that says "fixed src/foo.ts line 42" becomes a SOLUTIONS gotcha that says "the foo handler must validate input before dispatch; the previous skip-on-empty path allowed nulls to reach persistence."

## Summary (2-3 sentences)

**Primary sources:**
- `PRD.md` → Problem Statement (feature framing)
- `SPEC.md` → Context section + Technical Goals (what shipped)
- `DIAGNOSIS.md` → Bug Description + Impact (bug framing)
- `EXECUTION.md` → Summary section + Acceptance Criteria Trace

**Synthesis pattern:**
- **Feature pipeline:** "Shipped <capability> to <outcome>. <Acceptance-criteria trace count> criteria satisfied; commit-by-slice on <branch>."
- **Bug pipeline:** "Fixed <observed behaviour> affecting <audience>. Root cause: <one-line>. Reproduction: <test name>."
- **Mixed pipeline:** Two sentences — feature first, bug second.

**Pitfall:** Do not quote the SPEC's abstract goal verbatim — paraphrase into what actually shipped. A goal is a promise; the summary reports the delivery.

## Root Cause (bug + mixed only — 1-3 sentences)

**Primary sources:**
- `DIAGNOSIS.md` → Root Cause section, Causal Chain (name the violated invariant)
- `DIAGNOSIS.md` → Hypotheses table (the CONFIRMED row)

**Synthesis pattern:**
Rephrase the root cause as an invariant violation. The DIAGNOSIS says *"null dereference at src/checkout/coupon.ts:62"* — SOLUTIONS says *"`applyCoupon` reads `cart.items` before the load promise resolves; the handler assumes the cart is hydrated, but the entry point does not enforce this."* Name the boundary, name the invariant, name why the violation propagated.

**Behaviour-phrased** (good):
> "The validation middleware short-circuits on empty strings, which means `null` after coercion also skips validation; the database layer's `NOT NULL` check then rejects the insert with an opaque constraint error."

**File-anchored** (bad — rewrite):
> "Bug in src/middleware/validate.ts:42 — empty string check passes null."

## Approach (2-4 sentences)

**Primary sources:**
- `SPEC.md` → Key Decisions (mini-ADRs) — which ones actually drove implementation
- `SPEC.md` → Tracer-bullet Phases (the shape of the slicing)
- `EXECUTION.md` → Slice table + per-slice intent
- `DIAGNOSIS.md` → Suggested Fix section (for bug pipelines)
- `REVISION.md` → Patch plan (if the direction shifted post-review)

**Synthesis pattern:**
Feature: "Built <capability> in <N> vertical slices (<list>). Chose <approach> over <alternative> to optimise for <concern>." Add defense-in-depth note if multiple layers of validation were added.

Bug: "Fixed at <entry point | business layer | persistence>. Chose <root-cause | symptom | layered> fix because <reason>. Reproduction test added: <name>."

**Pitfall:** Don't catalogue every commit. The slice table is in EXECUTION; Summary mentions delivery; Approach explains the *shape* of the solution, not the step count.

## Key Decisions (compact mini-ADR)

**Primary sources:**
- `SPEC.md` → embedded mini-ADRs (Context / Decision / Alternatives / Consequences) — pull only the decisions that proved their weight
- `EXECUTION.md` → "unexpected decisions" or Confusion-resolution entries
- `REVISION.md` → Findings with severity P0/P1 that were patched (these are decisions, even if they don't say "decision")
- `CODE-REVIEW.md` → plan-alignment drift that was accepted rather than fixed (explicitly a decision)

**What "proved its weight" means:**
- The mini-ADR's decision was tested against a real constraint during EXECUTION (not just theoretical in SPEC).
- The alternative that was rejected would have been chosen by a reasonable reader without the rationale.
- A future maintainer reading the code and wondering "why not just X?" would find the answer here.

**Synthesis pattern** per decision:
- **What:** the decision as a behaviour / invariant — NOT "we used X library" (that's a dependency) but "every write path must carry a request dedup key" (that's an invariant).
- **Why:** the constraint it optimises for. Name the failure mode it prevents.
- **Trade-off accepted:** the honest cost. Every real decision has one. If the trade-off is "none," the alternative wasn't real.
- **Alternatives:** 1-2 genuine alternatives with a one-line reason for rejection. Required for Standard / Deep; optional for Lightweight.

**Example — real decision:**
> ### Decision: Idempotent coupon apply via request dedup key
> - **What:** every coupon-apply request carries a client-generated UUID; the server rejects duplicates with 409 within a 2-second window.
> - **Why:** users double-click Apply under slow network; without dedup, the coupon stacks and totals go negative.
> - **Trade-off accepted:** the client must generate a UUID per attempt, and legitimate retries of the same attempt return 409 — the UI must swallow silently.
> - **Alternatives considered:** (a) server lock on `user_id + coupon_code` — rejected: cross-session retries slip through; (b) at-least-once with reversal — rejected: eventual-consistency window confused the UI.

**Example — fake decision (delete):**
> ### Decision: Use TypeScript
> - **What:** the project uses TypeScript.
> - **Why:** type safety.

If you catch yourself writing something like the second example — it wasn't a real decision at this pipeline's scope. It belongs in project CLAUDE.md, not SOLUTIONS.

## Gotchas (behaviour-phrased invariants)

**Primary sources:**
- `EXECUTION.md` → NOTICED BUT NOT TOUCHING list (out-of-scope observations that surfaced during execution — often become future gotchas)
- `EXECUTION.md` → Concerns section (risks, low-confidence areas, post-deploy watches)
- `DIAGNOSIS.md` → Concerns section
- `CODE-REVIEW.md` → Residual risks + Testing gaps sections
- `SPEC.md` → Risks & Mitigations table (risks that materialised during implementation; skip the ones that didn't)

**Synthesis rule:**
Each gotcha must pass the **file-rename test**. Ask: *"if tomorrow someone renames every file in the module, is this gotcha still useful?"* If no, rewrite.

**Good gotchas:**
- "Cache invalidation must happen before the write commits — invalidating after leaves a window where concurrent readers see stale values."
- "`applyCoupon` assumes `cart.status === 'ready'`; callers that run during the loading state must gate before dispatching."
- "The totals endpoint returns HTTP 500 on empty carts instead of a semantic error; until that endpoint is versioned, upstream handlers must ensure cart.items.length > 0 before calling."

**Bad gotchas (rewrite before writing):**
- "See src/cache.ts:42 for the invalidation race." (anchors to file:line — dies on rename)
- "Be careful with the cache layer." (vague — no invariant named)
- "There's a bug in the legacy handler." (editorial — not an invariant)

**Empty-section rule:** If no gotchas surface after honest reading, state *"No gotchas surfaced during this pipeline. This section is intentionally empty."* with no padding. An honest empty section is always preferable to synthetic entries.

## What Didn't Work (dead-end map)

**Primary sources:**
- `EXECUTION.md` → any slice that triggered the 3-strike Step-Back protocol. Each attempt, each assumption, each reason it failed.
- `DIAGNOSIS.md` → REJECTED hypotheses in the Hypotheses table (evidence killed them — worth recording so the next session doesn't retread)
- `DIAGNOSIS.md` → INCONCLUSIVE hypotheses with the evidence ceiling noted
- `CODE-REVIEW.md` → Suppressed findings (flagged below 0.60 confidence — worth knowing they were considered and why they didn't make the bar)

**Synthesis pattern** per entry:
> **Attempted approach** — one-line summary. **Why it failed:** evidence / outcome. **Source:** EXECUTION slice N | DIAGNOSIS hypothesis N | CODE-REVIEW F-N.

**Good entry:**
> **Tried to solve with client-side debounce alone** — increased debounce to 500ms in `applyCoupon` dispatch. Why it failed: race condition persisted because the store was read synchronously before dispatch even fired; debounce only throttled the dispatch count, not the stale read. Source: EXECUTION slice 2, attempt 1-3 (3-strike Step-Back triggered).

**Bad entry (rewrite):**
> "First tried debouncing but it didn't work." (no evidence, no source — drop or rewrite)

**Empty-section rule:** Rare after a Standard or Deep pipeline, but legitimate after a clean Lightweight run. State *"No material dead ends surfaced during this pipeline."*

## Prevention (bug + mixed only)

**Primary sources:**
- `DIAGNOSIS.md` → Suggested Fix section (especially the defense-in-depth recommendation)
- `DIAGNOSIS.md` → Hotspots table (rephrased as "where to add boundary validation")
- `DIAGNOSIS.md` → Reproduction test (the primary regression guard)
- `CODE-REVIEW.md` → Testing gaps section (additional coverage worth adding)
- `EXECUTION.md` → regression red-green proof (BUG mode) — confirms the test actually protects

**Synthesis structure:**
- **Class of bug:** one-line label. Examples: "null propagation past a validation boundary", "race condition on an unobserved state transition", "invariant violation at an integration seam", "schema drift between ORM and database."
- **Where to add defense-in-depth:** name the boundary (entry / business / persistence / output) where layered validation would have caught this. Be concrete about what the layer checks.
- **Regression protection:** the DIAGNOSIS reproduction test is the primary guard — reference by test name, not file path. If CODE-REVIEW flagged additional testing gaps relevant to this class, name them.

**Good prevention entry:**
> - **Class of bug:** null propagation past a validation boundary.
> - **Where to add defense-in-depth:** entry-point validation enforcing `cart.status === 'ready'` before dispatch; schema-level `NOT NULL` on `cart.items` to fail fast at the database; error translation at the totals endpoint to return 4xx instead of 5xx on malformed input.
> - **Regression protection:** `applyCoupon throws when cart is still loading (GH#1234)` guards the entry-point fix. CODE-REVIEW flagged missing coverage for the legacy retry path — add `applyCoupon dedups repeat attempts within 2s`.

**Bad prevention entry (rewrite):**
> - "Fix it at the database."
> - "Add more tests."

Prevention that cannot be acted on is not prevention.

## Diagram (Deep required, Standard optional, Lightweight never)

**What to depict:**
The **durable mechanism**, not the file layout. Good diagrams capture:
- Sequence diagrams: the order of events and the invariant each step preserves.
- State diagrams: the state machine and the guards on each transition.
- Flowcharts: the decision tree with the condition on each branch.

**Bad diagrams:**
- Module-box diagrams showing `src/foo.ts` → `src/bar.ts` (ties to file layout — dies on refactor).
- ER diagrams copied verbatim from SPEC (add no synthesis value — reference the SPEC's diagram instead).
- Anything that would be clearer as three bullet points.

**Rule of thumb:** if you can delete the diagram and lose nothing, don't include it. If the diagram is doing work a paragraph can't, include it.

**Pulled from:**
- `SPEC.md` diagram (refine, not copy — SPEC's diagram shows the design; SOLUTIONS's shows what held)
- `DIAGNOSIS.md` diagram (often the sequence of the bug — invert it to show the invariant being protected post-fix)
- `EXECUTION.md` Integration Gate structure (rarely useful as a diagram — usually better as prose)

## References

**Primary source:** the folder listing itself. List every artifact read, with its date and status from frontmatter. Use relative links:

```
- [PRD.md](./PRD.md) — 2026-04-14, approved
- [SPEC.md](./SPEC.md) — 2026-04-15, approved
- [EXECUTION.md](./EXECUTION.md) — 2026-04-16, approved
- [CODE-REVIEW.md](./CODE-REVIEW.md) — 2026-04-17, approved
```

**Rule:** only list artifacts actually read. If `REVISION.md` is absent, omit the line — do not leave a placeholder. If a pipeline has no PRD (raw-mode `/execute`), list what's there.

## Relationship to Original Spec (mixed pipeline only)

**Primary sources:**
- `SPEC.md` → Test Strategy and Risks sections (was this edge case on the radar?)
- `DIAGNOSIS.md` → Relationship to Original Spec section (already captures the link — refine, don't copy)
- `EXECUTION.md` Acceptance Criteria Trace (which criteria did the bug violate?)

**Synthesis pattern:**
One paragraph naming (a) what the original SPEC covered, (b) what the bug revealed as missing, and (c) whether this is a gap in planning (SPEC under-specified), a regression (later change broke an invariant), or an edge case under the Test Strategy coverage bar.

Reference the `## Post-Release Bug Fix <DATE>` section that `/diagnosis` or `/execute` already appended to the SPEC — SOLUTIONS.md does not duplicate that; it synthesises the learning.

## Universal discipline

Across every section:

- **Verify.** Every claim in SOLUTIONS must be traceable to a quote, table row, or section in the source artifacts. If you can't cite it, don't write it.
- **Paraphrase for durability.** Source artifacts reference file:line freely; SOLUTIONS rewrites them as invariants.
- **Omit, don't pad.** A missing section (because a sub-artifact was absent, or because there's nothing to say) is better than a padded one. Say "intentionally empty" with a reason when honest.
- **One pass.** Draft every section before re-opening any artifact. Second-guessing from scratch wastes context — re-open only to verify a specific claim.
