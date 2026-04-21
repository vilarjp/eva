# Scope tiers — detailed rubric (EXECUTE)

Scope tunes slice count, question budget, ceremony depth, and how strict the integration gate is. It does **not** change the Iron Law — every tier still obeys test-first for behavior-bearing changes.

Scope is about **implementation effort**, not the severity of what's being shipped. A TRIVIAL diagnosis severity can map to Deep execution if the fix spans many components; a COMPLEX SPEC can map to Lightweight execution if the phase being implemented is narrow.

Pick the tier that matches **the work this session actually ships**, not the artifact as a whole. An execute call against a Deep SPEC is usually scoped to one or two SPEC phases at a time — match the tier to those phases.

## Decision flow

```
Single file, pattern already exists, no data-model change, no concurrency?     → Lightweight
3-10 files, clear SPEC phases or clear DIAGNOSIS root cause, no architecture?  → Standard
10+ files, cross-cutting, concurrency/migration/integration, or SPEC=HIGH?     → Deep
```

## Lightweight

**Signals:**
- 1-2 files likely to change.
- Pattern already exists in the codebase — implementation is primarily an application of an existing shape.
- No new public interface, no data-model change, no cross-module coordination.
- Failure mode is contained and reversible.
- Examples: typo fix, rename, a single added validation rule, missing import, one-liner bug fix, a config flag flip that doesn't change runtime behavior.

**Ceremony:**
- Phase 2 (Understand): 0-1 clarifying questions. Often zero.
- Phase 3 (Plan slices): 1 or 2 slices.
- Phase 4 (Self-review checklist): emit it — items that don't apply get `✓ (N/A)` with a one-word reason.
- Phase 5 (HARD GATE 1): a compact gate is fine; the slice plan might fit in 3 lines.
- Phase 6 (Execute): RED proof still mandatory when the change is behavior-bearing. Verification Mode is acceptable when the change truly isn't behavior-bearing.
- Phase 7 (Integration gate): full suite + lint + types still required; the gate is non-negotiable.
- Phase 8 (EXECUTION.md): complete, but short. The NOTICED-BUT-NOT-TOUCHING section can be `None.`

**Pitfalls:**
- "It looks trivial" ≠ "it is trivial." If the pre-scan surfaces something surprising, promote to Standard.
- Do not skip the reproduction test for a one-liner bug fix. Every regression is a good regression test.

**Exit signal:** an EXECUTION.md under ~115 filled lines. Findings triage
sections cut unless FIX/REFACTOR mode; per-slice details inline under one
shared heading; micro-spec cut unless RAW mode. Follow the trimmed
`templates/EXECUTION.md` verbatim — see `_shared/artifact-compactness.md`.

## Standard

**Signals:**
- 3-10 files.
- Implementation follows a clear SPEC with tracer-bullet phases, OR a clear DIAGNOSIS with a named root cause and suggested fix.
- Some cross-module coordination. No architectural change, no new concurrency model, no migration that requires staged rollout.
- Examples: a new endpoint and its tests, adding a feature flag to an existing flow, wiring a new event into an existing dispatcher, a moderate refactor to make room for tomorrow's feature (guarded by its own tests).

**Ceremony:**
- Phase 2: 1-3 clarifying questions. Usually 0-1 if a SPEC/DIAGNOSIS is in context.
- Phase 3: 2-4 slices. In FEATURE mode, usually one slice per SPEC phase.
- Phase 4: full checklist.
- Phase 5: full HARD GATE 1 with all slices summarized.
- Phase 6: RED-GREEN-REFACTOR per slice. Verification Mode only for genuinely non-behavior-bearing slices.
- Phase 7: full integration gate. Audit skipped tests. In BUG mode, capture regression red-green proof when practical.
- Phase 8: EXECUTION.md with a filled slice table and NOTICED-BUT-NOT-TOUCHING list.

**Pitfalls:**
- "Let me combine two slices to save a commit." Don't. Independent acceptance criteria → independent slices. Bisectability pays off the first time a regression lands.
- A slice that doesn't turn RED is a slice that isn't testing the right thing. Fix the test, not the verdict.

## Deep

**Signals:**
- 10+ files, or the change crosses module / process / runtime boundaries.
- Concurrency, migration, integration with a new external system, rollout requires staging.
- SPEC frontmatter marks complexity as HIGH, or REVISION adjusted the SPEC with at least one P0 finding.
- Examples: splitting a monolithic service, introducing a new data store, a schema migration with backfill, replacing an auth provider, adding distributed tracing end-to-end.

**Ceremony:**
- Phase 2: ≤3 clarifying questions. Confirm SPEC risk mitigations and migration plan before Slice 1.
- Phase 3: 2-4 slices is still the target. If the work genuinely needs more, stop and ask whether this should be split into multiple `/execute` sessions. One session = one coherent set of slices that can all land together.
- Phase 4: full checklist plus an explicit line: "I have read the SPEC's Risks & Mitigations table and have a plan for each risk that applies to this session's slices."
- Phase 5: HARD GATE 1 with a short risk-mitigation mapping per slice.
- Phase 6: full TDD per slice. Verification Mode almost never applies here. If a slice touches a shared boundary (auth, rate-limit, migration), add a defense-in-depth test at a second layer, per the SPEC's guidance.
- Phase 7: integration gate + extra care. Do not accept unexplained `skipped` tests. Regression red-green proof in BUG mode is required, not optional.
- Phase 8: EXECUTION.md with detailed per-slice evidence. Concerns section is mandatory and non-empty — Deep work always has things to watch post-deploy.

**Pitfalls:**
- "We'll do the migration in the same commit as the feature." Split. Migrations are their own slice, with their own test.
- "The SPEC covered this risk." That doesn't mean the code does. The test is the evidence.
- Deep scope ≠ freedom to scope creep. The minimal-diff constraint is strictest here, not loosest.

## Cross-cutting rules (all tiers)

- **Pre-flight is non-negotiable.** Branch check, date, bare-invocation clarify.
- **Pre-scan is internal.** Same budget regardless of tier (~2k tokens).
- **Iron Law holds.** Verification Mode is the only relaxation and requires genuine non-behavior.
- **3-strike Step-Back applies everywhere.** Lightweight slices almost never hit it; if they do, your tier was wrong.
- **No pushes, no PRs.** At any tier.
- **EXECUTION.md is always written.** It's how future readers (including you) know what shipped.

## Promoting or demoting mid-execution

Promotion happens the moment a pre-scan or a slice reveals the work is bigger than you thought:

- *Lightweight → Standard:* the "one-liner" touched three files and a test helper. Announce: `execute: scope promoted to STANDARD — <one-line reason>.` Re-emit the Phase 4 checklist before continuing.
- *Standard → Deep:* a slice revealed a concurrency window you didn't account for, or the SPEC's migration phase is landing in this session after all. Announce the promotion, consider splitting into multiple sessions, and confirm with the user before re-gating.
- *Deep → Standard:* rare. Only demote if the first slice proves the risk you were worried about is handled by an existing layer. Announce and continue.

## Quick reference

| Tier | Slices | Clarifying Qs | Verification Mode slices | Risk ceremony |
|------|--------|--------------|--------------------------|---------------|
| Lightweight | 1-2 | 0-1 | acceptable when non-behavior | minimal |
| Standard | 2-4 | 0-3 | narrow, named reasons | audit skipped tests |
| Deep | 2-4 (split sessions if more) | ≤3 | rare | mandatory risk mapping + regression proof |
