---
name: code-review
description: Audit current code changes through a two-stage multi-reviewer process covering plan compliance, code quality, conventions, test coverage, and (conditionally) security, performance, and adversarial review. Use when the user wants to review the current diff, audit changes before commit or PR, sanity-check work before merge, run a pre-commit review, ask "review my code", "check these changes", "audit this diff", "code review this", or invokes /code-review. Auto-detects diff scope (uncommitted > branch-vs-base > HEAD~1), smart-reuses adjacent spec folders when the diff maps to a known feature, runs plan-alignment as a blocking Stage 1, then fans out code-quality + convention + test + conditional security/performance in parallel; Deep scope adds a Stage 3 adversarial pass. Produces docs/YYYY-MM-DD-slug/CODE-REVIEW.md with severity-grouped findings, confidence-gated suppressed table, positives, residual risks, and testing gaps. On re-review, parses any prior `## Fixes applied <DATE>` sections (left by `/execute` in FIX mode) and verifies each claim against the current diff — addressed-but-still-present claims elevate to claim_mismatch findings; skipped-but-vanished claims become verification notes. Scope-adaptive (Lightweight / Standard / Deep), self-review checklist, HARD GATE on write. Pure reporter — never writes or edits code.
argument-hint: "[optional: path hint, scope override (lightweight|standard|deep), or 'vs <base>' to force branch-vs-base diff]"
---

# Code Review — Pure reporter, multi-reviewer, diff-aware

Audit the current code changes with a two-stage multi-reviewer pipeline, merge findings with dedup and confidence-gated suppression, and write a single durable artifact: `docs/YYYY-MM-DD-<slug>/CODE-REVIEW.md`.

This skill does NOT write or edit code. It reads, reviews, and reports. Fixes are the author's job — after the review lands.

## When to use

Trigger this skill when the user:
- says "review my code", "review this diff", "review my changes", "audit these changes", "check this before I commit", "sanity-check this", "pre-commit review", "review this PR before I open it", "code review"
- invokes `/code-review` (optionally with a scope hint or path)
- has just finished implementing a change and wants a pre-merge checkpoint
- references an in-progress PR and asks for a thorough look

Do NOT trigger for:
- authoring a new PRD (that's `prd`)
- authoring a new SPEC (that's `spec`)
- cross-reviewing a PRD/SPEC pair (that's `revision`)
- diagnosing a bug (that's `diagnosis`)
- implementing from a plan (that's `execute`)

## The Iron Law

**This skill is a pure reporter.** It does not write code. It does not apply fixes. It does not stage, commit, push, or edit any source file. The only file it creates is `CODE-REVIEW.md` at the approved path.

If you catch yourself about to apply a `safe_auto` fix "because it's obvious" — STOP. That is the author's decision. The review reports; the author acts.

## Core invariants

1. **Pure reporter.** No code edits. No staging. No commits. Only `CODE-REVIEW.md`.
2. **Diff-first.** Review only what changed. Pre-existing problems are surfaced in a separate table, not blamed on the author.
3. **Plan-alignment is Stage 1 and blocking.** On Standard/Deep, plan-alignment runs alone first. If it returns a P0, the orchestrator pauses and asks the user to resolve drift before Stage 2.
4. **Stage 2 runs in parallel.** All non-plan reviewers fan out in a single dispatch. Sequential Stage 2 is a protocol violation.
5. **Stage 3 (adversarial) runs only on Deep.** It receives Stage 2's merged findings as input and extends — never repeats.
6. **Every finding quotes verbatim evidence.** 5-30 word quote from diff, file, or plan. Paraphrase is suppressed.
7. **Confidence-gated suppression.** Findings with `confidence < 0.60` go to the Suppressed table. P0s with `confidence ≥ 0.50` stay top-bar. Never silently drop.
8. **Cross-reviewer agreement boost.** Findings with the same fingerprint from multiple reviewers get `+0.10` confidence per additional reviewer (capped at 1.0).
9. **Pre-existing separation.** Findings on unchanged code go to the "Pre-existing" table.
10. **Emit the self-review checklist to the user before Gate.** Skipping is a protocol violation.
11. **HARD GATE: do not write `CODE-REVIEW.md` until the user approves the finding set and the output path.**
12. **Smart folder reuse.** If the diff maps to an existing spec folder, write `CODE-REVIEW.md` inside it (next to `PRD.md` / `SPEC.md` / `DIAGNOSIS.md`). Re-reviews append under `## Re-review <DATE>`.
13. **Claim verification on re-review.** When an existing `CODE-REVIEW.md` contains prior `## Fixes applied <DATE>` sections (appended by `/execute` in FIX mode), parse them and verify each claim against the current diff and the fresh merged findings. Addressed claims that still reproduce become claim-mismatch findings (never suppressed). Skipped claims still present are expected; skipped claims that vanished are recorded as verification notes. Prior `## Fixes applied` sections are read-only history — never edited.

## Pre-flight (MANDATORY)

Before anything else:

1. Get today's date: `date +%Y-%m-%d` via Bash. Store as `<DATE>`.
2. Announce the skill is active: `code-review: starting (scope TBD)`.
3. If the invocation had a scope hint (`lightweight` / `standard` / `deep`) or path hint, capture it. The hint is a prior; Phase 0 may override it with a specific justification.

## Process

Execute phases in order. Do not skip.

### Phase 0 — Diff detection, resume, folder strategy, scope

**0.1 Diff-scope detection.** Determine what "current changes" means. Priority chain (mandatory order):

1. **Uncommitted changes.** Run `git status --porcelain` AND `git diff` AND `git diff --staged`. If either the unstaged or staged diff is non-empty → review those changes (both combined). Announce: `code-review: diff = uncommitted (staged + unstaged)`.
2. **Branch vs base.** If the working tree is clean, determine the branch's base:
   - If `git rev-parse --abbrev-ref --symbolic-full-name @{u}` resolves, use that upstream as base.
   - Else, default to `origin/main` (or `origin/master`). If neither exists, `main` locally. Confirm the base resolves with `git merge-base HEAD <base>`.
   - If the current branch is the base branch itself (e.g. on `main`), skip to step 3.
   - Review `git diff <base>...HEAD`. Announce: `code-review: diff = branch-vs-<base>`.
3. **HEAD~1 fallback.** If all else fails, `git diff HEAD~1 HEAD`. Announce: `code-review: diff = head-1`.

If the user passed `vs <base>` or an equivalent override, use that base for step 2 regardless.

If the diff is empty across all three strategies, STOP and tell the user: *"No changes detected. Point me at a diff with `/code-review vs <base>`, make changes first, or specify a different base branch."*

**0.2 Resume check.** After the folder strategy is determined (0.3), look for an existing `CODE-REVIEW.md` at the target path. If found, ASK:

```
Found an existing CODE-REVIEW.md at <path>. What would you like to do?
  A) Append — add a ## Re-review <DATE> section below the existing review [Recommended after a fix pass]
  B) Overwrite — start fresh (old review is preserved in git history)
  C) Write to a new file (CODE-REVIEW-N.md) — preserve both
```

Default is A. Overwrites require explicit confirmation.

**Prior-claim parsing (Append only).** If the user picks **A**, also scan the existing `CODE-REVIEW.md` for any `## Fixes applied <DATE>` sections (these are appended by `/execute` in FIX mode when it closes out a review). For each such section, extract two lists:

- `claimed_addressed` — one entry per row in the **Addressed** table: `{ finding_id, title, file, line, commit_sha, test_ref, applied_date }`.
- `claimed_skipped` — one entry per row in the **Skipped** table: `{ finding_id, title, severity, reason, applied_date }`.

If the file recovery for `file` / `line` fails (row malformed or stripped), fall back to reading the matching `### F-N` entry from the original review body to recover the anchor. If still unavailable, record the claim with `file: "<unknown>"` and flag for residual-risk note at Phase 6.8.

If there are no `## Fixes applied` sections, both lists are empty. These lists feed Phase 6.8 (claim verification). If Append is not chosen (Overwrite / new file), skip parsing — verification does not apply.

**0.3 Folder strategy (smart reuse).** Scan `docs/*/` for directories with `PRD.md`, `SPEC.md`, `DIAGNOSIS.md`, `REVISION.md`, or `EXECUTION.md`. For each candidate, check whether the diff's files are explicitly referenced, or whether the candidate's `topic`/frontmatter matches the diff's likely subject.

If a candidate clearly matches, ASK:

```
The diff appears to touch <feature/module> which already has a spec folder at <path>.
  A) Add CODE-REVIEW.md inside that folder [Recommended]
  B) Create a new dated folder (docs/<DATE>-<slug>/)
```

If the user picks A:
- Set `spec_directory_reuse: true` and `original_spec_dir: <path>` in the artifact frontmatter.
- This folder is now the write target. The plan file(s) in that folder become the **plan reference** for Stage 1.

If no candidate matches, derive a slug from the diff (kebab-case, 2-4 words: the most-modified module + the change kind — e.g. `checkout-coupon-refactor`, `auth-rate-limit`, `session-token-rotation`). Target becomes `docs/<DATE>-<slug>/`.

**0.4 Plan detection.** Check the target folder for plan documents. Capture which plan(s) exist:

- `PRD.md` — product framing.
- `SPEC.md` — architecture + decisions.
- `DIAGNOSIS.md` — bug investigation + suggested fix.
- `REVISION.md` — cross-review patches.
- `EXECUTION.md` — implementation log.

If ≥1 plan exists, Stage 1 (plan-alignment) will run. If none exist, Stage 1 is skipped and the orchestrator notes the absence in the artifact.

**0.5 Scope classification.** Inputs: LOC, file count, file paths, plan frontmatter complexity (if any). Decision rubric from `references/scope-tiers.md`:

| Tier | Criteria | Reviewers |
|------|----------|-----------|
| **Lightweight** | <50 LOC, 1-2 files, no user-input / auth / concurrency / migration. | quality, convention, test (if test files in diff), plan-alignment (if plan exists). |
| **Standard** | 50-500 LOC, 3-10 files, or non-trivial logic. | quality, convention, test, plan-alignment (if plan), security (auto-trigger), performance (auto-trigger). |
| **Deep** | 500+ LOC, 10+ files, OR touches concurrency / auth / payments / migrations / public APIs / cross-module. | all of the above + adversarial (required). |

Auto-trigger detection for security and performance on Standard:

- **Security trigger:** the diff includes any of: `req.body|req.query|req.params`, `fetch(` + user-bound args, `exec*(`, `spawn(`, `eval(`, raw SQL templates, `dangerouslySetInnerHTML`, new routes in `/api`, middleware changes, crypto/JWT/hash code, secret-handling files (`.env*`, config files with secrets), file-path joins with user input, new webhook handlers.
- **Performance trigger:** the diff includes any of: `.map(async`, `for (...) await`, new DB query calls, `Promise.all` over user-supplied arrays, new cron / queue consumer / batch job, recursive fn over user-supplied data, new React list rendering without virtualization keyword, new cache logic.

Run `Grep` against the diff hunks to detect triggers. If a trigger fires, the corresponding reviewer is added to Stage 2.

Announce the classification: `code-review: scope = <TIER> — <one-line reason>`.

### Phase 1 — Clarifying questions (≤3, only if ambiguous)

Most reviews need zero questions. Only ask when the auto-detected context is genuinely ambiguous:

1. **Base branch ambiguity** — multiple plausible bases (e.g., both `origin/main` and `origin/develop` exist).
2. **Folder reuse ambiguity** — multiple matching spec folders.
3. **Scope override** — auto-classified Standard but diff touches e.g. `src/auth/*`; confirm user wants Deep, or accept auto-added security.
4. **Resume mode** — existing `CODE-REVIEW.md` without a clear preference.

Rules:
- ONE question per turn. Prefer `AskUserQuestion` with multiple-choice.
- STOP asking as soon as the context is resolved.
- Do NOT ask about code quality, style, or convention preferences — the reviewers infer those from the code itself.

### Phase 2 — Grounding summary

Before dispatching reviewers, emit a short **grounding summary** (3-8 bullets) so the user can correct misreadings:

```
code-review: ready to dispatch

- Diff: <source>, N files, +X/-Y LOC
- Scope: <TIER>
- Plan: <path, or "none — reviewing on diff shape">
- Reviewers queued:
  - Stage 1: <plan-alignment (blocking) | skipped>
  - Stage 2: <list of parallel reviewers with trigger reasons>
  - Stage 3: <adversarial | skipped>
- Write target: <docs/<DATE>-<slug>/CODE-REVIEW.md>
```

Wait 1 beat for user correction. If the user corrects the scope or target, adjust and re-emit.

If the user does not respond within the normal cadence, proceed with the stated plan.

### Phase 3 — Stage 1: plan-alignment (blocking on Standard/Deep)

If no plan document exists, skip to Phase 4.

Dispatch `code-review-plan-alignment-reviewer` with:
- Plan document paths.
- Diff scope + file list + diff content.
- Scope tier.
- Repo root path.

Parse the returned JSON. If any finding has `severity: P0` and `pre_existing: false`:

**On Lightweight:** note the drift in the final artifact but proceed to Stage 2.

**On Standard or Deep:** PAUSE the pipeline. Emit:

```
code-review: plan-alignment returned P0 drift.

- <F-N> — <title>
  - Plan: <section quote>
  - Diff: <file:line quote>
  - Why it matters: <one sentence>

Options:
  A) Proceed with Stage 2 anyway — surface the drift in the artifact, review the implementation as-is
  B) Stop the review — fix the drift first, then re-run /code-review
  C) Update the plan (escalate to /revision) — drift reflects an intentional design change
```

Default recommendation: **B** (stop and resolve). If the user picks A, the orchestrator records the choice in the artifact and proceeds.

### Phase 4 — Stage 2: parallel fan-out

Dispatch all Stage 2 reviewers in a **single message** with multiple `Agent` tool calls. Reviewers to dispatch depend on tier and triggers:

**Lightweight:**
- `code-review-quality-reviewer`
- `code-review-convention-reviewer`
- `code-review-test-reviewer` (only if test files are in the diff; otherwise skip with a note)

**Standard:**
- `code-review-quality-reviewer` (always)
- `code-review-convention-reviewer` (always)
- `code-review-test-reviewer` (always)
- `code-review-security-reviewer` (if security trigger matched in Phase 0.5)
- `code-review-performance-reviewer` (if performance trigger matched)

**Deep:**
- All of the above, with `code-review-security-reviewer` and `code-review-performance-reviewer` **always dispatched** regardless of trigger.

Pass each reviewer:
- Diff scope + file list + diff content.
- Scope tier.
- Repo root path.
- For security/performance: the trigger pattern that brought them in.

Collect the returned JSON payloads. If any reviewer's output is malformed (not valid JSON, missing required fields), note the failure in the artifact's Stage results table and proceed with the reviewers that succeeded — do not block the pipeline on a malformed return.

### Phase 5 — Stage 3: adversarial (Deep only)

Skip entirely on Lightweight and Standard.

On Deep, after Stage 2 completes:

1. Merge Stage 2 findings (temporarily — full merge runs in Phase 6).
2. Dispatch `code-review-adversarial-reviewer` with:
   - Stage 2's merged findings JSON (so it does NOT repeat).
   - Diff + files + tier + plan.
3. Collect the JSON payload.

### Phase 6 — Merge, dedup, suppress, categorize

Process all reviewer outputs:

**6.1 Renumber IDs.** Each reviewer returned `<short>-<n>` IDs (e.g., `cq-1`, `cv-3`). Flatten into the skill's `F-N` sequence, preserving order by severity (P0 → P1 → P2 → P3) then by reviewer priority (plan-alignment > security > code-quality > convention > test > performance > adversarial).

**6.2 Deduplicate.** For each pair of findings, compute the fingerprint:
```
fingerprint = normalize(file) + line_bucket(line, ±3) + normalize(intent)
```
Findings with matching fingerprints merge:
- Keep title + fix from the higher-confidence reviewer (ties broken by reviewer priority).
- Merge `evidence` arrays, unique quotes only.
- Boost confidence: `min(1.0, max(confidences) + 0.10 * (reviewers_count - 1))`.
- Add `seen_by: [<reviewers>]` for the cross-reviewer-agreement badge.

**6.3 Suppress low-confidence.** For each finding:
- If `confidence < 0.60` AND NOT (`severity == P0` AND `confidence >= 0.50`), move to the **Suppressed** table.
- Never silently drop.

**6.4 Split pre-existing.** Findings with `pre_existing: true` move to the **Pre-existing** table.

**6.5 Categorize.** Main findings grouped by severity:
- **Blockers (P0)** — ship-blocking.
- **Major (P1)** — fix before merge or explicitly accept.
- **Minor (P2)** — worth fixing.
- **Nits (P3)** — preference.

**6.6 Collate.** Aggregate `positives`, `residual_risks`, `testing_gaps` across all reviewers. Unique entries only. Attribute to the producing reviewer.

**6.7 Audit.** Build the Diff inventory table: for each file in the diff, list which reviewers opened it (via their `files_reviewed` arrays). Flag any file that no reviewer opened.

### Phase 6.8 — Claim verification (re-review only)

Skip entirely unless Phase 0.2 recorded non-empty `claimed_addressed` or `claimed_skipped` lists. This phase is **passive** — no new reviewer invocations. It cross-references prior claims against the freshly merged findings (post-6.2 dedup) and the current diff via fingerprint matching.

**6.8.1 Build prior-claim fingerprints.** For each entry in `claimed_addressed` and `claimed_skipped`, compute:

```
fingerprint = hash(normalize(file) + line_bucket(line, ±3) + normalize(intent))
```

Recover `intent` from the original review's `### F-N` body if the Fixes-applied row does not carry it (the original review always has an `intent`-equivalent under its finding block — use the `title` + `why_it_matters` to reconstruct a normalized intent). If the original review is missing the entry (stale finding ID), mark the claim as `orphaned` and carry it into the verification table unchanged — it cannot be matched but must still be reported.

**6.8.2 Verify `claimed_addressed`.** For each prior-addressed finding:

1. Compare its fingerprint against every current merged finding (including any currently in the Suppressed set — claim mismatches override suppression).
2. **No match** → likely genuinely fixed. Status: `verified`. Notes: `No current finding at <file:line>`.
3. **Match** (same fingerprint, any severity) → **claim mismatch**. Status: `mismatch — still present`. Take the matched current finding and:
   - Set `verifies_claim: "<F-N>"` (the prior finding ID).
   - Set `claim_mismatch: true`.
   - If the current finding was in the Suppressed table, un-suppress it (promote to the main severity-grouped tables using its current severity).
   - Boost its confidence by `+0.15` (capped at 1.0) — a cross-revision recurrence is strong corroboration.
4. Additionally, check whether the claim's `file` appears in the current diff at all. **File untouched** and no match above → still `verified`. **File untouched** with a match above → keep `mismatch — still present`; add residual-risk note: *"Claimed addressed but current finding reopens the same fingerprint despite no touch this revision — prior fix may have been reverted or never landed."*

**6.8.3 Verify `claimed_skipped`.** For each prior-skipped finding:

1. Compare its fingerprint against the current merged findings.
2. **Match** → finding still present, consistent with "skipped". Status: `still present (expected)`. Tag the current finding with `verifies_claim: "<F-N>"` (no `claim_mismatch` — the skip is honest).
3. **No match** → the skip is now stale. Status: `vanished`. Do NOT create a new finding. Record as a verification note: *"Skipped finding F-N no longer reproduces — line removed or refactored."*

**6.8.4 Record the verification table.** Assemble `verification_table` with one row per prior claim (across both addressed and skipped). Columns:

| Prior claim | F-N (current) | Severity | Status | Notes |
|-------------|---------------|----------|--------|-------|

Status values: `verified`, `mismatch — still present`, `still present (expected)`, `vanished`, `orphaned`. This table is carried into Phase 9's re-review render and summarized in Phase 8's HARD GATE preview.

**6.8.5 No source-of-truth mutation.** Phase 6.8 never edits the prior `## Fixes applied` sections, the original review body, or any sibling plan document. All verification outcomes live inside the new `## Re-review <DATE>` section only. The original review and its appended Fixes-applied history remain immutable.

### Phase 6.9 — Lead Judgment (adjudication)

After 6.1–6.8 finish, the reviewers have each spoken in their lane. The orchestrator now speaks **last** — one pass over every surfaced finding (main tables + Suppressed; never Pre-existing) with a single decision each: `accept` · `downgrade P_X → P_Y` · `upgrade P_X → P_Y` · `reject → Suppressed` · `merge-into F-M`. Every reshaping decision carries a one-line rationale tied to evidence strength, cross-reviewer `seen_by` count, a plan anchor, an adversarial lens, or a prior-claim mismatch.

**Rules.** Default is **accept** — reviewers are trusted; the orchestrator resolves conflict and softens noise, it does not second-guess lanes. Downgrade when evidence is thinner than severity implies (P1 with single reviewer, `confidence 0.62`, no plan anchor → P2 candidate). Upgrade when agreement or claim-mismatch corroboration is stronger than merged confidence (P2 with `seen_by: [quality, security]` + `claim_mismatch: true` → P1 candidate). Reject only when a reviewer stepped out of lane or when two findings are genuinely the same finding dedup missed — rejected findings move to Suppressed with the rationale; never silently dropped.

**Render.** Reshaping entries (downgrade / upgrade / reject / merge) are rendered as a compact `### Lead adjudication` subsection directly under the severity-grouped findings. Plain accepts may be omitted. Adjudication is not a second review: no new code reads, no invented findings. If a decision requires new evidence, accept at the reviewer's severity and let the author decide.

### Phase 7 — Self-review checklist (MANDATORY, emit to user)

Before Gate, emit with `✓` or `✗` on each line:

- [ ] Diff scope detected correctly and announced
- [ ] Target folder chosen (new vs reused spec folder) and justified
- [ ] Scope tier classified and announced with reason
- [ ] Grounding summary emitted; user had a chance to correct
- [ ] Stage 1 ran if plan exists; result noted (clean / P0 drift handled)
- [ ] Stage 2 dispatched in parallel (single message, multiple `Agent` calls)
- [ ] Stage 2 reviewers matched the tier (no missing mandatory reviewers, no extra unjustified ones)
- [ ] Stage 3 ran on Deep; skipped correctly on Lightweight/Standard
- [ ] Every finding has severity + confidence + verbatim evidence
- [ ] Dedup applied; cross-reviewer agreement boost reflected in confidence
- [ ] Low-confidence findings moved to Suppressed (not dropped); P0 exception applied
- [ ] Pre-existing findings separated from diff findings
- [ ] (Re-review only) Prior `## Fixes applied` claims parsed at Phase 0.2 and verified at Phase 6.8
- [ ] (Re-review only) Claim mismatches elevated with `verifies_claim` + `claim_mismatch: true`; un-suppressed if they had been suppressed
- [ ] (Re-review only) `verification_table` assembled with a row per prior claim (no silent drops)
- [ ] Positives, residual risks, testing gaps collected and attributed
- [ ] Diff inventory built; files-not-reviewed (if any) flagged
- [ ] No reviewer output silently discarded; malformed returns noted in Stage results
- [ ] Every P0 and P1 has a concrete suggested fix (not "consider refactoring")
- [ ] Phase 6.9 Lead adjudication ran; every reshaping decision (downgrade / upgrade / reject / merge) carries a one-line rationale tied to evidence, `seen_by`, plan anchor, adversarial lens, or claim-mismatch status
- [ ] No `TODO`, `[TBD]`, placeholder text in the proposed artifact
- [ ] No code was edited, staged, committed, or pushed by the skill

If any check fails → STOP, fix, re-emit. Do not proceed with any `✗` unresolved.

### Phase 8 — HARD GATE: approve finding set + output path

Present via `AskUserQuestion`:

```
Ready to write CODE-REVIEW.md.

Summary:
- Scope: <TIER>
- Diff: <source>, <N> files, <+X/-Y> LOC
- Blockers (P0): <count>
- Major (P1): <count>
- Minor (P2): <count>
- Nits (P3): <count>
- Suppressed: <count>
- Pre-existing: <count>
- Prior-claim verification (re-review only): <verified N / mismatch N / still-present-expected N / vanished N / orphaned N>
- Reviewers: <list + one-line per result>
- Proposed path: <docs/<DATE>-<slug>/CODE-REVIEW.md | existing-folder/CODE-REVIEW.md | existing-folder/CODE-REVIEW.md ## Re-review append>

Approve and write, or describe changes?
  A) Approve — write at the path above [Recommended]
  B) Change a finding (remove, soften severity, edit fix)
  C) Re-dispatch a specific reviewer (e.g., rerun test with -v output)
  D) Change the slug / target path
  E) Abort — do not write
```

**STOP HERE.** Do not create the directory. Do not write `CODE-REVIEW.md`. WAIT for explicit approval.

If the user picks **B** or **C**, loop back to the relevant phase, then re-emit Phase 7 before re-gating.

### Phase 9 — Write CODE-REVIEW.md

Only after explicit approval:

1. `mkdir -p docs/<DATE>-<slug>/` (skip if reusing an existing spec folder).
2. Render the full artifact using `templates/CODE-REVIEW.md`. Replace every `{{placeholder}}` with real content. Set `status: approved` in frontmatter.
3. **If appending a re-review:** do NOT overwrite. Read the existing `CODE-REVIEW.md`, append a new section under the existing re-review append target:

```markdown
## Re-review {{DATE}}

**Scope:** <TIER>. **Reviewers:** <list>. **Findings:** P0:<n> P1:<n> P2:<n> P3:<n>.

### Verification of prior fixes

_Include this subsection ONLY when Phase 0.2 parsed prior `## Fixes applied <DATE>` sections and Phase 6.8 produced a `verification_table`. Otherwise omit._

Status values:
- **verified** — prior claim holds (addressed, not re-introduced).
- **mismatch — still present** — claimed addressed, but a current finding reopens the same fingerprint. Linked current finding carries `claim_mismatch: true`.
- **still present (expected)** — claimed skipped; still present, consistent with the skip.
- **vanished** — claimed skipped, but the code is now gone (refactor / removal).
- **orphaned** — claim references a finding ID no longer resolvable in the original review body; reported for the audit trail.

| Prior claim | F-N (current) | Severity | Status | Notes |
|-------------|---------------|----------|--------|-------|
| F-1 (addressed at `abc1234`) | — | P0 | verified | No current finding at `src/checkout/coupon.ts:42` |
| F-2 (addressed at `def5678`) | F-3 | P0 | **mismatch — still present** | Current finding quotes same line; confidence boosted `+0.15` |
| F-4 (skipped) | F-7 | P2 | still present (expected) | — |
| F-6 (skipped) | — | P3 | vanished | Line removed in refactor |

### What changed since last review

- <one-line delta> — e.g. "F-1 (P0) fixed in commit abc123; verified absent."
- <one-line delta>

### New findings

[... severity-grouped findings in the same shape as the main artifact ...]
```

4. If `spec_directory_reuse: true`, do NOT patch the sibling plan document — this skill is a pure reporter and only writes `CODE-REVIEW.md`. (The plan-alignment finding may mention the drift, but the act of updating the plan is the user's decision and is handled by `/revision`.)

5. Confirm to the user:
```
CODE-REVIEW written: <path>
  Blockers: <count> · Major: <count> · Minor: <count> · Nits: <count>
```

The artifact MUST be complete — no placeholders, no `TODO`, readable without this conversation.

### Transition

After writing, INFORM the user in one sentence: *"Review complete. Address the blockers, triage the majors, then either commit / push / open a PR, or re-run `/code-review` after fixes to confirm the delta is clean."*

Do NOT invoke any implementation skill. Do NOT edit, commit, or push. The skill is complete.

## Anti-patterns

See `references/anti-patterns.md`. Highlights:

- **"Just apply the safe_auto fixes."** No. Pure reporter. Fixes are the author's decision.
- **Skipping Lead adjudication because every finding looks fine.** Plain accepts are fine; the discipline is running the pass. Without it, the author reads seven reviewers' opinions instead of one coherent review.
- **Running reviewers serially "to save tokens".** Parallel is the design. Serial is slower AND costs more context.
- **Skipping the HARD GATE.** The artifact is never written without approval. Skipping breaks the contract.
- **Padding findings.** Clean passes are valid and expected. Synthetic findings corrupt the signal-to-noise ratio.
- **Going out of lane.** Each reviewer has a scope. Security concerns go to the security reviewer; style concerns to the convention reviewer; mixing creates dedup failures.
- **Overwriting an existing review without asking.** Re-reviews append under `## Re-review <DATE>`. The original review is history.
- **Silently accepting `## Fixes applied` claims.** A prior review that says "F-1 addressed at abc1234" is a claim, not a guarantee. Re-reviews must fingerprint-match each claim against the current findings. Trusting the table without verification is exactly how regressions hide between reviews.
- **Editing a prior `## Fixes applied` section.** It is historical record, appended by `/execute`. Never rewrite it — verification outcomes go in the new `## Re-review <DATE>` section.
- **Editing source files.** Under no circumstances. Every source file is read-only to this skill.
- **Flagging pre-existing code as the author's problem.** Move it to the Pre-existing table. Author is not responsible for code they did not write.
- **Missing grounding summary.** Without it, the user can't correct scope/target before reviewers fan out — wasted cycles if wrong.

## Red flags — self-check

STOP if you catch yourself doing any of these:

- You are about to apply a fix (any class — `safe_auto` included).
- You wrote `CODE-REVIEW.md` before Phase 8 approval.
- You dispatched Stage 2 reviewers sequentially across multiple messages.
- You dispatched Stage 3 (adversarial) on Lightweight or Standard tier without explicit user request.
- You dispatched Stage 3 without passing Stage 2's merged findings.
- Your final artifact has >30 findings on Lightweight scope — noise.
- Your final artifact has 0 findings on Deep scope — likely under-scrutinized.
- You silently dropped a low-confidence finding instead of moving it to Suppressed.
- You adjudicated a finding with no rationale, or with a rationale that does not reference evidence, `seen_by`, plan anchor, adversarial lens, or claim-mismatch status.
- Lead adjudication deleted a finding (reject must move to Suppressed, never drop).
- You merged two findings with different `intent` strings — probably a merge bug.
- You patched `PRD.md` / `SPEC.md` / `DIAGNOSIS.md` — pure reporter, never.
- You staged, committed, or pushed anything — pure reporter, never.
- The target folder has an existing `CODE-REVIEW.md` and you overwrote without asking.
- The existing `CODE-REVIEW.md` contains `## Fixes applied <DATE>` sections and you did not run Phase 6.8 claim verification.
- A current finding shares a fingerprint with a prior "addressed" claim and you did not elevate it with `verifies_claim` + `claim_mismatch: true` (or you left it suppressed).
- You edited the prior `## Fixes applied` section instead of appending verification outcomes to the new `## Re-review <DATE>` block.

## References

- `templates/CODE-REVIEW.md` — the output artifact template. Read only at Phase 9.
- `references/findings-schema.md` — shared JSON schema every reviewer returns. Dispatch sends reviewers to this file.
- `references/scope-tiers.md` — detailed Lightweight / Standard / Deep rubric, including security/performance auto-trigger patterns. Load when Phase 0.5 classification is ambiguous.
- `references/anti-patterns.md` — orchestration and reviewer failure modes. Load when tempted to skip a stage or pad findings.
- `agents/code-review-quality-reviewer.md` *(plugin root)* — Stage 2 code-quality reviewer.
- `agents/code-review-convention-reviewer.md` *(plugin root)* — Stage 2 convention reviewer.
- `agents/code-review-test-reviewer.md` *(plugin root)* — Stage 2 test quality reviewer.
- `agents/code-review-plan-alignment-reviewer.md` *(plugin root)* — Stage 1 blocking plan-alignment reviewer.
- `agents/code-review-security-reviewer.md` *(plugin root)* — Stage 2 conditional security reviewer.
- `agents/code-review-performance-reviewer.md` *(plugin root)* — Stage 2 conditional performance reviewer.
- `agents/code-review-adversarial-reviewer.md` *(plugin root)* — Stage 3 Deep-only adversarial reviewer.
