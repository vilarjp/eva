# No Workarounds — A shared smell catalog for eva skills

A cross-skill reference that names the seven most common ways a change silences a symptom without fixing a cause. Cited by `/diagnosis` before it proposes a fix, by `/execute` at the slice-plan gate, and by `code-review-quality-reviewer` as a smell catalog.

This reference does NOT prescribe a fix. It names the shape of the anti-pattern so the proposed change can be rejected, reshaped, or (in rare audited cases) accepted under the Escape Valve.

## Why it exists

A workaround buys short-term quiet for long-term debt. Every category below looks reasonable in isolation — `as any` moves a line past the type checker, `try / catch` moves a line past the test, a retry loop moves a line past the flake. Six months later the cause is unreachable and the symptom has mutated. eva skills catch the smell at the moment it is introduced rather than the moment it matures into an incident.

## Iron Law — Fix causes, not symptoms

Every proposed change must be able to answer one question in one sentence: *"What is the underlying cause this change addresses?"* A change whose only honest answer is *"it makes the error/failure/warning go away"* is a workaround. Treat the symptom as a pointer to the cause and chase the pointer.

## When to cite this reference

| Skill | Phase | Purpose |
|-------|-------|---------|
| `/diagnosis` | Phase 6.3 — before the Suggested Fix is written | Reject symptom-level fixes; force a root-cause fix tied to the Causal Chain. |
| `/execute` | Phase 3 — per-slice check before the plan is shown to the user | Block a slice whose planned `Impl:` silences the symptom rather than enforcing the invariant. |
| `code-review-quality-reviewer` | Stage 2 — finding emission | Tag the smell by category name so findings carry canonical vocabulary. |

Other skills may cite it as a secondary reference when a phase asks the user to reason about *why* a change is being proposed.

## The seven categories

Each category has a **signal** (how the smell appears in a diff or a discussion), a **gate question** (the one-sentence challenge that separates a workaround from a legitimate use), and **before → after** (the minimum example that shows the shape). The before/after pairs are illustrative, not prescriptive — the fix shape varies by codebase.

---

### 1. TYPE — Escaping the type system

**Signal.** `as any`, `@ts-ignore`, `@ts-expect-error` without an issue link, `# type: ignore`, `// @ts-nocheck`, unchecked casts, `Object` / `any` / `unknown` as a param type on a function that clearly expects structure, a generic `<T = any>` on a new function.

**Gate question.** *"Is the type system wrong about this value, or is the value wrong about this type?"*

If the value is wrong (the caller is passing the wrong thing, the data is malformed upstream, the API contract has drifted), the type system is correct and the fix is upstream. If the type system is wrong (the library lacks generics, the discriminated union is incomplete, the declaration file is stale), the fix is the type — not an escape hatch.

**Before (workaround):**
```ts
const user = response.data as any;
return user.profile.name;
```

**After (cause-level fix):**
```ts
// Parse once at the boundary; downstream code works with a real type.
const user = UserSchema.parse(response.data);
return user.profile.name;
```

Escape hatches that *are* legitimate: narrowing after a verified runtime check, crossing a genuinely untyped third-party boundary with a one-line rationale, a `@ts-expect-error` that is paired with a linked issue and reverted by a test.

---

### 2. LINT — Silencing the linter

**Signal.** `// eslint-disable-next-line`, `// eslint-disable`, `/* eslint-disable */`, `# noqa`, `# pylint: disable=...`, rule exceptions added to the config for the current file, suppression pragmas with no justifying comment.

**Gate question.** *"Is the linter wrong about this code, or is the code wrong about this rule?"*

Most disables exist because the rule caught a real smell the author did not want to fix in this PR. A disable is acceptable when the rule itself is genuinely unsuitable for the file (e.g. a test file exempt from a production-only rule, a generated file exempt from style rules) — and in those cases the exemption belongs in the config, scoped by glob, not sprinkled line-by-line.

**Before (workaround):**
```ts
// eslint-disable-next-line no-shadow
const user = fetchUser();
{
  const user = fetchUser(); // now shadows
  return user;
}
```

**After (cause-level fix):**
```ts
const outerUser = fetchUser();
{
  const innerUser = fetchUser();
  return innerUser;
}
```

Escape hatches that *are* legitimate: a config-scoped exemption for a whole directory with documented reason; a single-line disable paired with a one-line rationale and a link to the issue tracking the cleanup.

---

### 3. SWALLOW — Eating the failure

**Signal.** `try { ... } catch { /* ignore */ }`, `catch (e) { /* silent */ }`, `except: pass`, `.catch(() => {})`, a `Result` / `Either` failure branch that returns a synthesised success, a timeout that falls through to a "fake success" response, a silent `return null` inside a function whose contract is non-nullable.

**Gate question.** *"What does the caller need to do differently when this operation fails?"*

If the caller needs to do anything — retry, degrade, display an error, abort a transaction, emit a metric — the failure must propagate with enough information to do it. A swallowed exception hides the signal that would have told the caller to act. The legitimate form is a conscious, named degradation: the caller is given a typed failure and chooses what to do.

**Before (workaround):**
```ts
try {
  await publishEvent(userId);
} catch {
  // best-effort; ignore
}
```

**After (cause-level fix):**
```ts
try {
  await publishEvent(userId);
} catch (err) {
  logger.error({ err, userId }, 'event publish failed');
  metrics.increment('event.publish.fail');
  // Caller documents the degradation: user-facing flow is unaffected;
  // downstream analytics will be incomplete until the retry job drains.
}
```

Escape hatches that *are* legitimate: a deliberately non-blocking background task whose failure mode is documented and observable; a best-effort cache-invalidation where stale-on-failure is the intended behaviour.

---

### 4. TIMING — Waiting for the bug to not happen

**Signal.** `setTimeout(..., 100)` / `sleep(1)` inside production code (not a test), a retry loop with a "works most of the time" note, a polling interval that is there because something races, `await new Promise(r => setTimeout(r, N))` before calling an API that ought to be ready, a "warm-up" call that discards its result, a retry-on-generic-Error handler that catches everything.

**Gate question.** *"What event would let this code run exactly once, at the right time, without waiting?"*

Sleeps and retries hide synchronisation bugs. The cause is almost always a missing signal — a promise that was not awaited, a ready event that was not subscribed to, a mutex that was not taken, a handshake that was elided. The cause-level fix is to surface the signal.

**Before (workaround):**
```ts
await startServer();
await sleep(500); // give it time to bind
await fetch('http://localhost:3000/health');
```

**After (cause-level fix):**
```ts
await startServer(); // resolves when the listen callback has fired
await fetch('http://localhost:3000/health');
```

Escape hatches that *are* legitimate: an explicit rate limiter where the sleep IS the contract; an exponential backoff against a documented rate-limited API; a test harness that waits for an observable event with a timeout as a safety net.

---

### 5. PATCH — Special-casing the symptom

**Signal.** `if (userId === 'admin') { skip the check }`, `if (env === 'prod') { different behaviour }` added to fix a specific report, allow-lists that grow one entry per incident, environment-specific branches inside business logic, a flag named after a user / customer / ticket number, a conditional that exists to route around a known crash.

**Gate question.** *"Does this conditional encode a real domain rule, or is it a scar tissue around yesterday's bug?"*

Scar-tissue conditionals compound: each one makes the function harder to reason about and invites the next one. The cause-level fix is to either encode the rule as a first-class concept (a permission, a feature flag with a well-defined lifecycle, a tier) or to fix the underlying bug so the special case is unnecessary.

**Before (workaround):**
```ts
function processOrder(order) {
  if (order.customerId === 'cust-12345') {
    // Customer 12345 hits the validator bug; skip it.
    return applyDiscount(order);
  }
  validateOrder(order);
  return applyDiscount(order);
}
```

**After (cause-level fix):**
```ts
function processOrder(order) {
  validateOrder(order); // bug in validator fixed upstream; no special case
  return applyDiscount(order);
}
```

Escape hatches that *are* legitimate: a feature flag gating a real progressive rollout, with a sunset date; a tier-based code path where "admin" genuinely is a first-class concept in the domain.

---

### 6. SCATTER — Fixing the same bug in many places

**Signal.** The same defensive check repeated at 3+ call sites, the same `if (x == null) return` dotted across a module, repeated sanitisation of the same user-supplied string, the same null-guard being added to two files in the same PR, "I fixed it here too" in the commit message.

**Gate question.** *"Why does every caller need to defend against this? Should the defence live at the boundary instead?"*

When the same guard appears in many places, the real defect is a boundary that lets the bad value through. The cause-level fix is to enforce the invariant at one place — the constructor, the parser, the API boundary, the database row-loader — so callers can trust the value and stop guarding against it.

**Before (workaround):**
```ts
function displayName(user) {
  if (!user || !user.name) return 'Guest';
  return user.name;
}

function sendEmail(user) {
  if (!user || !user.name) throw new Error('no name');
  // ...
}

function logActivity(user) {
  if (!user || !user.name) return;
  // ...
}
```

**After (cause-level fix):**
```ts
// At the boundary: a user without a name is never constructed.
const user = UserSchema.parse(row); // throws if name is missing

// Callers trust the invariant:
function displayName(user) { return user.name; }
function sendEmail(user) { /* ... */ }
function logActivity(user) { /* ... */ }
```

Escape hatches that *are* legitimate: defence-in-depth at a genuine trust boundary (input validation + database constraint + output encoding is three layers by design, not a SCATTER); redundant assertions in a concurrency-critical path where each caller may run under different invariants.

---

### 7. CLONE — Copying instead of refactoring

**Signal.** A new function whose body is 80% identical to an existing function, "I copied this from X and changed two lines", two files with the same sanitiser logic, a helper that already exists being re-implemented because finding it felt harder than retyping it, a test file that duplicates setup from a sibling test file with no shared fixture.

**Gate question.** *"What is the single concept being expressed, and where should it live so it is expressed once?"*

Clones drift: each copy evolves separately, bugs get fixed in one but not the other, behaviour diverges, and eventually the two paths look alike but do not behave alike. The cause-level fix is to locate the canonical implementation (or extract one) and delete the clone.

**Before (workaround):**
```ts
// src/billing/format.ts
export function formatCurrency(n) {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(n);
}

// src/reports/format.ts
export function formatCurrency(n) {
  // copied from billing/format.ts; tweaked to allow cents-off
  return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', minimumFractionDigits: 0 }).format(n);
}
```

**After (cause-level fix):**
```ts
// src/shared/format.ts — one place, parameterised.
export function formatCurrency(n, { minimumFractionDigits = 2 } = {}) {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', minimumFractionDigits }).format(n);
}
```

Escape hatches that *are* legitimate: two implementations that currently look identical but are governed by different domains (billing-currency vs reports-currency may diverge later — duplication is preferable to a shared abstraction that forces them to stay coupled); a test fixture deliberately duplicated to keep a test self-contained for a cold reader.

---

## The Escape Valve — when a workaround is the right call

Reality occasionally demands a workaround. The Escape Valve is narrow, audited, and time-bounded. A change invoking it must satisfy **all five conditions** — not three of five, not four. A skill that would otherwise reject the change MAY accept it when every condition is explicitly met in the artifact (DIAGNOSIS.md's Suggested Fix, EXECUTION.md's per-slice intent, or CODE-REVIEW.md's rationale).

1. **The cause has been named.** The workaround does not pretend to fix the cause; the artifact states the cause in one sentence and states the workaround's scope in another.
2. **The cause is out of scope for this change.** The root fix belongs to another team, another PR, another release, or an external dependency — and the artifact says which.
3. **The workaround is reversible.** Removing it is a single-commit operation once the cause is fixed. No schema migrations, no data re-writes, no user-facing contract changes hang off it.
4. **The workaround is observable.** A log line, metric, comment referencing the tracking issue, or TODO with a ticket ID. The workaround advertises itself so the next maintainer finds it.
5. **The workaround has an expiry condition.** A date, a release, a dependency upgrade, a closed issue — something that will make the workaround's removal discoverable. "When we rewrite this module" is not an expiry condition.

A change that satisfies the Escape Valve is no longer a workaround in the pejorative sense — it is a deliberate, short-lived tactical decision with an audit trail. A change that *does not* satisfy all five is a workaround regardless of how reasonable it looks, and a skill citing this reference is entitled to reject it.

## Anti-patterns in usage

- **Citing this reference without a category.** "This looks like no-workarounds" is not a finding. Name the category (TYPE / LINT / SWALLOW / TIMING / PATCH / SCATTER / CLONE) so the author knows which gate question to answer.
- **Treating Escape-Valve conditions as optional.** Three of five is not four of five is not all five. A partial match is still a workaround.
- **Using the catalog as a rhetorical weapon.** The point is to surface the cause, not to score points. Every finding that cites this reference should also name the cause-level fix or ask the author for it.
- **Stacking categories.** A single change rarely deserves more than one category tag. If a change looks like TYPE + SWALLOW + SCATTER at once, step back — the real finding is probably "this change has no clear intent".

## References

This reference is cited from:

- `skills/diagnosis/SKILL.md` — Phase 6.3, before the Suggested Fix is written.
- `skills/execute/SKILL.md` — Phase 3 slice-plan gate, before the first line of code.
- `agents/code-review-quality-reviewer.md` — Stage 2 finding vocabulary.

A finding or proposed fix that does not name a category and does not satisfy the Escape Valve should be sent back for revision.
