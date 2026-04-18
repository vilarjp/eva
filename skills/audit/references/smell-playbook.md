# Smell Playbook — Full Catalogue for /audit Phase 2

The operating catalogue for Phase 2 of `/audit`. Every finding must cite one named smell from this playbook. The playbook covers three layers:

1. **Fowler vocabulary** (13 smells) — classical refactoring smells. Most P2 / P3 findings live here, with a handful of P1s when the shape blocks a foreseeable change.
2. **Workaround categories** (7 categories from `../_shared/no-workarounds.md`) — fix-discipline smells. These often clear P0 / P1 because they mask causes rather than solve them.
3. **Test-layer anti-patterns** (5 named patterns from `agents/code-review-test-reviewer.md`) — test smells that let bugs ship. Calibrated to the repo's posture (DAMP vs DRY, mocking style).

A finding whose smell is not in this playbook is either unnamed (drop or rewrite) or outside the audit's remit (hand off to `/diagnosis`, `/code-review`, or `/teardown`).

**How to use during Phase 2.** For each candidate finding, walk the gate question. If the answer says "this shape is legitimate here", suppress with a reason. Otherwise, emit the finding with the category name in `smell`, a 5-30-word verbatim quote in `evidence`, a one-sentence cost, and a one-sentence refactor shape. No code blocks in the artifact.

---

## Layer 1 — Fowler vocabulary

Thirteen named smells from the refactoring canon, tuned for audit posture. Each entry lists the signal that surfaces it, the gate question that separates smell from deliberate design, an example audit finding, and when NOT to flag.

### 1. Feature Envy

A function uses another type's data or methods more than its own — the logic belongs on the other type.

- **Signal.** A method that reads 3+ fields from a collaborator and none (or one) of its own; a chain of `other.x`, `other.y`, `other.z` inside the body with no reference to `this.*`; a module-level helper that takes a single collaborator and mostly unpacks it.
- **Gate question.** *"If this function moved to the collaborator, would most of its arguments disappear?"* If yes → Feature Envy.
- **Example finding.** `F-N | P2 | src/order/service.ts:142 | Feature Envy | OrderService.calculateTotal reads four fields from Cart and none of its own — move to Cart.calculateTotal.`
- **When NOT to flag.** Cross-boundary orchestrators (services that coordinate two collaborators) legitimately read from both; DTO mappers legitimately unpack; the pattern is only a smell when the logic could clearly live on the collaborator and doing so would collapse coupling.

### 2. Primitive Obsession

Domain concepts (money, ids, emails, durations, URLs) carried around as raw `string` / `number` / `Date`; validation scattered across every call site instead of living on a small type.

- **Signal.** Functions that take `(amount: number, currency: string)` repeatedly; validation regex repeated in 4+ files for the same shape; the same string coerced and re-parsed at every boundary; comments like `// must be positive int, cents`.
- **Gate question.** *"If the next currency / id format / unit is added, how many files need to change?"* If the answer is "more than one", the concept wants a type.
- **Example finding.** `F-N | P1 | src/billing/*.ts | Primitive Obsession | Money passed as (amount: number, currency: string) across 11 call sites; validation drifted twice in the last quarter per git log — extract Money value object and parse at the boundary.`
- **When NOT to flag.** Genuine one-off primitives (an internal cache key, a single-use sort index); when the language / framework idioms make value objects costly enough that the team has made a deliberate choice documented in an ADR.

### 3. Data Clumps

The same 3+ fields keep appearing together across parameter lists, DTOs, and function signatures — they want to be a type.

- **Signal.** `(startDate, endDate, timezone)` triples recurring in 5+ signatures; constructors that take `(street, city, postcode, country)` without an `Address`; DTOs that embed the same substructure inline rather than by reference.
- **Gate question.** *"Do these fields always travel together, always get validated together, and always get updated together?"* If yes → Data Clumps.
- **Example finding.** `F-N | P2 | src/scheduling/*.ts | Data Clumps | (startDate, endDate, timezone) appears in 7 function signatures; extracting a TimeWindow type removes 14 parameters and centralises timezone validation.`
- **When NOT to flag.** Two fields that happen to share a signature once are coincidence; three fields that appear together in two unrelated modules may be two different concepts — confirm by reading the usages before flagging.

### 4. Shotgun Surgery

A single logical change forces edits across many unrelated files; the concern is smeared, not encapsulated.

- **Signal.** *"Adding a currency would touch `format.ts`, `validate.ts`, `tax.ts`, `display.tsx`, `checkout.ts`, `reports.ts`, `mail.ts` ..."*; the same `if (currency === 'USD')` / `if (currency === 'EUR')` branch appearing in 5+ files; domain enums checked by their string values in render paths.
- **Gate question.** *"For the next likely change in this area (per PRD / roadmap / recent commits), how many files will need to be edited and in how many directories?"* If the count is high AND the edits look mechanical, the concern wants to be in one place.
- **Example finding.** `F-N | P1 | src/billing/ (11 files) | Shotgun Surgery | Currency-specific branches in 11 files; the multi-currency roadmap item would force edits in each — extract a Currency policy object; callers delegate formatting, tax, and display.`
- **When NOT to flag.** Cross-cutting concerns that are deliberately cross-cutting (logging calls, audit-trail writes, i18n markers) — those are supposed to touch many files because that's the contract.

### 5. Divergent Change

One module gets modified for multiple unrelated reasons — it is doing several jobs and should split.

- **Signal.** A file's git log shows edits for *"add coupon support"*, *"fix tax rounding"*, *"switch to new email provider"*, *"add retry"* all in the last three months; a class with 20+ methods across 4 domains (persistence, formatting, business rules, third-party); import statements spanning DB, HTTP, rendering, and math utilities in one file.
- **Gate question.** *"List the last five reasons this file was edited. Are they coherent (the same domain) or scattered (four unrelated reasons)?"* Scattered → Divergent Change.
- **Example finding.** `F-N | P1 | src/user/manager.ts:1-480 | Divergent Change | Last 10 commits split across auth, profile, billing notifications, and admin tooling — split into UserAuth, UserProfile, UserBillingEvents.`
- **When NOT to flag.** A "God file" that is genuinely a thin facade over composed concerns (the file is 40 lines of `re-export`) is not Divergent Change — it is a seam, possibly a Middle Man, possibly fine.

### 6. Long Parameter List

5+ positional parameters, or a boolean trap (`doThing(x, true, false, null)`) whose meaning is unreadable at the call site.

- **Signal.** Functions taking 6+ positional args; call sites that look like `fn(a, b, true, false, null, undefined, 0)`; optional param lists that have drifted to 8+ entries as features accrete.
- **Gate question.** *"Does the caller understand what each argument means without jumping to the definition?"* If not, the signature has too many positions.
- **Example finding.** `F-N | P2 | src/search/query.ts:88 | Long Parameter List | buildQuery takes 9 positional params; 6 call sites all mis-order at least one boolean — extract QueryOptions struct.`
- **When NOT to flag.** Math-style utilities that take many coordinates (`drawLine(x1, y1, x2, y2, thickness)`) — positional is idiomatic; hot-path constructors where an options object would allocate.

### 7. Message Chain

`a.b().c().d().e()` — fragile coupling to the internal shape of `a`; hide it behind an intent-named method on `a`.

- **Signal.** Chains of 3+ method calls on different receivers; tests that mock each link in the chain; callers that read `order.customer.address.country.code` throughout.
- **Gate question.** *"If `a`'s internal structure changed, how many call sites would need to know?"* Each link is a coupling; long chains lock many sites to the shape.
- **Example finding.** `F-N | P3 | src/order/render.tsx:34 | Message Chain | order.customer.address.country.code appears in 8 render paths; extract order.countryCode() so callers stop walking the shape.`
- **When NOT to flag.** Fluent APIs that are designed to be chained (query builders, test-data builders) — the chain IS the contract, not a smell.

### 8. Middle Man

A class whose methods just forward to a single collaborator — remove the wrapper or give it real work.

- **Signal.** A class with 8+ methods, each a one-line delegation to a single collaborator field; no caller uses the wrapper's identity for polymorphism; tests are mostly *"when I call `wrapper.x`, `collaborator.x` is called"*.
- **Gate question.** *"What does this class add that the collaborator doesn't provide?"* If the honest answer is "nothing, yet" → Middle Man.
- **Example finding.** `F-N | P2 | src/repo/user-facade.ts:1-120 | Middle Man | UserFacade forwards 14 methods 1:1 to UserRepository with no extra behaviour — inline callers to UserRepository directly.`
- **When NOT to flag.** The wrapper exists to cross a module boundary deliberately (adapter pattern, port/adapter layering) and the indirection is load-bearing even if each method is 1:1.

### 9. Refused Bequest

A subclass overrides parent methods to no-op or throw — inheritance is the wrong seam; prefer composition or a narrower interface.

- **Signal.** A subclass with several `override` methods that `throw new Error('not supported')` or return early with `return;`; a type hierarchy where 40% of the methods on the base don't apply to half the subclasses; tests that assert *"this method throws on this subclass"*.
- **Gate question.** *"Does the subclass genuinely IS-A the parent, or is it only using the parent's concrete bits?"* If the former with holes → Refused Bequest.
- **Example finding.** `F-N | P2 | src/notifier/sms-notifier.ts:22 | Refused Bequest | SmsNotifier overrides sendAttachment to throw; base class Notifier assumes attachment support — split into Notifier and AttachmentCapableNotifier.`
- **When NOT to flag.** Framework-imposed base classes (`React.Component` subclasses legitimately override only some lifecycle methods) — refusal is the contract.

### 10. Speculative Generality

An abstraction (factory, strategy, plugin, hook) with one implementation and no concrete second consumer — speculative scaffolding; overlaps with YAGNI.

- **Signal.** A `XxxFactory` with one `create` method returning one fixed type; a `Strategy` interface with one implementation; a plugin registry with one registered plugin; config fields that are read but whose values never vary in production.
- **Gate question.** *"Is there a second implementation coming in the next two quarters, or is this the only concrete one?"* If the only one → Speculative Generality.
- **Example finding.** `F-N | P2 | src/payment/processor-factory.ts:1-45 | Speculative Generality | PaymentProcessorFactory has one implementation (StripeProcessor) and no roadmap for a second; inline StripeProcessor at the call site.`
- **When NOT to flag.** Abstractions introduced deliberately at a boundary (public API, plugin surface) even if only one implementation exists today — the abstraction IS the contract for external consumers. Check for external consumers before flagging.

### 11. Temporary Field

A field that is `null` / `undefined` most of the time and only set in one code path — extract the "with-field" state into its own type.

- **Signal.** A class with a `discountCode?: string` that 90% of instances don't have; a struct with a `retryCount?: number` that only matters during retries; types with nullable fields that group semantically ("all these 4 fields are set together, or none are").
- **Gate question.** *"Does the existence of this field correlate with a specific state? If I split that state into its own type, do the fields stop being optional?"* If yes → Temporary Field.
- **Example finding.** `F-N | P2 | src/booking/order.ts:18 | Temporary Field | Order.refundReason, refundedAt, refundedBy are all null unless status === 'REFUNDED' — extract RefundedOrder with the fields required.`
- **When NOT to flag.** Genuinely optional user-provided fields (`notes?: string`) where null-or-present IS the domain.

### 12. Duplicate Code

Two or more blocks that compute the same thing with small drift — one of them is probably wrong; factor the shared behaviour.

- **Signal.** Two functions with 80%+ identical body and slightly different fix-ups; the same regex / parser repeated across files; the same sanitation logic in the controller and the service; "copied from X" comments.
- **Gate question.** *"If one of these copies had a bug, would the other copy have the same bug or have drifted past it?"* Drift already happened → Duplicate Code; hasn't yet → Duplicate Code waiting.
- **Example finding.** `F-N | P2 | src/reports/format.ts:12 and src/billing/format.ts:8 | Duplicate Code | formatCurrency implemented in both with divergent fraction-digit defaults; billing-callers occasionally import the reports version — move to src/shared/format.ts and parameterise.`
- **When NOT to flag.** "DAMP over DRY" tests (repo convention is deliberately repetitive test setup for readability); two modules that look alike but model different domains that will diverge later — duplication is cheaper than premature abstraction.

### 13. Comments as Deodorant

A long comment justifying a gnarly block — the comment is often the code screaming to be refactored; treat as a hint, flag the gnarly block.

- **Signal.** A comment like *"This is ugly but necessary because..."* followed by 40 lines; a block-comment larger than the block it describes; a comment that restates the control flow line by line; `// TODO: clean this up` older than six months.
- **Gate question.** *"If the comment were deleted, would the code be comprehensible?"* If no, the comment is propping up unclear code — the real smell is the code.
- **Example finding.** `F-N | P3 | src/scheduler/run.ts:72 | Comments as Deodorant | 18-line comment explains a nested conditional with 5 branches; extract to named predicate functions (isRetryEligible, isBackoffReady) and delete most of the comment.`
- **When NOT to flag.** Comments that explain *why* a decision was made (domain constraints, historical context, trade-off rationale) — those are the good kind. The smell is specifically *explaining-what-the-code-does-because-it-is-unclear*.

---

## Layer 2 — Workaround categories

Seven fix-discipline smells from `../_shared/no-workarounds.md`. These often clear P0 / P1 in audits because they mask causes and mature into incidents. Each entry reframes the shared reference for audit posture — the original reference is about "don't introduce this in a change"; here it's "this is already in the code; flag it".

For the full gate questions, before/after examples, and the five-condition Escape Valve, see `../_shared/no-workarounds.md`. This section is the audit-specific shorthand.

### TYPE — Escaping the type system

- **Signal in stable code.** `as any` / `@ts-ignore` / `# type: ignore` in production paths; unchecked casts on API responses; `unknown` propagated without a parse; old `@ts-expect-error` with no linked issue.
- **Audit gate.** *"Does the escape still hide a real type drift, or is the value now well-shaped at the boundary?"* If still hiding → P1 (likely Primitive Obsession or SCATTER adjacency).
- **Example finding.** `F-N | P1 | src/api/users.ts:34 | TYPE | response.data parsed as any then passed through 4 files; add UserSchema.parse at the boundary.`
- **When NOT to flag.** Narrowing casts after a verified runtime check with a one-line rationale; genuinely untyped third-party boundaries crossed once at a documented seam.

### LINT — Silencing the linter

- **Signal in stable code.** `// eslint-disable-next-line` with no justifying comment; rule exceptions piled into the config for one file; `# noqa` scattered across a module.
- **Audit gate.** *"Does each disable name the reason, and would the flagged rule catch a real smell if re-enabled?"* If no reason and the rule would catch a smell → P2.
- **Example finding.** `F-N | P2 | src/report/generator.ts:52 | LINT | Seven eslint-disable-next-line no-shadow across the file; renaming inner bindings removes the suppressions — actual fix is a rename pass.`
- **When NOT to flag.** Config-scoped exemptions for whole directories with a documented reason (test files exempt from production-only rules, generated files exempt from style rules).

### SWALLOW — Eating the failure

- **Signal in stable code.** `try { } catch { /* ignore */ }`, `.catch(() => {})`, `except: pass`; a `Result` failure branch that returns a synthesised success; a `return null` inside a function whose contract is non-nullable.
- **Audit gate.** *"If this call failed in production right now, would anything observable tell anyone?"* If no → P0 (incident one pager away) or P1 depending on path criticality.
- **Example finding.** `F-N | P0 | src/payments/capture.ts:118 | SWALLOW | catch{} on stripe.charge() — a failed capture is reported as success; no log, no metric, no retry; add a typed failure return and propagate.`
- **When NOT to flag.** Deliberately non-blocking best-effort tasks (cache invalidation, analytics fire-and-forget) whose failure mode is documented, observable, and accepted.

### TIMING — Waiting for the bug to not happen

- **Signal in stable code.** `setTimeout(..., N)` / `sleep(N)` inside production code; retry loops with *"works most of the time"* notes; polling intervals that exist because something races; warm-up calls that discard their result.
- **Audit gate.** *"What event would let this code run exactly once at the right time?"* If there is one and it isn't being used → TIMING.
- **Example finding.** `F-N | P1 | src/init/boot.ts:22 | TIMING | sleep(500) after startServer() — startServer already resolves on listen; remove the sleep.`
- **When NOT to flag.** Explicit rate limiters where the sleep IS the contract; documented exponential backoff against a rate-limited third-party API.

### PATCH — Special-casing the symptom

- **Signal in stable code.** `if (userId === 'admin')`, `if (customerId === 'cust-12345')`, `if (env === 'prod')` added to route around a specific report; allow-lists that grow per incident; conditionals named after tickets.
- **Audit gate.** *"Does this conditional encode a real domain rule or scar tissue from yesterday's bug?"* Scar tissue → P1 almost always (compounds fast).
- **Example finding.** `F-N | P1 | src/order/process.ts:47 | PATCH | if (order.customerId === 'cust-12345') skips validation — the validator bug is still in the tracker; fix upstream and remove the special case.`
- **When NOT to flag.** Feature flags gating a real progressive rollout with a sunset date; tier-based paths where "admin" is a first-class domain concept, not a workaround.

### SCATTER — Fixing the same bug in many places

- **Signal in stable code.** The same defensive check at 3+ call sites; the same `if (x == null) return` dotted across a module; repeated sanitisation of the same string; the same null-guard added in two files in the same commit.
- **Audit gate.** *"Why does every caller defend against this? Should the invariant live at the boundary?"* If yes → P1.
- **Example finding.** `F-N | P1 | src/user/*.ts (5 files) | SCATTER | if (!user || !user.name) return 'Guest' repeated in 5 callers — enforce UserSchema.parse at the loader; callers drop the guards.`
- **When NOT to flag.** Genuine defence-in-depth at a real trust boundary (input validation + DB constraint + output encoding is three layers by design); concurrency-critical paths where each caller's invariants genuinely differ.

### CLONE — Copying instead of refactoring

- **Signal in stable code.** Two functions with 80%+ identical bodies; *"copied from X"* comments; two files with the same sanitiser; helpers re-implemented because finding the existing one felt harder.
- **Audit gate.** *"What single concept is being expressed, and where should it live?"* If the answer is a single place and the clones have already drifted → P1.
- **Example finding.** `F-N | P2 | src/billing/format.ts:4 and src/reports/format.ts:4 | CLONE | formatCurrency implemented in both; drift already: reports uses minimumFractionDigits: 0; move to src/shared/format.ts, parameterise.`
- **When NOT to flag.** Two implementations in different domains that look alike today but will diverge later — premature abstraction couples them; leave duplicated and flag only if drift proves divergence was unsafe.

---

## Layer 3 — Test-layer anti-patterns

Five named patterns from `agents/code-review-test-reviewer.md`. Flag test smells ONLY when they match one of these — "this test could be cleaner" is out-of-lane for /audit. The five below are load-bearing: tests that exhibit them produce false green builds, which costs real production bugs.

### T1. Mock-behaviour assertion

The test asserts on what the mock returned — which the test itself configured — not on what production code produced.

- **Signal.** `expect(result).toEqual(mockReturn)` where `mockReturn` is the same literal the mock was configured with; spy assertions on internal calls whose arguments are already constants.
- **Audit gate.** *"If I mentally flip a branch in the production code, does this test fail?"* If no → toothless.
- **Example finding.** `F-N | P1 | src/checkout/coupon.test.ts:34 | Mock-behaviour assertion | expect(result).toEqual({ ok: true }) where mockService.validate mocked to return { ok: true } — assert on observable output (persisted row, emitted event) instead.`

### T2. Test-only production methods

A production class exposes a method / property / flag whose only caller is the test suite.

- **Signal.** Exported names like `_forTesting*`, `__internal*`, `resetForTests`, `getInternalState`, `overrideNow`; public members with no production consumers; default parameters only overridden by tests.
- **Audit gate.** *"Is the seam between test and production leaking into the production API?"* If yes → P2 (P1 if the test-only method is on a widely-imported class).
- **Example finding.** `F-N | P2 | src/clock/system-clock.ts:18 | Test-only production methods | setCurrentTime is only called from tests; extract a Clock interface and inject a fake in tests instead.`

### T3. Mocking without understanding

The author replaces a collaborator with a mock whose behaviour does not match the real collaborator's.

- **Signal.** A mock that always resolves when the real collaborator can reject / time out / return `null`; a mock for an HTTP call whose response shape is guessed, not captured from a real response; a library mocked with a function whose signature differs from the library's.
- **Audit gate.** *"Does this mock match the real contract, or the contract the test wishes existed?"* If wishes-existed → P1 on critical paths.
- **Example finding.** `F-N | P1 | src/payments/capture.test.ts:52 | Mocking without understanding | stripe.charges.create mocked to always resolve; real API rejects on 402/409/429; capture() has no branches for those — the test proves nothing about failure handling.`

### T4. Incomplete mocks

The mock implements the one method the test happens to call and nothing else; later refactors trip on unstubbed members silently.

- **Signal.** `jest.fn()` / `vi.fn()` returning `undefined` for members the production code may call on any branch; `as unknown as CollaboratorType` casts hiding missing members; three-method interfaces stubbed with only one method defined.
- **Audit gate.** *"If production called a second method on this mock tomorrow, would the test fail loudly or produce a misleading pass?"* Misleading pass → P2.
- **Example finding.** `F-N | P2 | src/queue/worker.test.ts:22 | Incomplete mocks | QueueClient stubbed with only dequeue(); ack() and nack() return undefined — a refactor that adds nack() to the worker silently passes.`

### T5. Tests-as-afterthought

Tests written after the production code, matching current behaviour (including bugs), structured to pass rather than pin the contract.

- **Signal.** Git log shows test file added strictly after production file in the same feature; hard-coded expected values that match the current (possibly buggy) output; bug-fix tests with no captured RED proof; tests that read like a restatement of the implementation.
- **Audit gate.** *"Does this test pin a contract the implementation could fail to meet, or does it just describe what the implementation currently does?"* If latter → P2 / P1 depending on the module's criticality.
- **Example finding.** `F-N | P2 | src/format/date.test.ts:1-40 | Tests-as-afterthought | All expected values mirror current implementation output verbatim (including 'Jan 1 2026' on a null-input branch that probably shouldn't render at all); re-derive each expectation from the documented contract.`

---

## Suppression guide

False-positive suppression keeps the audit honest. When a candidate finding looks like one of the smells above but is actually a deliberate choice, drop it AND record the drop under the artifact's **Suppressed candidates** section with a reason. Re-audits then don't re-surface the same drop.

Suppress when:

- **The pattern is documented in an ADR** naming it as a deliberate trade-off. Cite the ADR path in the suppression reason.
- **The repo's own convention prefers the flagged shape.** DAMP over DRY in tests, flat module structure over deep hierarchy, explicit factories over implicit wiring — calibrate against Phase 1.4 pattern exemplars. If the flagged shape IS the exemplar, suppress.
- **The smell is a stability decision at a public API boundary.** A `XxxFactory` with one implementation today may be the documented extension point for external consumers — suppress with "public API surface, extensibility is the contract".
- **The shape is forced by a framework or language idiom** that the repo has no freedom to work around (React's component pattern, Rails' fat-model convention, a typed ORM that requires certain shapes).
- **A recent commit intentionally introduced the pattern.** Read the commit message before flagging; the author may have chosen it with a stated rationale.

Do NOT suppress when:

- The only justification is *"this has always been this way"* — age is not a reason.
- The only justification is *"the person who wrote it is gone"* — absence is not a reason.
- The justification is *"a bigger refactor would fix it"* — that IS the refactor; the smell still stands, file it under the appropriate severity.
- The pattern is framed as "deliberate" but no ADR, comment, or convention exists to back the claim.

Suppression must be specific. *"Seems intentional"* is not a suppression reason. *"Per ADR-0007 — value objects held off pending multi-currency milestone"* is.

---

## Quick reference

| Layer | Smell | Typical severity ceiling |
|-------|-------|--------------------------|
| Fowler | Feature Envy | P2 |
| Fowler | Primitive Obsession | P1 (if drift already happened) |
| Fowler | Data Clumps | P2 |
| Fowler | Shotgun Surgery | P1 (if a roadmap change is pending) |
| Fowler | Divergent Change | P1 |
| Fowler | Long Parameter List | P2 |
| Fowler | Message Chain | P3 |
| Fowler | Middle Man | P2 |
| Fowler | Refused Bequest | P2 |
| Fowler | Speculative Generality | P2 |
| Fowler | Temporary Field | P2 |
| Fowler | Duplicate Code | P2 (P1 if drift already buggy) |
| Fowler | Comments as Deodorant | P3 |
| No-workarounds | TYPE | P1 |
| No-workarounds | LINT | P2 |
| No-workarounds | SWALLOW | P0 (critical path) / P1 |
| No-workarounds | TIMING | P1 |
| No-workarounds | PATCH | P1 |
| No-workarounds | SCATTER | P1 |
| No-workarounds | CLONE | P2 (P1 if drift is divergent) |
| Test | Mock-behaviour assertion | P1 |
| Test | Test-only production methods | P2 |
| Test | Mocking without understanding | P1 (critical path) |
| Test | Incomplete mocks | P2 |
| Test | Tests-as-afterthought | P2 (P1 for bug-fix commits w/o RED proof) |

Ceilings are defaults. Calibrate against actual cost per the `references/scope-tiers.md` severity rubric — a Shotgun Surgery on code nobody touches is P3; the same smell on the next roadmap item is P1.

---

## See also

- `../_shared/no-workarounds.md` — the canonical source for the seven workaround categories and the five-condition Escape Valve.
- `agents/code-review-quality-reviewer.md` — Fowler vocabulary for diff reviews (same set of 13 smells).
- `agents/code-review-test-reviewer.md` — canonical source for the five test anti-patterns.
- `references/scope-tiers.md` — severity rubric (P0 / P1 / P2 / P3) with cost bands and per-tier ceilings.
