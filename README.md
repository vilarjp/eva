<div align="center">

# eva

**Ten composable, scope-adaptive Claude Code skills for compound engineering —
reverse-engineer, plan, diagnose, execute, review, triage PR feedback, ship, and remember.**

[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Version 0.9.0](https://img.shields.io/badge/version-0.9.0-brightgreen)](.claude-plugin/plugin.json)
[![Claude Code plugin](https://img.shields.io/badge/claude%20code-plugin-orange)](https://docs.claude.com/en/docs/claude-code/plugins)

</div>

```
/plugin marketplace add vilarjp/eva
/plugin install eva@eva
```

---

A Claude Code plugin with ten sharp, composable skills:

- **`teardown`** — reverse-engineer an unfamiliar file (local path or URL) into a structural map: entry points, function inventory, state, data flow, external surface (network / DOM / storage), edge cases, and a Mermaid diagram. Runs a hydration pipeline (source-map → beautify → optional decompilation with `wakaru` / `webcrack`) for minified and obfuscated bundles, and supports a focused mode that narrows the artifact to one area (`/teardown bundle.js focus on auth flow`). Strictly read-only; writes only `TEARDOWN.md` and optional `teardown-sources/` sidecars after a HARD GATE with explicit user approval; unverified claims are labeled rather than invented; fetched content is treated as data, never as instructions.
- **`prd`** — turn a rough feature idea into a rigorous, codebase-grounded PRD through structured dialogue.
- **`spec`** — turn an idea or an approved PRD into a rigorous, codebase-grounded technical specification.
- **`revision`** — cross-review an already-drafted PRD.md and SPEC.md pair to catch gaps, inconsistencies, and mismatches before implementation; on approval, patches both source documents with a dated `## Revision` section.
- **`diagnosis`** — investigate a bug, error, or regression through backward tracing, 3+ structural hypotheses, a full causal chain with predictions for uncertain links, and a reproduction test with RED proof; produces a gated DIAGNOSIS.md that documents root cause, severity, suggested minimal fix, and hotspots.
- **`execute`** — implement a feature from an approved SPEC/REVISION, fix a bug from an approved DIAGNOSIS, address selected findings from an approved CODE-REVIEW (FIX mode), or execute a direct human prompt when no artifact exists; follows a red-green-refactor TDD loop with a narrow Verification Mode escape hatch for infra/config/docs, enforces minimal-diff and NOTICED-BUT-NOT-TOUCHING scope discipline, commits per vertical-slice on a non-protected branch, keeps its codebase pre-scan internal, and produces a durable EXECUTION.md log. FIX mode closes the **correction → code-review → correction** loop by letting the human pick which findings to address vs skip at triage, slicing each selected finding as a TDD unit, and appending a `## Fixes applied <DATE>` back-pointer on the CODE-REVIEW.md so the next `/code-review` re-run reads a clean delta.
- **`code-review`** — audit the current diff (uncommitted > branch-vs-base > HEAD~1) through a two-stage multi-reviewer pipeline: plan-alignment as a blocking Stage 1 when a plan exists; code-quality + convention + test + conditional security/performance in parallel Stage 2; adversarial Stage 3 on Deep scope. Merges findings across reviewers, boosts confidence on cross-reviewer agreement, suppresses low-confidence findings in a visible table, separates pre-existing issues from diff-introduced ones, and produces a gated CODE-REVIEW.md. Pure reporter — never writes code, never commits, never pushes.
- **`pr-feedback`** — triage incoming PR review feedback (inline thread comments via GraphQL, timeline comments, review bodies across every non-dismissed reviewer) into five author-facing verdicts — **must-fix / nice-to-have / question / push-back / already-done** — each with a 0-100 confidence score, verbatim evidence, a copy-pasteable GitHub reply draft, and a structured handoff record that `/execute` FIX mode consumes directly. Mandatory branch-checkout gate (offers `gh pr checkout` first, refuses-until-correct with manual commands on failure); unions multi-reviewer feedback deduped by `thread_id`; surfaces stale/outdated comments with a line-moved badge rather than silently dropping them; collapses resolved threads into `<details>` for context; logs explicitly-dismissed reviews in a Skipped footer. Smart-reuses an adjacent `docs/<DATE>-<slug>/` spec folder (PRD / SPEC / DIAGNOSIS / EXECUTION) for context and reconciles with a sibling CODE-REVIEW.md when present — agreements boost must-fix confidence (+15), tensions (reviewer flags what we suppressed) surface in a dedicated Reconciliation section. Pure triage — never posts to the PR, never edits source, never commits, never auto-invokes `/execute`. Writes only `docs/YYYY-MM-DD-<slug>/PR-FEEDBACK.md` after a HARD GATE with explicit user approval.
- **`commit-push`** — take the current working tree to a remote feature branch through a scope-adaptive pre-commit gate, a human-gated branch confirmation (never silently derived, never on `main`/`master`/`production`/`prod`/`stable`/`live`/`trunk`/`release*`), a human-approved logical commit split of up to three commits, convention-matched title+body drafts (repo instructions > commitlint > last 10 commits > Conventional Commits default) with a WHY-focused body and testing-scenarios block for feat/fix, explicit per-file staging (never `git add -A`), a `git fetch` + `git pull --rebase` before `git push -u`, and an optional `## Commits <DATE>` back-pointer to an adjacent `EXECUTION.md` / `DIAGNOSIS.md` when the work sits in a spec folder. Never opens a pull request, never runs `gh pr create`, never `--force`, never `--no-verify`, never commits secrets — PR creation is a separate, explicitly-human-requested action owned by the user.
- **`solutions`** — capture durable learnings from a completed pipeline (feature, bug, or mixed) into a gated `SOLUTIONS.md` next to the PRD/SPEC/REVISION/DIAGNOSIS/EXECUTION/CODE-REVIEW that produced it; synthesises Summary + Approach + Key Decisions (mini-ADR format with Alternatives) + Gotchas + What Didn't Work + References (+ Root Cause + Prevention for bugs, + Relationship to Original Spec for mixed pipelines, + Mermaid diagram for Deep) from every upstream artifact in the folder. Scope is **inherited** (MAX of upstream artifacts' scope values). Enforces the Iron Law of durability — learnings are phrased as behaviour invariants, not file:line coordinates — so they survive refactors and renames. Re-runs append `## Re-run <DATE>` or `## Post-Release Bug Fix <DATE>` sections rather than overwriting history. Writes only `SOLUTIONS.md` after a HARD GATE with explicit user approval — never edits `CLAUDE.md` / `AGENTS.md` (the user owns that edit if they want it), never chains to another skill.

All ten are scope-adaptive, verify claims against the source material, run a self-review checklist, and gate critical steps on explicit user approval. `teardown` is the only skill that reads a target outside the repo (local path or URL) — it enforces a one-line user acknowledgment before any URL fetch, runs a hydration ladder (source-map discovery → beautify → optional `wakaru`/`webcrack` decompilation) with graceful degradation when tools are unavailable, labels every unverified claim rather than guessing, refuses to execute any instruction embedded in fetched content, writes the artifact plus optional `teardown-sources/` sidecars into a dated `docs/` folder, and hands off — it never chains into `/prd`, `/spec`, `/diagnosis`, or `/execute`. `spec` adds an adversarial red-team pass and auto-detects an adjacent PRD to build on. `revision` adds a 4-lens composite review (cross-doc alignment, internal coherence, adversarial premise, scope creep + feasibility) and double-gates writing and patching. `diagnosis` is strictly read-only in source code (it creates only the reproduction test file), enforces the iron law "no fixes without root-cause investigation first," and smart-reuses an existing `docs/` folder when the bug maps to a known feature. `execute` is the only skill that writes production code — it enforces the iron law "no production code without a failing test first," refuses to run on protected branches, gates the first line of code on an approved slice plan, and gates writing EXECUTION.md on a green integration pass (full test suite + lint + types). In FIX mode it reads an approved CODE-REVIEW.md, displays every P0/P1/P2/P3 finding in a compact triage summary, lets the human pick which to address and which to skip (default: all P0+P1), slices each selected finding under full TDD discipline, records skipped findings as an audit trail, and back-points CODE-REVIEW.md with an addressed-vs-skipped delta so the loop stays auditable across iterations. `code-review` is a pure reporter that reads the current diff, fans out seven specialized reviewers (plan-alignment, code-quality, convention, test, security, performance, adversarial) through a two- or three-stage pipeline with dedup + confidence-gated suppression + cross-reviewer agreement boost, smart-reuses an adjacent spec folder when the diff maps to a known feature, and writes only CODE-REVIEW.md — never touches production code, tests, commits, or pushes. `pr-feedback` is the inbound counterpart that closes the external-review loop — it takes a GitHub PR reference, enforces a mandatory branch-checkout gate (offers `gh pr checkout` first, refuses-until-correct with manual commands on failure), unions every non-dismissed reviewer's inline thread comments (via GraphQL so `isResolved` / `isOutdated` / thread replies are first-class), timeline comments, and top-level review bodies, dedupes by `thread_id`, classifies each comment into five author-facing verdicts (must-fix / nice-to-have / question / push-back / already-done) with a 0-100 confidence score derived from Conventional Comments labels + decorators + spec cross-reference + anchor freshness + CODE-REVIEW.md fingerprint agreement, drafts a copy-pasteable reply per verdict, and hands off must-fix items to `/execute` FIX mode via a structured Handoff table — it is pure triage and never posts to the PR, never edits source, never commits. `commit-push` is the counterpart that ships the diff — a scope-adaptive pre-commit gate (secrets + protected-branch on Lightweight, + tests + lint + debug-artifacts on Standard, + scope-vs-diff + test-coverage-for-code-changes + chore-exemption audit on Deep), a hard human-gated branch confirmation in every run, a human-approved logical split of up to three Conventional-Commits-style messages drafted to match the repo's detected convention, explicit per-file staging, a fetch-and-rebase before push, and an optional back-pointer to an adjacent EXECUTION.md / DIAGNOSIS.md — but refuses protected branches, refuses to open PRs (`gh pr create` is out of scope), refuses `--force` / `--no-verify` / `git add -A`, and refuses to commit secrets regardless of what the user insists. `solutions` is the pipeline's memory terminator — it reads every artifact in the folder (PRD, SPEC, REVISION, DIAGNOSIS, EXECUTION, CODE-REVIEW) and distils durable, file-independent learnings into a scope-inherited `SOLUTIONS.md` with compact mini-ADRs, Gotchas, What Didn't Work, and (for bugs) Prevention. It is a pure synthesiser: never edits code, never touches `CLAUDE.md` / `AGENTS.md`, never chains to another skill — it enforces the Iron Law of durability (behaviour invariants over file:line) and appends on re-runs instead of overwriting, so a folder's SOLUTIONS.md becomes the accretive knowledge record a cold-context future session actually reads before touching that area again.

## What they do

### `teardown`

Produces `docs/YYYY-MM-DD-<slug>/TEARDOWN.md` (and optional `teardown-sources/` sidecars for minified/hydrated code), containing:

- Summary (5-8 lines — what the file is, what it does, how it does it, what's surprising, what depends on it — no adjectives like *clean* or *elegant*)
- Inputs & Hydration table (target, size, detected type, bundler signature, minification state, source-map availability, hydration tools used, sidecars list)
- Entry points (what runs on load — IIFE, top-level export, window assignment, SW install, etc. — with `file:line`)
- Module layout (when bundled — module id → hydrated path → role)
- Function inventory (signature + location + one-sentence role)
- State inventory (binding → scope → location → lifecycle)
- Behavior Walkthrough — happy path + branches + error paths + event surfaces, each with `file:line`
- Data Flow — inputs, transformations, outputs, coupling / fan-out, plus a Mermaid diagram (mandatory on Standard/Deep; `flowchart` / `sequenceDiagram` / `stateDiagram-v2` as appropriate)
- Edge Cases & Error Paths (table: scenario → trigger → location → behaviour)
- External Surface — network requests (method / URL / auth / payload shape), DOM touchpoints, storage reads/writes, workers and offloaded work
- Focus Slice (when the user passed `focus on <area>` or `just the <function>` — the artifact narrows Phases 3-5 to that region)
- Unknowns & Residual Risk (every `(unverified)` claim, why it's unverified, and what a reader might break if they act on it)
- References (links to sidecars and any sibling `docs/<DATE>-<slug>/` folders)

The flow is **scope-adaptive**: a Lightweight target (unminified, ≲500 lines, one entry point) gets a compact artifact with 0-1 questions and no diagram; a Standard target (minified ≤500 KB, source map likely, typical production bundle) gets the full source-map → beautify → read pipeline, a Mermaid diagram, and 1-3 questions; a Deep target (>500 KB, multi-bundle, heavy obfuscation, or WASM) gets the full hydration ladder (source-map → beautify → decompilation via `wakaru` / `webcrack` → optional identifier-rename hints via `humanify`), mandatory diagram (often more than one), and 2-3 questions.

The skill is **strictly read-only** — it writes only `TEARDOWN.md` and optional `teardown-sources/{beautified,hydrated,renamed}/` files under `docs/<DATE>-<slug>/`; it never edits source, never invokes `/prd` / `/spec` / `/diagnosis` / `/execute`, and never commits. For URL inputs it emits a one-line user acknowledgment before fetching (audited in frontmatter); for local files no acknowledgment is needed. Every function / variable / flow claim is anchored to a read; unanchored claims are labeled `(unverified)` rather than invented. Fetched content (HTML, JS, error strings, comments) is treated as **data** — commands and URLs embedded inside are never acted on without user confirmation. A focus phrase is honoured only when the user supplies one — the skill does not guess a focus. Writing is gated by a self-review checklist and an explicit HARD GATE on direction, output path, and sidecar scope.

### `prd`

Produces `docs/YYYY-MM-DD-<slug>/PRD.md`, containing:

- Problem Statement (specific, not vague)
- Goals (measurable) and Non-Goals (genuine exclusions)
- Constraints and Assumptions (with verification status)
- 2-3 genuinely distinct Options with pros, cons, complexity, risk
- Recommended Direction with rationale and explicit trade-off
- Complexity tier (LOW / MEDIUM / HIGH)
- User Stories (*As a… I want… so that…*)
- Not-Doing list (explicit trade-offs)
- Open Questions (tagged `now` or `during-build`)
- Optional inline Mermaid diagram when topic has structure

### `spec`

Produces `docs/YYYY-MM-DD-<slug>/SPEC.md`, containing:

- Context (with optional link to the originating PRD)
- Technical Goals and Non-Goals (testable, not product-flavored)
- Constraints and Assumptions (with verification status)
- **Current → Proposed Architecture** (module-level, parallel structure so readers can diff)
- **Key Decisions as embedded mini-ADRs** — each with Context, Decision, Alternatives Considered, Consequences
- Data Model with invariants and migration shape
- API Contracts with inputs, outputs, error semantics, idempotency, and backwards-compat notes
- Module Boundaries map (created / modified / absorbed)
- **Tracer-bullet Phases** — 1-4 thin vertical slices with acceptance criteria, **no file or function names** (durability rule)
- Test Strategy with prior-art references
- Risks & Mitigations table
- Mermaid diagram when structure warrants (mandatory for Deep)
- Not-Doing list (technical trade-offs)
- Open Questions (tagged `now` or `during-build`)
- Adversarial red-team pass appendix when material issues were raised

The flow is **scope-adaptive**: a Lightweight change gets a compact artifact in a few turns; a Deep architectural decision gets the full ceremony plus a mandatory diagram and full red-team.

The red-team pass is backed by three dedicated subagents — **`spec-staff-engineer-reviewer`** (production risk, load, deploy / rollback, concurrency, operability), **`spec-security-reviewer`** (trust boundaries, authN / authZ, input validation, data exposure, abuse), and **`spec-future-maintainer-reviewer`** (cold-read durability, unexplained *why*, rationalized decisions, untestable criteria). Dispatch is **scope-adaptive**: Lightweight runs inline one-liners, Standard offers opt-in parallel dispatch when the SPEC touches a public API surface, schema, or trust boundary, and Deep mandates parallel dispatch of all three. Each agent returns 3-5 section-anchored critiques with a Posture line and Bottom line, which are integrated into the SPEC or logged as Open Questions / Risks before the self-review checklist.

### `revision`

Produces `docs/YYYY-MM-DD-<slug>/REVISION.md` **in the same folder as the PRD/SPEC it reviews**, containing:

- Summary (clean pass vs material findings) with severity distribution
- Scope of Review (paths, inherited complexity, lenses run, mode)
- Grounding Summary (baseline reading before findings)
- Findings grouped by severity (P0 Blocker / P1 Major / P2 Minor / P3 Nit), each with:
  - Lens (`cross-doc` / `coherence` / `adversarial` / `scope-feasibility`)
  - Type (`error` / `omission`)
  - Target (`PRD` / `SPEC` / `PRD+SPEC`)
  - Evidence (≥1 direct quote from source, with section reference)
  - Why it matters (one-sentence risk)
  - Proposed fix (minimal, no new scope)
- Suppressed Findings table (low-confidence; visible, never silently dropped)
- Self-review Checklist
- Patch Plan (populated at Gate 2 approval)
- Optional Adversarial Lens output appendix

The 4-lens composite pass runs in a single sequential flow:

1. **Cross-doc alignment** — goal coverage matrix, non-goal respect, assumption preservation, user-story mapping. *(skipped in single-doc mode)*
2. **Internal coherence** — intra- and cross-doc contradictions, terminology drift, broken refs, ambiguity, orphan mini-ADRs.
3. **Adversarial premise challenge** — premise / unstated assumption / decision stress / simplification pressure / alternative blindness, with a rigorous-not-hostile stance. Dispatched to the `revision-adversarial-review` agent for a cold-context read; the skill integrates returned findings into the main sequence. Falls back to inline if the agent fails to dispatch.
4. **Scope creep + feasibility** — shadow-path tracing, migration shape, rollback story, data-model invariants, API-contract rigor, durability of phases.

The skill is **double-gated**: Gate 1 blocks writing REVISION.md until findings + path are approved; Gate 2 blocks patching PRD.md and SPEC.md until the patch plan is approved. A clean pass still writes REVISION.md (as an audit record) but never patches. Scope is **inherited** from the MAX of PRD/SPEC frontmatter complexity (HIGH wins). When only one of PRD / SPEC is present, the skill degrades to a single-doc review (coherence + adversarial only).

### `diagnosis`

Produces `docs/YYYY-MM-DD-<slug>/DIAGNOSIS.md` (or reuses an existing spec folder when the bug maps to a known feature), containing:

- Bug description (expected vs actual, who is affected, impact)
- Environment (runtime, OS, release/commit, reproducibility rate)
- Investigation Trail (table: every `Grep` / `Read` / `Bash` / `git` and what was found)
- Pattern Analysis (working example + line-level differences against the broken path)
- Hypotheses table — **minimum 3 structurally different** ("missing null check" ≠ "race condition" ≠ "wrong API contract"), each with evidence FOR, evidence AGAINST, and a CONFIRMED / REJECTED / INCONCLUSIVE verdict
- **Causal Chain** from original trigger to observed symptom, every link named with specific code (file:line) or specific event — **no "somehow X leads to Y" gaps**
- **Predictions for uncertain links** — if a prediction is wrong but the fix "works," you found a symptom, not the cause
- Root Cause (cites file:line and names the violated invariant)
- Reproduction test (file, name, status) — **mandatory RED proof block with literal command + literal output**, or UNTESTABLE with a one-paragraph justification
- Hotspots (table: file:function → why the implementer should focus here)
- Suggested Fix (minimal — addresses the root cause, not the symptom; optional defense-in-depth recommendation when layered validation is warranted)
- Concerns (risks of the fix, low-confidence areas, things to watch post-deploy)
- Severity classification — TRIVIAL / STANDARD / COMPLEX (independent of scope)
- Optional Mermaid diagram (sequence / flowchart / state) — mandatory for Deep scope
- Relationship to Original Spec section when reusing an existing `docs/` folder

The flow is **scope-adaptive**: a Lightweight bug (typo, clear type error) gets a compact artifact with 0-1 questions; a Standard bug (non-obvious cause, 2-5 files) gets full phases with ≥3 hypotheses; a Deep bug (race, distributed, architectural) gets full phases + mandatory diagram + multi-component boundary instrumentation guidance.

The skill is **strictly read-only in source code** — it may create only the reproduction test file. Writing is gated by a self-review checklist and an explicit HARD GATE. Issue-tracker references (GitHub `#123` / org/repo#123, Jira keys like `ABC-456`) are auto-fetched at Pre-Flight via `gh issue view` and the Atlassian MCP.

### `execute`

Produces `docs/YYYY-MM-DD-<slug>/EXECUTION.md` (reuses the SPEC/DIAGNOSIS/CODE-REVIEW folder when one exists), containing:

- Summary (what shipped, source of truth, integration gate result)
- Micro-spec (RAW mode only: Goal, Acceptance criteria, Modules touched, Risks)
- **Findings Addressed** (FIX mode only) — table of `F-N | severity | title | file:line | slice | commit | test` for every finding the user selected at Phase 0.5 triage
- **Findings Skipped** (FIX mode only) — table of `F-N | severity | title | reason` for every finding the user chose to defer; these persist as open issues for the next re-review
- **Slice table** — one row per vertical tracer-bullet slice (intent, mode `full-tdd | verification`, test file, commit SHA, result)
- **Per-slice detail** with literal RED proof, GREEN proof, files touched, commit SHA, acceptance criteria satisfied (Verification-Mode slices record a build + suite proof instead of RED)
- **Integration Gate** — literal output from a fresh full test suite, lint, and type-check; skipped-tests audit; regression red-green proof (BUG mode and FIX mode when practical — one proof per addressed P0 finding at minimum in FIX mode)
- **Acceptance Criteria Trace** — table mapping every SPEC criterion / DIAGNOSIS reproduction test / selected CODE-REVIEW finding to the slice and test that satisfies it
- **NOTICED BUT NOT TOUCHING** list — out-of-scope observations surfaced during execution and deliberately not acted on
- **Follow-ups** — concrete TODOs worth tracking after this lands
- **Concerns** — risks, low-confidence areas, post-deploy watches
- **Commits** table — SHA + conventional subject line per commit (none pushed)
- **Relationship to Source Artifact** — FEATURE/BUG/FIX/RAW disposition plus the back-pointer appended to SPEC.md (`## Implemented <DATE>`), DIAGNOSIS.md (`## Fixed <DATE>`), or CODE-REVIEW.md (`## Fixes applied <DATE>` with addressed-vs-skipped tables)

The flow is **scope-adaptive**: a Lightweight slice (typo, rename, one-liner) gets 1-2 slices with 0-1 questions; a Standard feature or bug (3-10 files, clear SPEC phases or named root cause) gets 2-4 slices with per-slice RED-GREEN-REFACTOR; a Deep change (10+ files, concurrency/migration/integration) gets 2-4 slices plus a risk-mitigation mapping and mandatory regression proof in BUG mode.

The skill **auto-detects the source of truth** in priority order: explicit path (filename routes to mode — `CODE-REVIEW.md → FIX`, `SPEC.md → FEATURE`, etc.) > FIX-mode prompt triggers ("address findings", "apply the code-review", "fix the blockers") matched against the newest `CODE-REVIEW.md` with `status: approved` > `REVISION.md` with `patches_applied: true` > `SPEC.md` with `status: approved` > `DIAGNOSIS.md` with `status: approved` > `PRD.md` alone (warns to run `/spec` first) > raw prompt (RAW mode). When the detected primary artifact sits in a folder that also has an approved CODE-REVIEW.md with open findings, the skill offers a one-question FIX-mode switch before advancing. Execute is the only eva skill that **writes production code**, and it is gated twice: HARD GATE 1 blocks the first line of code on approved slice plan + path; HARD GATE 2 blocks writing EXECUTION.md on a fully green integration pass. A protected-branch check at Pre-flight refuses `main`/`master`/`production`/`prod`/`stable`/`live`/`trunk`/`release*` before any work happens.

In **FIX mode**, the skill adds a dedicated triage phase (Phase 0.5) between input routing and the pre-scan: it parses every finding from the latest findings section of CODE-REVIEW.md (top-level tables, or the most recent `## Re-review <DATE>` append), subtracts any finding already addressed by a prior `## Fixes applied <DATE>` section, displays a compact P0/P1/P2/P3 summary, and asks the user to pick — default **A. All P0 + P1** (skip P2/P3), or **B. P0 + P1 + P2**, or **C. Every finding shown**, or **D. Custom** (include/skip by ID). Selected findings drive the slice plan (one slice per finding, or grouped when two findings share a file and fingerprint root); each slice still writes a failing test first. Skipped findings are recorded in the Findings Skipped table and remain open issues for the next re-review. The Suggested Fix on each finding is guidance, not gospel — disagreement triggers the Confusion Protocol rather than silent divergence. The CODE-REVIEW.md itself is treated as read-only; the skill only appends `## Fixes applied <DATE>` so prior findings stay intact as history.

### `code-review`

Produces `docs/YYYY-MM-DD-<slug>/CODE-REVIEW.md` (reuses an adjacent spec folder when the diff maps to a known feature), containing:

- **TL;DR posture** — green light / yellow (N P0/P1 to resolve) / red (blockers first); counts per severity; one-line verdict
- **Stage results table** — one row per reviewer with status (ran / clean / skipped with reason) and finding counts
- **Blockers (P0)** — each with severity, confidence, file:line, verbatim evidence, why it matters, suggested fix, autofix class, verification-needed flag, and a `seen_by` cross-reviewer-agreement badge when multiple reviewers independently flagged the same fingerprint
- **Major (P1)** — same shape as P0; fix before merge or explicitly accept the residual risk
- **Minor (P2)** — worth fixing; not ship-blocking
- **Nits (P3)** — preference; author may decline (single-line form to reduce noise)
- **Positives** — what the reviewers specifically liked, so it is not regressed on iteration
- **Residual risks** — concerns a reviewer surfaced but could not fully confirm
- **Testing gaps** — scenarios the tests under review do not cover
- **Suppressed findings** table — confidence < 0.60 findings (P0 exception at ≥ 0.50); visible, never silently dropped
- **Pre-existing findings** table — problems the diff happens near but did not introduce; triage separately
- **Plan alignment summary** — acceptance criteria satisfied / unmet, non-goals respected, mini-ADR consistency, silent decision changes (when a plan document is present)
- **Diff inventory** — every file with +/- LOC and which reviewers opened it; flags any file no reviewer read
- **Next steps** — ordered action list the author can scan in under a minute
- **Verification of prior fixes** (re-review only) — one row per prior claim parsed from `## Fixes applied <DATE>` sections, with status `verified` / `mismatch — still present` / `still present (expected)` / `vanished` / `orphaned`; addressed-but-still-present claims elevate the matching current finding to `claim_mismatch: true` with a `+0.15` confidence boost that bypasses the suppression gate

The flow is **two- or three-stage and scope-adaptive**:

- **Stage 1 (blocking on Standard/Deep)** — `code-review-plan-alignment-reviewer` verifies the diff implements the approved plan (PRD/SPEC/DIAGNOSIS/REVISION/EXECUTION). A P0 drift finding pauses the pipeline; the user picks between fix-the-drift, update-the-plan (escalate to `/revision`), or proceed-anyway-with-drift-recorded.
- **Stage 2 (parallel fan-out)** — `code-review-quality-reviewer`, `code-review-convention-reviewer`, and `code-review-test-reviewer` always run; `code-review-security-reviewer` and `code-review-performance-reviewer` are auto-added when the diff trips their trigger patterns (user-input handlers, auth, injection-prone primitives, DB queries, fan-out, unbounded loops, etc.) on Standard, and are mandatory on Deep. Reviewers are dispatched in a **single message with multiple Agent tool calls** — sequential dispatch is a protocol violation.
- **Stage 3 (Deep only)** — `code-review-adversarial-reviewer` receives Stage 2's merged findings as input and extends them — never repeats. Covers unstated assumptions, load/partition/rollback/concurrency stress, Hyrum's Law leakage, cross-module coupling, observability gaps.

Every reviewer returns findings in a shared JSON schema — severity (P0/P1/P2/P3), confidence (0.0-1.0), file/line, verbatim evidence, intent (file-line-independent problem class), why-it-matters, suggested-fix, autofix-class (safe_auto / gated_auto / manual / advisory), pre_existing flag, requires_verification flag. The orchestrator merges and deduplicates via a fingerprint (`normalize(file) + line_bucket(line, ±3) + normalize(intent)`), applies a **+0.10 confidence boost per additional reviewer** that flagged the same fingerprint (capped at 1.0), and suppresses findings below 0.60 confidence into a visible Suppressed table with a P0 exception at 0.50+.

The skill is **diff-aware**: auto-detects diff scope in priority order (uncommitted staged + unstaged > branch-vs-base > HEAD~1), with explicit overrides via `/code-review vs <base>`. If no changes are detected, it stops instead of reviewing nothing. It **smart-reuses** an existing `docs/<DATE>-<slug>/` folder when the diff clearly maps to a known feature — the CODE-REVIEW.md sits next to the PRD/SPEC/DIAGNOSIS. Re-reviews append under a `## Re-review <DATE>` section instead of overwriting (the original review is history). On re-review, any prior `## Fixes applied <DATE>` sections (left by `/execute` in FIX mode) are parsed and verified against the fresh findings via fingerprint matching — addressed-but-still-present claims elevate to `claim_mismatch` findings that bypass the suppression gate, and a Verification of prior fixes table summarizes each claim's outcome (verified / mismatch / still present expected / vanished / orphaned). Scope classification (Lightweight / Standard / Deep) tunes which reviewers run, how deep they go, and whether Stage 3 fires. The skill is **gated** by a self-review checklist and an explicit HARD GATE — nothing is written to disk until the user approves the finding set and the output path.

### `pr-feedback`

Produces `docs/YYYY-MM-DD-<slug>/PR-FEEDBACK.md` (or reuses an adjacent spec folder when the PR maps to a known feature), containing:

- **Summary** — PR identity (owner/repo#N, title, author), state (`OPEN` / `MERGED` / `CLOSED` / `DRAFT`), review decision (`APPROVED` / `CHANGES_REQUESTED` / `REVIEW_REQUIRED`), distinct reviewer list, branch-at-triage with short SHA + `matches PR head` label, scope tier + reason, spec-folder reuse note, reconciliation target (sibling CODE-REVIEW.md path, if any)
- **Verdict counts table** — one row per bucket (must-fix, question, push-back, nice-to-have, already-done, stale/outdated, suppressed low-confidence, resolved collapsed, dismissed skipped)
- **Must-fix (Mᴺ)** — each with reviewer handle, source kind (inline / timeline / review-body), permalink, SHA-pinned `file:line` link, the comment quoted verbatim, verdict + confidence, rationale, evidence quote with source (diff / spec / prior CODE-REVIEW), cross-ref notes ("corroborates CODE-REVIEW F-3" or "PRD §2.1 supports this"), a copy-pasteable **Suggested reply** draft in a fenced code block, and a **Handoff to /execute** block (file + line + intent + suggestion-block verbatim when the reviewer supplied one)
- **Questions (Qᴺ)** — same shape; replies cite spec anchors when a plan doc answers the question
- **Push-back (PBᴺ)** — same shape; replies quote the spec § or test name that justifies the disagreement; never condescending, never apologetic
- **Nice-to-have (NHᴺ)** — same shape; concise replies, declining gracefully when deferred
- **Already-done (ADᴺ)** — same shape; reply points to the commit SHA that addressed it
- **Stale / outdated** — comments whose `line == null` or `isOutdated == true`; every entry attempts `diffHunk` re-anchor and records `re_anchored_from: <orig-path>:<orig-line>` when successful
- **Reconciliation with CODE-REVIEW.md** — Agreements table (prior finding ↔ current verdict, with confidence boost recorded) + Tensions table (reviewer flags what we suppressed, worth re-examining) when a sibling `CODE-REVIEW.md` exists
- **Suppressed (confidence < 60)** — visible, never silently dropped; must-fix with confidence ≥ 50 stays top-bar anyway
- **Resolved threads** — collapsed inside a `<details>` block so the triage stays compact but context is preserved
- **Residual risks** — stale comments that re-anchoring failed on, multi-reviewer conflicts on the same line (escalates to Deep), push-back verdicts where confidence is marginal (<60) and the disagreement lacks a spec anchor
- **Skipped — dismissed reviews** — a footer table (reviewer, dismissed-at, comment count, reason if known) — audit trail, never hidden
- **Handoff table** — one row per must-fix (ID, file, line, intent, suggestion-block?); `/execute` FIX mode parses this directly
- **Branch-state-at-triage frontmatter** — `current_branch`, `head_sha`, `matches_pr_head` — so a future re-triage can detect branch drift and ask before proceeding

The flow is **scope-adaptive**: a Lightweight triage (≤5 comments, single reviewer, not `CHANGES_REQUESTED`, <100 LOC, no sensitive surface) takes a fast path with optional spec cross-reference and 1-2 sentence replies; a Standard triage (6-20 comments OR ≥2 reviewers OR `CHANGES_REQUESTED` OR 100-500 LOC) runs the full pipeline with mandatory spec cross-reference when a plan folder exists and 3-5 sentence replies; a Deep triage (>20 comments OR multi-reviewer contradiction on the same `path:line` OR sensitive surface — auth / payments / migrations / crypto / webhooks / schema — OR >500 LOC) adds a mandatory Residual Risks section with every ambiguous anchor enumerated, SHA-pinned permalinks in every reply, and a dedicated cross-reviewer-conflict subsection.

The skill is **pure triage** — it reads the PR's feedback, reconciles with adjacent artifacts, and writes one file. It never runs `gh pr comment`, never `gh pr review`, never stages / commits / pushes, never invokes `/execute`. Copy-pasteable replies are **drafts** — the human decides when (and whether) to post them. The mandatory **branch-checkout gate** is non-negotiable: Phase 1 inspects `git branch --show-current` + `git rev-parse HEAD`, compares against the PR's `headRefOid`, and if the working tree is not on the PR head, offers `gh pr checkout <N>` (the path that handles forks and cross-repo PRs cleanly); on decline or failure, the skill refuses with the exact manual recovery commands and stops. The comment fetch uses **GraphQL `reviewThreads`** because REST's `/pulls/<N>/comments` does not expose `isResolved` or `isOutdated` as first-class fields — threading, thread state, and diff hunks come back in one round-trip. The triage runs **single-lane** (no sub-agent fan-out) because the work requires full shared context (diff + spec + prior review) and parallel dispatch would load the same context N times for no gain. Writing is gated by a self-review checklist and an explicit HARD GATE; nothing is written until the user approves the verdicts and the output path.

### `commit-push`

Produces one to three Conventional-Commits-style commits on a human-confirmed non-protected feature branch, fetch-rebased onto the upstream default, pushed with `-u`, and (optionally) back-pointed to an adjacent artifact — the commits themselves are the durable output.

No new markdown file is written by default. When `docs/<DATE>-<slug>/EXECUTION.md` or `docs/<DATE>-<slug>/DIAGNOSIS.md` is adjacent to the committed work, a compact `## Commits <DATE>` section is appended to the primary artifact, listing:

- Commit SHA + subject line per commit
- Target branch

The flow is **scope-adaptive**:

- **Lightweight** (1–2 files, <50 lines, single concern) — secrets + protected-branch + debug-artifact scans only; single commit; subject-only message acceptable; skips the split question.
- **Standard** (3–10 files, 50–500 lines, single coherent concern) — full test suite + lint/formatter added to the gate; feat/fix requires a WHY body and a Testing scenarios block; split proposed only when files cluster by distinct concern.
- **Deep** (10+ files, >500 lines, mixed concerns, or changes under trust boundaries / migrations / concurrency) — adds scope-vs-diff audit, test-coverage-for-code-changes audit, and chore-exemption audit with a human-supplied one-sentence justification per exempted file recorded in the commit body; proposes a 2–3 commit split by concern.

The skill **always** gates on the human for the branch name — on invocation it detects the current branch (refuses if protected) and offers an A/B question: commit on the current branch, or create a new one and name it. The A/B answer counts as the explicit human confirmation; a silent proceed is a protocol violation. If the user picks B, the skill suggests a `<type>/<slug>` name derived from the argument-hint, adjacent `topic:` frontmatter, or top-level filename change — but waits for the human to confirm or override before creating.

The **pre-commit gate** is unconditional at its tier: secrets matches (API-key patterns, token/password/secret assignments, `.env` files being added, PEM private keys) stop the skill even if the user insists; detected debug artifacts (`console.log`, `debugger`, `TODO` without ticket, `FIXME`, `HACK`) ask the user to keep/remove/skip-scan before proceeding; test and lint failures stop the skill — no `--no-verify`, no "I'll fix the lint later"; out-of-scope files on Deep tier force a per-file keep/drop/defer decision; code files without a corresponding test change on Deep tier force a cover/chore decision, with `chore` requiring a justified audit entry.

The **commit-message draft** follows a strict precedence: repo instructions (CLAUDE.md / AGENTS.md / CONTRIBUTING.md) → commitlint config → pattern in the last 10 commits → Conventional Commits default (`<type>(<scope>): <subject>`). Subject is imperative, lowercase-first, ≤72 chars, no trailing period. Body explains **why** not **what** — the diff shows what. Testing scenarios block is mandatory for feat/fix on Standard/Deep. Footers (`Signed-off-by:`, `Co-Authored-By:`, generator promos) are only added when the repo's recent commits already use them.

The **split proposal** (up to 3 commits, max) is a single `AskUserQuestion` that lists the proposed clusters (C1 files + subject, C2 files + subject, …) and lets the user approve, collapse to one commit, or re-group. File-level clustering only — never `git add -p`, never hunk-splitting.

The **push flow** is unconditional: `git fetch origin` then `git pull --rebase origin <upstream-default>` (resolved from origin's HEAD branch > `gh repo view` > fallback `main`), then `git push -u origin <branch>`. Non-trivial conflicts stop the skill with a show-and-hand-back — trivial whitespace-only conflicts may be resolved in-skill, with a one-paragraph note. Push rejections trigger exactly one re-fetch-rebase retry — never `--force`, never `--force-with-lease` (that's a separate explicit human turn).

The skill is **gated twice**: the A/B branch question is a HARD HUMAN GATE that must resolve before Phase 3's pre-commit gate runs, and the pre-push HARD GATE emits a self-review checklist + an approve/change/abort question before any `git add` or `git commit`. Nothing stages, nothing commits, nothing pushes without an explicit approve in this conversation.

**Post-push**, the skill reports the SHAs, the pushed branch, and the back-pointer target (if any), then STOPS. It does not run `gh pr create`, does not invoke `/code-review`, does not suggest "shall I open the PR?" — PR creation is a separate, explicitly-human-requested action owned by the user.

### `solutions`

Produces `docs/YYYY-MM-DD-<slug>/SOLUTIONS.md` **in the same folder as the upstream artifacts it synthesises** (never a new dated folder), containing:

- **Summary** — 2-3 sentences naming what shipped (feature), what broke and how it was fixed (bug), or both (mixed)
- **Root Cause** (bug / mixed only) — failure mechanism as an invariant that was violated, not a line number
- **Approach** — what was done and why this path over the rejected alternatives, pulled from SPEC mini-ADRs and EXECUTION per-slice intent
- **Key Decisions** — compact mini-ADR per decision (What / Why / Trade-off accepted / Alternatives considered), promoted only when the decision proved its weight under implementation, emerged mid-execute, shifted during REVISION, or was deliberately accepted as drift during CODE-REVIEW
- **Gotchas** — behaviour invariants pulled from EXECUTION's NOTICED-BUT-NOT-TOUCHING and Concerns, DIAGNOSIS concerns, and CODE-REVIEW residual risks / testing gaps. Every entry phrased so it would survive a file rename
- **What Didn't Work** — a map of the territory for the next session, pulled from EXECUTION 3-strike Step-Back events, DIAGNOSIS REJECTED hypotheses, and CODE-REVIEW suppressed findings — each entry names the attempted approach, why it failed, and the source artifact
- **Prevention** (bug / mixed only) — class of bug, where to add defense-in-depth (entry point / business layer / persistence / output), and the regression-protection test referenced by name (not file path)
- **Diagram** (mandatory for Deep, optional for Standard, omitted for Lightweight) — Mermaid sequence / state / flowchart depicting the durable mechanism, not the file layout
- **References** — dated, status-tagged links to every upstream artifact in the folder
- **Relationship to Original Spec** (mixed pipelines only) — one paragraph naming the gap the original SPEC missed or the regression a later refactor introduced

The flow is **scope-inherited**: Lightweight when every upstream artifact classified Lightweight and EXECUTION had no Step-Backs (compact: Summary · Approach · Key Decisions · Gotchas · References); Standard when any upstream was Standard, or EXECUTION had a resolved Step-Back, or the pipeline was multi-slice (adds What Didn't Work, mini-ADR Alternatives mandatory, optional diagram); Deep when any upstream was Deep, concurrency / migrations / trust boundaries / architectural refactors were in play, or CODE-REVIEW ran Stage 3 adversarial (adds mandatory Mermaid diagram).

The skill is **strictly synthesis-only** — it reads every approved artifact in the folder (PRD / SPEC / REVISION / DIAGNOSIS / EXECUTION / CODE-REVIEW), extracts durable learnings, and writes `SOLUTIONS.md` next to them. It never edits code, never modifies any upstream artifact, never edits `CLAUDE.md` / `AGENTS.md` (if the learnings warrant durable project-wide instructions, the skill mentions the opportunity at the transition — the user owns the edit), and never chains to another skill. Re-runs **always append** under a new `## Re-run <DATE>` section (feature re-run, refactor re-run) or `## Post-Release Bug Fix <DATE>` section (when a DIAGNOSIS was appended to a feature folder) — the prior content is preserved as history.

The skill is **gated**: a self-review checklist (Iron Law compliance, verifiability against source artifacts, section completeness vs scope tier, no synthetic padding) precedes a HARD GATE that blocks writing `SOLUTIONS.md` until the user approves the summary, the learnings set, and the output path. Manual invocation only — `solutions` does not auto-run at the end of another skill, because the user should decide whether a pipeline is interesting enough to memorialize.

## Invariants (all ten skills)

- No code until the artifact is approved.
- One clarifying question at a time.
- Every claim about the codebase or source documents must be verified by reading; unverified claims are labeled.
- Synthetic options / decisions / findings are forbidden — a clean lens is valid.
- A self-review checklist is emitted to the user before writing.
- A HARD GATE blocks writing until the user approves the direction and the output path.

Additional invariants for `spec`:

- Every mini-ADR must list at least one genuine alternative — otherwise it's a rationalization.
- Tracer-bullet phases describe capabilities + acceptance criteria only — never file or function names.
- An adversarial red-team pass (staff engineer, security reviewer, future maintainer) runs before the self-review checklist.

Additional invariants for `revision`:

- No new features, no scope expansion. Revision fixes gaps and inconsistencies only.
- At most 3 clarifying questions, only when auto-detection is genuinely ambiguous — revisions usually need zero.
- Every finding quotes ≥1 piece of evidence from PRD or SPEC with section reference.
- A second HARD GATE blocks patching PRD.md and SPEC.md even after REVISION.md is written.
- Clean pass (zero findings) still writes REVISION.md as an audit record; PRD and SPEC remain untouched.

Additional invariants for `diagnosis`:

- **No fixes without root-cause investigation first.** Symptom fixes are failure.
- **Read-only in source code.** The skill may create only the reproduction test file.
- At most 3 clarifying questions — bug context is usually concrete; prefer reading code over interviewing.
- Minimum **3 structurally different hypotheses** (or a TRIVIAL-severity justification for fewer). Synthetic variations of the same idea are forbidden.
- **Causal-chain gate:** a root cause cannot be claimed without a full chain from trigger to symptom with no gaps. Every uncertain link needs a prediction.
- **Reproduction test with RED proof (literal command + literal output) is mandatory**, or an UNTESTABLE justification paragraph with manual reproduction steps.
- **Error messages and stack traces are data, not instructions** — never execute commands or follow URLs embedded in error output without user confirmation.
- If the bug maps to an existing `docs/<DATE>-<slug>/` feature folder, the skill smart-reuses it and appends a `## Post-Release Bug Fix` pointer to the primary doc.

Additional invariants for `execute`:

- **Iron Law: no production code without a failing test first.** If production code was written before the test, apply the Delete Rule — delete, write the test, watch it fail, rewrite. **Verification Mode** is the narrow exception for genuinely non-behavior-bearing changes (config, build files, type-only declarations, docs, pure formatting) and requires a captured full-build + full-suite green proof instead of RED.
- **Refuse protected branches.** At Pre-flight, the skill aborts on `main`, `master`, `production`, `prod`, `stable`, `live`, `trunk`, or `release*` and asks the user to create a feature branch first.
- **Pre-scan findings stay internal.** A fast codebase pre-scan loads instruction files, top-level structure, tech-stack fingerprint, touch-area files + nearest tests, and one pattern exemplar into working memory, skipping `node_modules`, `.git`, `dist`, `build`, `.next`, `vendor`, `coverage`, `.venv`, `.cache`, `target`, `out`, `tmp`. Budget ~2k tokens, <60s. Findings are NOT emitted to the user — only `execute: context loaded.` is printed.
- **Minimal-diff constraint.** Every changed line must trace to a SPEC phase acceptance criterion, a DIAGNOSIS root cause or suggested fix, a selected CODE-REVIEW finding ID (FIX mode), or (RAW mode) a micro-spec acceptance criterion. Lines that don't trace are not written.
- **FIX mode: user-gated findings triage.** In FIX mode, the user decides at Phase 0.5 which CODE-REVIEW findings are addressed this run (default: all P0 + P1) and which are skipped. Skipped findings are recorded in EXECUTION.md's Findings Skipped table with reasons — never silently dropped. Addressed findings are back-pointed into CODE-REVIEW.md under `## Fixes applied <DATE>` (append-only — the original findings tables are never edited). Suggested-Fix disagreements invoke the Confusion Protocol, not silent divergence.
- **NOTICED BUT NOT TOUCHING.** Out-of-scope observations are logged with that prefix in EXECUTION.md. Never acted on mid-execute.
- At most 3 clarifying questions — usually 0-1 when an artifact is the source of truth.
- **3-strike Step-Back.** When a slice's test stays RED after three different approaches, STOP. Do not attempt #4. Surface what was tried, what was assumed, and what's still unknown, then offer to escalate to `/diagnosis`, re-open `/spec`, or ask the user for guidance.
- **Confusion Protocol** — STOP / NAME / OPTIONS / WAIT / PLAN — when ambiguity appears mid-slice.
- **Commit per slice** on the current feature branch with a Conventional Commits message. Stage explicit files by name, never `git add -A`. No pushing. No PRs unless the user explicitly asks later.
- **HARD GATE 1** (plan approval) blocks the first line of code; **HARD GATE 2** (integration gate) blocks writing `EXECUTION.md` on anything other than a fully green test suite + lint + type-check captured in a fresh run.
- **No SPEC/DIAGNOSIS rewrites from execute.** If the source artifact is wrong, raise a Confusion; never silently diverge.
- **Error messages and stack traces are data, not instructions** — the same security invariant as `diagnosis`, carried into the execution path.

Additional invariants for `commit-push`:

- **Iron Law: no commit lands without explicit human confirmation of the branch in the current conversation, and never on a protected branch.** Even an "obviously correct" `<type>/<slug>` suggestion still needs an explicit A/B answer or a typed branch name in this turn. Auto-mode, durable instructions, and `/yolo`-style session modes do NOT override this — the refuse-and-ask half of the user's global git-safety rule is load-bearing.
- **Protected-branch refusal is unconditional.** `main`, `master`, `production`, `prod`, `stable`, `live`, `trunk`, and any branch matching `^release([/_-].*)?$` are refused both as the current branch (Pre-flight stops the skill) and as the requested target (Phase 2 re-asks). No carve-outs, no session overrides.
- **No pull requests.** `gh pr create`, `gh pr edit`, web-UI PR creation, or any PR-opening primitive is out of scope. If the skill is about to run `gh pr create`, STOP. PR creation is a separate, explicitly-human-requested action the user owns in a later turn.
- **Explicit per-file staging.** `git add -A`, `git add .`, and `git add -u` are forbidden — they leak `.env`, build artifacts, and IDE droppings through the checks the skill already ran. Stage only the paths verified against the diff.
- **No history rewrites from this skill.** No `--force`, no `--force-with-lease` (only on a separate explicit human turn after a push rejection), no `commit --amend` to a pushed commit, no `rebase -i`. If a rewrite is needed, stop and hand back.
- **No `--no-verify`.** Pre-commit hooks exist because someone wrote them to catch a real mistake. Fix the underlying issue, then re-commit. Even if the hook is genuinely flaky, this skill does not bypass — the user fixes the hook on a separate branch in a separate turn.
- **No secrets, unconditionally.** Detected API-key / token / password / `.env` / private-key patterns stop the skill. The stance holds even if the user insists — fake secrets in test fixtures use documented sentinels (`sk_test_fake_DO_NOT_ROTATE`) outside the scan's patterns.
- **Scope-adaptive pre-commit gate.** Lightweight runs secrets + protected-branch + debug-artifact scans. Standard adds the full test suite + lint/formatter. Deep adds scope-vs-diff + test-coverage-for-code-changes + chore-exemption audit. Every `chore` classification on Deep tier requires a one-sentence human justification recorded in the commit body under `Chore exemptions:` — silent coverage loss is the exact failure mode the audit exists to prevent.
- **Message-draft precedence.** Repo instructions (CLAUDE.md / AGENTS.md / CONTRIBUTING.md) > commitlint config > pattern in the last 10 commits > Conventional Commits default. Mixing styles in the same repo is a smell even when the conventional style would be "cleaner" — never upgrade a team's convention unilaterally.
- **Message review is human-gated.** The skill drafts; the human approves, edits, or rewrites via `AskUserQuestion` before the commit is created. A user-written message is still validated (imperative mood, subject length, body required on Standard/Deep feat/fix).
- **Logical split is human-approved, max 3 commits.** The skill proposes groupings when files cluster by distinct Conventional Commits type; the human approves, collapses, or re-groups. File-level clustering only — never `git add -p`, never hunk-splitting.
- **Rebase-then-push.** Before every push: `git fetch origin` + `git pull --rebase origin <upstream-default>`. Non-trivial conflicts stop the skill with a hand-back. Push rejections trigger exactly one re-fetch-rebase retry — never force.
- **Verification-before-claiming on every shell step.** Every test-pass / lint-pass / commit-success / push-success claim is backed by literal command output pasted in a fenced block from a fresh run in the same turn. "Tests passed earlier" is not evidence.
- **Source code is read-only.** The skill stages, commits, and pushes what's already in the working tree. It does not fix lint errors, write missing tests, or edit source to make the gate pass — those hand back to the user or `/execute`.
- **Optional back-pointer, never rewrite.** When an adjacent `EXECUTION.md` / `DIAGNOSIS.md` is found, the skill appends a `## Commits <DATE>` section with SHA + subject per commit. Existing sections are never rewritten; existing frontmatter is never mutated (e.g., no `status: merged` silent update).
- **One question per turn. Max 3–4 total.** Usually 2 (branch A/B + HARD GATE) plus at most one each for split and message-edit.

Additional invariants for `code-review`:

- **Pure reporter.** The only file the skill creates is `CODE-REVIEW.md`. Never edits, stages, commits, or pushes source code. Never applies `safe_auto` fixes automatically.
- **Diff-first.** Reviews the current diff and nothing else. Pre-existing problems surface in a separate **Pre-existing** table — the author is not blamed for code they did not write.
- **Plan-alignment is Stage 1 and blocking on Standard/Deep.** When a plan document exists, plan-alignment runs alone first; a P0 drift finding pauses the pipeline until the user decides whether to fix the drift, update the plan (escalate to `/revision`), or proceed with drift recorded.
- **Parallel Stage 2 is not optional.** All non-plan reviewers dispatch in a single message with multiple Agent tool calls. Sequential dispatch is a protocol violation.
- **Stage 3 receives Stage 2's output.** The adversarial reviewer runs only on Deep, and only after Stage 2 completes, with Stage 2's merged findings as input so it can extend rather than repeat.
- **Every finding quotes verbatim evidence.** 5-30 word quote from the diff, the file, or the plan document. Paraphrase is suppressed.
- **Confidence-gated suppression.** Findings below 0.60 confidence move to the visible Suppressed table — never silently dropped. P0 findings with confidence ≥ 0.50 stay top-bar.
- **Cross-reviewer agreement boost.** When two or more reviewers independently flag the same fingerprint (`normalize(file) + line_bucket(line, ±3) + normalize(intent)`), the merged finding's confidence is boosted by +0.10 per additional reviewer, capped at 1.0. The merged finding carries a `seen_by` badge.
- **Re-reviews append, never overwrite.** Subsequent `/code-review` runs on the same folder append under a `## Re-review <DATE>` section — the original review is preserved as history.
- **Claim verification on re-review.** When an existing `CODE-REVIEW.md` contains prior `## Fixes applied <DATE>` sections (appended by `/execute` in FIX mode), each claim is parsed at Phase 0.2 and fingerprint-matched at Phase 6.8 against the fresh merged findings. Addressed claims that still reproduce elevate the matching current finding to `claim_mismatch: true` with a `+0.15` confidence boost — these bypass the suppression gate because cross-revision recurrence is strong corroboration. Skipped claims that still reproduce are tagged as expected; skipped claims whose code has vanished become verification notes. A Verification of prior fixes table is rendered inside the new `## Re-review <DATE>` section. Prior `## Fixes applied` sections are read-only history — never edited.
- **Clean passes are valid.** Every reviewer may return `findings: []` with a one-line positive note. Padded synthetic findings are worse than a clean pass.
- **Smart spec-folder reuse.** When the diff maps to an existing `docs/<DATE>-<slug>/` feature folder, the CODE-REVIEW.md is written inside it; a new dated folder is created only when no match is found.
- **HARD GATE on write.** The skill never writes `CODE-REVIEW.md` until the user approves the finding set, the scope tier, and the output path.

Additional invariants for `pr-feedback`:

- **Pure triage. No PR posts, no code edits.** The only file the skill writes is `PR-FEEDBACK.md`. Never runs `gh pr comment` / `gh pr review` / `gh api .../comments`. Never stages, commits, or pushes. Never invokes `/execute`. Copy-pasteable replies are drafts — the human owns the posting turn.
- **Mandatory branch-checkout gate.** The working tree MUST be on the PR's head SHA before triage proceeds. First offer: run `gh pr checkout <N>` (handles forks and cross-repo PRs cleanly). On decline or failure: refuse with the exact manual `git fetch` / `git checkout` commands, stop, wait for the user to re-invoke. A silent proceed with a mismatched HEAD is a protocol violation — stale diff lines produce wrong verdicts.
- **PR state gate.** `MERGED` / `CLOSED` PRs refuse unless the user explicitly passes `--force-state`. `isDraft` warns and asks before proceeding. `OPEN` proceeds.
- **GitHub only, same repo only.** Parsed `owner/repo` must match the cwd's remote; cross-repo triage is refused. GitLab / Bitbucket are out of scope for v1. Bare `#N` invocations resolve `owner/repo` via `gh repo view` or are refused when the cwd has no GitHub remote.
- **Union non-dismissed reviews, dedupe by `thread_id`.** A PR may accumulate many reviews across reviewers; the triage unions them all. `DISMISSED` reviews are explicitly withdrawn — skipped, never hidden (logged in the Skipped footer with reviewer + dismissed-at + comment count). `PENDING` reviews are dropped (not yet published).
- **No silent drops.** Stale/outdated comments (`line == null` OR `isOutdated == true`) get a "line moved" badge and sort last inside their bucket; re-anchoring is attempted via `diffHunk` text-search against the current tree. Resolved threads are included in a collapsed `<details>` block. Suppressed verdicts (confidence < 60) stay visible with the score shown.
- **Verbatim evidence for every verdict.** A 5-30 word quote from the comment body, the diff, the plan document, or a prior CODE-REVIEW finding. Paraphrase is suppressed. Every quote carries a source tag (`comment` / `diff` / `spec:<section>` / `code-review:F-<N>`).
- **Five verdict buckets, not a free-form judgment.** must-fix / nice-to-have / question / push-back / already-done, with confidence factors derived from Conventional Comments label + decorator + spec cross-reference + prior CODE-REVIEW match + anchor freshness + code-level corroboration. See `references/taxonomy.md`.
- **Confidence-gated suppression with P0 exception.** Verdicts below 60 confidence move to the visible Suppressed table, EXCEPT `must-fix` with confidence ≥ 50 (keep top-bar so critical concerns surface under ambiguity). Never silently dropped.
- **Reconcile with adjacent CODE-REVIEW.md, never mutate it.** When a sibling `CODE-REVIEW.md` exists in the target folder, reviewer comments that match prior findings by fingerprint boost the must-fix confidence (+15); tensions ("reviewer flags what we suppressed") surface in a dedicated Reconciliation section. The prior review is **read-only** input — never edited, even in trivial ways.
- **Smart spec-folder reuse.** When the PR maps to an existing `docs/<DATE>-<slug>/` folder by branch slug, title overlap, or diff-file overlap ≥50%, write inside it. Otherwise derive a slug from the PR head branch and create a new dated folder.
- **Single-lane triage.** The skill runs inline — no sub-agent fan-out — because the work requires full shared context (diff + spec + prior review) and parallel dispatch would load the same context N times for no gain. Specialized analysis (security depth, perf depth) escalates the scope tier and cites the plan; it does not spawn a reviewer agent.
- **GraphQL for inline threads, REST for timeline + review bodies.** `reviewThreads` is the only way to get `isResolved` / `isOutdated` / reply chains first-class. REST `/pulls/<N>/comments` omits `isResolved` entirely and is not used.
- **Handoff record, not handoff call.** The Handoff table tells `/execute` FIX mode what to apply and where (file, line, intent, optional verbatim suggestion block) — the skill never invokes `/execute` itself. The human runs `/execute` in a separate turn when ready.
- **HARD GATE on write.** The skill never writes `PR-FEEDBACK.md` until the user approves the verdict set, the scope tier, and the output path. Re-triage on the same PR appends `## Re-triage <DATE>` rather than overwriting — prior triage is history.

Additional invariants for `solutions`:

- **Iron Law of durability: learnings are behaviour, not coordinates.** Every Gotcha, Key Decision, and What-Didn't-Work entry is phrased so it would survive a file rename or refactor. `src/checkout/cart.ts:42` belongs in the evidence column of the source artifact; SOLUTIONS.md states the invariant that held or was violated.
- **Synthesis-only. Source code is off-limits.** The skill never edits production code, tests, or any upstream artifact in the folder (PRD / SPEC / REVISION / DIAGNOSIS / EXECUTION / CODE-REVIEW are all read-only inputs).
- **Never edits `CLAUDE.md` / `AGENTS.md`.** If the learnings warrant a durable, repo-wide instruction, the skill mentions the opportunity in the transition paragraph — the user performs the edit on a separate turn. Silent drift into instruction files is the exact failure mode this invariant prevents.
- **Manual invocation only.** `solutions` never auto-runs at the end of another skill and never chains into another. The user decides when a pipeline is worth memorializing.
- **Scope is inherited, never self-classified.** The skill reads the `scope` frontmatter value of every upstream artifact and takes the MAX. Lightweight is never chosen on a folder where any artifact was Standard or Deep.
- **Verifiability before writing.** Every claim in SOLUTIONS.md (a Key Decision, a Gotcha, a What-Didn't-Work entry) must be traceable to a specific quote, table row, or section of an upstream artifact. Synthetic padding to hit section quotas is worse than an honest "intentionally empty" with a one-word reason.
- **Append on re-run, never overwrite.** Second and later runs add a new `## Re-run <DATE>` section (feature re-run / refactor) or `## Post-Release Bug Fix <DATE>` section (when a DIAGNOSIS was appended to a feature folder) below an HTML-comment anchor. Prior content is preserved as history; the frontmatter's `date` advances, `post_release_bug_fix_appended` flips to `true` where appropriate.
- **Writes inside the existing docs folder.** SOLUTIONS.md is placed next to the artifacts it synthesises (same `docs/<DATE>-<slug>/`), never a new dated folder. If the skill is invoked with no resolvable folder, it stops and asks which folder to synthesise.
- **HARD GATE on write.** The skill emits a self-review checklist (Iron Law compliance, upstream-artifact verifiability, section-vs-scope coverage, no synthetic padding) and blocks writing SOLUTIONS.md until the user approves the summary, the learnings set, and the output path.
- **One clarifying question at a time; usually zero.** A pipeline with clear artifacts in one folder is almost always self-describing — the skill defaults to zero questions and only asks when folder resolution or re-run disposition is genuinely ambiguous.

## Installation

### As a plugin (recommended)

```bash
# From Claude Code — add the marketplace, then install
/plugin marketplace add vilarjp/eva
/plugin install eva@eva
```

Or clone and symlink into `~/.claude/plugins/`:

```bash
git clone https://github.com/vilarjp/eva ~/.claude/plugins/eva
```

### As a project-local skill

Copy `skills/teardown/`, `skills/prd/`, `skills/spec/`, `skills/revision/`, `skills/diagnosis/`, `skills/execute/`, `skills/code-review/`, `skills/pr-feedback/`, `skills/commit-push/`, or `skills/solutions/` into your project's `.claude/skills/` directory.

## Usage

### `teardown`

```
/teardown path/to/bundle.js
/teardown https://cdn.example.com/app.min.js
/teardown https://example.com
```

Or with a focus phrase (narrows the artifact to one area of the target):

```
/teardown app.min.js focus on auth flow
/teardown app.min.js just the retry logic
```

Or from a natural description:

> "Reverse-engineer this bundle and map out its data flow."
> "Break down how this library handles auth — ignore the rest."

The skill will:

1. Detect input shape (local path vs URL) and classify scope (Lightweight / Standard / Deep) from size, minification, bundler signature, and obfuscation markers.
2. For URL inputs, emit a one-line permission acknowledgment question before fetching. For local files, skip the gate.
3. Fetch or read the target; record sha256 + byte count; detect bundler signature (Webpack / Rollup / Vite / esbuild / Parcel / esm.sh / none) and minification state.
4. Scan `docs/*/` for a related PRD/SPEC folder — if the torn-down file maps to an existing feature, offer smart-reuse of the folder (writes `TEARDOWN.md` next to the PRD/SPEC and adds a forward-reference).
5. Run the hydration pipeline: Rung 1 source-map discovery (`//# sourceMappingURL=`, sibling `.map`, common dev paths) → `shuji` extract; Rung 2 beautification (`prettier` → `js-beautify` → LLM fallback); Rung 3 (Deep only) decompilation / identifier renaming (`wakaru` for Rollup/esbuild, `webcrack` for Webpack + obfuscator.io, `humanify` for LLM-driven rename hints); Rung 4 read-as-is fallback with unverified labels. Every tool is optional — the pipeline degrades cleanly when `npx` is unavailable.
6. Save intermediate hydration sidecars under `docs/<DATE>-<slug>/teardown-sources/{beautified,hydrated,renamed}/` when the user approves.
7. Ask ≤3 clarifying questions, one at a time, only on genuine ambiguity (which bundle among many, intended outcome — understanding vs replication vs security, focus scope).
8. Build the structural map: entry points, module layout (if bundled), function inventory (signature + location + role), state inventory (scope + lifecycle).
9. Walk behaviour — happy path, branches, error paths, event surfaces — each step anchored to a `file:line`.
10. Map data flow — inputs, transformations, outputs, coupling — and produce a Mermaid diagram (mandatory on Standard/Deep).
11. Inventory external surface: network requests (method + URL template + auth + payload shape), DOM touchpoints, storage keys, workers, offloaded Promises, timers.
12. If focus mode is active, narrow Phases 3-5 to the focus slice and cross-link the rest with `file:line` pointers.
13. Enumerate unknowns — every `(unverified)` claim gets a row with its location and the reason it could not be verified — then add a residual-risk paragraph per unknown explaining what a reader might break if they act on it.
14. Emit a self-review checklist (every cited function was read; every identifier meaning is anchored in source-map / beautified cross-reference / call-site evidence; diagram matches narrative matches code; the artifact is obviously time-bound).
15. **HARD GATE** — propose the direction, the output path, and the sidecar scope; wait for approval.
16. Write `docs/<DATE>-<slug>/TEARDOWN.md` and any approved sidecars. Never edits source, never commits, never chains into another skill — the user decides when the artifact is ripe enough to compose with `/prd`, `/spec`, `/diagnosis`, or `/execute`.

### `prd`

```
/prd faster mobile checkout flow
```

Or from a natural description:

> "I want to brainstorm a PRD for a new notification system."

The skill will: classify scope, pre-scan the repo, ask up to 5 clarifying questions one at a time, frame the problem, generate 2-3 distinct options, recommend a direction, emit a self-review checklist, propose `docs/<DATE>-<slug>/PRD.md`, and wait for approval before writing.

### `spec`

```
/spec persisted checkout sessions
```

Or from a natural description:

> "Write a tech spec for the single-tap checkout approved in the PRD."

The skill will:

1. Detect an adjacent `PRD.md` in `docs/` — if one matches, offer to build on it (same folder as the PRD).
2. Classify scope (Lightweight / Standard / Deep) and pre-scan the repo.
3. Ask up to 5 clarifying questions, focused on SCALE / CONSISTENCY / INTEGRATION / COMPATIBILITY / OPERABILITY — fewer when PRD context already answers WHY / WHO / SUCCESS.
4. Frame the technical problem (goals, non-goals, constraints, verified assumptions).
5. Describe Current → Proposed architecture at the module level.
6. Capture every real decision as a mini-ADR with alternatives.
7. Detail data model, API contracts, module boundaries.
8. Draft 1-4 tracer-bullet phases with acceptance criteria.
9. Specify test strategy and risks.
10. Offer / include a Mermaid diagram (mandatory for Deep).
11. Run an adversarial red-team pass (staff engineer, security reviewer, future maintainer).
12. Emit a self-review checklist.
13. Propose a path and wait for approval.
14. Write `docs/<DATE>-<slug>/SPEC.md`.

### `revision`

```
/revision
```

Or with an explicit target:

```
/revision docs/2026-04-16-faster-checkout/
```

Or from a natural description:

> "Cross-check the PRD and SPEC in the faster-checkout folder — anything misaligned?"

The skill will:

1. Auto-detect the target folder (`docs/*/`) or accept an explicit path. If multiple PRD+SPEC pairs match, ask one question.
2. If only one of `PRD.md` / `SPEC.md` is present, offer a degraded single-doc review (coherence + adversarial only, cross-doc skipped).
3. If `REVISION.md` already exists in the folder, ask whether to resume (overwrite), create `REVISION-2.md`, or archive the old one.
4. Classify scope (Lightweight / Standard / Deep) by inheriting MAX complexity from PRD/SPEC frontmatter.
5. Ask up to 3 clarifying questions, one at a time, only if auto-detection is genuinely ambiguous.
6. Read both documents in full; emit a short Grounding Summary the user can correct.
7. Run the 4-lens composite pass (cross-doc alignment, internal coherence, adversarial premise, scope creep + feasibility).
8. Classify each finding with severity (P0/P1/P2/P3), finding type (error/omission), target (PRD/SPEC/PRD+SPEC), evidence quotes, and a minimal proposed fix.
9. Surface low-confidence findings in a Suppressed table rather than silently dropping them.
10. Emit a self-review checklist.
11. HARD GATE 1 — propose path and findings summary; wait for approval to write REVISION.md.
12. Write `docs/<DATE>-<slug>/REVISION.md`.
13. HARD GATE 2 — propose the patch plan; wait for approval to edit PRD.md and SPEC.md.
14. On approval, apply the minimal fixes and append `## Revision <DATE>` bullets (cross-referencing finding IDs) to each source document.

### `diagnosis`

```
/diagnosis checkout 500 after coupon apply
```

Or with an issue reference:

```
/diagnosis #1234
/diagnosis ABC-456
/diagnosis https://github.com/org/repo/issues/1234
```

Or from a natural description, including pasted stack traces and failing test output:

> "The integration test `coupon.integration.ts` started failing after the 2026-04-10 deploy. Stack trace attached."

The skill will:

1. Auto-detect issue-tracker references and fetch the report via `gh issue view` or the Atlassian MCP.
2. Resume-check `docs/` for an existing DIAGNOSIS on the same bug; offer to reuse or start fresh.
3. Scan `docs/*/` for a related PRD/SPEC folder — if the bug maps to a known feature, offer smart-reuse of the folder (appends a `## Post-Release Bug Fix` pointer to the primary doc on write).
4. Classify scope (Lightweight / Standard / Deep).
5. Ask ≤3 clarifying questions, one at a time, only on genuine ambiguity (expected vs actual, reproduction, recent changes, environment, prior attempts).
6. Reproduce consistently, check recent changes (`git log`, `git diff`), read errors carefully.
7. Trace data flow backward through the call chain to the original trigger — not just where the failure surfaces.
8. Find working examples of the same pattern and diff against the broken path.
9. Generate **≥3 structurally different hypotheses**, each with evidence FOR, evidence AGAINST, and a verdict (CONFIRMED / REJECTED / INCONCLUSIVE).
10. Build a full **causal chain** from trigger to symptom with no gaps; add **predictions for uncertain links** (if a prediction fails but the fix "works," the real cause is still active).
11. Write a **reproduction test** that fails RED for the right reason; paste literal command + literal output in a fenced proof block. UNTESTABLE only with a one-paragraph justification.
12. Classify severity (TRIVIAL / STANDARD / COMPLEX) and propose a minimal fix with hotspots and concerns.
13. Offer / include a Mermaid diagram (mandatory for Deep).
14. Emit a self-review checklist.
15. Propose a path and wait for approval.
16. Write `docs/<DATE>-<slug>/DIAGNOSIS.md` (or append inside an existing spec folder); never modify production source code.

### `execute`

```
/execute
```

Or with an explicit path:

```
/execute docs/2026-04-16-persisted-checkout/SPEC.md
/execute docs/2026-04-16-checkout-500-coupon/DIAGNOSIS.md
/execute docs/2026-04-17-auth-refactor/CODE-REVIEW.md
```

Or from a direct human prompt:

> "Improve the address form validation — zip codes should reject non-numeric input."
> "/execute I just found this 500 on `POST /api/checkout` after a coupon is applied, please fix."
> "/execute apply the code-review fixes in the auth-refactor folder."
> "/execute address the blockers from the last review."

The skill will:

1. Check the current git branch. Refuse to proceed on `main`, `master`, `production`, `prod`, `stable`, `live`, `trunk`, or `release*` and ask the user to create a feature branch first.
2. Auto-detect the source of truth in priority order: explicit path (`CODE-REVIEW.md → FIX`, `SPEC.md → FEATURE`, etc.) → FIX-mode prompt triggers + newest `CODE-REVIEW.md` with `status: approved` → `REVISION.md` with `patches_applied: true` → `SPEC.md` with `status: approved` → `DIAGNOSIS.md` with `status: approved` → `PRD.md` alone (warns and suggests `/spec` first) → raw prompt (RAW mode). When a detected primary artifact shares a folder with an approved CODE-REVIEW.md that has open findings, the skill offers a one-question FIX-mode switch before advancing.
3. Resume-check for an in-progress `EXECUTION.md` in the target folder; offer to resume or restart.
4. (FIX mode only) **Findings triage at Phase 0.5** — parse every finding in the latest findings section of CODE-REVIEW.md, subtract any finding already addressed by a prior `## Fixes applied <DATE>` section, and display a compact P0/P1/P2/P3 summary. Ask the user to pick: **A. All P0 + P1 (default)** / **B. P0 + P1 + P2** / **C. Every finding shown** / **D. Custom** (include/skip by ID) / **E. Abort**. Selected findings drive the slice plan; skipped findings are recorded with user-supplied reasons for the EXECUTION.md audit trail and remain open for the next re-review.
5. Run a fast, cheap codebase pre-scan — instruction files, top-level structure, tech-stack fingerprint, touch-area files + nearest tests, one pattern exemplar. Findings are held internally; the user sees only `execute: context loaded.`
6. Classify scope (Lightweight / Standard / Deep) based on signals (file count, concurrency, SPEC complexity, FIX-mode selected-finding tally).
7. Ask ≤3 clarifying questions, one at a time, only on genuine ambiguity (definition of done, scope boundaries, test baseline). Usually 0-1 when an artifact is the source of truth — FIX mode typically asks zero because the findings are the spec.
8. Derive a vertical-slice plan — one slice per SPEC phase in FEATURE mode, one slice (two if defense-in-depth was recommended) in BUG mode, one slice per selected finding (or grouped when findings share file + fingerprint root) in FIX mode, or an internal micro-spec + 1-4 slices in RAW mode.
9. Emit a self-review checklist.
10. **HARD GATE 1** — propose the slice plan, scope tier, branch, output path, commit style, and (FIX mode) the selected vs skipped findings; wait for approval before touching any file.
11. Execute slice-by-slice: **RED** (write the failing test, run it, capture literal output) → **GREEN** (write the minimum code to pass) → **REFACTOR** (only while green) → full suite + lint + types → **COMMIT** (Conventional Commits format — FIX mode commits reference the finding ID in the body — explicit files, no push, no PR). Non-behavior slices (config, build files, type-only, docs, pure formatting) may use **Verification Mode** — skip RED, capture a full-build + full-suite green proof instead.
12. On a 3-strike RED failure, STOP coding and invoke the Step-Back protocol — do not attempt a fourth approach; offer to escalate to `/diagnosis` or re-open `/spec`.
13. Log out-of-scope observations with the **NOTICED BUT NOT TOUCHING** prefix; never act on them mid-execute. In FIX mode, this applies doubly — tempting adjacent issues surfaced by the code review that the user did NOT select at triage must be logged, not fixed.
14. Run the integration gate — fresh full test suite, lint, and type-check; audit skipped tests; (BUG mode and FIX mode when practical) capture a regression red-green proof by reverting the fix → test RED → restoring the fix → test GREEN. In FIX mode, at least one red-green proof per addressed P0 finding.
15. **HARD GATE 2** — blocks writing `EXECUTION.md` on anything other than a fully green integration pass (or an explicit user-approved carve-out).
16. Write `docs/<DATE>-<slug>/EXECUTION.md` with literal test output for every slice, the integration gate log, acceptance criteria trace, NOTICED-BUT-NOT-TOUCHING list, follow-ups, concerns, the commit table, and (FIX mode) Findings Addressed + Findings Skipped tables. Append the back-pointer: `## Implemented <DATE>` on SPEC.md (FEATURE), `## Fixed <DATE>` on DIAGNOSIS.md (BUG), or `## Fixes applied <DATE>` on CODE-REVIEW.md with addressed-vs-skipped breakdown (FIX). Never push, never open a PR. Next step for the user in FIX mode: re-run `/code-review` to append a `## Re-review <DATE>` section showing the delta.

### `code-review`

```
/code-review
```

Or with a scope override or base hint:

```
/code-review deep
/code-review vs origin/develop
```

Or from a natural description:

> "Review my current changes before I commit."
> "Audit this diff against the SPEC."
> "Red-team this PR before I open it."

The skill will:

1. Detect the diff in priority order — uncommitted staged + unstaged (via `git status` + `git diff`) → branch-vs-base (`git diff <base>...HEAD`) → `HEAD~1` fallback. If no changes are found, stop and tell the user.
2. Scan `docs/*/` for an adjacent spec folder (PRD / SPEC / DIAGNOSIS / REVISION / EXECUTION). If the diff clearly maps to a feature folder, offer to write inside it; otherwise derive a new `docs/<DATE>-<slug>/`.
3. Resume-check for an existing `CODE-REVIEW.md` at the target path — offer to append a `## Re-review <DATE>` section (recommended), overwrite, or write to `CODE-REVIEW-N.md`. On **Append**, also parse any prior `## Fixes applied <DATE>` sections (left by `/execute` in FIX mode) into `claimed_addressed` / `claimed_skipped` lists for Phase 6.8 claim verification.
4. Detect plan document(s) in the target folder; if present, Stage 1 plan-alignment will run and block Stage 2 on P0 drift.
5. Classify scope (Lightweight / Standard / Deep) from LOC, file count, file paths, and plan complexity; auto-trigger the security and performance reviewers when the diff matches their sensitive-code patterns (user-input handlers, auth, injection-prone primitives, DB queries, fan-out, unbounded loops, cron / queue consumers, render paths).
6. Ask ≤3 clarifying questions, one at a time, only on genuine ambiguity (base branch, folder reuse, scope override, resume mode).
7. Emit a short Grounding Summary the user can correct — diff source, scope, reviewers queued, write target.
8. **Stage 1 (plan-alignment, blocking on Standard/Deep)** — dispatch the plan-alignment reviewer alone. If P0 drift is reported, pause and offer fix-drift / update-plan / proceed-with-drift.
9. **Stage 2 (parallel fan-out)** — dispatch code-quality + convention + test + conditional security/performance reviewers in a single message with multiple Agent tool calls. Collect JSON payloads.
10. **Stage 3 (Deep only)** — dispatch the adversarial reviewer with Stage 2's merged findings as input. Collect JSON.
11. Merge findings: renumber IDs, deduplicate by fingerprint (`normalize(file) + line_bucket(line, ±3) + normalize(intent)`), apply +0.10 cross-reviewer agreement boost per additional reviewer (capped at 1.0), suppress findings below 0.60 confidence (P0 exception at 0.50+), split pre-existing findings into their own table, collate positives / residual risks / testing gaps, audit the diff inventory for files no reviewer opened.
12. **(Re-review only) Claim verification** — fingerprint-match each prior claim against the merged findings. Addressed-but-still-present elevates to `claim_mismatch: true` with a `+0.15` confidence boost (bypassing the suppression gate); skipped-but-still-present is tagged `verifies_claim` without mismatch (expected); skipped-but-vanished becomes a verification note. Assemble a `verification_table` for the re-review render. Prior `## Fixes applied` sections are read-only and never edited.
13. Emit a self-review checklist.
14. **HARD GATE** — propose the finding set, scope tier, path, and (on re-review) the claim-verification summary; wait for approval.
15. Write `docs/<DATE>-<slug>/CODE-REVIEW.md` (or append a `## Re-review <DATE>` section into an existing one) with severity-grouped findings, the Suppressed table, Pre-existing table, plan-alignment summary, diff inventory, next-steps list, and — on re-review — a **Verification of prior fixes** subsection with one row per prior claim. Never edits, commits, or pushes.

### `pr-feedback`

```
/pr-feedback https://github.com/owner/repo/pull/123
/pr-feedback owner/repo#123
/pr-feedback #123
```

Or with a scope override:

```
/pr-feedback https://github.com/owner/repo/pull/123 deep
/pr-feedback owner/repo#123 lightweight
```

Or for a merged/closed PR (rare — post-merge retrospective triage):

```
/pr-feedback owner/repo#123 --force-state
```

Or from a natural description:

> "Triage the review comments on my PR."
> "Handle the feedback on #123."
> "Go through the comments on https://github.com/org/repo/pull/456 and tell me which ones matter."
> "I got review comments — help me address them."

The skill will:

1. Parse the PR reference into `{owner, repo, number}`. If a bare `#N` was passed, resolve `owner/repo` from the cwd's remote. If the parsed `owner/repo` does not match the cwd's remote, refuse with a `cd` hint — cross-repo triage is out of scope. Verify `gh auth status`.
2. Fetch PR metadata in a single `gh pr view` call (`state`, `isDraft`, `headRefName`, `headRefOid`, `headRepositoryOwner`, `baseRefName`, `author`, `reviewDecision`, `labels`, `mergeable`, `url`, `title`, `body`). Refuse `MERGED` / `CLOSED` without `--force-state`; warn + confirm on `isDraft`.
3. Resume-check for an existing `docs/*/PR-FEEDBACK.md` whose frontmatter `pr_url` matches; offer **A. Append** a `## Re-triage <DATE>` section (recommended after new reviewer activity) / **B. Overwrite** (old preserved in git history) / **C. Write to PR-FEEDBACK-2.md** / **D. Abort**.
4. Scan `docs/*/` for a related spec folder (PRD / SPEC / DIAGNOSIS / EXECUTION) — if the PR's branch slug, title, or diff-file set overlaps ≥50% with an existing folder, offer to write inside it (and reconcile with any sibling `CODE-REVIEW.md`); otherwise derive a slug from the PR head branch and create a new dated folder.
5. Classify scope provisionally (Lightweight / Standard / Deep) from base-rate signals (reviewer count from `gh pr view`, diff LOC via `gh pr diff --stat`, sensitive-surface heuristics on the touched paths, `reviewDecision == CHANGES_REQUESTED`); refine once comment counts land in Phase 2.6.
6. **HARD HUMAN GATE — branch checkout** (MANDATORY): inspect the current branch + HEAD SHA. If the working tree is NOT on the PR's `headRefOid`, offer **Option A — run `gh pr checkout <N>` for you now** (handles forks cleanly, recommended) or **Option B — refuse and print the manual commands** (you run them, re-invoke). On B or on Option A failure, STOP with the exact `git fetch <fork-url> <branch>:<local>` / `git checkout <local>` recovery commands — do not attempt triage with a mismatched HEAD.
7. Verify `git rev-parse HEAD == <headRefOid>` after checkout; abort and fall back to refuse-until-correct on mismatch.
8. Fetch all comments in parallel: a single GraphQL `reviewThreads` query (returns thread state + replies + `isResolved` / `isOutdated` / `diffHunk` / `author` / `body` / `url` — REST omits these first-class), a paginated REST call to `/issues/<N>/comments` (timeline), a paginated REST call to `/pulls/<N>/reviews` (top-level review bodies with `state`). Drop `DISMISSED` and `PENDING` reviews (log the skip in the audit trail with reviewer handle + dismissed-at + comment count).
9. Normalize every comment into a shared record shape (id, thread_id, path, line, originalLine, body, author, commit_id, original_commit_id, diffHunk, isResolved, isOutdated, minimizedReason, suggestion_blocks[], source_kind). Dedupe by `thread_id` across the multi-reviewer union.
10. Phase 2.6 scope refinement — if actual comment counts / reviewer count / multi-reviewer contradictions on same `path:line` escalate the tier, announce `pr-feedback: scope escalated <OLD> → <NEW> (<reason>)`; only de-escalate when the provisional tier was clearly inflated and no Phase 4 records exist yet.
11. Ask ≤3 clarifying questions, one at a time, only on genuine ambiguity (folder reuse, resume disposition on an existing PR-FEEDBACK.md, scope override).
12. Cross-reference each comment against the diff (via `gh pr diff`), the spec folder (PRD § / SPEC § / DIAGNOSIS root cause / EXECUTION acceptance criteria), and any sibling `CODE-REVIEW.md` findings by fingerprint (`normalize(file) + line_bucket(line, ±3) + normalize(intent)`). Attempt to re-anchor stale comments (`line == null` OR `isOutdated == true`) by text-searching the `diffHunk` snippet against the current working tree.
13. Triage each comment into one of five verdicts: **must-fix** / **nice-to-have** / **question** / **push-back** / **already-done**, computing a 0-100 confidence score from Conventional Comments label + decorator + spec cross-reference + prior CODE-REVIEW match + anchor freshness + code-level corroboration (see `references/taxonomy.md`).
14. Draft a copy-pasteable GitHub reply per verdict, in the PR author's voice (first-person, no "[reviewer's name]" placeholders, no reflexive thanks) — must-fix acknowledges and commits to a fix path; question answers directly and cites spec § when applicable; push-back is respectful and specific, quoting the spec § or test name that justifies the disagreement; nice-to-have is concise and declines gracefully when deferred; already-done points to the commit SHA.
15. For each must-fix, build a Handoff record — `file:line` (re-anchored when stale, `<unknown>` when re-anchor failed — which forces the user to resolve before `/execute` can consume), intent (what to change), and any verbatim ` ```suggestion ` block the reviewer supplied (so `/execute` applies it as-is).
16. Reconcile with sibling `CODE-REVIEW.md` when present — Agreements (prior finding ↔ current verdict, with confidence boost recorded) + Tensions (reviewer flags what we suppressed, worth re-examining in a future self-review).
17. Collate Residual Risks (stale comments that re-anchoring failed on, multi-reviewer conflicts on the same line, push-back verdicts with marginal confidence + no spec anchor) and a Skipped footer (dismissed reviews audit trail).
18. Suppress verdicts below 60 confidence into a visible Suppressed table (P0 exception: must-fix with confidence ≥ 50 stays top-bar), group resolved threads inside a collapsed `<details>` block.
19. Emit a self-review checklist (mandatory checkout gate cleared with HEAD match, every verdict has a verbatim quote, dismissed reviews logged not hidden, stale comments badged not dropped, confidence math cites the taxonomy factors, Handoff `file:line` exists for every must-fix or is explicitly `<unknown>`, no `/execute` was invoked, no PR posts were made).
20. **HARD GATE** — propose the verdict set, the scope tier, the reconciliation summary, and the output path; wait for approval.
21. Write `docs/<DATE>-<slug>/PR-FEEDBACK.md` (or append a `## Re-triage <DATE>` section into an existing one). The artifact contains severity-grouped verdicts with evidence and drafted replies, Reconciliation (when applicable), Suppressed, Resolved (collapsed), Residual Risks, Skipped (dismissed), and the Handoff table `/execute` FIX mode consumes.
22. Close with a one-sentence next-step hint: `/execute` in FIX mode on this folder to apply the must-fix items, posting the drafted replies is the human's separate turn. The skill never invokes `/execute`, never runs `gh pr comment`, never runs `gh pr review`.

### `commit-push`

```
/commit-push
```

Or with a free-text hint (branch name or subject-line hint):

```
/commit-push feat/refresh-rotation
/commit-push "rotate refresh tokens on every access-token refresh"
```

Or from a natural description:

> "Commit this and push."
> "Save my changes to a new branch."
> "Ship these changes to the remote."

The skill will:

1. Capture the current git state (status, diff, branch, last-10-commits, upstream default) in a single Pre-flight batch. If the working tree is clean, stop and ask whether the user meant to push already-committed work instead. If on a protected branch (`main` / `master` / `production` / `prod` / `stable` / `live` / `trunk` / `release*`), refuse and ask the user to create a feature branch first. If in detached HEAD, refuse and ask the user to create a branch.
2. Classify scope (Lightweight / Standard / Deep) from file count, line count, mixed-concern signals, and trust-boundary / migration / concurrency flags.
3. **HARD HUMAN GATE — branch decision**: emit an A/B question — A) commit on `<current-branch>` or B) create a new branch (user types the name next). A silent proceed without either answer is a protocol violation. Protected-branch names are re-refused if the user supplies one. Auto-mode does not override this gate.
4. Run the scope-adaptive **pre-commit gate**: secrets scan (matches stop the skill unconditionally), debug-artifact scan (asks keep/remove/skip-scan with record), test suite (Standard/Deep — detected from `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml` / `Gemfile` / project scripts), lint + formatter (Standard/Deep — ESLint / Prettier / Ruff / Rubocop / gofmt / rustfmt / etc.), scope-vs-diff audit (Deep — keep/drop/defer per out-of-scope file), test-coverage-for-code-changes audit (Deep — cover/chore per production-source file missing tests, with a one-sentence justification per `chore` that is recorded in the commit body).
5. Cluster changed files by concern. If ≥2 distinct clusters emerge (feat + refactor, or feat + test-infra), propose a split via `AskUserQuestion` listing each cluster's files and drafted subject — human approves, collapses, or re-groups. Max 3 commits per run. File-level clustering only.
6. Draft commit message(s): subject + body + Testing scenarios block (mandatory for feat/fix on Standard/Deep). Convention precedence: repo instructions (CLAUDE.md / AGENTS.md / CONTRIBUTING.md) → commitlint config → pattern in the last 10 commits → Conventional Commits default. Subject is imperative, lowercase-first, ≤72 chars, no trailing period. Show each draft via `AskUserQuestion` — approve / edit / rewrite. User-written messages are still validated.
7. Emit a self-review checklist covering the branch gate, pre-commit gate, no-secrets, staging plan, convention match, body requirements, split plan, back-pointer target, no-force / no-amend / no-no-verify / no-PR, and rebase plan. Any `✗` is fixed before advancing.
8. **HARD GATE — final confirmation**: emit an approve / change / abort question summarizing branch, scope, commits, staged file count, and back-pointer target. Nothing mutates until approved.
9. Stage explicit files by name (never `git add -A`) and commit each group with a heredoc-preserved message. If a commit fails a pre-commit hook, STOP and hand back — no `--no-verify`, no `--amend`.
10. Fetch + rebase: `git fetch origin` then `git pull --rebase origin <upstream-default>`. Non-trivial conflicts stop the skill with a show-and-hand-back; trivial whitespace-only conflicts may be resolved in-skill with a one-paragraph explanation.
11. Push: `git push -u origin <branch>`. On rejection, re-fetch-rebase and retry exactly once. Never `--force`, never `--force-with-lease` — if still rejected, hand back to the user.
12. **Optional back-pointer**: scan for an adjacent `docs/<DATE>-<slug>/EXECUTION.md` (status `completed` / `in_progress`, recently touched) or `DIAGNOSIS.md` (status `approved`, hotspots overlap this diff). If found, append a compact `## Commits <DATE>` section with SHA + subject per commit and the target branch. Existing sections are never rewritten; existing frontmatter is never mutated.
13. Report the commit summary (branch, SHAs, subject lines, pushed branch, back-pointer target) and STOP. No PR creation, no `/code-review` auto-invoke, no "shall I open the PR?" — the user owns the next step in a separate turn.

### `solutions`

```
/solutions
```

Or with an explicit target folder:

```
/solutions docs/2026-04-16-persisted-checkout/
```

Or from a natural description:

```
/solutions capture what we learned from the coupon-apply fix
```

> "Distil the learnings from the faster-checkout pipeline into a SOLUTIONS.md."
> "We're done with this folder — write the solutions doc."

The skill will:

1. Detect the target folder. If a single `docs/*/` folder was just worked in (recent activity + artifacts present), offer it. If multiple candidates exist, ask once. If no folder is resolvable, stop and ask.
2. Enumerate upstream artifacts in the folder (PRD.md / SPEC.md / REVISION.md / DIAGNOSIS.md / EXECUTION.md / CODE-REVIEW.md, including any `## Re-review <DATE>` / `## Re-run <DATE>` / `## Post-Release Bug Fix <DATE>` appended sections). Classify the pipeline: FEATURE / BUG / MIXED.
3. Resume-check: if `SOLUTIONS.md` already exists, offer **A. Append a new `## Re-run <DATE>` section** (default for a feature re-run / refactor) / **B. Append a new `## Post-Release Bug Fix <DATE>` section** (when a DIAGNOSIS was appended to a feature folder after SOLUTIONS was first written) / **C. Archive the old one as `SOLUTIONS-2.md`** / **D. Abort**. No overwrite-in-place option — prior synthesis is history.
4. Classify scope by inheriting MAX of upstream artifact `scope` frontmatter values (LIGHTWEIGHT / STANDARD / DEEP). Promote to Standard if any upstream artifact showed a multi-strike Step-Back, a REJECTED hypothesis, or P0/P1 CODE-REVIEW findings not otherwise reflected. Never downgrade.
5. Ask 0-2 clarifying questions, one at a time, only on genuine ambiguity (folder identity, re-run disposition, mixed-pipeline scope).
6. Read every upstream artifact in full. Extract: Summary / Approach candidates from SPEC mini-ADRs + EXECUTION per-slice intent; Root Cause from DIAGNOSIS; Key Decisions from SPEC mini-ADRs that proved their weight + REVISION patches + EXECUTION mid-flight decisions + CODE-REVIEW deliberate drift acceptance; Gotchas from EXECUTION NOTICED-BUT-NOT-TOUCHING + Concerns + DIAGNOSIS concerns + CODE-REVIEW residual risks + testing gaps; What Didn't Work from EXECUTION 3-strike Step-Back events + DIAGNOSIS REJECTED hypotheses + CODE-REVIEW suppressed findings; Prevention from DIAGNOSIS suggested fix + defense-in-depth recommendation + CODE-REVIEW testing gaps.
7. Apply the Iron Law — rewrite every extracted learning as a behaviour invariant. File paths and line numbers appear (if at all) as illustration inside prose clauses, never as the substance of a learning.
8. Draft SOLUTIONS.md content against the scope-tier template: Lightweight keeps Summary + Approach + Key Decisions + Gotchas + References (+ Root Cause + Prevention for bugs); Standard adds What Didn't Work and requires mini-ADR Alternatives, offers a Mermaid diagram once if structure warrants; Deep requires a Mermaid diagram (sequence / state / flowchart) of the durable mechanism and populates Relationship to Original Spec for mixed pipelines.
9. Emit a self-review checklist: Iron Law compliance (zero `src/...:42` references as the substance of any learning), every claim traceable to an upstream quote / table row, section coverage matches the scope tier, no synthetic padding (empty sections carry an explicit "intentionally empty — <one-word reason>"), re-run disposition is correct (append target chosen, not overwrite).
10. **HARD GATE** — propose the summary, the learnings set, the scope tier, and the output path; wait for approval.
11. Write `docs/<DATE>-<slug>/SOLUTIONS.md` (or append `## Re-run <DATE>` / `## Post-Release Bug Fix <DATE>` to the existing file, depending on Step 3's disposition). Update `date` frontmatter; flip `post_release_bug_fix_appended: true` where applicable.
12. Close with a one-sentence transition — if the synthesis surfaced a durable, project-wide learning worth promoting to `CLAUDE.md` / `AGENTS.md`, name the opportunity and hand back to the user. The skill never edits those files itself.

## Layout

```
eva/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── spec-staff-engineer-reviewer.md
│   ├── spec-security-reviewer.md
│   ├── spec-future-maintainer-reviewer.md
│   ├── revision-adversarial-review.md
│   ├── code-review-plan-alignment-reviewer.md
│   ├── code-review-quality-reviewer.md
│   ├── code-review-convention-reviewer.md
│   ├── code-review-test-reviewer.md
│   ├── code-review-security-reviewer.md
│   ├── code-review-performance-reviewer.md
│   └── code-review-adversarial-reviewer.md
└── skills/
    ├── teardown/
    │   ├── SKILL.md
    │   ├── templates/
    │   │   └── TEARDOWN.md
    │   └── references/
    │       ├── scope-tiers.md
    │       ├── hydration-pipeline.md
    │       └── anti-patterns.md
    ├── prd/
    │   ├── SKILL.md
    │   ├── templates/
    │   │   └── PRD.md
    │   └── references/
    │       ├── scope-tiers.md
    │       └── section-guides.md
    ├── spec/
    │   ├── SKILL.md
    │   ├── templates/
    │   │   └── SPEC.md
    │   └── references/
    │       ├── scope-tiers.md
    │       └── section-guides.md
    ├── revision/
    │   ├── SKILL.md
    │   ├── templates/
    │   │   └── REVISION.md
    │   └── references/
    │       ├── scope-tiers.md
    │       └── review-checklist.md
    ├── diagnosis/
    │   ├── SKILL.md
    │   ├── templates/
    │   │   └── DIAGNOSIS.md
    │   └── references/
    │       ├── scope-tiers.md
    │       ├── investigation-techniques.md
    │       └── anti-patterns.md
    ├── execute/
    │   ├── SKILL.md
    │   ├── templates/
    │   │   └── EXECUTION.md
    │   └── references/
    │       ├── scope-tiers.md
    │       ├── tdd-discipline.md
    │       └── anti-patterns.md
    ├── code-review/
    │   ├── SKILL.md
    │   ├── templates/
    │   │   └── CODE-REVIEW.md
    │   └── references/
    │       ├── findings-schema.md
    │       ├── scope-tiers.md
    │       └── anti-patterns.md
    ├── pr-feedback/
    │   ├── SKILL.md
    │   ├── templates/
    │   │   └── PR-FEEDBACK.md
    │   └── references/
    │       ├── scope-tiers.md
    │       ├── taxonomy.md
    │       └── github-api.md
    ├── commit-push/
    │   ├── SKILL.md
    │   └── references/
    │       ├── scope-tiers.md
    │       ├── commit-message-style.md
    │       └── anti-patterns.md
    └── solutions/
        ├── SKILL.md
        ├── templates/
        │   └── SOLUTIONS.md
        └── references/
            ├── scope-tiers.md
            ├── section-guides.md
            └── anti-patterns.md
```

## How `teardown`, `prd`, `spec`, `revision`, `diagnosis`, `execute`, `code-review`, `pr-feedback`, `commit-push`, and `solutions` compose

```
  idea ──▶ prd ──▶ PRD.md ──▶ spec ──▶ SPEC.md ──▶ revision ──▶ REVISION.md ──▶ execute ──▶ EXECUTION.md + commits ──▶ code-review ──▶ CODE-REVIEW.md ──▶ commit-push ──▶ commits + push ──▶ solutions ──▶ SOLUTIONS.md
                                                       │                            │                                      │                                 │                                     │
                                                       │                            └──▶ appends ## Implemented <DATE>     │                                 │                                     │
                                                       │                                 back-pointer to SPEC.md            │                                 │                                     │
                                                       │                                                                    │                                 │                                     │
                                                       └──▶ on approval, patches PRD.md + SPEC.md with ## Revision <DATE>   │                                 │                                     │
                                                                                                                             │                                 │                                     │
                                                                                              code-review reuses the same folder  ◀──────────────────────────┘ │                                     │
                                                                                              and writes CODE-REVIEW.md next to the PRD / SPEC                  │                                     │
                                                                                              (re-runs append a ## Re-review <DATE> section)                    │                                     │
                                                                                                                                                                 │                                     │
                                                                                              commit-push appends ## Commits <DATE> back-pointer  ◀─────────────┘                                     │
                                                                                              to the adjacent EXECUTION.md (or DIAGNOSIS.md in fix flows);                                             │
                                                                                              it never opens a PR and never touches a protected branch                                                │
                                                                                                                                                                                                       │
                                                                                              solutions reads every upstream artifact in the folder,  ◀──────────────────────────────────────────────┘
                                                                                              distils durable learnings (Summary, Approach, Key Decisions,
                                                                                              Gotchas, What Didn't Work, + Root Cause/Prevention for bugs,
                                                                                              + Mermaid for Deep) into SOLUTIONS.md in the same folder —
                                                                                              scope inherited from upstream MAX, re-runs append a
                                                                                              ## Re-run <DATE> section, Iron Law of durability enforced
                                                                                              (behaviour invariants, not file:line); never edits
                                                                                              CLAUDE.md / AGENTS.md, never chains to another skill

  ╭─────────────────────────  correction → review loop  ─────────────────────────╮
  │                                                                              │
  │  CODE-REVIEW.md  ──▶  execute (FIX mode)  ──▶  Phase 0.5 findings triage     │
  │       ▲                    │                  (user picks P0/P1/P2/P3 to     │
  │       │                    │                   address vs skip)              │
  │       │                    ▼                                                 │
  │       │            EXECUTION.md  +  commits                                  │
  │       │            (one slice per selected finding;                          │
  │       │             skipped findings logged in audit table)                  │
  │       │                    │                                                 │
  │       │                    ▼                                                 │
  │       │        CODE-REVIEW.md gains  ## Fixes applied <DATE>                 │
  │       │        (addressed + skipped tables; originals untouched)             │
  │       │                    │                                                 │
  │       │                    ▼                                                 │
  │       └──── /code-review  ──▶  verifies prior claims, appends               │
  │                               ## Re-review <DATE>                            │
  │                              (Verification of prior fixes table —            │
  │                               addressed-but-still-present elevates to a      │
  │                               claim_mismatch finding that bypasses           │
  │                               suppression; loop continues until clean)       │
  ╰──────────────────────────────────────────────────────────────────────────────╯

  ╭─────────────────────────  external review → fix loop  ───────────────────────╮
  │                                                                              │
  │  GitHub PR (teammate reviews)                                                │
  │       │                                                                      │
  │       ▼                                                                      │
  │  /pr-feedback  ──▶  branch-checkout gate (offers gh pr checkout)             │
  │       │             ──▶ GraphQL reviewThreads + REST timeline + reviews      │
  │       │             ──▶ union non-dismissed · dedupe by thread_id            │
  │       │             ──▶ classify into 5 verdicts (must-fix / nice-to-have /  │
  │       │                 question / push-back / already-done) with 0-100     │
  │       │                 confidence                                          │
  │       │             ──▶ reconcile with sibling CODE-REVIEW.md (agreements    │
  │       │                 +15, tensions surface)                               │
  │       ▼                                                                      │
  │  PR-FEEDBACK.md  ──▶  must-fix Handoff table  ──▶  /execute (FIX mode)       │
  │       │                                                 │                    │
  │       │                                                 ▼                    │
  │       │                                         EXECUTION.md + commits       │
  │       │                                         (same Phase 0.5 triage,      │
  │       │                                          same TDD discipline)        │
  │       ▼                                                 │                    │
  │  copy-paste drafted replies to the PR                   ▼                    │
  │  (human's separate turn — never auto-posted)    /commit-push  ──▶  push      │
  │                                                         │                    │
  │                                                         ▼                    │
  │                                             reviewer sees the commits,      │
  │                                             re-reviews or approves;         │
  │                                             a new /pr-feedback run on the   │
  │                                             next round appends              │
  │                                             ## Re-triage <DATE>              │
  ╰──────────────────────────────────────────────────────────────────────────────╯

  bug report ──▶ diagnosis ──▶ DIAGNOSIS.md ──▶ execute ──▶ EXECUTION.md + commits ──▶ code-review ──▶ CODE-REVIEW.md ──▶ commit-push ──▶ commits + push ──▶ solutions ──▶ SOLUTIONS.md
                     │                            │                                         │                                   │                                     │
                     │                            └──▶ appends ## Fixed <DATE> back-pointer │                                   │                                     │
                     │                                 to DIAGNOSIS.md                      │                                   │                                     │
                     │                                                                      │                                   │                                     │
                     └──▶ if the bug maps to an existing                                    │                                   │                                     │
                          docs/<DATE>-<slug>/ feature folder,                                │                                   │                                     │
                          reuses that folder and appends              code-review writes    │                                   │                                     │
                          ## Post-Release Bug Fix to SPEC.md / PRD.md next to the DIAGNOSIS ◀                                   │                                     │
                                                                      (plan-alignment uses it as the plan reference)            │                                     │
                                                                                                                                 │                                     │
                                                                      commit-push appends ## Commits <DATE> back-pointer ◀──────┘                                     │
                                                                      to DIAGNOSIS.md (fix flow); never opens a PR                                                     │
                                                                                                                                                                       │
                                                                      solutions populates Root Cause + Prevention from DIAGNOSIS,  ◀──────────────────────────────────┘
                                                                      What Didn't Work from REJECTED hypotheses, regression-protection
                                                                      test named (not file-pathed); when the bug fix was appended to a
                                                                      feature folder, the synthesis becomes a MIXED pipeline and the
                                                                      Relationship to Original Spec section is populated

  direct prompt ──▶ execute (RAW mode) ──▶ internal micro-spec + pre-scan → slices → EXECUTION.md + commits ──▶ code-review ──▶ CODE-REVIEW.md ──▶ commit-push ──▶ commits + push ──▶ solutions ──▶ SOLUTIONS.md
                                                                                                               (no plan reference → Stage 1 skipped, stages 2-3 run on diff shape)      (RAW mode pipelines are typically Lightweight;
                                                                                                                                                                                        solutions is optional but still useful when the change
                                                                                                                                                                                        surfaced a non-obvious invariant worth recording)

  current diff ──▶ code-review ──▶ CODE-REVIEW.md ──▶ commit-push ──▶ commits + push
                        │                                  │
                        │                                  └──▶ human-gated branch + per-file staging +
                        │                                       fetch+rebase before push; no PR, no force,
                        │                                       never on main/master/production/prod/stable/
                        │                                       live/trunk/release*
                        │
                        └──▶ auto-detects diff scope (uncommitted > branch-vs-base > HEAD~1)
                             and smart-reuses an adjacent spec folder when the diff maps to one

  any branch ──▶ commit-push ──▶ commits + push
                      │
                      └──▶ standalone entry-point: current working-tree diff through scope-adaptive
                           pre-commit gate → branch A/B question → commit split draft → human approve →
                           fetch+rebase → push -u. Works with or without any upstream eva artifact;
                           appends a back-pointer only if an adjacent EXECUTION.md or DIAGNOSIS.md exists.

  any folder ──▶ solutions ──▶ SOLUTIONS.md
                      │
                      └──▶ standalone synthesis entry-point: reads every upstream artifact in a
                           docs/<DATE>-<slug>/ folder (PRD / SPEC / REVISION / DIAGNOSIS /
                           EXECUTION / CODE-REVIEW, including any appended ## Re-run /
                           ## Re-review / ## Post-Release Bug Fix / ## Fixes applied sections),
                           inherits scope from upstream MAX, distils durable learnings into
                           Summary + Approach + Key Decisions + Gotchas + What Didn't Work +
                           References (+ Root Cause + Prevention for bugs, + Mermaid for Deep),
                           appends ## Re-run <DATE> on a second run rather than overwriting,
                           and writes SOLUTIONS.md inside the folder. Never edits source code,
                           never edits upstream artifacts, never edits CLAUDE.md / AGENTS.md —
                           mentions the opportunity in the transition if warranted; the user
                           owns any durable-instruction edit on a separate turn.
```

Each artifact is standalone and durable:

- **TEARDOWN** answers *what does this unfamiliar file actually do* — a structural map of the target (entry points, function inventory, state, data flow, external surface, edge cases), anchored to `file:line` pointers into the original or hydrated source, with a Mermaid diagram of the durable mechanism and a residual-risk section listing every `(unverified)` claim and what a reader might break by acting on it. The only eva artifact that reads input from outside the repo (local path or URL) and the only one that writes sidecars (`teardown-sources/`). Strictly read-only: never edits source, never commits, never opens a PR. Composes one-way with the downstream skills — on approval the user can run `/prd` to brainstorm a replacement, `/spec` to design a port, or `/diagnosis` to investigate a regression — but `teardown` itself never auto-invokes any of them.
- **PRD** answers *what and why* — readable by PMs and engineers.
- **SPEC** answers *how, technically* — readable by engineers, grounded in the codebase, full of embedded mini-ADRs, with tracer-bullet phases that survive refactors.
- **REVISION** answers *where the pair disagrees and what to fix* — evidence-grounded findings with severity, minimal proposed fixes, and an audit record that the review happened (clean pass or otherwise). The adversarial lens is dispatched to a dedicated `revision-adversarial-review` sub-agent so its read is not biased by the orchestrator's narrative.
- **DIAGNOSIS** answers *what broke, why, and how to prove it* — an investigation trail, 3+ structural hypotheses, a full causal chain with predictions, a reproduction test with RED proof, a root cause pinned to file:line, a suggested minimal fix, hotspots, and concerns. Strictly read-only in source code.
- **EXECUTION** answers *what shipped, how, and what's proven* — literal RED and GREEN proofs per vertical slice, an integration-gate log with full suite + lint + types, acceptance-criteria trace mapping each SPEC criterion / DIAGNOSIS reproduction test / selected CODE-REVIEW finding to the slice that satisfies it, NOTICED-BUT-NOT-TOUCHING list, follow-ups, concerns, the Conventional Commits table, and (FIX mode) the Findings Addressed + Findings Skipped tables that give the next reviewer a clean audit trail of what was handled vs deferred. The only eva artifact produced alongside real code changes.
- **CODE-REVIEW** answers *what a careful pre-merge review would surface* — severity-grouped findings from up to seven specialist reviewers, verbatim evidence for every finding, merged across reviewers with fingerprint dedup and cross-reviewer agreement boosts, confidence-gated suppression in a visible table, pre-existing problems separated into their own table, plan-alignment summary when a plan document is present, diff inventory mapping every changed file to the reviewers who opened it, and a next-steps action list. Pure reporter — never writes, commits, or pushes code.
- **PR-FEEDBACK** answers *which reviewer comments actually need action, and how should I reply* — five-bucket triage (must-fix / nice-to-have / question / push-back / already-done) over the union of every non-dismissed reviewer's inline comments (GraphQL `reviewThreads` so `isResolved` / `isOutdated` / reply chains are first-class), timeline comments, and review bodies. Each verdict carries a 0-100 confidence score from Conventional Comments label + decorator + spec cross-reference + prior CODE-REVIEW fingerprint agreement + anchor freshness, verbatim evidence, a first-person copy-pasteable reply draft, and (for must-fix) a Handoff record that `/execute` FIX mode consumes directly. Stale comments are badged not dropped, resolved threads are collapsed not hidden, dismissed reviews are logged in a Skipped footer, low-confidence verdicts (<60) surface in a visible Suppressed table (P0 exception at ≥50). Reconciles with a sibling CODE-REVIEW.md when present — agreements boost must-fix confidence (+15), tensions surface for future self-review. Pure triage — never posts to the PR (the human owns posting the replies), never edits source, never commits, never invokes `/execute`. The mandatory branch-checkout gate (offer `gh pr checkout` first, refuse-until-correct with manual commands on failure) is non-negotiable — a mismatched HEAD produces wrong verdicts. The only artifact it writes is `PR-FEEDBACK.md`; re-triage on the same PR appends `## Re-triage <DATE>` rather than overwriting.
- **`commit-push` produces no standalone artifact** — it takes the working-tree diff through a scope-adaptive pre-commit gate (secrets scan + protected-branch refusal on every tier; + full test suite + lint + debug-artifact scan on Standard; + scope-vs-diff + test-coverage + chore-exemption audit on Deep), a human-gated A/B branch question (always — never silently derived, never on `main` / `master` / `production` / `prod` / `stable` / `live` / `trunk` / `release*`), a human-approved logical split of up to three commits, a convention-matched title + body draft per commit (repo instructions > commitlint > last 10 commits > Conventional Commits default), a fetch + pull `--rebase` before `push -u`, and an optional back-pointer appended to an adjacent EXECUTION.md or DIAGNOSIS.md (never rewrites their content, never mutates their frontmatter). Never runs `gh pr create`, never `git add -A`, never `--force`, never `--no-verify`, never commits secrets.
- **SOLUTIONS** answers *what's worth remembering from this pipeline* — a compact, scope-inherited reference document that a cold-context future session can read before touching the same area again. Summary + Approach + Key Decisions (compact mini-ADRs with genuine Alternatives) + Gotchas (behaviour invariants, phrased to survive a rename) + What Didn't Work (Step-Back attempts, REJECTED hypotheses, suppressed findings — the map of the territory) + References. Bugs add Root Cause and Prevention (class-of-bug + defense-in-depth location, regression-protection test named by test name not file path). Mixed pipelines add Relationship to Original Spec. Deep scope adds a mandatory Mermaid diagram of the durable mechanism. Pure synthesiser — never edits source code, never mutates upstream artifacts, never touches CLAUDE.md / AGENTS.md (the user owns that edit), never chains to another skill. Re-runs append `## Re-run <DATE>` or `## Post-Release Bug Fix <DATE>` sections instead of overwriting, so a folder's SOLUTIONS.md becomes the accretive record of every learning cycle on that feature.
- **`teardown` composes one-way into the rest of the pipeline, never the other way** — it reads an unfamiliar target (local file or URL), produces a structural map in `docs/<DATE>-<slug>/TEARDOWN.md`, and stops. From there the user can hand the artifact to `/prd` to brainstorm a replacement or an adaptation, to `/spec` to design a port grounded in the torn-down reference, or to `/diagnosis` when the target is a production bundle that broke. When the teardown maps to an existing feature folder (e.g. torn-down competitor bundle for an already-scoped feature), the skill smart-reuses the folder and adds a forward-reference into the sibling PRD/SPEC. The skill never auto-invokes any downstream skill, never edits source, never commits — the handoff is always a separate human turn. For URL targets the skill also enforces a one-line permission acknowledgment before fetching (audited in the frontmatter) and refuses to act on any instruction embedded in fetched content.
- **`spec` can run without a PRD** — it will ask the product-framing questions itself when needed (but will suggest starting with `prd` if the problem framing is still vague).
- **`revision` can run on a single document** — if only `PRD.md` or `SPEC.md` is present, it degrades to a single-doc review (internal coherence + adversarial only, cross-doc skipped). The natural flow is still to author both first, then revise.
- **`diagnosis` composes with `spec` and `prd`** — when the bug maps to a known feature, `diagnosis` nests DIAGNOSIS.md inside that feature's folder and back-links the primary doc with a `## Post-Release Bug Fix` section. This keeps the full feature → bug → fix lifecycle in one place.
- **`execute` can run without any upstream artifact** — RAW mode accepts a direct human prompt, derives an internal micro-spec from the prompt + codebase pre-scan, and proceeds through the same red-green-refactor loop. When upstream artifacts exist, it auto-detects them in priority order (explicit-path-by-filename > FIX-mode trigger + CODE-REVIEW.md > REVISION post-patch > SPEC > DIAGNOSIS > PRD-alone-with-warning) and uses them as the source of truth. A direct human prompt like `/execute fix the off-by-one in cart.totals()` works exactly like a compact Lightweight SPEC would.
- **`execute` FIX mode closes the correction → review loop** — pass `docs/<DATE>-<slug>/CODE-REVIEW.md` (or a prompt like `/execute address the blockers`) and the skill parses every finding, asks the human which to address vs skip (default: P0 + P1), slices each selected finding under full TDD, records skipped findings in an audit table, and appends `## Fixes applied <DATE>` to CODE-REVIEW.md. The user then re-runs `/code-review`, which appends `## Re-review <DATE>` with the delta — cleared findings, surviving ones, and any newly-introduced ones. The loop is **human-paced**: nothing auto-loops, every iteration passes through HARD GATE 1 (plan approval) and HARD GATE 2 (green integration gate), and the user can stop at any iteration by simply not running the next skill.
- **`code-review` can run with or without any plan document** — when a plan (PRD / SPEC / DIAGNOSIS / REVISION / EXECUTION) exists in the target folder, Stage 1 plan-alignment uses it as the source of truth and can block Stage 2 on P0 drift; when none exists, Stage 1 is skipped and the pipeline reviews on diff shape alone. The review is always diff-first: scope auto-detects in priority order (uncommitted > branch-vs-base > HEAD~1), smart-reuses the feature's spec folder when the diff maps to one, and re-reviews append under `## Re-review <DATE>` rather than overwriting. It composes naturally as the final gate after `execute` — once implementation commits land, `code-review` audits the exact diff before merge — but is useful standalone too: a `/code-review` on a quick WIP branch surfaces blockers before the author even writes a plan.
- **`pr-feedback` is the inbound counterpart to `code-review` — it closes the external-review loop** — after the author pushes a branch and opens a PR, teammates review on GitHub; `/pr-feedback <pr-url>` then pulls the full comment set, classifies each one into a five-bucket verdict, drafts a reply per comment, and hands the must-fix items to `/execute` FIX mode via the same Handoff table shape that `code-review` produces. The skill **pairs cleanly with an adjacent `CODE-REVIEW.md`** when one exists: reviewer comments that match prior self-review findings by fingerprint strengthen the must-fix bucket (+15 confidence), and tensions ("reviewer flags what we suppressed") surface in a Reconciliation section so the author can re-examine the suppression in a future self-review. The **mandatory branch-checkout gate** distinguishes this skill from every other one in the pipeline — because the diff on disk must match the PR HEAD for anchors to resolve correctly, the skill offers `gh pr checkout` first and refuses-until-correct with exact manual commands on failure; a silent proceed with a mismatched HEAD would produce wrong verdicts. It is **pure triage**: never posts to the PR (the human copy-pastes the drafted replies at their discretion), never edits source, never commits, never runs `/execute` — the handoff is always a separate human turn. Re-triage on the same PR appends `## Re-triage <DATE>` rather than overwriting, so the history of a contentious review is preserved. The skill composes with `/execute` FIX mode (which consumes the Handoff table) → `/code-review` (on the resulting diff) → `/commit-push` (push the fixes) → re-run `/pr-feedback` after the reviewer looks again, forming a loop that mirrors the correction → review loop but for external reviewers.
- **`commit-push` is the last mile and works from any entry-point** — invoked at the end of the pipeline (after `code-review` clears) it commits the working tree and pushes; invoked in the middle of a feature it commits intermediate progress without pushing context the human is not ready to share; invoked standalone on a branch with no eva artifacts at all it still applies the full gate (secrets + protected-branch + scope-adaptive checks + branch A/B question + split draft + message draft + fetch+rebase+push). The skill is deliberately **not** a PR-opening shortcut — after a successful push it reports the remote SHA and hands the human a plain-text hint ("open a PR when you're ready"); it never drafts a PR body in chat and never runs `gh pr create`, so PR creation stays a separate, explicit human action. The only side-effect it writes into eva's docs tree is a `## Commits <DATE>` back-pointer appended to the adjacent EXECUTION.md or DIAGNOSIS.md when one exists — it never rewrites those artifacts' content, never touches their frontmatter, never changes their status.
- **`solutions` is the pipeline's memory terminator** — the last, optional, manually-invoked step after the commits ship. It composes naturally after `commit-push` (a just-shipped feature is the freshest surface to harvest learnings from), after `code-review` even without a shipping step (when the review itself produced durable Gotchas), or months later on an old folder whose learnings the team only now realizes were worth preserving. It **reads every artifact in the folder** — the original PRD / SPEC / REVISION, the DIAGNOSIS (if a bug was handled), the EXECUTION with its per-slice detail and NOTICED-BUT-NOT-TOUCHING list, the CODE-REVIEW with its findings and suppressed table, and any appended `## Re-run`, `## Re-review`, `## Post-Release Bug Fix`, or `## Fixes applied` sections — then distils the durable parts into SOLUTIONS.md in the same folder. Scope is **inherited** (MAX of upstream `scope` frontmatter values) so ceremony matches what the pipeline earned. The Iron Law of durability (behaviour invariants, never file:line) makes the document survive refactors. Re-runs **always append** under `## Re-run <DATE>` or `## Post-Release Bug Fix <DATE>` rather than overwrite, so the SOLUTIONS.md for a long-lived feature accretes across every cycle. The skill **never edits code, never mutates upstream artifacts, never touches `CLAUDE.md` / `AGENTS.md`**, and **never chains to another skill** — if the synthesis surfaces a learning worth promoting into durable project-wide instructions, the skill names the opportunity in a closing sentence and hands back; the user performs that edit on a separate turn. Manual invocation only — `solutions` runs when the human decides a pipeline is worth memorializing, not at the silent end of every other skill.

## Express lane — skipping ceremony for quick wins and urgent hotfixes

Not every change needs the full pipeline. Two cases the skills deliberately support without adding a new skill:

- **Quick-win feature** — a one-line change, rename, copy tweak, or config adjustment. You already know what the code should look like.
- **Urgent production hotfix** — the root cause is obvious at read time (typo, null deref on a named line, clearly-broken config key) and production is bleeding. You already know where and what to change.

For both, skip every upstream skill and go straight to `/execute`. The floor that remains is the minimum viable safety net — not every gate, but enough to keep "quick win" from becoming "quick regression."

### Quick-win feature

```
/execute "<the task in one sentence>"
```

RAW mode + Lightweight tier will:

- Ask 0-1 clarifying questions (usually zero when the task is a one-liner)
- Derive an internal micro-spec from the prompt + codebase pre-scan
- Plan 1-2 vertical slices
- Run RED → GREEN → REFACTOR → commit per slice (or Verification Mode for non-behavior slices — config, build files, type-only declarations, docs, pure formatting)
- Run the integration gate (full suite + lint + types) before writing `EXECUTION.md`

No PRD, no SPEC, no REVISION. `/execute` does not push. When you are ready to share the work, follow up with `/commit-push` — it takes the branch through the protected-branch refusal, the secrets scan, the Lightweight pre-commit gate, the A/B branch question, a convention-matched message draft, and `push -u` after fetch+rebase. **The `/execute` floor is the designed minimum for the code change; the `/commit-push` gate is the designed minimum for the push. Going below either is not supported.**

### Urgent production hotfix

Skip `/diagnosis` only when the root cause is obvious at read time. Then:

```
/execute "<bug description + suspected file:line>"
```

The hotfix path inherits every `/execute` invariant: protected-branch refusal (`main` / `master` / `production` / `prod` / `stable` / `live` / `trunk` / `release*`), RED proof (or Verification Mode), the full integration gate, commit per vertical slice, no push. When the fix is green and committed, run `/commit-push` — the same protected-branch list is enforced again at the push gate, the A/B branch question is still asked (even on a hotfix, a silent push to the wrong branch is how incidents cascade), and the convention-matched message captures why the fix exists so the next on-call read of `git log` tells the full story.

**You own the risk.** By skipping `/diagnosis`, you are stating that you already hold the full causal chain in your head and do not need the investigation trail, the 3+ structurally different hypotheses, the verdicts, or the predictions for uncertain links. If the fix ships and the prediction chain in your head was wrong — it "works" but the real cause is still active — you just shipped a symptom fix. That is the exact failure mode `/diagnosis` exists to prevent.

When in doubt, run `/diagnosis`. Its Lightweight scope + TRIVIAL severity path already gives you a compact artifact with ≤1 hypothesis and no mandatory diagram, so "the cause is obvious" still has a home inside the skill. The express lane is only for when you are certain enough to skip the artifact entirely.

### When not to use the express lane

- The change touches trust boundaries, authN/authZ, injection-prone primitives, schema migrations, or data retention → run `/spec` first. The red-team passes exist for exactly this surface, and skipping them is how security regressions ship.
- The bug reproduces in more than one code path, the stack trace points to a different module than you suspect, or a pattern-analysis diff against a working example would change your hypothesis → run `/diagnosis`. You are not certain enough to skip it.
- The fix crosses more than ~3 files or touches a module you haven't read this week → run `/spec` or the Lightweight tier of `/prd`. RAW mode's internal micro-spec assumes you already hold the context.
- You find yourself typing a multi-paragraph prompt into `/execute` to explain the task → that is a SPEC in disguise. Write one.

After a hotfix lands, run `/code-review` on the diff as a cheap sanity check. Stage 1 plan-alignment is skipped (no plan), but Stages 2-3 still catch the regressions you were too rushed to spot — a two-minute `/code-review` on a hotfix diff is the cheapest insurance in the pipeline. Once it clears, `/commit-push` takes the diff to the remote on a non-protected branch, with a message that makes the post-incident write-up cheaper to draft.

## License

MIT — see [LICENSE](LICENSE).
