# Review checklist — full rubric for the 4 lenses

Load this file when any lens feels under-specified or when the Phase 5 self-review flags a lens as weak. It contains the detailed check list for each lens, the severity rubric with concrete thresholds, the adversarial stance, and good/bad examples of findings.

Table of contents:

1. [Lens 1 — Cross-doc alignment](#lens-1--cross-doc-alignment)
2. [Lens 2 — Internal coherence](#lens-2--internal-coherence)
3. [Lens 3 — Adversarial premise challenge](#lens-3--adversarial-premise-challenge)
4. [Lens 4 — Scope creep + feasibility](#lens-4--scope-creep--feasibility)
5. [Severity rubric](#severity-rubric)
6. [Adversarial stance](#adversarial-stance)
7. [How to phrase a finding](#how-to-phrase-a-finding)
8. [Good / bad finding examples](#good--bad-finding-examples)

---

## Lens 1 — Cross-doc alignment

Skipped in single-doc mode. Otherwise mandatory.

### Checks

- **Goal coverage matrix.** For each PRD goal, list ≥1 SPEC section that satisfies it (tracer-bullet phase, mini-ADR, API contract, data model). Goals with zero SPEC coverage → `P0`.
- **Commitment trace.** For each major SPEC commitment (mini-ADR, tracer-bullet phase, new API), name the PRD goal, constraint, or trade-off that justifies it. Uncited commitments → `P1` (scope creep).
- **Non-goal respect.** Parse PRD Non-Goals. For each, check SPEC has NOT absorbed it. SPEC violating a PRD non-goal → `P0`.
- **Assumption preservation.** Each PRD assumption should appear in SPEC (same wording, or explicitly superseded with rationale). A PRD assumption silently dropped → `P1`.
- **Constraint preservation.** Same rule as assumptions. PRD constraints (timeline, tech stack, backwards-compat, regulatory) must be honored in SPEC.
- **User-story mapping.** Each PRD user story should be satisfiable at a specific tracer-bullet phase. A user story with no phase that covers it → `P0`.
- **New SPEC assumption sanity.** Any assumption introduced in SPEC that contradicts PRD is a `P0`.
- **Complexity-tier consistency.** PRD.LOW paired with SPEC.HIGH (or vice versa) requires an explicit rationale in SPEC. Uncorroborated tier mismatch → `P1` (coherence-adjacent).
- **Open Questions reconciliation.** A `now`-tagged PRD Open Question must be resolved in SPEC (or re-surfaced). Dropped Open Questions → `P1`.

### What a cross-doc finding looks like

- Evidence must cite BOTH documents (or explicitly say "SPEC is silent on X" with the PRD quote).
- Proposed fix is almost always a small SPEC addition, a small PRD edit, or both.
- Target is typically `SPEC` (when SPEC is missing coverage) or `PRD+SPEC` (when both need to align on terminology).

---

## Lens 2 — Internal coherence

Always runs. Catches contradictions, drift, ambiguity, and orphans.

### Checks

- **Intra-doc contradictions.** The PRD claims X in one section and ¬X in another. Same for SPEC.
- **Cross-doc contradictions.** PRD says "one-tap flow"; SPEC describes a two-request handshake. Numbers, counts, names, and deadlines are the most common culprits.
- **Terminology drift.** PRD says "customer"; SPEC says "user". PRD says "session token"; SPEC says "auth credential". Same concept, different word — document which one wins.
- **Broken internal references.** PRD mentions "Option B" but the Options section only has A and C. SPEC mentions "D-5" but only D-1..D-4 exist.
- **Ambiguous wording.** "The system should handle high load." (What's high? What's handle?) Two reasonable readers would implement differently.
- **Empty sections / `[TBD]`.** Any section that should have content but has placeholders or is blank.
- **Orphaned mini-ADRs.** A decision that no other SPEC section references. Either it's non-load-bearing (delete or demote) or other sections are missing the reference.
- **Numbered-list drift.** PRD Goals are G-1..G-3 but SPEC references G-4. Or PRD renumbers Goals between drafts and SPEC still points at old IDs.
- **Section-order drift from template.** If the docs were written by `prd` or `spec`, section ordering should match the canonical template. Material deviations → coherence finding unless explicitly rationalized.

### What a coherence finding looks like

- Evidence is usually two quotes from the same document (intra-doc contradiction) or one quote from each (cross-doc).
- Proposed fix is a one-line edit: rename, correct count, resolve ambiguity, delete stray reference.

---

## Lens 3 — Adversarial premise challenge

**Note:** the main skill dispatches this lens to the `revision-adversarial-review` agent (`agents/revision-adversarial-review.md`) for a cold-context read. This section stays here as (a) the agent's calibration reference for severity and stance, and (b) the inline fallback rubric when the agent fails to dispatch.

Always runs at Standard and Deep. At Lightweight, runs but may legitimately produce zero findings.

### Techniques

Apply each technique in turn. For each, either produce a finding or explicitly state "no material concern."

1. **Premise challenge.** Is the problem as stated in the PRD actually true?
   - *Good challenge:* "PRD Problem Statement cites '38% mobile abandonment' but no source is given. Evidence?"
   - *Bad challenge:* "The problem might not be real." (vague; unactionable)

2. **Unstated assumption.** What does the SPEC silently assume that isn't named?
   - *Good:* "SPEC's `POST /checkout/tap` assumes the service is single-tenant; PRD implies a multi-tenant deployment in its Non-Goals list."
   - *Bad:* "SPEC makes assumptions." (self-evident; no information)

3. **Decision stress.** For each mini-ADR, what happens under load / partition / rollback / concurrent access?
   - *Good:* "D-2 (server-side transaction) holds a DB connection while calling Stripe. At 500 rps, pool starvation likely; risks table does not cover this."
   - *Bad:* "D-2 could be slow." (no mechanism)

4. **Simplification pressure.** Does every abstraction earn its keep? Could the same acceptance criteria be met with one fewer module?
   - *Good:* "SPEC introduces a new `TapOrchestrator` module, but the two calls it coordinates already live in `CheckoutController`. The new module earns nothing for this specific scope."
   - *Bad:* "This is over-engineered." (generic; no lever)

5. **Alternative blindness.** Is there a credible option neither doc considered?
   - *Good:* "Neither doc considers a background-job reconciliation approach — the 3-tap flow could stay client-side with an async correction pass. Worth naming even if rejected."
   - *Bad:* "There might be other options." (no candidate)

### Stance

From Karpathy's framing + vilarjp's revision-adversarial-review agent:

> *You are not hostile. You are rigorous. A plan that survives your scrutiny is a plan worth executing. Every challenge must be specific. "This might not scale" is useless. "This loads all records into memory — at 10K records the response time exceeds 5s" is actionable.*

### Iron rules for adversarial findings

- Ground every challenge in actual document text. A premise challenge without a PRD quote to object to is speculation.
- If you cannot make the challenge specific, drop it.
- Recommend a direction for the fix, even if tentative.
- Do not multiply findings to look thorough. Three sharp adversarial findings beat ten generic ones.

---

## Lens 4 — Scope creep + feasibility

Always runs.

### Scope-creep checks

- **SPEC vs PRD recommended direction.** Is the SPEC describing the option the PRD chose, or has it drifted toward a richer design?
- **Abstraction earning its keep.** Every new module or layer should trace to a PRD goal, a PRD constraint, or an explicit SPEC-only technical concern. Nothing else.
- **Tracer-bullet phase proliferation.** If the SPEC has more phases than the PRD has goals, investigate. Often a sign of decomposing for thoroughness rather than vertical slicing.
- **New open questions introduced.** Each SPEC `now` Open Question that has no corresponding PRD Open Question is a possible drift.

### Feasibility checks

- **Shadow paths.** For each mini-ADR and each API contract, trace the happy / nil / empty / error path. Does the SPEC handle all four? Does the risks table cover the failure modes?
- **Migration shape.** For any schema or contract change, is the migration shape named? Additive, destructive-with-backfill, or breaking-with-compat-window?
- **Rollback story.** Every mini-ADR should name the rollback cost. If rollback is missing, add it as a finding.
- **Data-model invariants.** Uniqueness, foreign keys, cascading rules, nullability. A data-model section without invariants is under-specified (usually `P1` or `P2`).
- **API-contract rigor.** Every new/modified endpoint specifies inputs, outputs, errors, idempotency, and backwards-compat. Gaps → `P1`.
- **Risks-table realism.** "Bugs" is not a risk. "Unexpected issues" is not a risk. Every row must name a mechanism.
- **Durability of phases.** Any tracer-bullet phase that names a file path or function name → `P2` (durability violation).

### What a scope-feasibility finding looks like

- Evidence is usually a SPEC quote; occasionally PRD + SPEC together for scope-creep.
- Proposed fix is a small SPEC edit (add invariant, name a failure mode, trim an abstraction). It should NEVER be "redesign the architecture" — that's too big for a revision.

---

## Severity rubric

Calibrate severity against the specific thresholds below. If your severity distribution is all-P0 or all-P3, recheck.

### P0 — Blocker

At least one of:
- **Contradicts the user's explicit ask.** The PRD/SPEC violates a stated requirement from the conversation that produced them.
- **PRD goal has zero SPEC coverage.** A goal the PRD named explicitly isn't addressed in any SPEC section.
- **SPEC violates a PRD non-goal.** The SPEC absorbs scope the PRD excluded.
- **Security or data-integrity gap at plan level.** PII exposure, missing authN/authZ boundary, idempotency gap that could double-charge, data loss risk in migration.
- **Acceptance criterion is untestable.** A criterion phrased as "better" / "faster" / "easier" without a measurable bound.
- **Acceptance criterion references an implementation detail.** e.g., "function X returns Y" — durability violation.

### P1 — Major

At least one of:
- **Key assumption unverified and blocks direction.** A `verified: NO` assumption that the SPEC's direction actually depends on.
- **Mini-ADR missing alternatives.** A decision presented without at least one genuine rejected alternative. Rationalization masquerading as deliberation.
- **Scope creep with real cost.** A SPEC commitment that adds implementation burden without matching a PRD goal.
- **Risks table misses a real failure mode.** The pair describes a scenario whose common failure mode is absent from risks.
- **Terminology drift that could cause misimplementation.** Same concept named two different things across the pair, in a way that would produce divergent code.
- **Cross-doc assumption drop.** A PRD assumption silently dropped in SPEC.

### P2 — Minor

At least one of:
- **Ambiguous wording.** Two reasonable readers would implement differently, but neither would ship something broken.
- **Broken internal reference.** Pointer to a section or ID that doesn't exist.
- **Section drift from template.** Canonical `prd`/`spec` template ordering violated without rationale.
- **Missing prior-art reference.** Test-strategy section names no existing test file to pattern against; mini-ADR names no prior convention.
- **Tracer-bullet phase names a file.** Durability violation, but the phase's behavior is still correct.

### P3 — Nit

At least one of:
- Typos, capitalization, grammar.
- Inconsistent casing on proper nouns (`GitHub` vs `Github`).
- Missing optional fields (e.g., `author:` absent from frontmatter).
- Oxford-comma drift in a list-heavy section.

P3 findings should never be more than ~30% of the finding set. If they are, you are padding; re-check whether the real issues are deeper.

---

## Adversarial stance

Repeat from Lens 3, verbatim, because it governs multiple lenses:

> *You are not hostile. You are rigorous. Your goal is to surface weaknesses in the pair so they can be addressed before implementation begins. A pair that survives your scrutiny is a pair worth building from.*

### Iron rules

1. Every challenge must be specific. Generic pessimism is noise.
2. Ground in the actual text. No paraphrases pretending to be quotes.
3. Recommend direction. If you cannot propose a fix, the finding probably isn't actionable.
4. Do not multiply findings. Three sharp findings beat ten generic ones.
5. A clean pass is valid. Forced findings corrupt the review.

---

## How to phrase a finding

Every finding has the same shape:

- **Title** — one line, descriptive. No severity prefix (severity is metadata).
- **Evidence** — at least one quote from PRD or SPEC, 5–20 words, with section reference.
- **Why it matters** — one sentence. Names the risk or cost if unfixed.
- **Proposed fix** — one or two sentences. Minimal. No new scope.

Keep the prose in the voice of the source docs. If the PRD uses "customer", the finding uses "customer" too. Consistency reduces cognitive load for the person reading REVISION.md alongside the originals.

---

## Good / bad finding examples

### Good — Cross-doc alignment (P0)

> **F-3 — PRD goal G-2 has no SPEC coverage**
>
> - Lens: cross-doc
> - Type: omission
> - Target: SPEC
> - Evidence:
>   - `PRD §Goals G-2`: > "Maintain ≤800ms p95 checkout completion latency."
>   - `SPEC`: no latency target appears in §Goals, §Risks, §Test Strategy, or any tracer-bullet phase.
> - Why it matters: The PRD's latency commitment has no corresponding SPEC-level mechanism or test, so the implementation has no way to know when it has met the goal.
> - Proposed fix: Add a latency goal to SPEC §Goals and a load-test entry to §Test Strategy targeting 800ms p95 under 500 rps.

### Good — Coherence (P2)

> **F-7 — Terminology drift: "customer" vs "user"**
>
> - Lens: coherence
> - Type: error
> - Target: PRD+SPEC
> - Evidence:
>   - `PRD §User Stories`: > "As a repeat mobile customer…"
>   - `SPEC §API Contracts`: > "Input: `{ userId: UUID, cardId: UUID }`"
> - Why it matters: The same actor is called "customer" in PRD and "user" in SPEC. In implementation this becomes two fields or two confusingly-named types. Fix once, everywhere.
> - Proposed fix: Pick one term (prefer "customer" to match the PRD's reader framing). Replace `userId` with `customerId` in SPEC §API Contracts, §Data Model, and §Tracer-bullet Phases.

### Good — Adversarial (P1)

> **F-12 — D-2 ignores DB pool starvation under load**
>
> - Lens: adversarial
> - Type: omission
> - Target: SPEC
> - Evidence:
>   - `SPEC §D-2`: > "Expose POST /checkout/tap that performs session lookup, payment authorization, and order creation within a single server-side transaction."
>   - `SPEC §Risks`: no entry mentions DB-pool behavior.
> - Why it matters: Holding a DB connection while calling Stripe (network round-trip, 50–300ms) at 500 rps will exhaust the pool. The decision is silent on this. Under real load the system will return 503 before it returns 422s.
> - Proposed fix: Add R-3 to SPEC §Risks (mechanism + mitigation: release DB connection before Stripe call, reacquire for insert; load-test at 2× projected rps). Update D-2 §Consequences to name this trade-off.

### Bad — too vague

> **F-X — SPEC might not scale**
>
> *(No evidence, no mechanism, no proposed fix. Drop or sharpen.)*

### Bad — introduces new scope

> **F-X — Add Apple Pay support**
>
> *(Revision does not add features. If Apple Pay is missing and warranted, file a new PRD.)*

### Bad — duplicates another finding

> **F-X — D-2 doesn't explain rollback cost**
> **F-Y — D-2 missing rollback analysis**
>
> *(Merge into one finding.)*
