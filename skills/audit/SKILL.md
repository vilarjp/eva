---
name: audit
description: "Audit existing, stable code for structural debt — not diff regressions, not reported bugs — and produce a handoff-ready list of refactor opportunities. Use when the user says \"audit this module\", \"survey the tech debt\", \"find smells in src/checkout\", \"what needs refactoring here\", \"audit for code smells\", \"inventory the debt\", or invokes /audit. Produces docs/YYYY-MM-DD-slug/AUDIT.md with findings grouped by severity (P0 blocker debt / P1 major / P2 minor / P3 nit), each carrying file:line evidence, a named smell category from Fowler's vocabulary or _shared/no-workarounds (TYPE / LINT / SWALLOW / TIMING / PATCH / SCATTER / CLONE), a one-sentence suggested refactor shape, and a handoff record that /execute REFACTOR mode consumes directly. Scope-adaptive (Lightweight / Standard / Deep); Deep fans out 3-5 parallel Explore agents across the target's submodules and consumer surfaces. Read-only — never edits source, never commits, never invokes /execute. Distinct from /code-review (which reads a diff), /diagnosis (which investigates a reported bug), and /teardown (which maps unfamiliar behaviour); /audit surveys stable code that works but could be cleaner, safer, or more evolvable."
argument-hint: "[target path, directory, or glob, optionally followed by 'focus on <area>' or a scope override (lightweight|standard|deep)]"
---

# Audit — Survey stable code for structural debt

Turn an existing module, directory, or file surface into a ranked list of refactor opportunities through codebase pre-scan, smell inventory, architecture observation, and severity calibration. The durable output is a single file: `docs/YYYY-MM-DD-<slug>/AUDIT.md`.

This skill does NOT write code. It reads, classifies, and reports. Applying the refactors is a separate turn, via `/execute` in REFACTOR mode.

## When to use

Trigger this skill when the user:
- says "audit this module", "audit src/<path>", "find smells in <area>", "survey the tech debt", "inventory the debt", "what needs refactoring", "walk the debt in <module>"
- invokes `/audit` (optionally with a path, directory, or focus phrase)
- wants a static, snapshot-level view of refactor opportunities in stable code — code that works but could be cleaner, safer, or more evolvable

Do NOT trigger for:
- A review of a diff or PR — that is `/code-review` or `/pr-feedback`
- Investigation of a reported bug — that is `/diagnosis`
- Reverse-engineering unfamiliar code to understand *what it does* — that is `/teardown`
- Actually applying the refactors — that is `/execute` in REFACTOR mode, after this skill completes

## The Iron Law

**No smell without code evidence. No severity without a stated cost.**

Every audit finding must quote `file:line` and name the specific shape being flagged. A finding that says *"this module feels complex"* without a named smell and a quoted line is noise. Every severity tier must be justified by a cost the next reader can recognise — an incident shape, a change-cost, a future migration pain. *"This is debt"* is not a cost; *"adding a fourth currency forces edits across 11 files"* is.

## Core invariants

1. **Read-only.** No source edits, no commits, no pushes. Writes only `AUDIT.md`.
2. **Named smells only.** Every finding cites a category from the Fowler vocabulary or `../_shared/no-workarounds.md`. Unnamed smells get rewritten or dropped.
3. **Verbatim evidence.** Every finding includes a 5-30-word quote from the target code (with `file:line`). Paraphrase is suppressed.
4. **Severity is a claim, not a vibe.** Each severity tier (P0 / P1 / P2 / P3) carries a stated cost per `references/scope-tiers.md`. A severity without a cost is downgraded or dropped.
5. **Static snapshot, not diff.** The target is stable code that already merged — not a current change set. If the user asks to audit a diff, redirect to `/code-review`.
6. **No refactor code in the artifact.** The Suggested-refactor column is one sentence describing the *shape* of the fix, never a code block. Code lives in the eventual `/execute` REFACTOR slices.
7. **Separate blocker debt from active bugs.** If a finding is a real current bug (crash, data loss, security hole), it goes in a dedicated **Bugs surfaced** section at the top of the artifact and the skill recommends running `/diagnosis` — an audit does not pretend to be a bug tracker.
8. **Ask ONE question at a time. Max 3 total.** The target code is the source of truth; prefer reading to interviewing.
9. **Emit the self-review checklist** to the user before Gate. Skipping is a protocol violation.
10. **HARD GATE: do not write `AUDIT.md` until the user approves the findings and the output path.**
11. **Handoff record, not handoff call.** The artifact tells `/execute` REFACTOR what to apply and where; the skill never invokes `/execute` itself.
12. **Pre-scan findings are internal during the Deep parallel fan-out.** Sub-agent reports merge into the findings list only after the skill has classified and severity-calibrated them.

## Pre-flight (MANDATORY)

Before anything else:

1. Get today's date: `date +%Y-%m-%d` via Bash. Store as `<DATE>`.
2. If the invocation has no target (bare `/audit`), ask: *"What should I audit? A file, a directory, a module path, or a glob."* Do not proceed without a concrete target.
3. Resolve the target:
   - **Path or glob:** verify at least one file matches. If none, STOP and ask.
   - **Focus phrase** (*"audit the checkout flow"*, *"focus on concurrency in the scheduler"*): record as `audit_focus` — the phrase narrows what findings are surfaced, not which files are read.
4. Announce the skill is active: `audit: starting (target <resolved>, scope TBD)`.

## Process

Execute phases in order. Do not skip.

### Phase 0 — Resume + scope

**0.1 Resume check.** Look in `docs/*/` for an existing `AUDIT.md` whose frontmatter `target` matches this target. If found, ASK:

```
Found existing AUDIT.md at <path> for this target. What would you like to do?
  A) Append — add a ## Re-audit <DATE> section below the existing audit [Recommended after refactor work]
  B) Overwrite — start fresh (old audit preserved in git history)
  C) Write to a new file (AUDIT-N.md) — preserve both
```

Default A. Overwrites require explicit confirmation.

**0.2 Scope classification.** Use target size + structure:

| Tier | Criteria | Ceremony |
|------|----------|----------|
| **Lightweight** | 1-3 files, <500 LOC total, single module. | Single-pass read. Findings table only. Skip architecture observations. 0-1 clarifying Qs. |
| **Standard** | 4-15 files, 500-3000 LOC, one cohesive module or feature. | Full phases. Architecture observations required. 1-2 Qs. |
| **Deep** | 15+ files, 3000+ LOC, cross-module, shared kernels, or "audit the whole service". | Full phases + parallel `Explore` fan-out in Phase 1 + mandatory consumer-surface sweep + Hyrum's-Law section. |

Announce: `audit: scope = <TIER>`. Full rubric: `references/scope-tiers.md`.

### Phase 1 — Codebase pre-scan

**1.1 Constraint check.** Read `AGENTS.md`, `CLAUDE.md`, `README.md` for workflow / style / scope constraints that should calibrate findings (e.g., *"we prefer DAMP over DRY in tests"* changes which Duplicate-Code findings matter).

**1.2 Target read.** For Lightweight / Standard: read every file in the target surface end-to-end, plus the nearest test file for each production file. For Deep: fan out.

**1.3 Deep scope parallel fan-out.** Deep scope **fans out the pre-scan into 3-5 parallel `Explore` agent calls in a single message**:
- One `Explore` call per distinct submodule under the target.
- One `Explore` call for the primary consumer surface(s) — the callers that depend on the target.
- One `Explore` call for shared utilities / helpers the target imports.

Each agent returns: inventory (functions / classes / exported API), call graph within its slice, and unreplaced conventions it noticed. The orchestrator merges the reports into a single working-memory model before Phase 2. This shortens the total read while covering more surface — a single-pass read on 15+ files under-audits.

**1.4 Pattern exemplar.** Find ONE working example per smell category you plan to flag (e.g., a place in the codebase where a value object was correctly extracted, or where a boundary validator was used instead of SCATTER). Read it. Calibrate: *"what does 'good' look like in THIS codebase?"* A smell judged against an external standard produces false positives; a smell judged against the repo's own best examples is actionable.

**1.5 Test-shape pre-read.** Skim 2-3 test files covering the target. Note the test framework, assertion style, and fixture convention — findings about test-layer smells must match the repo's posture, not an imagined one.

Announce: `audit: context loaded — <n> files, <n> submodules <if Deep>, <n> consumer surfaces.`

### Phase 2 — Smell inventory (individual findings)

Walk the target through the two smell catalogues. Each candidate finding must:

1. **Cite `file:line` + a 5-30-word quote.**
2. **Name the smell category** — one of the Fowler vocabulary entries or one of `TYPE / LINT / SWALLOW / TIMING / PATCH / SCATTER / CLONE` from `../_shared/no-workarounds.md`. Unnamed = dropped.
3. **State the cost** — *"adding a currency forces edits in 11 files"*, *"every new caller re-implements the same null guard"*, *"the swallowed error hides a retry that never fires"*.
4. **Propose a refactor shape in one sentence** — *"move `calculateTotal` to `Cart` — the four reads it does belong there"*, *"parse `response.data` through `UserSchema` at the boundary, drop the downstream null-guards"*. No code blocks.

See `references/smell-playbook.md` for the full catalogue with gate questions and disambiguation.

**Suppress false positives.** If a candidate smell was deliberate (documented ADR, known trade-off, shared-kernel constraint), drop it from the inventory. Over-surfacing drowns the real findings.

### Phase 3 — Architecture observations

Cross-cutting concerns that are bigger than any single smell. Required for Standard + Deep, optional for Lightweight.

Walk these lenses:

- **Layering violations.** Does a domain module import from infrastructure? Does a shared kernel depend on a feature module?
- **Missing seams.** Is there a concept stretched across multiple classes that wants to be one type (Primitive Obsession scaled up)? Is there a workflow implemented as a series of orchestration calls that wants a single state machine?
- **Cohesion drift.** A module whose public API exports 12 unrelated concerns is doing too many jobs (Divergent Change). A module with one concern smeared across 6 files is doing one job in too many places (Shotgun Surgery).
- **Test layer smells.** Tests that assert on mocks, tests coupled to implementation detail, branches that are provably unexercised. Cite the test-reviewer's five named anti-patterns (`agents/code-review-test-reviewer.md`) by name.
- **Evolvability.** What change is likely in the next 2 quarters (per PRD / roadmap / comments)? Does the current shape welcome it or resist it? A shape that resists a likely change is a P1 observation even if no individual smell is P1.

Architecture observations do NOT replace individual findings — they sit above them. If an architecture observation reduces to "Shotgun Surgery across `src/billing/`", emit both: one architecture observation (the shape) and the individual Shotgun Surgery findings that prove it.

### Phase 4 — Severity calibration

For each finding (individual + architecture), assign a tier:

| Tier | Cost band | Signal |
|---|---|---|
| **P0 — Blocker debt** | Active incident risk. A user-visible failure mode is one pager away, or a security invariant is implicit. | Unenforced validation at the boundary (SCATTER with a null check), a SWALLOW on a critical path, a TIMING retry masking a race that hits in prod, a test that proves nothing on a module the user relies on. |
| **P1 — Major debt** | Likely to cause an incident in a quarter, OR blocks a known-likely change. | Shotgun Surgery for a feature currently on the roadmap; Primitive Obsession on a domain concept whose validation has already drifted twice; Feature Envy that forces a coming feature to touch three classes. |
| **P2 — Minor debt** | Makes maintenance harder. No active risk. | Middle Man class with no foreseeable consumer; Long Parameter List on a cold path; occasional Data Clumps without active change pressure. |
| **P3 — Nit** | Small cleanup. | Naming drift, Comments as Deodorant, a single Duplicate Code instance with no drift yet. |

**Calibration rules:**
- A finding without a named cost does not clear P2.
- A finding flagged as P0 must identify the incident shape, not just the risk.
- P1s cap at ~10 per Standard audit, ~20 per Deep. If you exceed, re-calibrate — some are actually P2.
- **Surfaced bugs leave the audit.** If a finding is a real current bug (data loss, crash, security hole), move it to the top-level **Bugs surfaced** section and flag `/diagnosis` as the next step. Audit findings are about debt, not live incidents.

### Phase 5 — Handoff records

For every finding (P0, P1, P2, P3 — not architecture observations, which are human-read), build a handoff record for `/execute` REFACTOR mode:

```
{
  id: "F-N",
  severity: "P0 | P1 | P2 | P3",
  file, line,
  smell: "<category name>",
  title: "<one-line finding title>",
  intent: "<one-line suggested refactor shape>",
  evidence: "<verbatim quote from source>"
}
```

Architecture observations are described in prose in the artifact and are not handed off directly — they inform which individual findings to prioritise, which is a human call.

### Phase 6 — Self-review checklist (MANDATORY, emit to user)

Before Gate, emit with `✓` / `✗` on each line:

- [ ] Target is a stable snapshot (not a current diff); redirected to `/code-review` if the user wanted a diff review
- [ ] Every finding cites a named smell category (Fowler or `../_shared/no-workarounds.md`) — no unnamed findings
- [ ] Every finding quotes `file:line` with a 5-30-word verbatim excerpt
- [ ] Every finding states a concrete cost, not a vibe
- [ ] Every severity (P0 / P1 / P2 / P3) is justified by the cost band in `references/scope-tiers.md`
- [ ] P0 findings name the incident shape, not just the risk
- [ ] Real current bugs moved to **Bugs surfaced** section with a `/diagnosis` pointer
- [ ] Suggested-refactor column is one sentence per finding — NO code blocks
- [ ] Architecture observations (Standard + Deep) name the shape and cite the individual findings that prove it
- [ ] Deep scope: parallel `Explore` fan-out was used (3-5 agents); merged findings before classification
- [ ] Deep scope: Hyrum's-Law review performed on exported surfaces
- [ ] Handoff records built for every individual finding (id, severity, file, line, smell, intent, evidence)
- [ ] False positives suppressed — documented trade-offs, intentional patterns, shared-kernel constraints are not flagged
- [ ] P1 count ≤ 10 for Standard / ≤ 20 for Deep — if higher, re-calibrated
- [ ] No source file was edited; no commit or push was made
- [ ] No `TODO`, `[TBD]`, placeholder text in the proposed artifact

If any `✗` → STOP, fix, re-emit. Do not advance.

### Phase 7 — HARD GATE: approve findings + output path

Present via `AskUserQuestion`:

```
Ready to write AUDIT.md.

Summary:
- Target:     <path | glob>
- Focus:      <phrase | "full audit">
- Scope:      <LIGHTWEIGHT | STANDARD | DEEP>
- Findings:
    P0 Blocker: <n>
    P1 Major:   <n>
    P2 Minor:   <n>
    P3 Nit:     <n>
    Bugs surfaced: <n>  (→ /diagnosis)
    Architecture observations: <n>
- Proposed path: docs/<DATE>-<slug>/AUDIT.md

Approve and write, or describe changes?
  A) Approve — write at the path above [Recommended]
  B) Reclassify a finding (change severity, drop, or merge)
  C) Add a focus — narrow to one area
  D) Change the slug / target path
  E) Abort — do not write
```

**STOP HERE.** Do not create the directory. Do not write `AUDIT.md`. WAIT for explicit approval.

If the user picks **B** or **C**, loop back to the relevant phase, then re-emit Phase 6 before re-gating.

### Phase 8 — Write AUDIT.md

Only after explicit approval:

1. `mkdir -p docs/<DATE>-<slug>/` (skip if reusing an existing folder).
2. Render using `templates/AUDIT.md`. Replace every `{{placeholder}}`. Set `status: approved` in frontmatter.
3. **If appending a re-audit:** do NOT overwrite. Append a `## Re-audit <DATE>` section with the same shape as a fresh audit. Prior sections are read-only history.
4. Confirm:

```
AUDIT written: <path>
  P0: <n> · P1: <n> · P2: <n> · P3: <n>  (Bugs surfaced: <n>)
```

The artifact MUST be complete — no placeholders, readable without this conversation.

### Transition

After writing, inform the user in one sentence:

*"Audit complete. To apply selected findings as refactors, run `/execute` in REFACTOR mode on this folder — it will triage which findings to address, slice them TDD-style, and back-point this file with a `## Refactors applied <DATE>` section. Surfaced bugs (if any) should run through `/diagnosis` first."*

Do NOT invoke `/execute`. Do NOT invoke `/diagnosis`. The skill is complete.

## Anti-patterns

- **Auditing a diff.** The wrong skill. Redirect to `/code-review`.
- **Auditing without named smells.** Every finding must name a category. "This feels complex" is not a finding.
- **Treating the audit as a bug tracker.** Real current bugs go to **Bugs surfaced** with a `/diagnosis` pointer. Audit surveys debt, not live incidents.
- **Proposing refactor code.** Never. One sentence per finding; code lives in `/execute` REFACTOR slices.
- **Over-flagging test files.** Test-layer smells exist, but *"this test could be DRY-er"* is a nit at best. Flag tests only when the anti-pattern is one of the test-reviewer's five named patterns.
- **Ignoring the repo's own conventions.** If the codebase prefers DAMP tests and you flag DRY opportunities, you are auditing against the wrong standard. Calibrate against Phase 1.4 pattern exemplars.
- **Architecture observations without supporting findings.** If you claim "Shotgun Surgery across `src/billing/`", cite at least two individual Shotgun-Surgery findings that prove it.
- **Flagging Speculative Generality on code that has two consumers.** Look before you flag — Speculative Generality means ONE implementation and NO foreseeable second.
- **Running `/execute` automatically after writing.** Pure reporter. The human decides which findings to address.

## Red flags — self-check

STOP if you catch yourself:

- A finding carries no named smell category
- A P0 finding has no stated incident shape, just "risky"
- You quoted a line from memory instead of reading the file
- You audited a diff instead of the stable snapshot
- You wrote a code block in the Suggested-refactor column
- You edited a source file, staged, committed, or pushed
- You wrote `AUDIT.md` before Phase 7 approval
- You surfaced a real current bug as a P0 debt finding instead of routing it to `/diagnosis`
- Your P1 count exceeds 10 (Standard) or 20 (Deep) — under-calibrated
- You flagged a pattern as a smell without finding the repo's own exemplar of the alternative
- Deep scope and you did NOT use the parallel `Explore` fan-out — single-pass read under-audits at this scale
- You invoked `/execute` or `/diagnosis` automatically after writing
- You called this "the complete list of problems" — audits are samples calibrated to the scope tier, not exhaustive sweeps

## References

- `templates/AUDIT.md` — the output template. Read only at Phase 8.
- `references/scope-tiers.md` — detailed Lightweight / Standard / Deep rubric with severity bands and per-tier ceilings.
- `references/smell-playbook.md` — full Fowler + `_shared/no-workarounds` catalogue with gate questions, disambiguation, and example findings. Load in Phase 2.
- `../_shared/no-workarounds.md` — seven workaround categories (TYPE / LINT / SWALLOW / TIMING / PATCH / SCATTER / CLONE) and the five-condition Escape Valve. Findings in this catalogue are often P0 / P1 because they mask root causes.
- `agents/code-review-quality-reviewer.md` — the Fowler vocabulary entries (Feature Envy, Primitive Obsession, Data Clumps, Shotgun Surgery, Divergent Change, etc.) are the same set used here.
- `agents/code-review-test-reviewer.md` — the five named test anti-patterns (mock-behaviour assertion, test-only production methods, mocking without understanding, incomplete mocks, tests-as-afterthought) are the vocabulary for Phase 3 test-layer observations.
