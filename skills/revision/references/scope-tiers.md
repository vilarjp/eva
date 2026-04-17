# Scope tiers — detailed rubric (REVISION)

Match ceremony to the real risk of the PRD/SPEC pair being reviewed. Revision inherits complexity from the source documents first; it never invents its own tier out of thin air.

The revision tier is the **MAX** of the two source documents' declared complexity:

| PRD tier | SPEC tier | Revision tier |
|----------|-----------|---------------|
| LOW | LOW | Lightweight |
| LOW | MEDIUM | Standard |
| MEDIUM | LOW | Standard |
| MEDIUM | MEDIUM | Standard |
| LOW | HIGH | Deep |
| HIGH | LOW | Deep |
| MEDIUM | HIGH | Deep |
| HIGH | MEDIUM | Deep |
| HIGH | HIGH | Deep |

If either doc lacks a `complexity:` frontmatter field, fall back to these heuristics:

- **Lightweight fallback:** both docs under ~200 lines each, fewer than 4 sections each, no mini-ADRs, no data-model section, no API-contracts section.
- **Standard fallback:** at least one doc 200–500 lines, 1–3 mini-ADRs, ≤ 4 tracer-bullet phases, no public-API change.
- **Deep fallback:** either doc over 500 lines, ≥ 4 mini-ADRs, schema migration, public-API breaking change, or a new architectural pattern.

## Lightweight

**Use when:**
- Both PRD and SPEC are LOW complexity
- Obvious 1:1 mapping between PRD goals and SPEC sections
- No migration, no public-API surface change
- Either doc is under ~200 lines

**Process adjustments:**
- Phase 1 (clarifying Qs): 0 Qs. Auto-detected context should be sufficient. Only ask if Phase 0 detection genuinely fails.
- Phase 2: read both docs in full anyway. The documents are small; there is no excuse.
- Phase 3 (4 lenses): run all four, but keep the pass compact. 1–4 findings is typical.
  - Cross-doc: quick goal↔section map, quick non-goal check.
  - Coherence: spot-check terminology; one pass for contradictions.
  - Adversarial: run it but accept a clean pass ("no material concern") without apology.
  - Scope-feasibility: confirm no hidden expansion and that tracer-bullet phases stay durable.
- Phase 4 (classify): most findings will be P2 or P3.
- Phase 5 (self-review): full checklist; can pass quickly if the pair is clean.

**Exit signal:** a compact REVISION.md with 0–4 findings, emitted in under 5 user turns.

## Standard

**Use when:**
- At least one doc is MEDIUM complexity
- Normal feature or bounded design with real decisions to cross-check
- 3–10 files likely affected by the underlying work
- At least one of: multiple mini-ADRs, data-model change, API surface addition

**Process adjustments:**
- Phase 1: ≤3 Qs, one at a time. Usually 0 — use the budget only on genuine ambiguity.
- Phase 2: read both docs in full. Summarize structure in Grounding summary (3–8 bullets).
- Phase 3: run all four lenses at full depth. 3–10 findings is typical.
  - Cross-doc: formal goal-coverage matrix. Trace every PRD goal to SPEC sections.
  - Coherence: check terminology drift across the pair, broken refs, orphan mini-ADRs.
  - Adversarial: run all five techniques (premise, unstated, decision stress, simplification, alternative). Integrate material critiques into findings.
  - Scope-feasibility: shadow-path tracing for any mini-ADR that touches error paths.
- Phase 4: P0/P1/P2/P3 distribution should span at least two tiers. Suppressed findings table is usually non-empty.
- Phase 5 (self-review): full checklist. Re-emit if any `✗`.

**Exit signal:** a focused REVISION.md with 3–10 findings, emitted in 6–12 user turns.

## Deep

**Use when:**
- At least one doc is HIGH complexity
- Cross-cutting, strategic, or irreversible work (schema migration, public API, new architectural pattern)
- 10+ files affected, multiple mini-ADRs, ≥ 3 tracer-bullet phases
- The decision constrains multiple quarters of future work

**Process adjustments:**
- Phase 1: ≤3 Qs. Expect to use the budget on genuine ambiguity — Deep revisions often pair docs from multiple authors or across sessions.
- Phase 2: read both docs in full. Grounding summary is 5–8 bullets, not 3.
- Phase 3: run all four lenses at full depth with explicit integration.
  - Cross-doc: goal-coverage matrix is mandatory. Every mini-ADR must trace to a PRD goal or an explicit SPEC-only concern. Complexity-tier consistency check runs explicitly.
  - Coherence: full terminology pass, all internal refs checked, orphan-ADR scan.
  - Adversarial: every technique runs. `P0`/`P1` adversarial findings must be addressed or logged as `now` Open Questions.
  - Scope-feasibility: migration shape validated explicitly; data-model invariants checked; API-contract backwards-compat checked; risks table checked against the mini-ADRs.
- Phase 4: expect 5–20 findings. Every `P0` and `P1` must have a proposed fix, not just a description of the gap.
- Phase 5 (self-review): strict. Every `✗` must be resolved before Gate 1.
- Phase 6 (Gate 1): present the full severity distribution. Highlight any suppressed findings in the summary line.
- Phase 8 (Gate 2): for Deep, strongly prefer the full-patch path (option A). A partial patch on Deep revision leaves the pair in an inconsistent state.

**Exit signal:** a thorough REVISION.md with 5–20 findings, emitted in 12–20 user turns. The pair should be fit to gate a substantial PR or migration.

## Re-classification mid-process

If during Phase 2 or Phase 3 the scope reveals itself as larger (or smaller) than the inherited tier suggests, announce the re-classification explicitly:

```
revision: scope reclassified STANDARD → DEEP (reason: SPEC introduces a schema migration that PRD's LOW tier understates)
```

Then continue with the ceremony appropriate to the new tier. Re-emit the scope line so the user tracks it.

## When the declared tier disagrees with the docs' content

If PRD frontmatter says LOW but the PRD body describes a cross-cutting strategic change, the revision should:

1. Treat this as a **finding** (coherence lens): "PRD complexity frontmatter (`LOW`) contradicts PRD body content (cross-cutting)."
2. Run the revision at the HIGHER tier implied by the body.
3. Surface the mismatch as a fixable finding in REVISION.md so the PRD's frontmatter can be corrected at patch time.

This is the *only* situation where revision edits complexity frontmatter — and it only happens via the normal Gate 2 approval, never silently.

## Interaction with single-doc mode

Single-doc mode (only PRD.md or only SPEC.md present) always runs at the tier of the single document, regardless of what the missing sibling might imply. Cross-doc alignment is skipped, so the rubric is naturally lighter.

| Single doc tier | Revision tier |
|-----------------|---------------|
| LOW | Lightweight |
| MEDIUM | Standard |
| HIGH | Deep |

## When in doubt

- Between Lightweight and Standard → **Standard**. The overhead is small. Under-revising a real feature is worse than over-revising a simple one.
- Between Standard and Deep → ask one disambiguating question about blast radius in Phase 1.
- Never classify as Lightweight just to avoid the work. The rubric must match the pair's real risk.
