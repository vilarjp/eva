# Scope tiers — pr-feedback

Classify the triage ceremony by the density and contentiousness of reviewer feedback, plus the surface area of the change. Provisional at Phase 0.4, refined at Phase 2.6 once real comment counts are available.

## Inputs

- `total_comments` — inline thread comments + timeline issue comments + review-body summaries, post-dedup, post-dismissed-filter.
- `reviewer_count` — distinct `author.login` across all non-dismissed reviews.
- `review_decision` — `APPROVED | CHANGES_REQUESTED | REVIEW_REQUIRED | null`.
- `diff_loc` — lines added + deleted across the PR (`gh pr diff --stat`).
- `sensitive_surface` — diff touches any of: `src/auth*`, `src/payments*`, `migrations/*`, `src/api/public*`, concurrency primitives, crypto / JWT / hash code, webhook handlers, new database indices, schema changes, cron / queue consumers.
- `multi_reviewer_disagreement` — two or more reviewers' comments on the same `path:line` carry contradictory asks (detected at Phase 4.3).

## Tier criteria

| Tier | All of | Ceremony |
|------|--------|----------|
| **Lightweight** | `total_comments <= 5` AND `reviewer_count == 1` AND `review_decision != CHANGES_REQUESTED` AND `diff_loc < 100` AND NOT `sensitive_surface`. | Fast path. Spec cross-reference optional if no plan folder exists. Reconciliation with CODE-REVIEW.md: fingerprint-match only; skip tension surfacing unless ≥1 contradiction fires. Reply-draft tone: 1-2 sentences. Residual-risks section may be "(none)". |
| **Standard** | `total_comments 6-20` OR `reviewer_count >= 2` OR `review_decision == CHANGES_REQUESTED` OR `diff_loc 100-500`. | Full pipeline. Spec cross-reference required if a plan folder exists. CODE-REVIEW.md reconciliation full (agreement + tension). Reply drafts full-voice (3-5 sentences when citing spec). |
| **Deep** | `total_comments > 20` OR `multi_reviewer_disagreement` OR `sensitive_surface` OR `diff_loc > 500`. | Full pipeline + mandatory Residual Risks section with every ambiguous anchor listed. Reconciliation must quote CODE-REVIEW.md entries by `F-N`. Reply drafts include SHA-pinned `file:line` permalinks. Cross-reviewer conflict gets its own Reconciliation subsection. |

## Escalation signals (up only)

Escalate the tier *upward* if any of these surface during Phase 4:

- Two or more reviewers disagree about the same `path:line` → escalate to at least **Standard**.
- A reviewer cites a spec §, PRD §, or ADR → escalate to at least **Standard** (reply drafts need spec anchors).
- A reviewer marks a comment `(blocking)` via Conventional Comments → escalate to at least **Standard**.
- ≥3 `push-back` verdicts emerge → escalate to **Deep** (push-back requires rigorous evidence and spec anchors).
- Reviewer flags a concern touching `sensitive_surface` even if the diff didn't originally → escalate to **Deep**.

Announce any mid-pipeline re-escalation: `pr-feedback: scope escalated <OLD> → <NEW> (<reason>)`. Do not de-escalate once escalated within a single run.

## De-escalation (Phase 2.6 only)

Phase 2.6 may de-escalate the *provisional* tier if real counts don't justify it. Example: provisional was **Deep** because the diff touches `src/auth/`, but all comments are `APPROVED` / `praise` / `nice-to-have` — de-escalate to **Standard** with announcement.

Do not de-escalate after Phase 4 has produced triage records — a record assigned Deep confidence factors carries them forward.

## Ceremony mapping — what each tier actually changes

| Pipeline step | Lightweight | Standard | Deep |
|---------------|-------------|----------|------|
| Spec cross-reference (Phase 3.2) | Optional | Required if plan exists | Required, quoted verbatim |
| CODE-REVIEW.md reconciliation (Phase 3.3) | Fingerprint match only | Agreement + tension surfaced | Agreement + tension + every F-N quoted |
| Re-anchoring stale comments (Phase 4.1.2) | Best-effort | Required | Required + residual-risk entry on fail |
| Reply-draft length | 1-2 sentences | 3-5 sentences with citations | 3-5 sentences + SHA-pinned permalinks |
| Residual Risks section | Optional ("(none)" ok) | Required if ≥1 ambiguity | Required with every ambiguity enumerated |
| Handoff records | One line per must-fix | Structured block per must-fix | Structured block + suggestion_block verbatim |

## Anti-signals (DO trigger these as escalation, NOT as suppression)

- A single-reviewer PR with 40+ nitpick comments → still **Deep** via `total_comments`. Volume is a signal even when density is low.
- A `CHANGES_REQUESTED` PR with only 2 comments → **Standard** (state trumps count). The reviewer deliberately chose to block merge.
- A PR touching `migrations/*` with zero reviewer comments → refuse at Phase 0 (no feedback to triage). Do not synthesize.
