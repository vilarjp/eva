---
name: solutions
description: Capture durable learnings from a completed pipeline into a single synthesis document future sessions can read before touching the same code. Distills PRD / SPEC / REVISION / DIAGNOSIS / EXECUTION / CODE-REVIEW in the active spec folder into Summary, Approach, Key Decisions (mini-ADRs), Gotchas (behaviour invariants), What Didn't Work, and References. Use at the end of any pipeline, when the user says "capture learnings", "wrap up the pipeline", "post-mortem", "lessons learned", "what did we learn", or invokes /solutions. Produces docs/YYYY-MM-DD-slug/SOLUTIONS.md. Refuses to run when no eva artifact exists in the target folder. Never edits CLAUDE.md, AGENTS.md, or any project instruction file.
argument-hint: "[path to docs/<DATE>-<slug>/, slug fragment, or blank to auto-detect newest folder]"
---

# Solutions — Durable learnings, not a trip report

Turn a just-completed pipeline into a single reference future sessions can read cold. The durable output is `docs/YYYY-MM-DD-<slug>/SOLUTIONS.md` — a synthesis of every eva artifact in the folder, phrased so it survives a refactor.

This skill does NOT write code, run tests, edit PRD/SPEC/DIAGNOSIS/EXECUTION/CODE-REVIEW, touch CLAUDE.md/AGENTS.md, push, or open PRs. It reads, synthesises, and writes one file.

## When to use

Trigger this skill when the user:
- says "solutions", "learnings", "capture learnings", "write it up", "wrap up the pipeline", "close out this work"
- says "post-mortem", "retrospective", "lessons learned", "document what we learned", "record the gotchas"
- says "compound what we did", "capture this for next time", "what did we learn from this"
- invokes `/solutions` (optionally with a path or slug)
- has just finished an `/execute` + `/code-review` pass and wants the folder buttoned up before moving on

Do NOT trigger for: planning (`/prd`, `/spec`), cross-doc review (`/revision`), bug investigation (`/diagnosis`), implementation (`/execute`), or pre-merge audit (`/code-review`). Solutions assumes the doing is done and the folder is ready to be distilled.

Do NOT auto-chain from `/execute` or `/code-review`. Solutions is always **manually invoked** — the user decides when a pipeline is ready to be captured.

## The Iron Law

**Describe every learning by behaviour, not by file path or line number.**

A gotcha that says *"src/cache.ts:42 has a race"* is dead the moment the file is renamed or the line moves. A gotcha that says *"the cache must be invalidated before the write, not after — otherwise concurrent readers see stale values"* survives every refactor, because it names the invariant, not the address.

Every item in SOLUTIONS.md — Root Cause, Key Decisions, Gotchas, What Didn't Work, Prevention — must pass this test: **would this still be useful if the file were renamed tomorrow?** If not, rewrite it as an invariant before writing.

References to specific files are allowed only in the `## References` section (pointing at upstream artifacts) and as illustrative *examples* inside a prose paragraph — never as the substance of a learning.

## Core invariants

1. **Iron Law of durability.** Every learning is phrased as behaviour / invariant / mechanism. File:line as substance is a protocol violation.
2. **Inline synthesis only.** No subagents. No parallel dispatch. Solutions reads the folder and writes one file. The heavy lifting already happened upstream.
3. **Read-only outside the write target.** Solutions reads every artifact in the folder but never edits PRD.md, SPEC.md, REVISION.md, DIAGNOSIS.md, EXECUTION.md, or CODE-REVIEW.md. It writes exactly one file: `SOLUTIONS.md`.
4. **No instruction-file edits.** Solutions NEVER writes or edits `CLAUDE.md`, `.claude/CLAUDE.md`, `AGENTS.md`, `.cursorrules`, or any top-level rules file. It may *mention* the opportunity once in the transition text; the user does the edit themselves if they want it.
5. **Artifacts required.** At least one of `PRD.md`, `SPEC.md`, `REVISION.md`, `DIAGNOSIS.md`, `EXECUTION.md`, or `CODE-REVIEW.md` must exist in the target folder with `status: approved`. If none do, refuse — there is nothing to synthesise.
6. **No tacit-knowledge interview.** Solutions does not ask "what's a gotcha you're carrying in your head?" All content comes from the artifacts on disk. The user opted for synthesis, not interrogation.
7. **At most 2 disambiguation questions.** Only when auto-detection is genuinely ambiguous (which folder when many match, resume policy when `SOLUTIONS.md` already exists). Never to extract content. Usually 0-1.
8. **Verify before claiming.** Every Key Decision, Gotcha, and What-Didn't-Work entry must be traceable to a quote or table row in the source artifacts. Unverified paraphrase is a protocol violation.
9. **Synthetic learnings are forbidden.** A clean pass is valid — if there were no gotchas, say *"No gotchas surfaced during this pipeline. This section is intentionally empty."* Padding with platitudes is worse than emptiness.
10. **Emit the self-review checklist** to the user before writing. Skipping it is a protocol violation.
11. **HARD GATE: do not write `SOLUTIONS.md` until the user approves the synthesis AND the output path.** No creation, no directory touch, no file write before that approval.
12. **Re-run append, never overwrite.** If `SOLUTIONS.md` already exists, the default is to append a dated section under the original content — never to rewrite the prior synthesis silently.

## Pre-flight (MANDATORY)

Before anything else, in order:

1. **Today's date.** Run `date +%Y-%m-%d` via Bash. Store as `<DATE>`.
2. **Clarify on bare invocation.** If `/solutions` was called with no path and Phase 0 auto-detect finds zero eligible folders, ask: *"Which pipeline should I capture? Pass a folder path (e.g. `docs/2026-04-16-faster-checkout/`), a slug fragment, or tell me what the work was about."* Do not proceed without a target.
3. Announce: `solutions: starting (scope TBD)`.

## Process

Execute phases in order. Do not skip ahead.

### Phase 0 — Folder detection, artifact audit, scope classification

**0.1 Resolve the target folder.** Priority order (first match wins):

1. **Explicit path** — the user passed `docs/<DATE>-<slug>/` (with or without trailing slash) or `docs/<DATE>-<slug>/SOLUTIONS.md`. Verify the directory exists. Use it.
2. **Slug hint** — the user passed a word or phrase (e.g. `checkout-coupon`). Glob `docs/*/` and match on the slug segment case-insensitively. If multiple match, ask ONE disambiguation question listing the candidates newest-first.
3. **Newest-folder auto-detect** — no path, no slug. Glob `docs/*/` and pick the most recent directory whose frontmatter `topic` or filename slug is non-empty. If multiple equally-recent folders exist, ask ONE disambiguation question.

If no `docs/*/` directory exists, STOP and tell the user: *"No `docs/*/` folders found. Run `/prd`, `/spec`, `/diagnosis`, `/execute`, or `/code-review` first — solutions synthesises an existing pipeline; it does not generate one."*

**0.2 Artifact audit.** In the target folder, check for each of:
- `PRD.md` (`status: approved` required)
- `SPEC.md` (`status: approved` required)
- `REVISION.md` (`status: approved` and `patches_applied: true` preferred)
- `DIAGNOSIS.md` (`status: approved` required)
- `EXECUTION.md` (`status: approved` required)
- `CODE-REVIEW.md` (`status: approved` required)

Record the list of present artifacts. If zero of the six are present with `status: approved`, refuse: *"`<folder>` has no approved eva artifact. Solutions synthesises what the other skills produced — there's nothing to distill here. Run `/prd`, `/spec`, `/diagnosis`, `/execute`, or `/code-review` first."* STOP.

Announce: `solutions: folder = <path>, artifacts = [<list>]`.

**0.3 Pipeline classification.** The artifact set determines whether this is a feature pipeline, a bug pipeline, or both:

- `DIAGNOSIS.md` present → **bug pipeline**. `Root Cause` and `Prevention` sections become required.
- `SPEC.md` (or `REVISION.md` with `patches_applied: true`) present and no `DIAGNOSIS.md` → **feature pipeline**. `Root Cause` and `Prevention` are omitted.
- Both `DIAGNOSIS.md` and `SPEC.md` present (feature folder where a bug was also diagnosed and fixed) → **mixed pipeline**. All sections apply; `Relationship to Original Spec` section is included.

Store as `pipeline: [feature]`, `[bug]`, or `[feature, bug]`.

**0.4 Resume check.** If `SOLUTIONS.md` already exists in the folder:

- If `DIAGNOSIS.md` exists in the folder AND its `date` in frontmatter is newer than SOLUTIONS.md's `date` → default to **appending `## Post-Release Bug Fix <DATE>`** with the bug-pipeline sections nested inside.
- Otherwise → default to **appending `## Re-run <DATE>`** with the full current section set.

Emit ONE confirmation question:
```
Found existing SOLUTIONS.md at <path>.

  A) Append <## Post-Release Bug Fix <DATE> | ## Re-run <DATE>> — prior content preserved [Recommended]
  B) Write to SOLUTIONS-2.md — keep this run separate
  C) Overwrite — discard the prior synthesis
```

Default = A. Selecting C requires the user to type "overwrite" verbatim (eva's convention for destructive choices).

**0.5 Scope classification.** Inherit the MAX complexity across the present artifacts' frontmatter `scope` fields. Promotion rules:

| Highest scope found | Solutions tier | Ceremony |
|---|---|---|
| LIGHTWEIGHT only | **Lightweight** | Compact SOLUTIONS. Summary + Approach + Key Decisions + Gotchas + References. 0-1 disambiguation Qs. No diagram. |
| STANDARD or mixed | **Standard** | All sections that apply. 0-2 disambiguation Qs. Diagram optional (offer once if structure warrants). |
| DEEP anywhere | **Deep** | All sections that apply + **mandatory Mermaid diagram** of the durable mechanism. 0-2 Qs. |

If no artifact carries a `scope` field in frontmatter (rare — most eva artifacts do), infer from the largest artifact: a SPEC with ≥3 tracer-bullet phases → Standard; anything with concurrency/migration/integration language → Deep; else Lightweight.

Announce the classification: `solutions: scope = <TIER>, pipeline = <feature | bug | mixed>`. Detailed rubric in `references/scope-tiers.md`.

### Phase 1 — Read every artifact in full

Read every artifact identified in Phase 0.2, in this order (a reader's chronology):

1. `PRD.md` — problem framing, goals, non-goals, recommended direction.
2. `SPEC.md` — architecture, mini-ADRs, tracer-bullet phases, risks.
3. `REVISION.md` — findings and patches applied (only if `patches_applied: true`).
4. `DIAGNOSIS.md` — investigation trail, hypotheses, causal chain, root cause, suggested fix, concerns.
5. `EXECUTION.md` — slice table, per-slice RED/GREEN proofs, integration gate log, acceptance-criteria trace, NOTICED-BUT-NOT-TOUCHING list, concerns, commits.
6. `CODE-REVIEW.md` — findings by severity, positives, residual risks, testing gaps, suppressed, pre-existing, plan-alignment summary.

Take structured notes as you read:
- Key decisions surfaced mid-pipeline that were NOT already mini-ADRs in SPEC
- Gotchas phrasable as invariants (pull from Concerns, Risks, NOTICED-BUT-NOT-TOUCHING)
- Dead ends (3-strike Step-Back events in EXECUTION, REJECTED hypotheses in DIAGNOSIS, suppressed findings in CODE-REVIEW)
- Prevention levers (DIAGNOSIS hotspots, CODE-REVIEW testing gaps, defense-in-depth recommendations)

Do NOT start drafting sections until every artifact is read — synthesis quality falls apart if one input is missed. Loading guide: `references/section-guides.md`.

### Phase 2 — Synthesise sections

Draft the full content, section by section. Every item must pass the Iron Law (behaviour, not file:line) and the Verify-Before-Claiming rule (traceable to a specific artifact).

**2.1 Summary** — 2-3 sentences: what shipped (feature pipeline) / what was fixed (bug pipeline) / both. Pull from PRD problem statement, SPEC context, DIAGNOSIS bug description, EXECUTION summary.

**2.2 Root Cause** *(bug and mixed pipelines only)* — 1-3 sentences describing the **mechanism** of failure as an invariant. Pull from DIAGNOSIS's root cause and causal chain, rephrased. Example: *"The validation middleware skips empty strings, allowing null values to reach the database layer after the ORM coerces them."* NOT *"See src/middleware/validate.ts:42."*

**2.3 Approach** — 2-4 sentences on what was done and why this path was chosen over alternatives. Pull from SPEC mini-ADRs and EXECUTION's per-slice intent. For bug pipelines, include the shape of the fix (symptom vs root cause addressed; defense-in-depth if applied).

**2.4 Key Decisions** — compact mini-ADR format. Pull from:
- SPEC mini-ADRs that proved their weight during implementation (not every ADR — only the ones that mattered)
- EXECUTION "unexpected decisions" / Confusion resolutions
- REVISION patches that shifted the direction
- CODE-REVIEW plan-alignment drift that was accepted rather than fixed

Each decision uses this shape:

```
### Decision: <one-line name>

- **What:** <the decision, as a behaviour or invariant>
- **Why:** <rationale — what it optimises for>
- **Trade-off accepted:** <what it costs — the honest downside>
- **Alternatives considered:** <1-2 genuine alternatives, one-line each>
```

Omit Alternatives for Lightweight scope. Synthetic alternatives are forbidden — a decision without a real alternative is a rationalisation, not a decision.

**2.5 Gotchas** — bulleted list, each phrased as a behaviour invariant. Pull from EXECUTION's NOTICED-BUT-NOT-TOUCHING, Concerns, and DIAGNOSIS concerns; CODE-REVIEW residual risks and testing gaps. Examples:

- *"The `cart_items` cache must be invalidated before the write commits; invalidating after leaves a window where concurrent readers see stale values."*
- *"`applyCoupon` assumes `cart.items` is hydrated; callers that run during the loading state must gate on `cart.status === 'ready'`."*

**2.6 What Didn't Work** — bulleted list. Pull from:
- EXECUTION 3-strike Step-Back events (each attempt and why it failed)
- DIAGNOSIS REJECTED hypotheses (what the evidence killed, so future sessions don't retread)
- CODE-REVIEW suppressed findings (things flagged but not confirmed — worth knowing they were considered)

Each entry: *Attempted approach + why it failed.* Not a criticism — a map of the territory. Survivors-only maps waste the next session's time.

**2.7 Prevention** *(bug and mixed pipelines only)* — how to avoid this class of bug next time. Pull from DIAGNOSIS suggested fix + hotspots, CODE-REVIEW testing gaps, EXECUTION regression proof:

- **Class of bug** — name the category (e.g. "null propagation past a validation boundary").
- **Where to add defense-in-depth** — entry point, business layer, persistence, or output. If DIAGNOSIS recommended layered validation, carry it here.
- **Regression test** — the reproduction test from DIAGNOSIS is the primary guard. Note any additional coverage CODE-REVIEW's testing gaps flagged.

**2.8 Diagram** *(Deep mandatory, Standard optional, Lightweight never)* — a Mermaid block depicting the durable mechanism. Prefer sequence diagrams for cross-component flows, state diagrams for state machines, flowcharts for decision trees. The diagram captures the *why this happens*, not the *where it lives*.

**2.9 References** — bulleted list of every artifact read, with date and status. Links are relative to `SOLUTIONS.md`:

- `[PRD.md](./PRD.md)` — 2026-04-14, approved
- `[SPEC.md](./SPEC.md)` — 2026-04-15, approved
- `[EXECUTION.md](./EXECUTION.md)` — 2026-04-16, approved
- `[CODE-REVIEW.md](./CODE-REVIEW.md)` — 2026-04-17, approved

**2.10 Relationship to Original Spec** *(mixed pipeline only — DIAGNOSIS appended to a feature folder)* — one paragraph describing how the bug related to the original feature (edge case the SPEC missed, regression introduced by a later refactor, invariant that needed defense-in-depth).

Section-by-section drafting guidance: `references/section-guides.md`.

### Phase 3 — Visual companion offer (conditional)

For **Deep** scope: a diagram is mandatory — do not ask, include it.

For **Standard** scope: if the synthesis has sequence/state/decision structure worth visualising, ASK ONCE:
```
This synthesis has <sequence | state | flow> structure that a diagram would clarify.
Include an inline Mermaid diagram in SOLUTIONS.md?
  A) Yes — <sequence | flowchart | state> diagram [Recommended if structure is non-linear]
  B) No — text is sufficient
```

For **Lightweight** scope: never include a diagram.

### Phase 4 — Self-review checklist (MANDATORY, emit to user)

Before proposing the gate, emit this checklist with `✓` or `✗` on each line. Items marked N/A need a one-word reason.

- [ ] Target folder resolved and at least one approved eva artifact present
- [ ] Every artifact in the folder was read, not skimmed
- [ ] Summary names what shipped (feature) and/or what was fixed (bug) in 2-3 sentences
- [ ] Root Cause (bug pipelines) describes the mechanism as an invariant, not a file:line
- [ ] Approach states what was done and why this path over alternatives
- [ ] Every Key Decision has a rationale and an honestly-named trade-off (not "we decided to …")
- [ ] Every Key Decision lists ≥1 genuine alternative considered (Standard / Deep)
- [ ] Gotchas are phrased as behaviour invariants — each would survive a file rename
- [ ] What Didn't Work captures dead ends (Step-Backs, REJECTED hypotheses, suppressed findings) — not a criticism, a map
- [ ] Prevention (bug pipelines) names the class of bug and where defense-in-depth belongs
- [ ] References section lists every artifact read with date and status
- [ ] No learning is anchored to a file:line as its substance (file:line only as illustration inside prose)
- [ ] Scope tier (LIGHTWEIGHT / STANDARD / DEEP) matches ceremony and diagram rule
- [ ] For Deep scope, a Mermaid diagram is drafted and depicts the durable mechanism
- [ ] No synthetic padding — empty sections say "intentionally empty" with a reason
- [ ] No placeholder text, no `[TBD]`, no `TODO`
- [ ] If appending to an existing SOLUTIONS.md, the prior content is preserved and the new section is dated

If any check fails → STOP, fix, re-emit. Do not proceed with any `✗` unresolved.

### Phase 5 — HARD GATE: approve synthesis + path

Propose the output target: either the existing `<target-folder>/SOLUTIONS.md` (new write or dated append), or `<target-folder>/SOLUTIONS-2.md` if the user selected option B at Phase 0.4.

Present the gate via `AskUserQuestion`:

```
Ready to write SOLUTIONS.

Summary:
- Pipeline: <feature | bug | mixed>
- Scope: <LIGHTWEIGHT | STANDARD | DEEP>
- Folder: docs/<DATE>-<slug>/
- Artifacts synthesised: <list>
- Mode: <fresh write | append ## Post-Release Bug Fix <DATE> | append ## Re-run <DATE> | SOLUTIONS-2.md | overwrite>
- Sections: <list of sections populated>
- Diagram: <included | not included>
- Proposed path: <path>

Approve and write, or describe changes?
  A) Approve — write SOLUTIONS at the path above [Recommended]
  B) Change a section (Summary / Approach / Key Decisions / Gotchas / What Didn't Work / Prevention / Diagram / References)
  C) Change the path or the append mode
```

**STOP HERE.** Do not create the file. Do not touch the directory. Do not modify any other artifact. WAIT for user approval.

If the user picks B or C, loop back to the relevant phase, then re-emit the Phase 4 checklist before re-gating.

### Phase 6 — Write SOLUTIONS.md

Only after explicit approval:

1. **Fresh write.** Use `templates/SOLUTIONS.md`. Replace every `{{placeholder}}` with actual content. Set `status: approved` in frontmatter. Omit sections that don't apply (Root Cause / Prevention / Relationship to Original Spec / Diagram) instead of leaving them with placeholder text.
2. **Append mode.** Read the existing `SOLUTIONS.md`. Preserve everything above the `<!-- Re-run append target -->` comment (or after the original body if no comment exists). Append the dated section under a level-2 heading: `## Post-Release Bug Fix <DATE>` or `## Re-run <DATE>`. Update frontmatter `date` to today and append to the `pipeline` array (e.g. `[feature]` → `[feature, bug]`). Preserve the original `topic` and `scope` — if scope escalated (e.g. Standard → Deep), update `scope` to the new MAX.
3. **SOLUTIONS-2.md.** Treat as a fresh write at the `-2` path. Add frontmatter `supersedes: SOLUTIONS.md` for audit trail.
4. Confirm to the user with a single line:
   ```
   SOLUTIONS written: <path>
   (Optional: mention the discoverability opportunity — see Transition.)
   ```

SOLUTIONS.md MUST be complete — no placeholders, no TODOs, no synthetic padding. It MUST be standalone: a reader opening it cold should understand what happened and what to watch without ever reading PRD/SPEC/DIAGNOSIS/EXECUTION/CODE-REVIEW.

### Transition

After writing, INFORM the user in one line: *"Solutions written and approved. Future sessions working on this area can now open `<path>` before touching the code."*

If the user has no `docs/*/SOLUTIONS.md` reference in their project's `CLAUDE.md`, `AGENTS.md`, or equivalent rules file, mention ONCE (no edit, no gate — a plain suggestion):

*"Optional: adding `# Past solutions and decisions are in docs/*/SOLUTIONS.md` to your CLAUDE.md (or AGENTS.md) makes these discoverable to future agent sessions. I will not edit that file — run the edit yourself if you want it."*

Do NOT invoke any other skill. Do NOT start coding. Do NOT push. The skill is complete; the pipeline is captured.

## Anti-patterns

See `references/anti-patterns.md` for the full list. Highlights:

- **"Nothing surprising happened, so there's nothing to capture."** Every pipeline produces decisions and trade-offs worth recording. Write it. Empty sections are allowed when honest; "nothing happened" across the whole doc is never honest.
- **"The commit messages already say this."** Commit messages say *what* changed. SOLUTIONS says *why*, *what to watch*, and *what we tried that didn't work*.
- **"I'll write it later when I have more perspective."** You have maximum context right now. Later means never. The artifacts are still fresh; the synthesis is cheap now and expensive in two weeks.
- **"The Gotchas section is empty because there were none."** A multi-slice Deep execution with zero gotchas is not looking hard enough. Re-read EXECUTION's Concerns and NOTICED-BUT-NOT-TOUCHING. Re-read CODE-REVIEW's residual risks and testing gaps. Something is there.
- **"Let me point at the specific files so the reader knows where to look."** References to `src/foo.ts:42` die the moment `foo.ts` is renamed. Phrase the learning as an invariant; use the References section for pointers to artifacts, not to source.
- **"This key decision is so obvious it doesn't need a rationale."** Every key decision has a rationale. If the rationale is obvious, writing it is trivial — write it. If it isn't obvious, the decision was real and the rationale is the whole point.
- **"I'll just edit the CLAUDE.md to mention the docs folder while I'm here."** No. Instruction-file edits are a non-goal of this skill. Mention the opportunity in the transition line; the user decides.
- **"Synthesising from EXECUTION is enough; I can skip PRD/SPEC."** Wrong. Key Decisions pulls from SPEC mini-ADRs; Prevention pulls from DIAGNOSIS hotspots; What Didn't Work pulls from CODE-REVIEW suppressed findings. Skipping any input produces a thin synthesis.

## Red flags — self-check

STOP if you catch yourself doing any of these:

- You wrote a Gotcha that names a specific file:line as its substance (not just illustration)
- You listed a Key Decision with no rationale or with "we decided to …" as the rationale
- You invented a third alternative for a Key Decision that wasn't actually considered
- You left Gotchas empty after a Deep pipeline with a multi-slice EXECUTION (look harder)
- You skipped PRD or SPEC because "EXECUTION has everything" (it doesn't)
- You proposed an edit to CLAUDE.md / AGENTS.md (never — mention only, never write)
- You started drafting before every artifact in the folder was read
- You are about to overwrite a prior SOLUTIONS.md without an explicit "overwrite" directive from the user
- You are about to write SOLUTIONS.md before the user approved Phase 5
- You asked the user "what gotchas are you carrying in your head?" (solutions does synthesis, not interrogation)
- You synthesised a What-Didn't-Work entry that was never attempted, just to avoid an empty section
- You started chaining — called `/execute` or another skill after writing (the skill is complete; the pipeline is captured)

## References

- `templates/SOLUTIONS.md` — the output template. Read only at Phase 6.
- `references/scope-tiers.md` — detailed Lightweight / Standard / Deep rubric for solutions. Load at Phase 0.5 if the automatic classification is ambiguous.
- `references/section-guides.md` — per-section "what to pull from which artifact" map, with worked examples of behaviour-phrased learnings vs file:line anti-examples. Load at Phase 2 if any section feels thin.
- `references/anti-patterns.md` — the full rationalisation catalogue and the honest-empty-section rule. Load when tempted to pad, skip, or shortcut.
