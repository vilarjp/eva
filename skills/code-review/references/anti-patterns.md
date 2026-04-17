# Anti-patterns — Code Review

Load this when a reviewer agent feels tempted to pad findings, when the orchestrator is tempted to skip a stage, or when the review is producing generic boilerplate instead of specific, actionable findings.

The cost of a bad code review is not "no review" — it is **false confidence**. A review that passes a broken PR because every reviewer returned "looks good" is worse than no review at all.

---

## The Iron Law, restated

**Specific + actionable > thorough-looking.** A review that surfaces 3 real problems the author will fix is a successful review. A review that produces 40 findings of which 36 are style preferences and 4 are real is a failure — the signal is buried.

If you cannot name *what specifically breaks* and *where specifically to change it*, drop the finding or lower the confidence.

---

## Reviewer anti-patterns

### Generic-pessimism findings

**Smell:** the finding reads like it could have been written without reading the diff.

- ❌ *"Consider adding error handling here."*
- ✅ *"`fetchOrder` calls `res.json()` without checking `res.ok` — a 500 from the upstream will be parsed as JSON and propagate an unhelpful `SyntaxError`; check at src/api/order.ts:34."*

The test: if you paste the finding into a different PR and it still reads plausibly, it is generic. Drop it.

### "This could be better" findings

Every line of code could be better. The reviewer's job is to name the specific thing that *goes wrong* if left as-is.

- ❌ *"This function could be cleaner."*
- ✅ *"`calculateTotal` has three `if` branches that mutate `total`; callers in `Cart.tsx:88` and `Checkout.tsx:104` pass overlapping input shapes that make the precedence ambiguous — this is the call path the bug at GH#1892 went through."*

### Padding clean lenses

If a reviewer has nothing material to say, it must return `findings: []` with a one-line `positives` or a clean-pass note. **Do not invent a P3 to look thorough.** The orchestrator treats padding as noise that drowns real signal.

A clean pass is valid and expected. It means the lens ran and the diff survived.

### Duplicate findings across reviewers

If `code-quality` and `convention` both independently notice the same naming issue, the orchestrator will merge them — but only if they return the **same `intent` string and the same file/line window**. If each describes the problem differently, the merge misses and the author sees two near-identical findings in different sections.

When writing a reviewer: use plain, consistent `intent` phrasing. Don't prefix it with the reviewer's name, don't hedge ("maybe unchecked null") — state the problem class directly.

### Out-of-lane findings

Each reviewer has a lane. Going out of lane produces:
- Confusion (author sees a security finding under "code quality").
- Dedup failures (the orchestrator expects security findings from the security reviewer).
- Loss of trust (the reviewer appears unfocused).

Staying in lane:

| Reviewer | Lane | Out of lane |
|----------|------|-------------|
| `code-quality` | Correctness, readability, bug risk, dead code, complexity. | Auth checks, injection, capacity planning. |
| `convention` | Matches the repo's existing patterns, naming, file layout, import style. | Business-logic bugs, performance. |
| `test` | Test quality, coverage gaps, TDD adherence, brittle assertions, mocking discipline. | Implementation bugs outside tests (unless the test *proves* them). |
| `plan-alignment` | Does the diff implement the approved plan? Gaps, over-scope, drift. | Code style, security depth (note and defer). |
| `security` | Trust boundaries, auth, validation, injection, data exposure, secrets, rate limiting, crypto. | Style, convention drift, generic quality. |
| `performance` | Hot paths, N+1, fan-out, memory, pool starvation, capacity. | Readability. |
| `adversarial` | What did the other reviewers miss? Unstated assumptions, failure modes under load, Hyrum's-Law leakage, rollback hazards. | Findings the other reviewers already produced. |

If you see a finding that belongs to a different lane, either drop it with a note (`"observed security concern — deferred to security reviewer"`) or, in Phase 2 dispatch, escalate to the orchestrator so the correct reviewer is added.

### Prescribing a rewrite

Reviewers flag. They do not redesign.

- ❌ *"Extract a new `CouponValidator` class that owns state transitions."*
- ✅ *"The `applyCoupon` function mixes validation, state transition, and HTTP call; if you ever need to re-run validation without firing the HTTP call, you cannot — consider splitting validation out. Low confidence on the specific split without more context."*

The second form surfaces the concern without inventing a design the reviewer does not have context for.

### Fabricated confidence

- ❌ Every finding has `confidence: 0.95`.
- ❌ Every finding has `confidence: 0.5`.

Honest calibration is what makes the Suppressed table useful. If you actually read the file and traced the call, `0.80+` is fair. If you inferred from the diff alone, `0.60-0.79`. If you are guessing, `0.40-0.59` and label `requires_verification: true`.

### Evidence paraphrase

Every finding needs a verbatim quote. "The code dereferences cart.items without a null check" is paraphrase, not evidence. Quote the line:

```
"const count = cart.items.length; // src/checkout/coupon.ts:62"
```

If you cannot produce a verbatim quote, the finding is not grounded. Drop it.

### Ignoring the pre-existing distinction

A finding in code the diff did not touch is `pre_existing: true`. This does not mean "ignore it" — it means "report it in the pre-existing table so the author is not blamed for code they did not write, and so they can triage whether to fix it in this PR or a follow-up."

Marking a real-change finding as `pre_existing: true` lets the author off the hook. Marking a pre-existing finding as `false` (the default) blames the author unfairly. Both are wrong.

---

## Orchestrator anti-patterns

### Running reviewers serially "to save tokens"

Stage 2 reviewers are **parallel** by design. Running them sequentially triples wall-clock time with no quality gain. The orchestrator must dispatch Stage 2 reviewers in a single message with multiple `Agent` tool calls.

### Merging Stage 1 and Stage 2 into one batch

Stage 1 (plan-alignment) is **blocking** on Standard and Deep tiers. Its job is to catch plan drift BEFORE the other reviewers waste cycles on code that does not implement the approved plan. Running plan-alignment in parallel with code-quality defeats that purpose.

Lightweight may collapse Stage 1 into Stage 2 (no real blocking needed on 50-LOC diffs).

### Running adversarial without Stage 2's findings

Stage 3 (adversarial) exists to find what Stage 2 **missed**. If the adversarial reviewer doesn't receive the Stage 2 output as context, it will repeat Stage 2's findings — wasting Stage 3 entirely.

The orchestrator must pass Stage 2's merged findings to the adversarial reviewer's input prompt.

### Skipping the HARD GATE

No `CODE-REVIEW.md` is written until the user approves the finding set and the output path. Skipping the gate and writing first is a protocol violation — announce it and stop if you catch yourself.

### Writing partial findings to disk during the review

The review is a transient, in-memory process. Nothing is persisted until Phase 9. An "intermediate CODE-REVIEW.md" is a bug — the merge/dedup/suppress pipeline must run to completion first, then one atomic write.

### "Smart reuse" that overwrites an existing review

If the user opts to append a re-review to an existing `CODE-REVIEW.md`, the orchestrator must append under a `## Re-review <DATE>` heading — it does NOT overwrite the prior content. The prior review is history; erasing it is destructive.

### Silently changing the scope tier

If the orchestrator promotes from Standard to Deep mid-review, it must announce: `code-review: scope promoted to DEEP — <one-line reason>`. Silent promotion confuses the user about what was actually reviewed.

### Stuffing every lens into every diff

A 10-line README change does not need a security reviewer. A schema migration does not need a "does this match convention" reviewer if there's no code style to compare against. The scope-tier rubric exists — follow it.

Over-dispatching reviewers burns context, time, and produces dilute noise. Each reviewer's return must be justified by the diff's shape.

---

## Author-facing anti-patterns (for the final artifact)

### The artifact that reads as prose, not as a checklist

`CODE-REVIEW.md` is read by authors deciding *what to fix before merging*. Every finding must be scannable:

- Severity prefix visible.
- Title is a one-liner.
- File:line clickable.
- Fix is concrete enough to act on in under 2 minutes of reading.

A review that reads like an essay is a review the author will not act on.

### Surfacing 40 findings with no prioritization

The orchestrator groups findings by severity (P0 → P1 → P2 → P3) AND by category (Blockers / Major / Minor / Nits). Within each, cross-reviewer-agreement findings float to the top. Without grouping, 40 findings in the order reviewers produced them is a dump, not a review.

### Hiding the positives

If a reviewer says the test coverage is good, that observation goes in the **Positives** section of the artifact — not dropped. Authors need to know what is right so they do not regress it on iteration.

### Hiding the residual risks

A reviewer that *considered* a concern but could not confirm it should still surface it in **Residual Risks**. Silently dropping it because "no finding fires" is worse than naming the gap.

### Missing the "what to run next"

The artifact ends with a **Next steps** section: fix the P0s, consider the P1s, triage the rest, optionally re-run `/code-review` after fixes. Without this, the author stares at 30 findings and asks "now what."

---

## Red-flag self-check (for the orchestrator)

STOP and reconsider if:

- You produced ≥ 30 findings in Lightweight scope — noise, recalibrate.
- You produced 0 findings in Deep scope — reviewers under-scrutinized, re-dispatch.
- Every finding is P3 — calibration is off, or the diff is genuinely trivial (then why Deep?).
- Every finding is P0 — calibration is off, or the diff is genuinely broken (then stop reviewing and tell the user to re-plan).
- The Pre-existing table is larger than the main findings table — the diff may not be the right scope to review in isolation.
- Adversarial has the same findings as the other reviewers — Stage 3 was not fed Stage 2's output, re-dispatch.
- You wrote `CODE-REVIEW.md` before the user approved Phase 8 — broken protocol, disclose.
- A reviewer's `findings` JSON is malformed or missing required fields — merge will corrupt. Ask that reviewer to re-run with the schema spec in hand.
- You are about to auto-apply a `safe_auto` fix — this skill is PURE REPORTER. No fixes. Never.

Pass all → proceed. Fail any → loop back.
