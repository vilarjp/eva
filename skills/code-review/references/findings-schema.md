# Findings Schema — Shared Contract for Code Review Reviewers

Every reviewer agent dispatched by the `code-review` skill returns findings in the same JSON-shaped structure. The orchestrator merges, deduplicates, and suppresses findings based on this shape. Deviations break the merge pipeline.

A reviewer's output is TWO fenced blocks:

1. A `findings` fenced code block (language tag `json`) — the machine-readable payload the orchestrator parses.
2. A short `## Notes` narrative — human-readable context the reviewer wants to surface (e.g., "ran grep for X, found 3 matches, none material"). Optional but encouraged.

The orchestrator consumes the JSON. Prose outside the JSON block is advisory, not authoritative.

---

## Schema

```jsonc
{
  "reviewer": "code-quality | convention | test | plan-alignment | security | performance | adversarial",
  "scope_observed": "uncommitted | branch-vs-base | head-1",
  "files_reviewed": ["<path/a>", "<path/b>"],
  "findings": [
    {
      "id": "<reviewer-short>-<n>",
      "severity": "P0 | P1 | P2 | P3",
      "confidence": 0.0,
      "title": "<one-line — no severity prefix>",
      "file": "<path/relative/to/repo-root>",
      "line": 0,
      "intent": "<file/line-independent description of the problem class>",
      "evidence": [
        "<5–30 word verbatim quote from the diff or file>",
        "<optional: second quote, other file>"
      ],
      "why_it_matters": "<one sentence — risk or cost if unfixed>",
      "suggested_fix": "<one or two sentences — minimal; no new scope>",
      "autofix_class": "safe_auto | gated_auto | manual | advisory",
      "pre_existing": false,
      "requires_verification": false,
      "verifies_claim": null,
      "claim_mismatch": false
    }
  ],
  "positives": [
    "<one-line observation the reviewer wants to call out positively>"
  ],
  "residual_risks": [
    "<one-line risk the reviewer considered but could not confirm>"
  ],
  "testing_gaps": [
    "<one-line — a scenario the tests under review do not cover>"
  ]
}
```

---

## Field definitions

### `reviewer`
The dispatching agent name, without the `code-review-` prefix. Used for attribution in the merged artifact.

### `scope_observed`
What slice of code the reviewer actually looked at. Must match the scope the orchestrator told it to look at; if the reviewer had to widen (e.g., following a call chain into an untouched file), mark it here so the orchestrator knows why.

### `files_reviewed`
Every path the reviewer opened. Used for auditability and for the orchestrator's "files in diff but reviewed by no one" check.

### `id`
Reviewer-local ID. Format: `<short>-<n>` where `<short>` is one of `cq` (code-quality), `cv` (convention), `tt` (test), `pa` (plan-alignment), `sc` (security), `pf` (performance), `av` (adversarial). Examples: `cq-1`, `sc-3`. The orchestrator renumbers these into `F-1`, `F-2`, … during the merge.

### `severity`
Four tiers. Calibrate against the rubric in `references/scope-tiers.md#severity-rubric`.

- **P0 (Blocker)** — bug, data loss, security breach, regression, violates stated plan, failing test claim. Ship is blocked.
- **P1 (Major)** — real defect or design problem; fixable without replanning; would be caught in a thorough human review. Ship should not proceed until addressed or explicitly accepted.
- **P2 (Minor)** — quality, clarity, small convention violation, missing small test. Worth fixing; not shipping-blocking.
- **P3 (Nit)** — style, naming, formatting, preference. Author may decline.

### `confidence`
A float in `[0.0, 1.0]` expressing how certain the reviewer is that this is a real problem.

- `≥ 0.80` — verified (read the code, traced the path, matches a known bad pattern).
- `0.60–0.79` — reasoned (consistent with the diff + conventions, but one step of inference).
- `0.40–0.59` — speculative (plausible; needs verification).
- `< 0.40` — hunch.

**Suppression thresholds** (applied by the orchestrator):

- Findings with `confidence < 0.60` are moved to the **Suppressed** table — visible but not top-surface.
- **P0 exception:** a `P0` finding with `confidence ≥ 0.50` stays in the main table (a blocker is worth naming even if uncertain).
- **Cross-reviewer agreement boost:** if two reviewers flag the same problem (same fingerprint), the merged finding's confidence is boosted by `+0.10` per additional reviewer, capped at `1.00`. This can lift a finding out of the Suppressed table.

### `title`
One line. Descriptive. No severity prefix, no reviewer name. Example: *"`applyCoupon` dereferences `cart.items` before the promise resolves"* — not *"P1: apply coupon bug"*.

### `file`
Path relative to the repo root. If the finding is about a diff-wide or project-wide concern with no single file anchor, use `"<cross-cutting>"`.

### `line`
Line number in the **current file state** (post-diff, as it would be committed). Use `0` when the finding is cross-cutting or when the file no longer contains the anchor (e.g., "missing guard" at a caller site — use the caller's line).

### `intent`
A file/line-independent description of the problem class. Used for deduplication across reviewers who may anchor the same issue to different lines.

Good: *"unchecked `null` on `cart.items` before property access"*. Bad: *"bug on line 62"*.

### `evidence`
An array of 1+ verbatim quotes from the diff, the file, or a referenced plan document. 5–30 words each. No paraphrase. No "the code does X". Quote the words.

For plan-alignment findings, evidence can cite the plan document with section reference: `"SPEC §D-2: 'all coupon applications must be idempotent'"`.

### `why_it_matters`
ONE sentence. Names the concrete risk or cost. Must NOT say "this is bad practice" — say *what bad thing happens if unfixed*.

Good: *"If `cart.items` is ever `null` on first render, the coupon page crashes instead of showing an empty-cart message."*
Bad: *"This violates defensive-programming principles."*

### `suggested_fix`
One or two sentences. Minimal. Names the concrete change. Must NOT propose new features, new abstractions, or "while you're here" cleanup. If the right fix needs context the reviewer doesn't have, say *"defer to author — need to know X"* and drop confidence accordingly.

### `autofix_class`
How the orchestrator (or the author) should treat this finding if they want to auto-apply fixes:

- **`safe_auto`** — mechanical, deterministic, no judgment required (missing import, wrong type annotation, trailing whitespace). The fix is the finding.
- **`gated_auto`** — the fix is mechanical but the *decision to apply* needs review (rename a symbol, switch a lib call to its typed variant). Auto-suggest; don't auto-commit.
- **`manual`** — requires human judgment, design choice, or project knowledge. Majority of real findings.
- **`advisory`** — no fix proposed; the finding is "watch this" or "consider the alternative" or a residual risk. Must not have a `suggested_fix`.

### `pre_existing`
`true` if the problematic code was **not modified in the diff under review**. The diff only surrounds / touches / happens-near the pre-existing problem.

Pre-existing findings are reported (visibility matters) but the orchestrator moves them to a separate **Pre-existing** table in the output, so the author is not blamed for code they did not write in this diff.

### `requires_verification`
`true` if the reviewer could not fully verify the finding with the tools available (e.g., would need to run the code, check a DB schema, or observe production behavior). The orchestrator surfaces these in a **Verification needed** callout so the author knows where to look.

### `verifies_claim`
Optional. Set by the orchestrator (never by a reviewer agent). On re-review, when a current finding's fingerprint matches a prior claim parsed from a `## Fixes applied <DATE>` section, this field is set to the prior finding's ID (e.g., `"F-2"`). Used to link verification-table rows back to the specific current finding.

- `null` (or omitted) — not a verification-linked finding.
- `"F-N"` — this finding corresponds to the prior claim F-N (either an addressed-but-still-present mismatch, or a skipped-and-still-present expected match).

Reviewer agents MUST leave this field as `null` / omitted. The orchestrator populates it at Phase 6.8.

### `claim_mismatch`
Optional. Set by the orchestrator at Phase 6.8. `true` when a current finding matches the fingerprint of a prior-claimed-addressed finding (the prior review said "fixed", the current review disagrees). Mismatched findings are never suppressed — they bypass the `confidence < 0.60` gate and surface at their severity regardless.

- `false` (default) — not a mismatch (or not applicable).
- `true` — mismatch confirmed. Render in the artifact with an inline marker, e.g., *"(claim mismatch — F-2 was marked addressed at commit `def5678` but the same pattern reproduces here)"*.

A `claim_mismatch: true` finding also receives a `+0.15` confidence boost (capped at 1.0) because cross-revision recurrence is strong corroboration.

---

## Deduplication (orchestrator-side)

When the orchestrator merges findings from multiple reviewers, it computes a **fingerprint** per finding:

```
fingerprint = hash(normalize(file) + line_bucket(line, ±3) + normalize(intent))
```

- `normalize(file)` — strip `./`, lowercase, unify separators.
- `line_bucket(line, ±3)` — group by 7-line windows so two reviewers pointing at line 62 and line 64 of the same `intent` merge.
- `normalize(intent)` — lowercase, strip leading articles, collapse whitespace.

When two findings share a fingerprint:

1. Keep the title + suggested_fix from the HIGHER-confidence reviewer (ties broken by reviewer priority: plan-alignment > security > code-quality > convention > test > performance > adversarial).
2. Merge the `evidence` arrays (unique quotes only).
3. Set the merged `confidence` to `min(1.0, max(confidences) + 0.10 * (reviewers_count - 1))`.
4. Record `seen_by: ["<reviewer-a>", "<reviewer-b>"]` in the merged finding — the main artifact shows this as a **cross-reviewer agreement** badge.

---

## Empty and clean returns

A reviewer with **nothing to say** returns the full schema with empty arrays:

```jsonc
{
  "reviewer": "security",
  "scope_observed": "uncommitted",
  "files_reviewed": ["src/checkout/coupon.ts", "src/checkout/coupon.test.ts"],
  "findings": [],
  "positives": ["All user input is validated with the existing Zod schema at src/checkout/types.ts:18."],
  "residual_risks": [],
  "testing_gaps": []
}
```

This is a valid, expected result. The orchestrator records the clean pass in the CODE-REVIEW.md artifact as *"<reviewer>: clean — <one-line why>"*. Padded findings are worse than a clean pass.

---

## Anti-patterns (schema violations)

- **Severity prefix in title** (`"P1: unchecked null"`) — severity is its own field. Duplication is noise.
- **Confidence set to a default value** (always `0.8`, always `1.0`) — calibrate honestly. Uncalibrated confidence is worse than none.
- **Evidence paraphrased** (`"the code doesn't check for null"`) — quote the actual source.
- **`suggested_fix` proposes new scope** (`"extract a new `CouponValidator` class"`) — reviewers flag; they do not redesign. Drop the finding or lower confidence and describe the gap.
- **`pre_existing: false` on untouched lines** — be honest. If the diff just *passes by* a problem, it is pre-existing.
- **`advisory` with a concrete fix** — either the fix is real (`manual`/`gated_auto`/`safe_auto`) or it is advice. Do not blur.
- **Findings that duplicate another lens's territory** — plan-alignment does not flag naming drift; convention does not flag security holes. Respect the reviewer's lane.

---

## Example finding (well-formed)

```json
{
  "reviewer": "code-quality",
  "scope_observed": "uncommitted",
  "files_reviewed": ["src/checkout/coupon.ts", "src/checkout/coupon.test.ts"],
  "findings": [
    {
      "id": "cq-1",
      "severity": "P1",
      "confidence": 0.82,
      "title": "`applyCoupon` dereferences `cart.items` before the loading promise resolves",
      "file": "src/checkout/coupon.ts",
      "line": 62,
      "intent": "unchecked property access on cart.items before cart state has loaded",
      "evidence": [
        "const count = cart.items.length; // src/checkout/coupon.ts:62",
        "if (cart.status === 'loading') return; // src/checkout/coupon.ts:40 — guard exists upstream but is not enforced at call site"
      ],
      "why_it_matters": "When the user clicks Apply Coupon during initial cart load, the handler crashes instead of showing the empty-cart message, surfacing a blank error toast.",
      "suggested_fix": "Guard at the top of `applyCoupon`: `if (cart.status !== 'ready') return;` — mirrors the existing guard at line 40 in the cart-update path.",
      "autofix_class": "manual",
      "pre_existing": false,
      "requires_verification": false
    }
  ],
  "positives": [
    "The new Zod schema for `CouponRequest` is consistent with the existing validation pattern in src/checkout/types.ts."
  ],
  "residual_risks": [
    "Intermittent failures on slow network where `cart.status` transitions through `loading → ready → loading` — not observable without integration test."
  ],
  "testing_gaps": [
    "No test covers the `applyCoupon` call during the `loading` state."
  ]
}
```

That is the contract. A reviewer that returns this shape merges cleanly. A reviewer that returns prose, a markdown table, or half the fields does not.
