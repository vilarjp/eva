---
name: audit
date: "{{DATE}}"
status: "{{draft | approved}}"
topic: "{{short topic — what was audited}}"
target: "{{path | glob | directory}}"
focus: "{{omit if full audit, else: short focus phrase}}"
scope: "{{LIGHTWEIGHT | STANDARD | DEEP}}"
target_stats: "{{files: N, LOC: approx}}"
stack: "{{language + framework, e.g. 'TypeScript / React / Vitest'}}"
---

# Audit: {{topic}}

**Target:** `{{target}}` — {{N files, approx LOC}}. **Focus:** {{full | phrase}}.
**Scope:** {{LIGHTWEIGHT | STANDARD | DEEP}}. **Stack:** {{stack}}.

## TL;DR

**Posture:** {{one sentence — "clean surface, P3 only" / "material debt, N P0+P1" / "structural — refactor track"}}

| P0 | P1 | P2 | P3 | Bugs surfaced | Arch observations |
|---:|---:|---:|---:|--------------:|------------------:|
| {{n}} | {{n}} | {{n}} | {{n}} | {{n}} *(→ /diagnosis)* | {{n}} |

{{one-line verdict — e.g. "Route bugs to /diagnosis first; /execute REFACTOR on P0 + P1."}}

## Bugs surfaced

_Live defects found during the audit — route to `/diagnosis`, not debt. Omit heading if none._

| # | File:line | Shape | Evidence |
|---|-----------|-------|----------|
| B-1 | `{{path:line}}` | {{crash/data loss/silent/security}} | `{{verbatim quote}}` |

## Findings

One row per finding. Severity blocks keep headings; empty blocks are cut.

### Blocker debt (P0) — active incident risk

| ID | File:line | Smell | Evidence | Cost | Refactor shape |
|----|-----------|-------|----------|------|----------------|
| F-1 | `{{path:line}}` | {{smell from 3-layer vocab}} | `{{verbatim 5-30-word quote}}` | {{one sentence}} | {{one sentence, NOT code}} |

### Major debt (P1) — incident-in-quarter risk OR blocks known change

| ID | File:line | Smell | Evidence | Cost | Refactor shape |
|----|-----------|-------|----------|------|----------------|
| F-3 | `{{path:line}}` | {{smell}} | `{{quote}}` | {{cost}} | {{shape}} |

### Minor debt (P2) — maintenance drag, no active risk

| ID | File:line | Smell | Refactor shape |
|----|-----------|-------|----------------|
| F-5 | `{{path:line}}` | {{smell}} | {{one sentence}} |

### Nits (P3) — small cleanup, defer freely

- **F-N** `{{path:line}}` — {{title}}. {{smell}}. {{one-line shape}}.

## Architecture observations

_Cross-cutting shapes bigger than any single finding. Omit on Lightweight or when nothing cross-cutting emerged._

| ID | Shape | Proof | Cost | Refactor shape | Hint |
|----|-------|-------|------|----------------|------|
| A-1 | {{one-sentence concern}} | F-{{N}}, F-{{M}} | {{concrete pain}} | {{one-sentence extraction}} | {{P0/P1/P2}} |

## Positives

- {{file / module}}: {{one-line observation — calibration anchor}}

## Suppressed candidates

_Considered and dropped — keeps the audit honest. Omit if empty._

| Candidate | Smell | Why dropped |
|-----------|-------|-------------|
| `{{path:line}}` | {{smell}} | {{ADR-3 / shared-kernel / exemplar}} |

## Handoff — `/execute` REFACTOR mode

| ID | Severity | File:line | Smell | Intent |
|----|----------|-----------|-------|--------|
| F-1 | P0 | `{{path:line}}` | {{smell}} | {{refactor shape}} |

Architecture observations and bugs surfaced are NOT in this table.

## Next steps

1. {{e.g. route B-1 via `/diagnosis` before refactor}}
2. {{e.g. `/execute` REFACTOR on F-1, F-2, F-3; defer P2/P3}}
3. {{e.g. re-run `/audit` after REFACTOR to append `## Re-audit <DATE>`}}

<!--
Optional sections — include ONLY when they add signal:
- Target inventory (table of files read + smells-found) — on Standard/Deep
  when the scope is ambiguous
- Diagram (mermaid) — rarely useful for audits; skip by default

Cut any section above if empty. See _shared/artifact-compactness.md.

Re-audit append target: new `## Re-audit <DATE>` sections are added below on
subsequent runs. Each block follows audit SKILL.md Phase 0.1:

## Re-audit {{DATE}}

**Scope:** {{TIER}}. **Target:** {{path | unchanged}}.
**Findings:** P0:{{n}} P1:{{n}} P2:{{n}} P3:{{n}}.

### Verification of prior findings

| Prior finding | Status | Notes |
|---------------|--------|-------|
| F-1 (addressed at `abc1234`) | verified / still present / vanished / orphaned | {{notes}} |

### What changed since last audit

- {{one-line delta}}

### New findings

[... same shape as main artifact ...]
-->
