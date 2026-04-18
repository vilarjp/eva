# Verdict taxonomy — pr-feedback

Five author-facing buckets. Each verdict is derived from: Conventional Comments label + decorator (if present in the reviewer's body), spec cross-reference, prior CODE-REVIEW.md match, anchor freshness, and code-level corroboration.

## The five buckets

### 1. must-fix

Reviewer is correct; the code must change before merge.

**Positive signals (any of):**
- `(blocking)` decorator on the comment.
- Conventional `issue` or `todo` label without a `(non-blocking)` decorator.
- Review state is `CHANGES_REQUESTED` AND the comment carries a specific, actionable code ask (not just a question).
- Reviewer cites a real bug, missing input validation, spec-invariant break, security risk, data-loss risk, or a test that will fail.
- Reviewer's diagnosis is factually correct when checked against HEAD.

**Confidence factors (additive, floor 50, cap 100):**
- `+20` if review state is `CHANGES_REQUESTED`.
- `+15` if the comment explicitly cites a spec §, ADR, or PRD § we can verify.
- `+15` if a prior `CODE-REVIEW.md` finding corroborates (same fingerprint).
- `+10` if the reviewer supplied a ```suggestion block and it parses cleanly.
- `-15` if the comment is outdated / anchor moved and we re-anchored by text search.
- `-25` if the comment references code that no longer exists and we couldn't re-anchor.

**Reply tone:** acknowledging, specific, commits to a fix. Cite `file:line`. If applying a suggestion block verbatim, say so.

**Example reply:**
> Good catch — you're right that the retry path bypasses the rate limiter. Pushing a fix to route retries through `rateLimiter.acquire()` (see `src/http/retry.ts:48`). Will update when it lands.

---

### 2. nice-to-have

Valid observation but non-blocking.

**Positive signals:**
- `(non-blocking)` or `(if-minor)` decorator.
- Conventional `suggestion` / `nitpick` / `thought` / `note` / `praise` / `chore` labels.
- Body begins with `nit:` or `style:` prefix.
- Style, consistency, polish — does not violate any spec invariant.
- Reviewer phrases as preference ("I'd prefer...", "consider...").

**Confidence factors:**
- `+15` if the comment is explicitly labeled `(non-blocking)` or starts with `nit:`.
- `+10` if the suggestion is an improvement but the current code passes all tests and spec.
- `-10` if the comment sounds like a preference but actually cites a spec invariant (likely misclassified; may be must-fix).

**Reply tone:** appreciative, non-committal unless the author wants to apply. If declining, explain briefly — do not apologize.

**Example reply (accepting):**
> Good point — will apply in the next push. Thanks.

**Example reply (declining):**
> Thanks — I'll skip this one to keep the diff focused on the core change. Happy to revisit in a follow-up if it bothers you.

---

### 3. question

Reviewer wants information, not a code change.

**Positive signals:**
- Interrogative syntax ("is this intentional?", "why did you...", "wouldn't it be clearer to...", body ends with `?`).
- Conventional `question` label.
- Comment asks about intent, trade-off, or alternatives without demanding a change.

**Confidence factors:**
- `+20` if the comment is grammatically a question.
- `+15` if a plan doc (PRD / SPEC / ADR) answers the question — include the quote in the reply.
- `+10` if the question is open-ended and genuinely needs author judgment (not a "why didn't you use X" with X being worse).
- `-10` if the comment is a question in form but a demand in substance (likely must-fix or nice-to-have).

**Reply tone:** direct, cite the spec if it answers the question, offer to add a code comment if the question is likely to recur.

**Example reply:**
> Yes — SPEC §3.2 specifies the coupon is applied before tax for tax-inclusive regions. I'll add an inline comment here so the next reader doesn't have to dig.

---

### 4. push-back

Reviewer is wrong, OR the suggested change would violate the spec, cause a regression, or break a test.

**Positive signals:**
- Reviewer's diagnosis contains a factual error about code behavior, API contract, or data flow.
- A plan doc (PRD / SPEC / ADR) documents the opposite choice with stated rationale.
- The suggested change would break a specific named test that's currently passing.
- The suggestion contradicts a prior `CODE-REVIEW.md` decision that the team already deliberated.

**Confidence factors:**
- `+20` if a spec § explicitly documents the opposite choice.
- `+20` if a prior `CODE-REVIEW.md` entry resolved this exact concern.
- `+15` if the reviewer's claim is falsifiable by reading the diff (they say "this doesn't handle X" and the diff shows it does).
- `+10` if the suggestion would break a specific named test.
- `-20` if the disagreement is a matter of preference without a documentable anchor (spec, test, ADR) — reclassify to `nice-to-have` if confidence stays under 60.

**Reply tone:** respectful, specific, quote the spec § or test name, offer to discuss in a sync if the reviewer still disagrees. Never condescending.

**Example reply:**
> Looked at this — SPEC §2.4 documents that we accept both email and username (product scope decision, see PRD §1.3). The alternative you're suggesting would break `auth.test.ts:42`, which asserts username-only login still works. Open to discussing in a sync if you want to revisit the scope decision.

---

### 5. already-done

Concern was already addressed — either in a later commit on this branch, or the comment anchors to code that has since been removed or rewritten.

**Positive signals:**
- `is_outdated == true` AND re-anchoring finds the concern is gone in the new code.
- A later commit on the branch (after `original_commit_id`) contains the fix.
- Comment references code the diff no longer contains (successful re-anchor returns "removed").
- PR author already replied with a commit SHA.

**Confidence factors:**
- `+25` if we can identify the specific commit SHA that addressed it.
- `+15` if a later commit message matches the reviewer's ask.
- `+10` if the anchor line's current content clearly differs from the quoted `diff_hunk`.
- `-20` if the anchor is stale but we can't confirm what was originally there.

**Reply tone:** short; point to the commit or new line. No justification needed — the code speaks.

**Example reply:**
> Addressed in `<commit-sha>` — see `src/checkout/coupon.ts:52`.

---

## Conventional Comments mapping

The [Conventional Comments](https://conventionalcomments.org/) spec uses labels + decorators:

| Conventional label | Default bucket | With `(blocking)` | With `(non-blocking)` / `(if-minor)` |
|--------------------|----------------|-------------------|---------------------------------------|
| `praise` | nice-to-have | — (nonsensical) | nice-to-have |
| `nitpick` | nice-to-have | must-fix | nice-to-have |
| `suggestion` | nice-to-have | must-fix | nice-to-have |
| `issue` | must-fix | must-fix | nice-to-have |
| `todo` | must-fix | must-fix | nice-to-have |
| `question` | question | question | question |
| `thought` | nice-to-have | nice-to-have | nice-to-have |
| `chore` | nice-to-have | must-fix | nice-to-have |
| `note` | nice-to-have | nice-to-have | nice-to-have |

When a comment has no Conventional label, classify by content using the positive signals above.

## Suppression

Any verdict with `confidence < 60` moves to the Suppressed table, EXCEPT `must-fix` with `confidence >= 50` (keep top-bar so critical concerns surface under ambiguity). Suppressed verdicts are still rendered — with confidence visible — never dropped.

## Stale / outdated handling

Outdated comments (`is_outdated == true` OR REST `line == null`) get a "line moved" badge regardless of bucket. Inside each bucket, they sort last.

If re-anchoring (text-searching `diff_hunk` against the current tree) succeeds, use the new `file:line` for the handoff record and note `re_anchored_from: <original-path>:<original-line>` in the record.

If re-anchoring fails:
- Keep the original anchor for reference in the record.
- Set the handoff `file: "<unknown>"` (must-fix will not auto-hand off without a file).
- List the record in the Residual Risks section.

## Reply-tone conventions

Across all buckets:

- Write as the PR author, first-person ("I'll", "I looked at this"). Never "the author should".
- Cite `file:line` or spec § when the reply references code.
- Do not apologize for push-back. Do not grovel on nice-to-have.
- Do not add "Thanks for the review!" to every reply — it becomes noise. Use it once in a top-level response to the review body if at all.
- Do not include "[reviewer's name]" placeholders — replies are posted as thread replies; GitHub resolves attribution.
- For must-fix with a known commit that will fix it: commit to a timeline ("will land in next push", "pushing now") only if you know the author actually intends to. Otherwise say "will update when it lands."

## Forbidden performative phrases

A reply draft must communicate what the author will do (or won't do) and why — it is not a forum for praising the reviewer or performing agreement. The following phrases and their close variants **never** appear in a generated reply. They read as insincere at best and sycophantic at worst, and they displace the one line that actually answers the comment.

| Forbidden phrase (and close variants) | Why it's forbidden | What to write instead |
|---|---|---|
| *"You're absolutely right"* · *"You're 100% right"* · *"Totally right"* · *"Great catch"* · *"Excellent point"* · *"Fantastic feedback"* | Praises the reviewer instead of answering. Performs agreement rather than demonstrating it. | State the concrete action: *"Fixed — routed retries through `rateLimiter.acquire()` at `src/http/retry.ts:48`."* |
| *"Thanks for the review"* (in a per-comment reply) · *"Thanks for catching this"* · *"Appreciate the thorough review"* · *"Thanks so much"* | Noise per-comment. Gratitude openers dilute the reply and imply the reviewer needs reassurance. | Use `"Thanks — "` at most **once**, in a single top-level response to the review body, not on individual threads. Most per-thread replies need no thanks at all. |
| *"Good point"* · *"Fair point"* · *"That makes sense"* (as an opener) | Ceremony; adds zero information. The reviewer already knows their point is a point. | Skip the opener. Lead with the answer: *"Agreed — SPEC §3.2 specifies this ordering. Adding the inline comment now."* |
| *"I really appreciate …"* · *"I'm so glad you brought this up"* | Emotional performance. Reviewers are doing their job; the reply is not a thank-you card. | Delete. If genuine warmth is warranted (a reviewer went far beyond scope), one sentence at the top of the review-body response covers it. |
| *"Sorry for the confusion"* · *"Apologies for …"* · *"My bad"* (when explaining a design, not an actual mistake) | Unnecessary apology for a deliberate choice — signals the design is indefensible even when it isn't. | For push-back, **defend the choice**. For must-fix, **state the fix**. Apologise only for real mistakes (e.g. a committed secret), and make the apology one line. |
| *"I hope this makes sense"* · *"Let me know if this doesn't answer your question"* · *"Happy to discuss further if you have questions"* (as a boilerplate tail) | Filler tail. If the reply is clear, it doesn't need a hedge; if it isn't, the hedge doesn't fix it. | Drop the tail. The reviewer can reply if they disagree — that is what the thread is for. |
| *"I'll take a look and get back to you"* (on a must-fix) · *"Let me think about this"* | Vague. A must-fix reply needs a specific action or a specific acknowledgement with a timeline. | *"Fixing — will update this thread when the commit lands."* |

A reply containing any of these phrases **must** be regenerated before being put in the artifact. The rule is mechanical, not stylistic: even if the phrase "fits" in isolation, its presence signals the draft was optimising for politeness over information.

Exception: when the user's repo has an explicit reviewer-etiquette convention (e.g. `CONTRIBUTING.md` mandates "lead with thanks"), honour it. Otherwise, default to the list above.
