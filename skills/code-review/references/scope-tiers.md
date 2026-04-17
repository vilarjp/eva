# Scope Tiers — Detailed Rubric for Code Review

Code reviews come in wildly different shapes. A two-line typo fix and a 600-line architectural change both enter through `/code-review`, but they deserve different ceremony. The scope tier tunes which reviewers run, how deep they go, whether the adversarial Stage 3 fires, and how long the orchestrator's merge phase takes.

**Scope classifies the diff, not the fix.** A tiny diff inside a concurrent code path can still be Deep. A large generated-code diff can still be Lightweight. Read the diff before classifying.

## Decision flow

```
Is the diff < 50 LOC AND in 1-2 files AND no user input / no security surface / no concurrency? → Lightweight
Is the diff 50-500 LOC OR crosses 3-10 files OR touches real logic?                            → Standard
Does the diff touch concurrency, auth/auth, payments, migrations, public APIs,
  cross-module boundaries, OR exceed 500 LOC OR span 10+ files?                                → Deep
```

If the diff has a corresponding plan document (PRD/SPEC/DIAGNOSIS), the plan's complexity (`LOW | MEDIUM | HIGH`) is a strong prior for the scope tier. Use `MAX(plan_complexity, diff_signals)`.

---

## Tier: Lightweight

**Signals:**
- Diff < 50 LOC across 1-2 files.
- No user-input-facing change, no new endpoint, no auth/permission change.
- No concurrency, no queue, no migration, no new dependency.
- Examples: README touch-up, a typo fix, renaming a local variable, adding a unit test for existing code, a small refactor with no behavior change, updating a constant.

**Reviewers that run:**
- `code-quality` (required)
- `convention` (required)
- `test` (required only if test files are in the diff; otherwise returns a "no tests in diff" note)
- `plan-alignment` **only** if a plan exists AND the orchestrator can confidently map the diff to a phase/acceptance-criterion. Otherwise skip.
- `security`, `performance`, `adversarial` — skipped by default.

**Orchestrator ceremony:**
- Stage 1 (plan-alignment) runs only if plan exists.
- Stage 2 (quality + convention + test) runs in parallel.
- Stage 3 (adversarial) is skipped unless the user explicitly asks for Deep review.
- Merge phase runs. Suppression applied. Artifact written.
- Typical report size: 3-10 findings total, mostly P2/P3.

**Pitfalls:**
- "It's just a rename" is not the same as "the rename is safe." Check call sites.
- A one-line fix in a payment path is NOT Lightweight — promote to Deep regardless of LOC.
- README or docs-only diffs are valid Lightweight; don't inflate ceremony because the diff is "in the codebase".

---

## Tier: Standard

**Signals:**
- 50-500 LOC across 3-10 files.
- New logic, new branches, modified business rules, new or edited tests.
- Touches one or two domain modules but not cross-cutting infrastructure.
- May introduce user-visible behavior — but not in the security-critical or concurrency-critical zones.
- Examples: a new endpoint with existing auth, a refactor of a service layer, adding a feature to an existing flow, updating a form component with new fields.

**Reviewers that run:**
- `code-quality` (required)
- `convention` (required)
- `test` (required)
- `plan-alignment` (required if plan exists — even imperfect mapping counts)
- `security` (required if the diff touches any of: user input handlers, auth, permissions, sessions, secrets, rendered output, file paths, URLs, database queries, third-party calls)
- `performance` (required if the diff touches: loops over user-controlled input, DB queries, API fan-out, render paths with >1000 expected items, cron/batch jobs)
- `adversarial` — skipped by default.

The orchestrator auto-detects the `security` and `performance` triggers using grep/glob on the diff. If either trigger fires, the corresponding reviewer is added to Stage 2.

**Orchestrator ceremony:**
- Stage 1 (plan-alignment) is BLOCKING. If plan-alignment returns a P0 (diff does not implement the approved plan), Stage 2 does not run until the author decides: fix the drift or update the plan.
- Stage 2 (quality + convention + test + conditional security + conditional performance) runs in parallel.
- Stage 3 (adversarial) is skipped unless the user explicitly asks for Deep.
- Merge phase runs. Dedup across reviewers. Suppression applied. Artifact written.
- Typical report size: 8-25 findings, mix of all severities, usually 0-2 P0s.

**Pitfalls:**
- Don't auto-skip security "because it's just a CRUD endpoint" — user input + DB queries = security scope.
- Don't auto-skip performance "because it's small" — N+1 queries hide in 20-line diffs.
- If the plan exists and the diff obviously does not implement it, Stage 2 is wasted effort — report the drift and stop.

---

## Tier: Deep

**Signals:**
- 500+ LOC, OR 10+ files, OR the diff touches any of:
  - Concurrency, races, locks, queues, workers, cron schedules.
  - Authentication, authorization, session management, RBAC.
  - Payments, refunds, financial ledgers, invoicing.
  - Migrations (forward or reverse), schema changes, index changes.
  - Public or semi-public APIs (external callers depend on the shape).
  - Cross-module boundaries (the diff edits 2+ domain modules).
  - Security-critical code paths (input validation at trust boundaries, crypto, signing, secret handling).
  - Infrastructure / deploy / CI / release scripts.
- Or: any diff the user explicitly asks to review as "Deep" / "thorough" / "red-team".

**Reviewers that run:**
- `code-quality` (required)
- `convention` (required)
- `test` (required)
- `plan-alignment` (required if plan exists; if no plan exists but the diff is Deep, the orchestrator surfaces this as a risk — Deep work without a spec is worth naming)
- `security` (required, regardless of trigger auto-detection)
- `performance` (required, regardless of trigger auto-detection)
- `adversarial` (required — Stage 3 fires)

**Orchestrator ceremony:**
- Stage 1 (plan-alignment) is BLOCKING. P0 drift halts the pipeline.
- Stage 2 (quality + convention + test + security + performance) runs in parallel.
- Stage 3 (adversarial) runs AFTER Stage 2 completes. The adversarial reviewer is given Stage 2's merged findings as input so it does not repeat them — its job is to surface what the other reviewers missed (unstated assumptions, failure modes under load, rollback hazards, Hyrum's-Law leakage).
- Merge phase runs on the full three-stage output. Full dedup. Confidence boost for cross-reviewer agreement. Suppression applied.
- Typical report size: 15-50 findings. 1-5 P0s are not unusual for Deep diffs.

**Pitfalls:**
- A Deep diff with zero P0s and zero P1s is suspicious — reviewers likely under-scrutinized. Re-read the most-touched file and re-run the adversarial reviewer.
- The adversarial reviewer must receive the Stage 2 findings — otherwise it will repeat them instead of extending.
- If three P0s all point to the same architectural pattern, stop — the design is wrong and the review should say so before nitpicking the code.

---

## Cross-cutting rules (all tiers)

- **The orchestrator NEVER writes code.** Pure reporter.
- **Pre-existing problems are reported separately.** The author should not be blamed for code they did not write. The orchestrator's "Pre-existing" table makes this distinction visible.
- **Verification-needed findings are surfaced.** A finding the reviewer could not fully confirm is still reported, with the gap flagged so the author knows where to look.
- **Confidence < 0.60 → Suppressed table.** Except P0s with `confidence ≥ 0.50`.
- **Clean passes are valid.** A reviewer with nothing to say returns empty arrays; the orchestrator records `"<reviewer>: clean"` in the artifact. Padded findings are worse than a clean pass.
- **HARD GATE at Phase 8.** Regardless of tier, writing `CODE-REVIEW.md` requires explicit user approval of the finding set and output path.

---

## Severity rubric (cross-tier)

| Severity | Meaning | Typical finding types |
|----------|---------|----------------------|
| **P0** | Ship-blocker. Bug, regression, security hole, data loss, plan drift, failing test that claims to pass. | Null dereference in user path, missing auth check, broken migration, wrong API contract, test file is green but doesn't actually assert anything. |
| **P1** | Major defect. Fixable without replanning; must resolve before merge. | Race condition, missing idempotency key on retried write, N+1 query hitting a paginated list, convention violation that will force a later refactor, missing test for a branch that actually matters. |
| **P2** | Minor quality. Worth fixing; not ship-blocking. | Small duplication, unclear variable name, missing small test case, helpful log that isn't structured, lint-style issue. |
| **P3** | Nit. Preference; author may decline. | Capitalization, one-word rename, inline comment preference, formatting. |

Calibrate against this rubric in every reviewer agent. A review that flags everything as P1 or everything as P3 is miscalibrated and should be re-run.

---

## Promoting / demoting a tier mid-review

It is fine — often necessary.

- Classified Lightweight and the diff turns out to touch a payment path → promote to Deep. Announce: `code-review: scope promoted to DEEP — diff edits src/payments/authorize.ts`.
- Classified Deep but the diff is 800 LOC of generated Protobuf that the user explicitly said to skim → stay Deep (the *investigation* already earned its ceremony), but note the reduced human-review weight in the artifact.

Promotions mid-review re-fire any skipped reviewers. Demotions do not retroactively drop already-run reviewers (their findings still count).

---

## Quick reference

| Tier | LOC | Files | Stage 1 | Stage 2 | Stage 3 | Conditional |
|------|-----|-------|---------|---------|---------|-------------|
| Lightweight | <50 | 1-2 | plan-alignment if plan | quality + convention + test | skip | security/performance skipped |
| Standard | 50-500 | 3-10 | plan-alignment if plan | quality + convention + test | skip | security/performance auto-added if triggers match |
| Deep | 500+ or 10+ files or risk-tagged | varies | plan-alignment if plan; risk-flag if no plan | quality + convention + test + security + performance | adversarial required | all reviewers required |

Scope tunes depth. Severity classifies findings. They are independent.
