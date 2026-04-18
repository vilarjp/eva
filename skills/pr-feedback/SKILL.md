---
name: pr-feedback
description: "Triage incoming PR review feedback through a structured intake pipeline — fetches all inline review-thread comments (GraphQL), timeline issue comments, and top-level review bodies for a given GitHub PR, cross-references the current diff against any adjacent docs/YYYY-MM-DD-<slug>/ spec folder (PRD.md / SPEC.md / DIAGNOSIS.md / EXECUTION.md) plus an optional sibling CODE-REVIEW.md for agreement/tension reconciliation, then classifies each comment into five verdict buckets — must-fix, nice-to-have, question, push-back, already-done — with confidence scoring, verbatim evidence, a copy-pasteable GitHub reply, and a handoff record that /execute FIX mode can consume. Surfaces stale/outdated comments with a line-moved badge (never silently dropped), includes resolved threads collapsed for context, unions multi-reviewer feedback deduped by thread_id, and skips only explicitly-dismissed reviews (logged, not hidden). Use when the user wants to process a PR review, says 'triage this review', 'handle PR feedback', 'go through the comments on my PR', 'address the review', 'I got review comments', pastes a GitHub PR URL or `owner/repo#N` reference, or invokes /pr-feedback with a PR link. Produces docs/YYYY-MM-DD-<slug>/PR-FEEDBACK.md with severity-grouped verdicts, reconciliation notes, residual risks, and a handoff section. Mandatory branch-checkout gate (offers to run gh pr checkout for the user first; refuses-until-correct with the exact manual commands if auto-checkout declines or fails). Smart-reuses an adjacent spec folder when the PR maps to a known feature; on re-triage appends a ## Re-triage <DATE> section. Scope-adaptive (Lightweight / Standard / Deep), self-review checklist, HARD GATE on write. Pure triage — never posts to the PR, never edits source, never commits, never invokes /execute. Hands off to /execute FIX mode for the actual fixes."
argument-hint: "[pr-url, owner/repo#N, or bare #N in a matching repo, optionally followed by scope override (lightweight|standard|deep) or --force-state]"
---

# pr-feedback — Triage reviewer comments, never generate them

Turn a PR's review feedback — inline comments, timeline comments, review bodies across every non-dismissed reviewer — into a single cold-readable artifact: `docs/YYYY-MM-DD-<slug>/PR-FEEDBACK.md`. For each comment: a verdict (must-fix / nice-to-have / question / push-back / already-done), a 0-100 confidence score, verbatim evidence from the diff or spec, a copy-pasteable GitHub reply, and a handoff record that `/execute` FIX mode can consume directly.

This skill does NOT post to the PR. It does NOT edit source code. It reads, triages, and reports. Applying the fixes is a separate turn, via `/execute` in FIX mode.

## When to use

Trigger this skill when the user:
- says "triage this review", "handle PR feedback", "go through the review comments", "address the review", "I got review comments on my PR", "triage these comments"
- pastes a GitHub PR URL, an `owner/repo#N` reference, or a bare `#N` when the cwd's remote matches
- invokes `/pr-feedback` (with a PR reference)

Do NOT trigger for:
- Generating a review on a diff (that's `/code-review`)
- Applying the fixes (that's `/execute` in FIX mode, after this skill completes)
- Creating or updating a PR (out of scope — this skill assumes the PR exists)
- Replying to the PR automatically (out of scope — the artifact is copy-pasteable; the author posts replies themselves)
- Non-GitHub platforms (GitLab, Bitbucket — out of scope for v1)

## The Iron Law

**Pure triage. No PR posts. No code edits. Artifact only.**

The skill reads the PR feedback, classifies it, and writes one file. It does not `gh pr comment`, it does not `gh pr review`, it does not `git add` / `commit` / `push`, it does not invoke `/execute`. Copy-pasteable replies live inside the artifact — the human decides when and whether to post them. Crossing any of these lines is a protocol violation.

## Core invariants

1. **Pure triage.** No PR posts, no inline review replies, no source edits, no commits. Only `PR-FEEDBACK.md`.
2. **Mandatory branch-checkout gate.** The working tree MUST be on the PR's head commit before triage proceeds. First offer: ask to run `gh pr checkout <N>` (handles forks and cross-repo PRs cleanly). On decline or failure: refuse with the exact manual commands, stop, wait for the user to re-invoke.
3. **PR state gate.** `MERGED` or `CLOSED` PRs: refuse unless the user passes `--force-state`. `isDraft`: warn and confirm before proceeding. `OPEN`: proceed.
4. **Union non-dismissed reviews, dedupe by thread_id.** A PR may accumulate many reviews; the triage covers all of them. `DISMISSED` reviews are explicitly withdrawn — skip them, but log the skip.
5. **No silent drops.** Stale/outdated comments get a "line moved" badge. Resolved threads are included in a collapsed `<details>` block. Dismissed reviews are listed in a Skipped footer, never hidden.
6. **Verbatim evidence for every verdict.** 5-30 word quote from the comment itself, the diff, the spec, or a prior self-review. Paraphrase is suppressed.
7. **Five verdict buckets.** `must-fix` / `nice-to-have` / `question` / `push-back` / `already-done`. Derived from Conventional Comments labels + decorators + spec cross-reference + anchor freshness — see `references/taxonomy.md`.
8. **Confidence-gated suppression.** Verdicts with `confidence < 60` move to the Suppressed table, EXCEPT `must-fix` with `confidence >= 50` (keep top-bar so critical concerns surface even under ambiguity). Never silently dropped.
9. **Reconcile with adjacent CODE-REVIEW.md (if present).** Reviewer comments that match prior self-review findings strengthen the must-fix bucket (+15 confidence); tension — "reviewer flags X we suppressed" — surfaces under a Reconciliation section. Never mutates CODE-REVIEW.md.
10. **Smart folder reuse.** If the PR maps to an existing `docs/YYYY-MM-DD-<slug>/` folder by branch slug, title overlap, or diff-file overlap, write inside it. Otherwise derive a slug from the PR head branch and create a new dated folder.
11. **Emit the self-review checklist to the user before Gate.** Skipping is a protocol violation.
12. **HARD GATE: do not write `PR-FEEDBACK.md` until the user approves the verdicts and the output path.**
13. **Handoff record, not handoff call.** The artifact tells `/execute` what to apply and where; the skill never invokes `/execute` itself.
14. **Single-lane triage.** This skill runs inline (no sub-agent fan-out). The triage requires full shared context — diff + spec + prior review — and parallel dispatch would load the same context N times for no gain. If a reviewer's comment demands specialized analysis (security, perf), escalate tier and cite the plan — do not spawn a reviewer agent.

## Pre-flight (MANDATORY)

Before anything else:

1. Get today's date: `date +%Y-%m-%d` via Bash. Store as `<DATE>`.
2. If the invocation has no PR reference (bare `/pr-feedback`), ask: *"Which PR? Paste a GitHub URL, `owner/repo#N`, or a bare `#N` if the repo is obvious from the cwd."* Do not proceed without a target.
3. Parse the PR reference into `{owner, repo, number}` — patterns in `references/github-api.md`. If a bare `#N` was passed, resolve `owner/repo` from `gh repo view --json nameWithOwner`; refuse if the cwd has no GitHub remote.
4. If the parsed `owner/repo` does not match the cwd's remote (`gh repo view --json nameWithOwner -q .nameWithOwner`), STOP: *"This PR is in `<other-repo>` but your cwd is `<cwd-repo>`. Please `cd` into `<other-repo>` before re-invoking."* Do not attempt cross-repo triage.
5. Verify `gh auth status` succeeds. If it fails, stop and tell the user to run `gh auth login`.
6. Announce the skill is active: `pr-feedback: starting (PR <owner>/<repo>#<N>, scope TBD)`.

## Process

Execute phases in order. Do not skip.

### Phase 0 — Resume, folder strategy, scope

**0.1 PR metadata fetch (minimal, for folder strategy and state gate).** Run once:

```bash
gh pr view <N> --repo <owner>/<repo> --json number,title,state,isDraft,headRefName,headRefOid,headRepositoryOwner,baseRefName,author,body,url,reviewDecision,labels,mergeable
```

Store the JSON. Do not fetch comments yet (that's Phase 2).

**0.2 Resume check.** Look in `docs/*/` for an existing `PR-FEEDBACK.md` whose frontmatter `pr_url` matches this PR. If found, ASK:

```
Found existing PR-FEEDBACK.md at <path> for this PR. What would you like to do?
  A) Append — add a ## Re-triage <DATE> section below the existing triage [Recommended after new reviewer activity]
  B) Overwrite — start fresh (old triage preserved in git history)
  C) Write to a new file (PR-FEEDBACK-N.md) — preserve both
```

Default is A. Overwrites require explicit confirmation.

**0.3 Folder strategy (smart reuse).** Scan `docs/*/` for directories containing `PRD.md`, `SPEC.md`, `DIAGNOSIS.md`, `REVISION.md`, or `EXECUTION.md`. Score each candidate by:
- PR head-branch slug overlap with the folder slug (e.g., `feat/checkout-coupon` ↔ `2026-04-10-checkout-coupon`)
- PR title token overlap with the folder's plan `topic` frontmatter
- Files in the PR's diff overlapping with paths referenced in the plan

If one candidate clearly matches, ASK:

```
This PR appears to touch <feature> which has a spec folder at <path>.
  A) Add PR-FEEDBACK.md inside that folder [Recommended]
  B) Create a new dated folder (docs/<DATE>-<slug>/)
```

If A: set `spec_directory_reuse: true` and `original_spec_dir: <path>` in frontmatter. This folder is the write target. Any sibling `CODE-REVIEW.md` becomes a reconciliation input at Phase 3.

If no candidate matches, derive a slug from the PR head branch (drop `feat/` / `fix/` / `chore/` prefix, kebab-case, 2-4 words). Target: `docs/<DATE>-<slug>/`.

**0.4 Scope classification (provisional).** Using PR metadata only — comment counts will refine at Phase 2.6:

| Tier | Criteria | Ceremony |
|------|----------|----------|
| **Lightweight** | `reviewDecision != CHANGES_REQUESTED`, 1 reviewer (from review state listing), diff <100 LOC, no sensitive surface. | Fast path. Spec cross-reference optional if no plan folder. Reply-draft tone brief. |
| **Standard** | `reviewDecision == CHANGES_REQUESTED` OR 2+ reviewers OR diff 100-500 LOC. | Full pipeline. Spec cross-reference required if plan exists. |
| **Deep** | Diff 500+ LOC OR touches auth/payments/migrations/public APIs/concurrency/crypto OR multi-reviewer disagreement suspected. | Full pipeline + mandatory residual-risks section + CODE-REVIEW.md reconciliation must quote F-N. |

Provisional tier is announced as: `pr-feedback: scope = <TIER> (provisional, refined after fetch)`. Full rubric: `references/scope-tiers.md`.

### Phase 1 — PR state gate + branch checkout gate

**1.1 PR state gate.**

- `state == "MERGED"` or `state == "CLOSED"` → STOP unless the invocation carried `--force-state`. Message: *"This PR is <state>. Triaging feedback after merge/close is usually wasted work — the feedback is historical. If you still want to triage, re-invoke with `--force-state`."*
- `isDraft == true` → WARN via `AskUserQuestion`: *"This PR is a draft. Reviewers may still be iterating. Proceed anyway?"* (Yes / No).
- Otherwise proceed.

**1.2 Branch checkout gate (MANDATORY).** The working tree must be on the PR's head commit before triage proceeds. This guarantees `already-done` verdicts are verifiable, spec cross-references are accurate, and any stale-anchor re-anchoring reflects the real HEAD.

Inspect local state:

```bash
git branch --show-current
git rev-parse HEAD
git status --porcelain
```

Compare current branch + HEAD to the metadata's `headRefName` + `headRefOid`. Three paths:

**Path 1 — already on the correct branch AND HEAD matches `headRefOid`:** announce `pr-feedback: on PR head (<branch> @ <short-sha>), matches` and proceed to Phase 2.

**Path 2 — on the correct branch, HEAD differs from `headRefOid`:** user is ahead or behind. ASK:

```
You're on <branch> but HEAD is <short-sha-local>; PR head is <short-sha-remote>.
  A) Run `git fetch origin && git pull --ff-only` to sync with the PR's head [Recommended]
  B) Proceed anyway — triage against your current HEAD (already-done verdicts may differ)
  C) Abort
```

Default A. If A's fast-forward fails (divergent history, dirty tree), fall back to refuse-until-correct: print the exact recovery commands, stop, wait for re-invocation.

**Path 3 — on a different branch entirely (includes forks and first-time checkout):** offer-to-run first. ASK:

```
You're on <current-branch>; PR head is <pr-branch>.
  A) Run `gh pr checkout <N>` [Recommended — handles forks correctly]
  B) I'll do it myself — abort, I'll re-invoke after checkout
  C) Proceed on <current-branch> anyway — triage may be inaccurate
```

Default A. On A: run `gh pr checkout <N> --repo <owner>/<repo>`, verify HEAD matches `headRefOid`, announce. On any failure (dirty tree, fetch error, gh auth): fall back to refuse-until-correct with the exact manual commands printed.

Record the final branch + HEAD in artifact frontmatter as `branch_state_at_triage`. If the user chose B or C above and the local HEAD is not the PR head, add a warning banner to the artifact: `WARNING: triage produced against <current-branch> @ <short-sha>, not PR head — verdicts may be stale.`

### Phase 2 — Fetch and normalize

Fetch three comment sources and merge into a flat `Comment[]` array. Queries and fields: `references/github-api.md`.

**2.1 Inline review-thread comments.** Single GraphQL call via `gh api graphql` — the query in `references/github-api.md` returns every thread with `isResolved`, `isOutdated`, `isCollapsed`, `path`, `line`, `originalLine`, plus every comment in the thread (author, body, createdAt, `pullRequestReview.state`, `replyTo`, `diffHunk`).

Why GraphQL: `isResolved` is NOT exposed in REST. One call gets thread state + comment bodies + reply chains + review state.

**2.2 Timeline issue comments.** `gh api repos/<owner>/<repo>/issues/<N>/comments --paginate`. These are not anchored to files — treat as `source: "timeline"`, `path: null`, `line: null`.

**2.3 Review bodies.** `gh api repos/<owner>/<repo>/pulls/<N>/reviews --paginate`. Each review has `state` (APPROVED / CHANGES_REQUESTED / COMMENTED / DISMISSED / PENDING) and `body`. Treat as `source: "review-body"`, `path: null`, `line: null`.

**2.4 Normalize to Comment[].** For every comment across the three sources, produce:

```
{
  id,
  source: "inline-thread" | "timeline" | "review-body",
  review_id, review_state, pr_review_state,
  path, line, original_line, diff_side,
  body (raw markdown),
  author,
  created_at,
  thread_id, in_reply_to,
  is_resolved, is_outdated,
  suggestion_blocks: [...],   // parsed ```suggestion fences
  url, diff_hunk
}
```

**2.5 Filter dismissed + dedupe.**
- Drop any comment whose `review_state == "DISMISSED"`. Log the skip count by reviewer.
- Drop any `PENDING` review body (not yet published).
- Dedupe by `id` — a comment may appear in both the reviewThreads graph and the REST reviews listing.
- Group inline comments by `thread_id` so replies stay nested under the top-level comment.

**2.6 Confirm scope.** Recompute Phase 0.4 classification with real comment counts and reviewer counts:

- `total_comments` = inline + timeline + review-body (post-filter).
- `reviewer_count` = distinct authors across all non-dismissed reviews.

Apply full `references/scope-tiers.md` rubric. If the provisional tier was wrong, re-announce: `pr-feedback: scope = <TIER> (was <PROVISIONAL>, revised — <reason>)`.

### Phase 3 — Cross-reference context

Load in this order:

**3.1 PR diff.** `gh pr diff <N> --repo <owner>/<repo>`. Store as the canonical diff. Used for:
- Verifying `already-done` verdicts (was the line changed since the comment?).
- Identifying handoff `file:line` for must-fix records.
- Re-anchoring stale comments (text-search `diff_hunk` against the current diff).

**3.2 Adjacent plan documents** (only if `spec_directory_reuse: true`).

- `PRD.md` — product framing. Quote §s that justify a contested product decision for push-back / question verdicts.
- `SPEC.md` — architecture + ADRs. Quote invariants that a reviewer either corroborates (strengthens must-fix) or contradicts (strengthens push-back).
- `DIAGNOSIS.md` / `EXECUTION.md` — fix/build log. Scan for commit SHAs that could ground an `already-done` verdict.

**3.3 Adjacent CODE-REVIEW.md (reconciliation input, read-only).** If the chosen folder has a `CODE-REVIEW.md`, parse its severity-grouped findings into `prior_self_findings[]`:

```
{ id: "F-N", title, severity, file, line, fingerprint }
```

where `fingerprint = normalize(file) + line_bucket(line, ±3) + normalize(intent)` — matching `/code-review`'s shape so cross-skill fingerprints align. The skill NEVER edits `CODE-REVIEW.md`; reconciliation outcomes live only in the new `PR-FEEDBACK.md`.

**3.4 Announce grounding.** Emit:

```
pr-feedback: context loaded
- PR: <owner>/<repo>#<N> (<state>, <reviewDecision>)
- Comments: <total> (inline <n> / timeline <n> / review-body <n>; <n> outdated, <n> resolved, <n> dismissed-skipped)
- Reviewers: <@a>, <@b>, <@c>
- Diff: <N> files, +X/-Y LOC
- Spec dir: <path | "none — triaging against diff only">
- Prior CODE-REVIEW.md: <yes, N findings | no>
- Scope: <TIER>
```

Wait one beat for correction. If the user corrects the scope or target, adjust and re-emit.

### Phase 4 — Triage (single-lane, inline)

For each comment in `Comment[]`, produce a triage record. Single-lane — not multi-agent — because the triage needs shared context (diff + spec + prior review) and parallel dispatch would load the same context N times.

**4.1 Per-comment decision procedure.**

1. **Read the body end-to-end.** Parse any ```suggestion fences; store their content as `suggestion_blocks[]`.
2. **Anchor to current code.** If `is_outdated == true` OR `line == null`, mark `anchor_stale: true`; use `diff_hunk` + `original_line` to identify the original context, then text-search the current tree for the hunk snippet. If re-anchoring succeeds, record the new `file:line` and add `re_anchored_from`; if not, the record keeps the original anchor and lists under Residual Risks.
3. **Cross-reference spec.** If a plan file defines an invariant, requirement, or ADR that touches the comment's concern, locate a 5-30-word quote.
4. **Cross-reference prior CODE-REVIEW.md.** If any `prior_self_findings[i].fingerprint` matches, record `corroborates_finding: "F-N"` (agreement) or `contradicts_finding: "F-N"` (tension — prior self-review took the opposite stance).
5. **Classify.** Apply the five-bucket rubric from `references/taxonomy.md`. Each bucket has explicit signals and confidence factors.
6. **Score confidence (0-100)** per the factors in `references/taxonomy.md`. Floor 50, ceiling 100.
7. **Draft the copy-pasteable GitHub reply.** Tone varies by bucket (see `references/taxonomy.md`). Cite `file:line` or spec § where relevant. Exactly the message the author should post as a thread reply — no "[author]" placeholders.
8. **Build the handoff record** for `/execute` FIX mode, populated ONLY for `must-fix` verdicts (and the rare `push-back-with-counterfix` case):

```
{
  file, line,
  intent: "<one-line imperative — what to change>",
  suggestion_block: "<verbatim payload from a ```suggestion fence if present, else null>",
  verdict: "must-fix",
  evidence: "<quote from diff or spec>"
}
```

**4.2 Confidence suppression.** Move records with `confidence < 60` to the Suppressed table, EXCEPT `must-fix` with `confidence >= 50` (keep top-bar). Never drop silently.

**4.3 Collate reconciliation notes.** Aggregate `corroborates_finding` / `contradicts_finding` tags into the Reconciliation section:
- **Agreement** — "Reviewer and prior self-review both flagged `F-N` at `file:line`; must-fix confidence boosted to <X>."
- **Tension** — "Reviewer flags <concern>; prior self-review suppressed `F-N` at `file:line` with reasoning `<quote>`. Worth re-examining."

### Phase 5 — Assemble artifact

Group triage records by verdict bucket:

1. **must-fix (M-N)**
2. **question (Q-N)**
3. **push-back (PB-N)**
4. **nice-to-have (N-N)**
5. **already-done (AD-N)**
6. **stale / outdated (S-N)** — rendered with a "line moved" badge
7. **suppressed (SU-N)** — low confidence
8. **resolved** — rendered inside a collapsed `<details>` block

Ordering inside each bucket: primary by reviewer-state weight (`CHANGES_REQUESTED > COMMENTED > APPROVED`), secondary by `created_at`.

Build:
- **Summary block** — counts, scope, branch state, reviewers, reconciliation stats.
- **Verdict sections** — full triage record per entry.
- **Reconciliation section** — agreement + tension lists.
- **Residual Risks** — unresolvable stale anchors, ambiguous verdicts at confidence 50-60, contradictory multi-reviewer asks.
- **Handoff section** — flat list of every must-fix handoff record, ready for `/execute` FIX mode to consume.
- **Skipped footer** — dismissed reviews (author + count + reason if known).

### Phase 6 — Self-review checklist (MANDATORY, emit to user)

Before Gate, emit with `✓` or `✗` on each line:

- [ ] PR reference parsed (owner/repo/number) and cwd repo matches
- [ ] `gh auth status` verified
- [ ] PR state gate passed (not merged/closed, or user opted into `--force-state`)
- [ ] Branch checkout gate resolved (on PR head + HEAD matches `headRefOid`, OR user accepted the WARNING banner)
- [ ] All three comment sources fetched (inline threads via GraphQL + timeline + review bodies)
- [ ] DISMISSED / PENDING reviews filtered out (and logged in the Skipped footer)
- [ ] Comments deduped by id; inline comments grouped by thread
- [ ] Stale/outdated comments flagged with "line moved" badge (never silently dropped)
- [ ] Resolved threads preserved in a collapsed `<details>` block (never excluded)
- [ ] Spec folder reuse decided and announced
- [ ] Scope tier confirmed with real counts (provisional revised if needed)
- [ ] Every verdict has: confidence 50-100, verbatim evidence, copy-pasteable reply
- [ ] Every must-fix verdict has a handoff record (file + line + intent + suggestion-block if present)
- [ ] Reconciliation with prior CODE-REVIEW.md completed (if file exists), tension listed even when agreement dominates
- [ ] Low-confidence verdicts moved to Suppressed (not dropped); must-fix exception applied
- [ ] Copy-pasteable replies don't scold on nice-to-have / praise; don't grovel on push-back
- [ ] No code was edited, staged, committed, or pushed by the skill
- [ ] No PR comment was posted by the skill
- [ ] `CODE-REVIEW.md` / plan docs were read-only
- [ ] No `TODO`, `[TBD]`, placeholder text in the proposed artifact

If any check fails → STOP, fix, re-emit. Do not proceed with any `✗` unresolved.

### Phase 7 — HARD GATE: approve verdicts + output path

Present via `AskUserQuestion`:

```
Ready to write PR-FEEDBACK.md.

Summary:
- PR: <owner>/<repo>#<N> — <title>
- Scope: <TIER>
- Comments triaged: <total> (inline <n> / timeline <n> / review-body <n>)
- Reviewers: <list>
- Verdicts:
    must-fix:      <n>
    question:      <n>
    push-back:     <n>
    nice-to-have:  <n>
    already-done:  <n>
    stale:         <n>
    suppressed:    <n>
    resolved:      <n>
- Reconciliation with CODE-REVIEW.md: <n agreements, n tensions> | <none — no prior review>
- Proposed path: <docs/<DATE>-<slug>/PR-FEEDBACK.md | existing-folder/PR-FEEDBACK.md | existing-folder/PR-FEEDBACK.md ## Re-triage append>

Approve and write, or describe changes?
  A) Approve — write at the path above [Recommended]
  B) Change a verdict (reclassify, edit reply, edit handoff)
  C) Re-triage a specific comment (with a hint)
  D) Change the slug / target path
  E) Abort — do not write
```

**STOP HERE.** Do not create the directory. Do not write `PR-FEEDBACK.md`. WAIT for explicit approval.

If the user picks **B** or **C**, loop back to Phase 4 for the affected record, then re-emit Phase 6 before re-gating.

### Phase 8 — Write PR-FEEDBACK.md

Only after explicit approval:

1. `mkdir -p docs/<DATE>-<slug>/` (skip if reusing an existing spec folder).
2. Render using `templates/PR-FEEDBACK.md`. Replace every `{{placeholder}}`. Set `status: approved` in frontmatter.
3. **If appending a re-triage:** do NOT overwrite. Read the existing file and append a `## Re-triage <DATE>` section under the existing body with the same shape as a fresh triage (summary + verdicts + reconciliation + residual risks). Prior sections are read-only history.
4. If `spec_directory_reuse: true`, do NOT patch any sibling plan document. Pure reporter. (If reviewer feedback motivates a plan revision, the author runs `/revision` separately.)
5. Confirm to the user:

```
PR-FEEDBACK written: <path>
  must-fix: <n> · question: <n> · push-back: <n> · nice-to-have: <n> · already-done: <n>
```

The artifact MUST be complete — no placeholders, no `TODO`, readable without this conversation.

### Transition

After writing, inform the user in one sentence:

*"Triage complete. To apply the must-fix items, run `/execute` in FIX mode on this folder. Post the copy-pasteable replies to the PR at your discretion — they are drafts, edit to taste."*

Do NOT invoke `/execute`. Do NOT `gh pr comment`. Do NOT commit. The skill is complete.

## Anti-patterns

- **"Let me auto-post the replies."** No. Pure triage. Replies are for the author to post. The skill never `gh pr comment`s or `gh pr review`s.
- **Dropping stale/outdated comments because "the line moved."** The concern often survives a rebase. Triage, badge as stale, let the author decide.
- **Excluding resolved threads.** Authors reopen threads constantly. Collapse ≠ omit.
- **Skipping the branch checkout gate.** If the working tree isn't on the PR head, `already-done` verdicts are unverifiable and spec cross-references are untrustworthy.
- **Treating a `suggestion` block as "auto-apply this."** Still triage it. The suggestion is the payload, not the verdict. If must-fix, the handoff record carries the exact payload for `/execute`.
- **Skipping the HARD GATE.** The artifact is never written without approval.
- **Editing adjacent `CODE-REVIEW.md` to "reconcile."** Never. Reconciliation goes in `PR-FEEDBACK.md`'s Reconciliation section.
- **Posting back to the PR on the author's behalf.** Never.
- **Triaging merged/closed PRs silently.** Refuse unless the user opts in with `--force-state`.
- **Filling all five buckets to look thorough.** Empty buckets are fine — write "(none)" and move on.
- **Running sub-agents for the triage.** Single-lane. Parallel dispatch duplicates context for no gain.
- **Paraphrasing a comment into "the reviewer said X".** Verbatim quote or suppress.

## Red flags — self-check

STOP if you catch yourself doing any of these:

- You are about to `gh pr comment`, `gh pr review`, or post anything to GitHub.
- You are about to edit a source file, stage, commit, or push.
- You wrote `PR-FEEDBACK.md` before Phase 7 approval.
- The branch checkout gate didn't run, OR the user declined and you proceeded without a WARNING banner in the artifact.
- A comment is stale (`is_outdated == true` or `line == null`) and you dropped it instead of badging it.
- A thread is resolved and you excluded it entirely instead of collapsing it.
- A `DISMISSED` review's comments appear in the triage — filter and log them.
- You triaged a comment without quoting its body verbatim in the artifact.
- You "reconciled" with `CODE-REVIEW.md` by editing `CODE-REVIEW.md`.
- You dispatched a sub-agent for triage (single-lane is the design).
- You invoked `/execute` automatically after writing.
- Your artifact has 0 must-fix AND 0 push-back AND 0 already-done on a `CHANGES_REQUESTED` PR — likely under-triaged.
- Your must-fix count exceeds the reviewer's actual comment count — likely over-classified.

## References

- `templates/PR-FEEDBACK.md` — the output artifact template. Read only at Phase 8.
- `references/scope-tiers.md` — Lightweight / Standard / Deep criteria, escalation signals.
- `references/taxonomy.md` — five-bucket verdict rules, Conventional Comments mapping, confidence factors, reply-tone conventions.
- `references/github-api.md` — GraphQL query + REST endpoints, field semantics (line vs position vs original_line, outdated detection, resolve state), PR-reference parsing.
