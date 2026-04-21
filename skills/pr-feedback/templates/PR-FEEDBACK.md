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

- **PR:** [{{pr_owner}}/{{pr_repo}}#{{pr_number}}]({{pr_url}}) — {{pr_title}} · @{{pr_author}}
- **State:** {{pr_state}}{{draft_badge}} · **Decision:** {{pr_review_decision}} · **Reviewers:** {{reviewers_formatted}}
- **Branch at triage:** `{{branch_at_triage}}` @ `{{head_sha_short}}` ({{head_matches_label}})
- **Scope:** {{scope}} — {{scope_reason}} · **Spec folder:** {{spec_dir_block}} · **Reconciled with:** {{reconciled_block}}

{{branch_warning_banner}}

| must-fix | question | push-back | nice-to-have | already-done | stale | suppressed | resolved | dismissed |
|---------:|---------:|----------:|-------------:|-------------:|------:|-----------:|---------:|----------:|
| {{n_must_fix}} | {{n_question}} | {{n_push_back}} | {{n_nice_to_have}} | {{n_already_done}} | {{n_stale}} | {{n_suppressed}} | {{n_resolved}} | {{n_dismissed_skipped}} |

## Verdicts

One record per reviewer comment. Severity-equivalent ordering: must-fix first,
then question, push-back, nice-to-have, already-done, stale. Empty buckets have
no heading.

### Must-fix

{{must_fix_records}}

<!--
Per-record shape (same for every bucket; prefix changes: M- / Q- / PB- / N- / A- / S-):

#### M-1 — {{record_title}}

- **By** @{{author}} · {{source_kind}} · [permalink]({{comment_url}}) · **File** [`{{path}}:{{line}}`]({{file_url}}){{stale_badge}}
- **Comment:** > {{body_verbatim}}
- **Verdict:** must-fix ({{confidence}}/100) — {{rationale}}
- **Evidence:** `{{evidence_quote}}` ({{evidence_source}}) {{cross_ref_notes}}
- **Reply (copy-paste):**
  ```
  {{reply_draft}}
  ```
- **Handoff:** `{{handoff_path}}:{{handoff_line}}` — {{handoff_intent}} {{handoff_suggestion_block_or_none}}
-->

### Questions

{{question_records}}

### Push-back

{{push_back_records}}

### Nice-to-have

{{nice_to_have_records}}

### Already-done

{{already_done_records}}

### Stale / outdated

{{stale_records}}

## Handoff — `/execute` FIX mode

Must-fix items ready to consume:

{{handoff_summary}}

<!--
| ID  | File                       | Line | Intent                                              | Suggestion |
|-----|----------------------------|-----:|-----------------------------------------------------|------------|
| M-1 | `src/http/retry.ts`        | 48   | Route retry through `rateLimiter.acquire()` first   | yes (verbatim from reviewer) |
| M-2 | `src/checkout/coupon.ts`   | 112  | Null-check `coupon.discount` before multiplication  | no         |
-->

Run `/execute` in FIX mode on this folder — the skill parses the table above.
Post the copy-pasteable replies to GitHub at your discretion; they're drafts.

<!--
Optional sections — include ONLY when non-empty:

### Reconciliation with CODE-REVIEW.md

**Agreements (+15 confidence to must-fix):**

| Prior finding | Current verdict | Notes |
|---------------|-----------------|-------|
| F-3 (P0 — "missing rate limit") | M-1 (conf 92) | Reviewer independently flagged same line |

**Tensions (prior stance disagrees with reviewer):**

| Prior finding | Reviewer ask | Prior stance | Worth re-examining? |
|---------------|--------------|--------------|---------------------|
| F-7 (suppressed, conf 52) | Wants explicit error message | Low-confidence nitpick | Yes — reviewer's framing elevates this |

### Suppressed (low confidence)

{{suppressed_records}}

### Resolved threads (collapsed)

<details>
<summary>Resolved threads ({{n_resolved}})</summary>

{{resolved_records}}

</details>

### Residual risks

- Stale comment S-2 — couldn't re-anchor; reviewer's concern may still apply if logic migrated
- PB-1 — conf 58; reviewer and prior self-review disagree; no spec anchor cited
- Multi-reviewer conflict on `src/auth/session.ts:88` — @a wants shorter TTL, @b wants longer; author decision required

### Skipped — dismissed reviews

| Reviewer    | Dismissed at         | Comments | Reason                           |
|-------------|----------------------|---------:|----------------------------------|
| @reviewer-x | 2026-04-17T14:30Z    | 3        | Approved after subsequent changes |

Cut any section above if empty. See _shared/artifact-compactness.md.
-->
