---
name: code-review
date: "{{DATE}}"
status: "{{draft | approved}}"
topic: "{{short topic — what was reviewed}}"
scope: "{{LIGHTWEIGHT | STANDARD | DEEP}}"
diff_source: "{{uncommitted | branch-vs-<base> | head-1}}"
diff_stats: "{{files: N, +LOC / -LOC}}"
reviewers_run: "{{code-quality, convention, test[, plan-alignment][, security][, performance][, adversarial]}}"
plan_reference: "{{omit unless a plan existed, else: docs/YYYY-MM-DD-<slug>/SPEC.md}}"
spec_directory_reuse: {{false | true}}
original_spec_dir: "{{omit unless spec_directory_reuse is true, else: docs/YYYY-MM-DD-<slug>/}}"
---

# Code Review: {{topic}}

**Diff:** `{{diff_source}}` — {{N}} files, +{{added}}/-{{removed}} LOC.
**Scope:** {{LIGHTWEIGHT | STANDARD | DEEP}}. **Reviewers:** {{list}}.
**Plan:** {{path or "none"}}.

## TL;DR

**Posture:** {{one sentence — green / yellow (N P0+P1) / red (blockers)}}

| P0 | P1 | P2 | P3 | Suppressed | Pre-existing |
|---:|---:|---:|---:|-----------:|-------------:|
| {{n}} | {{n}} | {{n}} | {{n}} | {{n}} | {{n}} |

{{one-line verdict — e.g. "Fix P0s, triage P1s, merge." / "Clean pass — green."}}

## Stage results

| Stage | Reviewer | Status | P0/P1/P2/P3 |
|-------|----------|--------|-------------|
| 1 | plan-alignment | {{ran / skipped — reason / clean}} | {{n/n/n/n}} |
| 2 | code-quality | {{ran / clean}} | {{n/n/n/n}} |
| 2 | convention | {{ran / clean}} | {{n/n/n/n}} |
| 2 | test | {{ran / clean}} | {{n/n/n/n}} |
| 2 | security | {{ran — trigger / skipped / clean}} | {{n/n/n/n}} |
| 2 | performance | {{ran — trigger / skipped / clean}} | {{n/n/n/n}} |
| 3 | adversarial | {{ran / skipped — not Deep}} | {{n/n/n/n}} |

## Findings

One row per finding. Severity headings kept; empty blocks cut.

### Blockers (P0) — must resolve before merge

| ID | File:line | Conf | Seen by | Evidence | Why it matters | Fix |
|----|-----------|-----:|---------|----------|----------------|-----|
| F-1 | `{{path:line}}` | 0.00-1.00 | {{reviewer(s)}} | `{{verbatim quote}}` | {{one sentence — concrete risk}} | {{one sentence — minimal}} |

### Major (P1) — fix or explicitly accept residual risk

| ID | File:line | Conf | Seen by | Evidence | Why it matters | Fix |
|----|-----------|-----:|---------|----------|----------------|-----|
| F-3 | `{{path:line}}` | {{n}} | {{...}} | `{{quote}}` | {{risk}} | {{fix}} |

### Minor (P2) — worth fixing, not blocking

| ID | File:line | Fix |
|----|-----------|-----|
| F-5 | `{{path:line}}` | {{one sentence}} |

### Nits (P3) — preference; author may decline

- **F-M** `{{path:line}}` — {{title}}. {{one-line fix}}.

## Positives

- {{reviewer}}: {{one-line observation}}

## Residual risks & testing gaps

- **Risk:** {{one-line, reviewer attribution}}
- **Testing gap:** {{what is untested and where}}

## Plan alignment

_Only when a plan document is present._

- **Plan:** `{{path}}`
- **Criteria satisfied:** {{N of M}}. **Unmet:** {{list or "none"}}
- **Non-goals respected:** {{yes / see F-N}}. **ADRs consistent:** {{yes / see F-N}}
- **Silent decision changes:** {{none / see F-N}}

## Next steps

1. {{e.g. address F-1, F-2 (P0)}}
2. {{e.g. decide F-3 (P1): accept residual risk or fix}}
3. {{e.g. optional re-run after fixes}}

<!--
Optional sections — include ONLY when non-empty:

### Suppressed findings (< 0.60 confidence)

| ID | Sev | Conf | Title | File:line | Reviewer |
|----|-----|-----:|-------|-----------|----------|
| S-1 | P2 | 0.55 | {{title}} | `{{path:line}}` | {{reviewer}} |

### Pre-existing findings (outside this diff)

| ID | Sev | Title | File:line | Reviewer |
|----|-----|-------|-----------|----------|
| PE-1 | P1 | {{title}} | `{{path:line}}` | {{reviewer}} |

### Diff inventory

| File | +LOC | -LOC | Reviewed by |
|------|-----:|-----:|-------------|
| `{{path}}` | {{n}} | {{n}} | {{reviewers}} |

Files no reviewer opened: {{none / list — a gap signal}}.

Cut any section above if empty. See _shared/artifact-compactness.md.

Re-review append target: new `## Re-review <DATE>` sections are added below
on subsequent runs. Each block follows code-review SKILL.md Phase 9:

## Re-review {{DATE}}

**Scope:** {{TIER}}. **Reviewers:** {{list}}.
**Findings:** P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}}.

### Verification of prior fixes

| Prior claim | F-N current | Sev | Status | Notes |
|-------------|-------------|-----|--------|-------|
| F-1 (at `abc1234`) | — | P0 | verified / mismatch / still present / vanished / orphaned | {{notes}} |

### What changed since last review

- {{one-line delta}}

### New findings

[... same shape as main ...]
-->
