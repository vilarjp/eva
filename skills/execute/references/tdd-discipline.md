# TDD discipline — the full ceremony

Load this file when a slice stalls on test design, when you're unsure whether a change is behavior-bearing, when a test passes on first run, or when you're tempted to skip RED.

## The Iron Law

**NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.**

This is absolute. Every line of behavior-bearing production code MUST be justified by a test that failed without it. The test is not a verification step — it is the specification.

**The Delete Rule.** If production code was written before the test: delete the production code. Write the test. Watch it fail. Rewrite the production code. No exceptions. Don't "keep it as reference." Don't "adapt it while writing the tests." Delete means delete.

## The cycle

```
RED:      Write a test. Run it. It MUST fail for the right reason.
GREEN:    Write the MINIMUM code to pass. Run the test. It MUST pass.
REFACTOR: Clean up while green. Tests MUST still pass after.
COMMIT:   Stage explicit files, write a Conventional message, move to the next slice.
```

One acceptance criterion → one slice → one cycle. Do not batch. Do not "write all tests first." That's horizontal slicing and it produces tests that verify *imagined* behavior instead of *actual* behavior.

## RED — watch it fail

Mandatory. Never skip.

A genuine RED meets all of:

1. The test runs.
2. The test **fails** (not errors — fails on an assertion).
3. The failure message describes the missing behavior. Good: `expected "valid" but got undefined`. Bad: `TypeError: cannot read property 'x' of null` — that's a setup bug, not a red test.
4. You ran the test in this turn and pasted the literal output.

If the test passes on first run, one of these is true:
- The behavior already exists → the slice may be unnecessary; re-check the SPEC.
- The test is tautological (`expect(fn).toBeDefined()`, `expect(true).toBe(true)`) → fix the assertion.
- The test is exercising the wrong path → fix the test to hit the real entry point.

Do NOT proceed to GREEN until you have a genuine failure.

## GREEN — minimum code to pass

Write the smallest amount of code that makes the failing test pass. Do NOT:

- Handle edge cases not covered by a test
- Add error handling "you know you'll need"
- Build an abstraction for a single use
- Optimize prematurely
- Modernize syntax in files you're only reading
- Reformat surrounding code
- Add configuration options no one asked for

If you find yourself writing code "you know you'll need for the next slice," stop. Commit this slice, start the next, write that slice's test first.

## REFACTOR — green only

Clean up locally, behaviorally neutral. Run the test after each edit. Tests must stay green.

**Never refactor while RED.** Get to GREEN first. A refactor that lands mid-RED hides which change actually fixed the test.

REFACTOR scope:
- Rename locals for clarity
- Extract a small helper when duplication is real (third-use rule)
- Collapse conditional noise

NOT refactor scope:
- Extract a cross-module abstraction
- Reorganize imports in a file you weren't touching
- Introduce a "flexibility" hook for a future slice

## Verification Mode — the narrow escape hatch

The ONLY relaxation of the Iron Law. Applies to changes that are provably non-behavior-bearing:

- Config files (`tsconfig.json`, `vite.config.*`, `.eslintrc*`, `vitest.config.*`, `biome.json`)
- Build files and scripts that don't change runtime output shape
- Type-only declarations (`*.d.ts`, `type` aliases with no runtime impact)
- Docs, READMEs, changelogs
- Pure formatting (prettier/eslint-style changes)
- Environment variable templates (`.env.example`)
- Database migrations *when paired with an integration test elsewhere* (the test is the proof)

**Verification Mode protocol:**

1. Make the change.
2. Run the full build. Confirm it compiles / transpiles without errors.
3. Run the full test suite. Confirm no existing tests break.
4. If the change affects build output, verify the output (e.g. bundle size didn't explode, types still resolve).
5. Record `mode: verification` in the slice's EXECUTION entry and name the reason in one short clause.

**Verification Mode does NOT apply when:**

- The change affects logic that runs at runtime, even indirectly
- The change removes a type-check that was catching something
- The change modifies a migration that alters production data
- You are unsure — **when in doubt, default to Iron Law.** It's always safe to test more.

## The Prove-It Pattern (BUG mode)

When fixing a bug, the bug IS the failing test. The DIAGNOSIS skill has already written the reproduction test (or documented UNTESTABLE with manual steps). Your job as executor is:

```
1. REPRODUCE: Run the existing reproduction test. Confirm RED for the documented reason.
2. CONFIRM:   The failure message matches the DIAGNOSIS Root Cause narrative.
3. FIX:       Implement the minimum change named in DIAGNOSIS's Suggested Fix.
4. VERIFY:    Run the test. It MUST now pass.
5. REGRESS:   Run the full suite. No regressions.
6. RED-GREEN PROOF (recommended): Revert the fix → test RED → restore fix → test GREEN → capture both outputs in EXECUTION.md.
```

If the reproduction test doesn't fail for the documented reason, STOP. Something has changed since DIAGNOSIS was written, or the diagnosis was incomplete. Raise a Confusion.

If the Suggested Fix doesn't make the test pass after 2 attempts, STOP. Go to the Step-Back protocol. Do not improvise.

## Test quality

### Survive a refactor

A good test asserts **observable behavior** through the public interface. A bad test asserts implementation details.

- ✅ "User can submit a checkout with a valid coupon" — survives a refactor of `applyCoupon`.
- ❌ "applyCoupon calls `discountStrategy.computeDiscount` exactly once" — breaks when you inline the helper.

Red flags:
- Mocking internal collaborators (things you own)
- Testing private methods
- Asserting on call counts or call order of your own code
- Test breaks when you refactor without changing behavior
- Test name describes HOW, not WHAT

### Preference order for test doubles

**Real > Fake > Stub > Mock.**

Prefer real implementations whenever practical. Use fakes (e.g. in-memory repo) at boundaries you own. Use stubs for deterministic external inputs. Reserve mocks for verifying interactions with systems you do NOT own (payment provider, email sender, clock, randomness).

Mocking rule: mock at **system boundaries** only. Do not mock:
- Your own classes or modules
- Internal collaborators
- Things whose contract you control

### DAMP over DRY in tests

**DAMP** = Descriptive And Meaningful Phrases. **DRY** = Don't Repeat Yourself.

In production code, DRY wins. In tests, DAMP wins. Each test should tell a complete, self-contained story. Duplication in setup is acceptable when it makes each test independently understandable. A test that requires you to traverse 4 files of shared fixtures to understand is a test the model will get wrong.

### One logical assertion per test

One behavior per test. "User can register AND log in AND access dashboard" is three tests, not one. Narrow failures are diagnostic; wide failures are mysterious.

### Survey existing tests before writing new ones

Before writing a test, read the nearest existing test file for the module. Match its:
- Fixture/factory style
- Describe/context/it nesting
- Assertion library and matchers
- Setup/teardown pattern
- Naming convention

Your new tests must be indistinguishable from the existing ones in style. This is non-negotiable — a test that looks foreign is a test that will be ignored or rewritten.

## The System-Wide Test Check (Deep scope)

Inject these 5 questions before writing the test, for any Deep slice:

1. **What fires when this code runs?** — Callbacks, middleware, observers, event listeners, lifecycle hooks. Trace two levels out from the change.
2. **Do tests exercise the real chain?** — Not just the unit in isolation. Integration tests with real objects through the callback/middleware chain are where Deep bugs live.
3. **Can a test failure leave orphaned state?** — If your code persists state (DB row, cache, file) before an external call, does retry produce duplicates?
4. **What other interfaces expose this?** — Mixins, HOCs, alternative entry points (Agent vs Chat vs ChatMethods, HTTP vs GraphQL vs RPC). If parity is needed, add it in the same slice — not as a follow-up.
5. **Do error strategies align across layers?** — Retry middleware + application fallback + framework error handling. Do they conflict? Create double execution?

## Step-Back protocol (3-strike)

When a slice's test stays RED:

- **Attempt 1 failed.** Adjust and try again. You may have made a typo or missed a detail.
- **Attempt 2 failed.** STOP coding. Write down what was tried, what assumption was made, and why the assumption might be wrong. Try a FUNDAMENTALLY different approach — different data flow, different boundary, different decomposition. Not a variation of the same idea.
- **Attempt 3 failed.** STOP. Tell the user what was tried, what was assumed, and what's still unknown. Offer: (a) escalate to `/diagnosis` — the bug is deeper than the slice; (b) re-open `/spec` — the design is ambiguous; (c) ask the user for guidance.

Do NOT attempt #4. Three rejected approaches means the mental model is wrong, or the architecture is wrong, and attempting the same thing with more thrashing will compound the error.

## Captured test output

Every RED proof, every GREEN proof, every integration-gate run must include **literal stdout/stderr** from a fresh run. Not paraphrases. Not "tests pass." Paste the real output:

```
$ pnpm test tests/checkout/coupon.test.ts
 RUNS  tests/checkout/coupon.test.ts
 FAIL  tests/checkout/coupon.test.ts > applyCoupon > throws when cart is still loading
  expected function to throw 'CartNotReadyError', got 'TypeError: Cannot read properties of undefined'

Tests:  1 failed, 7 passed, 8 total
```

The machine-readable counts line (`Tests: 1 failed, 7 passed, 8 total`) is what EXECUTION.md's integration gate will cite. A plain "tests pass" sentence is not acceptable evidence.

## Final discipline check

Before writing EXECUTION.md:

- [ ] Every behavior-bearing slice has a RED proof captured
- [ ] Every Verification-Mode slice has a build+suite proof captured
- [ ] No slice was written to GREEN without a RED first
- [ ] No production code exists that isn't exercised by at least one test
- [ ] Tests assert on observable behavior, not internal structure
- [ ] Style matches existing tests in the module
- [ ] No mocks on collaborators you own
- [ ] No tautological assertions
- [ ] Full suite + lint + types are GREEN in a fresh run captured in this turn

If any item is `✗` → fix before Phase 8.
