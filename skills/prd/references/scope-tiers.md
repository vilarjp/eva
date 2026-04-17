# Scope tiers — detailed rubric

Match ceremony to the real size and ambiguity of the work. A 20-minute PRD for a 5-minute change is waste; a 5-minute PRD for a cross-cutting rewrite is negligent.

## Lightweight

**Use when:**
- The change is small and well-bounded (likely 1-3 files)
- The pattern already exists in the codebase — you are reusing, not inventing
- Requirements are already clear from the user's description; there is little ambiguity
- The work is reversible and low-risk

**Process adjustments:**
- Phase 1 (clarifying questions): 0-2 Qs. Skip entirely if the description is unambiguous and success criteria are obvious.
- Phase 2 (codebase scan): single Grep for the topic; confirm no duplicate exists.
- Phase 4 (options): one recommended approach with a brief "why not X, Y" paragraph is enough; 2-3 full options only when real alternatives exist.
- Phase 7 (visual): skip unless the user explicitly asks.
- Sections in PRD.md: keep everything compact. User Stories can be 1-3 items. Not-Doing still required.

**Exit signal:** a 1-page PRD in under 5 user turns.

## Standard

**Use when:**
- The change is a normal feature or bounded refactor with real decisions to make
- Likely 3-10 files affected
- There are multiple reasonable implementations to choose from
- Some cross-module coordination but no deep architectural change

**Process adjustments:**
- Phase 1: 2-5 Qs, one at a time.
- Phase 2: read `AGENTS.md` / `CLAUDE.md` / `README.md` for constraints, grep for topic, skim prior art.
- Phase 4: 2-3 genuinely distinct options with full pros/cons.
- Phase 7: offer a diagram only if topic is spatial/systemic.
- Full Phase 8 self-review checklist.

**Exit signal:** a focused PRD with real option analysis, typically in 6-12 user turns.

## Deep

**Use when:**
- Cross-cutting, strategic, or highly ambiguous work
- Likely 10+ files or a new architectural pattern
- Trade-offs are significant and irreversible (vendor lock-in, schema migrations, public API surface)
- The decision constrains future directions

**Process adjustments:**
- Phase 1: lean into the full 5 clarifying Qs.
- Phase 2: map module boundaries, read multiple prior-art PRDs, document interfaces likely to change.
- Phase 4: mandatory 3 options + council-style debate (staff engineer / PM / skeptic one-line critiques per option).
- Phase 7: diagram is **mandatory**, not optional. Do not ask — just include one in the PRD.
- Phase 8 self-review is strict: every `✗` must be resolved before Phase 9.
- Consider flagging in the PRD that a follow-up architecture review may be warranted.

**Exit signal:** a thorough PRD that could credibly be the input to a planning phase or RFC, typically 12-20 user turns.

## When in doubt

- If you are unsure between Lightweight and Standard → Standard. The overhead is small.
- If you are unsure between Standard and Deep → ask one disambiguating question about scope before committing.
- Never classify as Lightweight to avoid doing the work. If the scope is genuinely Lightweight, the output will naturally be short.

## Re-classification mid-process

If during Phase 1 or Phase 2 the scope reveals itself to be larger than initially classified, announce the re-classification explicitly:

```
prd: scope reclassified LIGHTWEIGHT → STANDARD (reason: <one sentence>)
```

Then continue with the ceremony appropriate to the new tier.
