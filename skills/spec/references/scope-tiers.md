# Scope tiers — detailed rubric (SPEC)

Match ceremony to the real size, risk, and blast radius of the technical work. A 30-minute SPEC for a one-file change is waste; a 5-minute SPEC for a cross-cutting schema migration is negligence.

If a PRD is in context, the PRD's complexity tier is a strong prior, not a mandate. A LOW-complexity PRD can still spawn a Standard SPEC when the *engineering* decisions (data model, API shape, integration) are non-obvious.

## Lightweight

**Use when:**
- The change is small and well-bounded (1-3 files, one module)
- The pattern already exists in the codebase — the SPEC is primarily *describing* an existing shape for a new use
- No schema change, no new public interface, no cross-module coordination
- Reversible; failure mode is contained

**Process adjustments:**
- Phase 1 (clarifying Qs): 0-2 Qs. Skip entirely if description + PRD already answer WHAT / WHY / HOW-AT-HIGH-LEVEL.
- Phase 2 (pre-scan): single Grep + read the one or two files to be touched.
- Phase 5 (mini-ADRs): often just 1. Two at most. If you can't find a real decision, write zero — the SPEC can still be valuable as a shape-document.
- Phase 6 (data model / API / boundaries): concise. Skip the Data Model section entirely if the change doesn't touch data. Skip API Contracts if no interface changes.
- Phase 7 (phases): often 1 phase is enough. Two max.
- Phase 8 (diagram): skip unless the user asks.
- Phase 9 (red-team): run it but expect short output — it's fine to end a role with "no material concern."

**Exit signal:** a 1-2 page SPEC in under 6 user turns.

## Standard

**Use when:**
- Normal feature or bounded design with real decisions to make
- 3-10 files affected
- At least one of {API shape, data model, integration point, error semantics, concurrency model} requires deliberation
- Some cross-module coordination; no deep architectural change

**Process adjustments:**
- Phase 1: 2-5 Qs, one at a time. When PRD is in context, spend the budget on SCALE / CONSISTENCY / INTEGRATION / COMPATIBILITY / OPERABILITY.
- Phase 2: full constraint check + topic scan + interface map for touched modules.
- Phase 5: 2-4 mini-ADRs — every real decision, no padding.
- Phase 6: complete all three sub-sections (Data Model, API Contracts, Module Boundaries).
- Phase 7: 2-4 tracer-bullet phases. Each must deliver independently verifiable behavior.
- Phase 8: offer a diagram once when structure is spatial or flow is non-linear.
- Phase 9: full red-team, one line per role, integrate material concerns.
- Full Phase 10 self-review checklist.

**Exit signal:** a focused SPEC with real decision analysis, typically 8-15 user turns.

## Deep

**Use when:**
- Cross-cutting, strategic, or irreversible
- 10+ files affected, new architectural pattern introduced, or external-API surface changes
- Trade-offs are significant and hard to reverse (schema migrations, vendor lock-in, public API, backwards-compat breaks)
- The decision constrains future directions for multiple quarters

**Process adjustments:**
- Phase 1: lean into the full 5 Qs even with a PRD in context — the engineering surface is wide.
- Phase 2: map module boundaries exhaustively, read prior-art SPECs in `docs/`, locate relevant ADRs. Document interfaces likely to change.
- Phase 5: 3+ mini-ADRs. Name at least one decision that could have gone the other way with a credible argument.
- Phase 6: full coverage. Data model includes migration shape and rollout sequence. API contracts include idempotency + backwards-compat for every endpoint.
- Phase 7: 3-5 tracer-bullet phases. Include an explicit "foundation" or "scaffold" phase if infrastructure is needed.
- Phase 8: diagram is **mandatory**. Prefer a component diagram for cross-module work, sequence for multi-actor flows, or state for lifecycle changes. Do not ask — include one.
- Phase 9: full red-team, and any `HIGH`-impact critique must either be addressed in-SPEC or logged as a `now` Open Question.
- Phase 10 self-review: every `✗` must be resolved before Phase 11. No exceptions.
- Consider flagging in the SPEC that a follow-up architecture review (or ADR split into separate files) may be warranted.

**Exit signal:** a thorough SPEC that could credibly gate a PR for a significant migration or new subsystem, typically 15-25 user turns.

## When in doubt

- Between Lightweight and Standard → **Standard**. The overhead is small. Downgrading later is free; upgrading in the middle loses momentum.
- Between Standard and Deep → ask ONE disambiguating question about blast radius (e.g. "Will this change the public API shape or require a DB migration?") before committing.
- Never classify as Lightweight just to avoid the work. If the scope is genuinely Lightweight, the output will naturally be short.

## Re-classification mid-process

If during Phase 1 or Phase 2 the scope reveals itself as larger (or smaller) than initially classified, announce the re-classification explicitly:

```
spec: scope reclassified LIGHTWEIGHT → STANDARD (reason: touching a public interface requires decision on backwards-compat)
```

Then continue with the ceremony appropriate to the new tier. Re-emit the scope line so the user tracks it.

## Interaction with the PRD's complexity tier

| PRD tier | Default SPEC tier | When to deviate |
|----------|-------------------|------------------|
| LOW | Lightweight | Promote to Standard if the PRD hand-waves over a real API/data/integration decision |
| MEDIUM | Standard | Promote to Deep if engineering path introduces migration, vendor lock-in, or new pattern |
| HIGH | Deep | Never downgrade |

Announce any deviation: `spec: PRD=LOW, but scope=STANDARD (reason: …)`.
