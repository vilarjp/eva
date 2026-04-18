---
name: PR Feedback Triage — {{pr_title}}
pr_url: {{pr_url}}
pr_number: {{pr_number}}
pr_repo: {{pr_owner}}/{{pr_repo}}
pr_title: {{pr_title}}
pr_author: {{pr_author}}
pr_state: {{pr_state}}
pr_is_draft: {{pr_is_draft}}
pr_review_decision: {{pr_review_decision}}
pr_head_branch: {{pr_head_branch}}
pr_head_sha: {{pr_head_sha}}
pr_base_branch: {{pr_base_branch}}
date: {{date}}
scope: {{scope}}
scope_reason: {{scope_reason}}
reviewers: {{reviewers_yaml_list}}
spec_directory_reuse: {{spec_directory_reuse}}
original_spec_dir: {{original_spec_dir}}
reconciled_with_code_review: {{reconciled_with_code_review_path}}
branch_state_at_triage:
  current_branch: {{branch_at_triage}}
  head_sha: {{head_sha_at_triage}}
  matches_pr_head: {{head_matches}}
verdict_counts:
  must_fix: {{n_must_fix}}
  question: {{n_question}}
  push_back: {{n_push_back}}
  nice_to_have: {{n_nice_to_have}}
  already_done: {{n_already_done}}
  stale: {{n_stale}}
  suppressed: {{n_suppressed}}
  resolved: {{n_resolved}}
  dismissed_skipped: {{n_dismissed_skipped}}
reconciliation:
  agreements: {{n_agreements}}
  tensions: {{n_tensions}}
status: approved
---

# PR Feedback Triage — {{pr_title}}

## Summary

- **PR:** [{{pr_owner}}/{{pr_repo}}#{{pr_number}}]({{pr_url}}) — {{pr_title}}
- **Author:** @{{pr_author}}
- **State:** {{pr_state}}{{draft_badge}} · **Review decision:** {{pr_review_decision}}
- **Reviewers:** {{reviewers_formatted}}
- **Branch at triage:** `{{branch_at_triage}}` @ `{{head_sha_short}}` ({{head_matches_label}})
- **Scope:** {{scope}} — {{scope_reason}}
- **Spec folder:** {{spec_dir_block}}
- **Reconciled with:** {{reconciled_block}}

{{branch_warning_banner}}

### Verdict counts

| Bucket | Count |
|--------|-------|
| must-fix | {{n_must_fix}} |
| question | {{n_question}} |
| push-back | {{n_push_back}} |
| nice-to-have | {{n_nice_to_have}} |
| already-done | {{n_already_done}} |
| stale / outdated | {{n_stale}} |
| suppressed (low confidence) | {{n_suppressed}} |
| resolved (collapsed) | {{n_resolved}} |
| dismissed (skipped) | {{n_dismissed_skipped}} |

---

## Must-fix ({{n_must_fix}})

{{must_fix_records}}

<!--
Per-record template for each bucket (must-fix / question / push-back / nice-to-have / already-done / stale):

### M-1 — {{record_title}}

- **Reviewer:** @{{author}} · **Source:** {{source_kind}} · **Comment:** [permalink]({{comment_url}})
- **File:** [`{{path}}:{{line}}`](https://github.com/{{pr_owner}}/{{pr_repo}}/blob/{{pr_head_sha}}/{{path}}#L{{line_start}}-L{{line_end}}){{stale_badge}}
- **Comment (verbatim):**
  > {{body_verbatim}}
- **Verdict:** must-fix (confidence {{confidence}}/100)
- **Rationale:** {{rationale}}
- **Evidence:** `{{evidence_quote}}` ({{evidence_source}})
- **Cross-ref:** {{cross_ref_notes}}   # e.g. "corroborates CODE-REVIEW F-3" or "PRD §2.1 supports this"
- **Suggested reply (copy-pasteable):**
  ```
  {{reply_draft}}
  ```
- **Handoff to /execute:**
  - File: `{{handoff_path}}:{{handoff_line}}`
  - Intent: {{handoff_intent}}
  - Suggestion block: {{handoff_suggestion_block_or_none}}
-->

## Questions ({{n_question}})

{{question_records}}

## Push-back ({{n_push_back}})

{{push_back_records}}

## Nice-to-have ({{n_nice_to_have}})

{{nice_to_have_records}}

## Already-done ({{n_already_done}})

{{already_done_records}}

## Stale / outdated ({{n_stale}})

{{stale_records}}

## Reconciliation with CODE-REVIEW.md

{{reconciliation_block}}

<!--
Reconciliation block shape (omit entire section if no prior CODE-REVIEW.md):

### Agreements ({{n_agreements}})

| Prior finding | Current verdict | Notes |
|---------------|-----------------|-------|
| F-3 (P0 — "missing rate limit") | M-1 (must-fix, conf 92) | Reviewer independently flagged the same line; boosted must-fix confidence +15. |

### Tensions ({{n_tensions}})

| Prior finding | Reviewer ask | Our prior stance | Worth re-examining? |
|---------------|--------------|------------------|---------------------|
| F-7 (suppressed, conf 52) | Wants explicit error message | We suppressed as low-confidence nitpick | Yes — reviewer's framing elevates this; consider re-reviewing. |
-->

## Suppressed (low confidence)

{{suppressed_records}}

<details>
<summary>Resolved threads ({{n_resolved}})</summary>

{{resolved_records}}

</details>

## Residual risks

{{residual_risks}}

<!--
Residual-risks shape:

- Stale comment S-2 — couldn't re-anchor `diffHunk` snippet to any current file. Original anchor: `src/old.ts:42`. Reviewer's concern may still be valid if the logic migrated elsewhere.
- PB-1 — confidence 58; reviewer and prior self-review disagree but neither cites a spec anchor. Worth a sync-chat before merge.
- Multi-reviewer conflict on `src/auth/session.ts:88` — @reviewer-a wants shorter TTL, @reviewer-b wants longer. Author decision required.
-->

## Skipped — dismissed reviews

{{dismissed_skipped_block}}

<!--
Shape:

| Reviewer | Dismissed at | Comment count | Reason (if known) |
|----------|--------------|---------------|-------------------|
| @reviewer-x | 2026-04-17T14:30Z | 3 | Approved after subsequent changes |
-->

## Handoff

The following must-fix items are ready for `/execute` FIX mode to consume. Each entry maps directly to a targeted change:

{{handoff_summary}}

<!--
Handoff shape:

| ID | File | Line | Intent | Suggestion block? |
|----|------|------|--------|-------------------|
| M-1 | `src/http/retry.ts` | 48 | Route retry path through `rateLimiter.acquire()` before the HTTP call | yes — verbatim from reviewer |
| M-2 | `src/checkout/coupon.ts` | 112 | Add null-check on `coupon.discount` before multiplication | no |
-->

To apply: run `/execute` in FIX mode on this folder. The skill will parse this Handoff table and open each file at the listed line.

Post the copy-pasteable replies to GitHub at your discretion — they are drafts, edit to taste before sending.
