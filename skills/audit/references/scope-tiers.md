# Scope Tiers — Detailed Rubric for /audit

An audit's depth is set by the target, not the findings. A single file and a 20-module service both enter through `/audit`, but they deserve different ceremony. The scope tier tunes which phases run, whether the Deep parallel `Explore` fan-out fires, the severity-count ceilings, and how long the skill spends on architecture observations.

**Scope classifies the target, not the debt.** A single 50-line file in a concurrent path can still be Standard. A 3000-line generated-types directory can still be Lightweight. Read the target shape before classifying.

## Decision flow

```
Is the target 1-3 files AND < 500 LOC AND single cohesive module?              → Lightweight
Is the target 4-15 files OR 500-3000 LOC OR one feature / bounded module?       → Standard
Is the target 15+ files OR 3000+ LOC OR cross-module OR "audit the whole X"?   → Deep
```

If the target touches concurrency, auth, payments, migrations, or public APIs — promote one tier regardless of size (same logic as `/code-review`'s risk-tag promotion).

---

## Tier: Lightweight

**Signals:**
- 1-3 files, < 500 LOC total, one cohesive module.
- No cross-module dependencies, no shared-kernel code, no concurrency.
- Examples: a single service file, a small utility module, a React component directory with its test, a single migration script.

**Phases that run:**
- Phase 0 (resume, scope) — required.
- Phase 1 (pre-scan) — single-pass read; no fan-out.
- Phase 2 (smell inventory) — required.
- Phase 3 (architecture observations) — **skip unless a shape emerged**; Lightweight targets rarely produce cross-cutting observations.
- Phase 4 (severity calibration) — required.
- Phase 5 (handoff records) — required.
- Phase 6 / 7 / 8 (checklist, gate, write) — required.

**Ceremony:**
- No parallel `Explore` fan-out.
- 0-1 clarifying questions (usually zero — the target is small enough to read).
- Typical findings count: 2-8 total. P0 is uncommon.
- P1 count ceiling: 5. If higher, re-calibrate.

**Pitfalls:**
- "It's just a utility file" does not mean it is clean. Read the call sites — a utility with wide fanout amplifies any smell.
- Lightweight does not mean *"skip smells"*. It means *"survey one file honestly, write one short artifact"*.

**Exit signal:** an AUDIT.md under ~80 filled lines. Architecture observations
cut unless a cross-cutting shape emerged; Target inventory cut unless the
scope is genuinely ambiguous; Suppressed candidates cut if empty. Follow the
trimmed `templates/AUDIT.md` verbatim — see
`_shared/artifact-compactness.md`.

---

## Tier: Standard

**Signals:**
- 4-15 files, 500-3000 LOC, one feature or bounded module.
- Cohesive unit — a feature, a service boundary, a sub-package.
- May touch one external boundary (a library, a queue, a framework surface), but not many.
- Examples: `src/checkout/`, `src/auth/session/`, a slice of a GraphQL schema + its resolvers, a React feature directory with its hooks + tests.

**Phases that run:**
- Phase 0, 1, 2, 4, 5, 6, 7, 8 — all required.
- Phase 3 (architecture observations) — **required**; Standard targets almost always produce at least one cross-cutting observation.

**Ceremony:**
- Pre-scan: single-pass read (no parallel fan-out unless the user explicitly escalates).
- Phase 1.4 pattern-exemplar sweep is mandatory — find the best-shaped analogue in the target itself to calibrate what "good" means here.
- 1-2 clarifying questions if the target is ambiguous.
- Typical findings count: 8-25 total.
- P1 count ceiling: 10. If higher, re-calibrate (likely P2s mis-tiered).

**Pitfalls:**
- Skipping Phase 3 because *"nothing cross-cutting jumped out"* — look harder. A Standard module with zero architecture observations is under-audited.
- Inflating P2s to P1 to make the audit look thorough. P1 means "incident in a quarter OR blocks a known-likely change" — hold the line.
- Flagging smells the repo deliberately embraces (DAMP tests, flat module structure, explicit factories) — calibrate against Phase 1.4 exemplars.

---

## Tier: Deep

**Signals:**
- 15+ files, OR 3000+ LOC, OR any of:
  - Cross-module (the target spans two or more domain modules).
  - Shared kernel / shared utility surface (consumed by many features).
  - Concurrency, workers, queues, cron.
  - Public API surface (other services or external consumers depend on the shape).
  - Infrastructure layer (database access, framework wiring, auth middleware).
  - "Audit the whole service" / "audit the backend" / "audit everything".
- Or: any target the user explicitly asks to audit as "Deep" / "thorough".

**Phases that run:**
- All phases (0-8) required.
- **Phase 1 parallel `Explore` fan-out is mandatory** — 3 to 5 agents dispatched in a single message:
  - One per distinct submodule under the target.
  - One per primary consumer surface (callers the target exposes a contract to).
  - One per shared utility cluster the target imports.
- **Hyrum's-Law review is mandatory** — what observable details (field order, error message text, status-code choice, timestamp format) may external consumers have started depending on? Surface as architecture observations.
- **Consumer-surface sweep is mandatory** — for every exported symbol, grep the consumer surface and note any caller that depends on implementation detail.

**Ceremony:**
- Parallel fan-out in a single message (not sequentially — sequential fan-out wastes the advantage).
- Phase 1.4 pattern-exemplar sweep + Phase 1.5 test-shape pre-read are both mandatory.
- 2-3 clarifying questions if the target's boundaries are genuinely ambiguous.
- Typical findings count: 20-60 total.
- P1 count ceiling: 20. Architecture observations: 2-5 is typical; zero is suspicious.

**Pitfalls:**
- Running the fan-out sequentially — defeats the purpose. One message, multiple `Explore` calls.
- Producing 50 P1s in a 3000-LOC audit — calibration is wrong. Re-read the severity rubric and demote.
- Skipping the consumer-surface sweep and producing findings about an exported symbol without checking how it is used — a symbol's callers constrain which refactors are actually safe.
- Deep audit, zero architecture observations — you under-scrutinised. Re-run Phase 3 with the cohesion + layering lenses sharper.

---

## Severity rubric (cross-tier)

Detailed bands, per Phase 4:

| Severity | Cost band | Recognition test | Typical example |
|---|---|---|---|
| **P0 — Blocker debt** | Active incident risk; a user-visible failure mode is one pager away, OR a security/data-invariant is implicit. | *"If nothing changes, we will page on this shape within a month."* | A SWALLOW on the payment-capture path; a SCATTER of null-guards where the invariant is one deep call away from being violated; a mock-behaviour test masking a real contract drift in a critical module. |
| **P1 — Major debt** | Likely to cause an incident in a quarter, OR blocks a known-likely upcoming change. | *"The next time someone changes this area, they will hit this."* | Shotgun Surgery across a module the roadmap says is about to get a new currency; Primitive Obsession on a domain concept whose validation has already drifted twice; Feature Envy that forces the next feature to touch three classes. |
| **P2 — Minor debt** | Makes maintenance harder. No active risk. | *"This is annoying when you read it; it does not hurt anything today."* | Middle Man with no foreseeable consumer; Long Parameter List on a cold path; a single Data Clumps instance without change pressure. |
| **P3 — Nit** | Small cleanup. | *"Worth doing if someone has five minutes; otherwise drop."* | Naming drift; a single Duplicate Code instance that has not drifted; Comments as Deodorant on a function with no complexity. |

Calibrate against this rubric. An audit where everything is P1 is noise; an audit where everything is P3 under-serves the next maintainer.

---

## Cross-cutting rules (all tiers)

- **The skill NEVER writes code.** Pure reporter.
- **Deliberate patterns are not smells.** If a finding's alleged smell is called out in an ADR, a comment, or the repo's own convention file, suppress it and list it under Suppressed candidates with the reason.
- **Real current bugs leave the audit.** If a finding turns out to be a live crash / data loss / security hole, it goes to the top-level **Bugs surfaced** section and the skill recommends `/diagnosis`. The audit does not masquerade as a bug tracker.
- **Clean passes are valid.** An audit with zero P0/P1 findings is a useful result — the artifact calls the surface stable and lists Positives. Padded findings are worse than a clean pass.
- **HARD GATE at Phase 7.** Regardless of tier, writing `AUDIT.md` requires explicit user approval of the finding set and output path.

---

## Promoting / demoting a tier mid-audit

- Classified Lightweight and the target turns out to be imported by 20 consumer files → promote to Standard. Announce: `audit: scope promoted to STANDARD — target is a shared kernel.`
- Classified Deep and the target turns out to be 600 LOC of generated types the user explicitly said to skim → stay Deep (the investigation earned the ceremony), but reduce the human-read weight in the artifact (Positives-heavy, Nits-light).

Promotions mid-audit re-fire any skipped phases (parallel fan-out, consumer-surface sweep). Demotions do not retroactively drop already-produced findings.

---

## Quick reference

| Tier | Files | LOC | Fan-out | Architecture obs | Consumer sweep | P1 ceiling |
|------|-------|-----|---------|------------------|----------------|------------|
| Lightweight | 1-3 | <500 | — | skip unless shape emerged | — | 5 |
| Standard | 4-15 | 500-3000 | — | required | optional | 10 |
| Deep | 15+ | 3000+ | 3-5 `Explore` agents in one message | required | required | 20 |

Scope tunes depth. Severity classifies findings. They are independent.
