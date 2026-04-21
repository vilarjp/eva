# Scope Tiers — Detailed Rubric for Solutions

A SOLUTIONS.md's ceremony should match the pipeline that produced it. A one-line config change should not produce the same artifact shape as a multi-slice concurrency refactor. Scope is the dial.

Solutions **inherits** its scope from the artifacts already in the folder. Take the MAX of the `scope` values in the frontmatter of PRD / SPEC / REVISION / DIAGNOSIS / EXECUTION / CODE-REVIEW. Never downgrade — if any upstream artifact was Deep, the learnings deserve Deep treatment. Occasionally promote: a pipeline that all five skills classified Lightweight but which produced a multi-strike Step-Back in EXECUTION probably has enough surface area to warrant Standard.

## Decision flow

```
Every upstream artifact classified Lightweight AND EXECUTION shows no Step-Backs    → Lightweight
Any upstream artifact Standard, OR EXECUTION shows Step-Backs, OR multi-slice       → Standard
Any upstream artifact Deep (concurrency / distributed / integration / architectural) → Deep
```

## Tier: Lightweight

**Signals (inherited):**
- PRD / SPEC / DIAGNOSIS / EXECUTION all classified Lightweight in their frontmatter.
- EXECUTION has 1-2 slices with clean RED-GREEN proofs and no Step-Back events.
- CODE-REVIEW (if present) is a clean pass or has only P2 / P3 findings.
- Typical pipelines: a copy tweak, a config flag flip, a rename, a one-line bug fix, a clear type error.

**Ceremony:**
- 0-1 disambiguation questions (usually zero — the folder is obvious).
- Sections populated: Summary · Approach · Key Decisions · Gotchas · References.
- Sections omitted: Root Cause (unless the pipeline is bug-only — then required), What Didn't Work (unless dead ends exist — rare), Prevention (unless bug), Diagram.
- Mini-ADR Alternatives are optional — a Lightweight decision often has one obvious choice and no serious alternative. Say so explicitly: *"No genuine alternative — the `null` check at the entry point was the only candidate."*
- Self-review checklist still emitted. Items that don't apply get `✓ (N/A) — <one-word reason>`.
- HARD GATE still required.

**Pitfalls:**
- "It's so small, just skip the gate." No. The gate is the contract between synthesis quality and the user's trust. A 30-second gate on a 3-minute write is worth it.
- "Lightweight means skip the Iron Law." No. Phrase even a trivial gotcha as behaviour, not `src/foo.ts:42`. The file will be renamed next quarter.
- "No gotchas, no decisions, skip the doc." If the pipeline produced artifacts at all, there are decisions in SPEC mini-ADRs and concerns in EXECUTION. Write it.

**Exit signal:** a SOLUTIONS.md under ~55 filled lines. Diagram and
Relationship-to-Original-Spec sections cut. Gotchas / What-Didn't-Work /
Prevention cut if they'd be empty. Follow the trimmed
`templates/SOLUTIONS.md` verbatim — see `_shared/artifact-compactness.md`.

## Tier: Standard

**Signals (inherited):**
- At least one of SPEC / DIAGNOSIS / EXECUTION classified Standard in its frontmatter.
- EXECUTION has 2-4 slices, possibly including a Step-Back that resolved after a plan adjustment.
- CODE-REVIEW (if present) has P1 or P0 findings that were addressed.
- Typical pipelines: a feature with 3-10 files touched, a bug with a real causal chain, a refactor with a reproduction test and a genuine alternative approach considered.

**Ceremony:**
- 0-2 disambiguation questions, one at a time (usually 0-1).
- Sections populated: Summary · Approach · Key Decisions · Gotchas · What Didn't Work · References · (Root Cause + Prevention if bug).
- Mini-ADR Alternatives required — Standard decisions should have at least one genuine alternative considered and rejected with a reason.
- Diagram optional. Offer once if the synthesis has sequence / state / decision structure.
- Full self-review checklist.
- HARD GATE mandatory.

**Pitfalls:**
- "Standard without a diagram." Fine, but offer it once. Don't skip the offer silently.
- Flat Key Decisions with single-line rationales that read like "we decided X because X is good." Rewrite until the rationale names a specific trade-off being optimised for.
- What Didn't Work listing only the 3-strike Step-Back and nothing else. REJECTED hypotheses from DIAGNOSIS and suppressed findings from CODE-REVIEW are also dead ends — check both.

## Tier: Deep

**Signals (inherited):**
- Any upstream artifact classified Deep in its frontmatter.
- Concurrency, races, distributed systems, cross-process state, schema migrations, authN/authZ, trust boundaries, or architectural refactors in play.
- EXECUTION includes BUG mode regression proof, or FEATURE mode with ≥3 slices touching shared state.
- CODE-REVIEW ran Stage 3 adversarial reviewer.

**Ceremony:**
- 0-2 disambiguation questions. Prior-run awareness is critical ("did this fix a previous bug in the same folder?").
- Sections populated: all that apply. Relationship to Original Spec is usually populated for Deep.
- Mini-ADR Alternatives mandatory. Each key decision names the rejected alternative with evidence or reasoning, not just a label.
- **Diagram: MANDATORY.** Prefer sequence diagrams for cross-component flows, state diagrams for state-machine bugs, flowcharts for decision trees. The diagram depicts the durable mechanism, not the file layout.
- What Didn't Work is usually substantial — concurrency bugs produce multiple rejected hypotheses.
- Full self-review checklist + extra diagram check.
- HARD GATE mandatory.

**Pitfalls:**
- "Deep scope but the Gotchas section is short." Re-read. Concurrency, migrations, and cross-module refactors always leave invariants that matter. If after honest re-reading there truly are none, say so and move on — but verify first.
- "The diagram shows the file structure." No — the diagram shows the mechanism. A sequence diagram should name the invariant being preserved at each step ("cart.status == ready before applyCoupon dispatches"), not the module that holds the logic.
- "Prevention = the reproduction test exists." Not enough. Name the class of bug and where defense-in-depth belongs, even if the current fix is single-point. A future reader asking "how do I avoid this again?" needs the class, not just the test.

## Cross-cutting rules (all tiers)

- **Iron Law of durability always applies.** Learnings are behaviour, not file:line. Lightweight does not exempt you from this — it just means fewer sections.
- **Verify before claiming.** Every Key Decision, Gotcha, What-Didn't-Work entry must be traceable to a specific quote or table row in the source artifacts.
- **HARD GATE at Phase 5.** Regardless of tier, writing SOLUTIONS.md requires explicit user approval.
- **No instruction-file edits at any tier.** Solutions never touches CLAUDE.md / AGENTS.md. The transition text may mention the opportunity; the user owns the edit.
- **Synthetic padding is worse than empty.** An empty section with *"intentionally empty"* and a one-word reason is honest. Padding to hit section quotas is a red flag.

## Promoting / demoting a tier mid-synthesis

Rarely needed — the scope is inherited, and it's right most of the time. Still:

- If you classified Lightweight and Phase 1 reading surfaces a Step-Back or a REJECTED DIAGNOSIS hypothesis, promote to Standard. Announce: `solutions: scope promoted to STANDARD — <reason>`.
- If you classified Deep and the synthesis turns out to be straightforward (the concurrency turned out to be surface-level), stay Deep. The ceremony is already earned and a Deep SOLUTIONS.md never costs much once the reading is done.

## Mixed pipelines (feature folder + post-release bug fix)

A folder with both SPEC.md (approved) AND DIAGNOSIS.md (approved, reusing the folder) is a **mixed pipeline**. Scope is the MAX of the two. The synthesis must cover both:

- **Summary** covers the feature that shipped, then the bug that was fixed.
- **Root Cause + Prevention** are populated for the bug.
- **Key Decisions** include both planning-time decisions that proved their weight (from SPEC mini-ADRs) and the fix-shape decision (from DIAGNOSIS suggested fix).
- **Relationship to Original Spec** paragraph is required — name the gap the original SPEC missed or the regression a later refactor introduced.

## Quick reference

| Tier | Inputs | Sections | Mini-ADR alts | Diagram | Disambiguation Qs |
|------|--------|----------|---------------|---------|------------------|
| Lightweight | All upstream LIGHTWEIGHT, no Step-Backs | Summary · Approach · Key Decisions · Gotchas · References (+ Root Cause / Prevention if bug) | Optional | No | 0-1 |
| Standard | Any upstream STANDARD, or Step-Back, or multi-slice | Above + What Didn't Work | Required | Optional (offer once) | 0-2 |
| Deep | Any upstream DEEP | Above + Relationship to Original Spec (if mixed) | Mandatory | **Mandatory** | 0-2 |

Solutions is a reference document, not a narrative. Length discipline: a Lightweight SOLUTIONS should fit on one screen; a Deep one should fit on two. If you're past that, you've started narrating — cut the prose and keep the invariants.
