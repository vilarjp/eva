---
name: prd
description: Turn a rough feature idea or problem into a rigorous, codebase-grounded PRD document. Use when the user wants to brainstorm a new feature, write a PRD, draft requirements, or invokes /prd. Produces docs/YYYY-MM-DD-slug/PRD.md with problem framing, goals, non-goals, 2-3 options with trade-offs, recommended direction, user stories, and open questions. Distinct from /spec (technical design, not product framing).
argument-hint: "[feature idea or problem to explore]"
---

# PRD — Write a rigorous Product Requirements Document

Turn a rough idea into a codebase-grounded PRD through structured dialogue, option exploration, and an explicit approval gate. The durable output is a single file: `docs/YYYY-MM-DD-<slug>/PRD.md`.

This skill does NOT write code. It explores, challenges, and documents — then hands off.

## When to use

Trigger this skill when the user:
- says "brainstorm", "write a PRD", "draft requirements", "explore this idea", "help me think through X", "what should we build"
- invokes `/prd` (optionally with a feature description)
- describes a vague or ambitious feature request, presents a problem with multiple valid solutions, or seems unsure about scope or direction

## Core invariants

1. **No code until the PRD is approved.** No implementation, no scaffolding, no edits to source files.
2. **Ask ONE question at a time. Max 5 total.** Prefer multiple-choice via `AskUserQuestion` when natural options exist.
3. **Verify before claiming.** Any factual claim about the codebase must be confirmed by reading files. Unverified claims must be labeled as assumptions.
4. **Synthetic options are forbidden.** If only one credible approach exists, say so — do not manufacture fake alternatives.
5. **Emit the self-review checklist to the user** before writing. Skipping it is a protocol violation.
6. **HARD GATE: do not write PRD.md until the user approves the recommended direction AND the proposed slug.**

## Pre-flight (MANDATORY)

Before anything else:

1. Get today's date: `date +%Y-%m-%d` via Bash. Store as `<DATE>`.
2. If the user's invocation had no feature description (e.g., bare `/prd`), ask: *"What would you like to explore? Describe the feature, problem, or improvement you're thinking about."* Do not proceed until you have a description.
3. Announce the skill is active with a single line: `prd: starting (scope TBD)`.

## Process

Execute phases in order. Do not skip ahead.

### Phase 0 — Resume & Scope

**0.1 Resume check.** Look in `docs/` for any directory matching `<DATE>-*` or a prior date whose topic obviously matches. If a matching `PRD.md` exists, ask the user:
```
Found an existing PRD at <path>. Resume from it or start fresh?
  A) Resume and update [Recommended if topic matches]
  B) Start fresh — create a new dated directory
```
If resuming, read the file, summarize current state, and continue from open questions. Update the existing file rather than creating a duplicate.

**0.2 Scope classification.** Use the feature description plus a quick glob/read to classify:

| Tier | Criteria | Ceremony |
|------|----------|----------|
| **Lightweight** | Small, well-bounded, low ambiguity. 1-3 files likely affected. | Compact PRD. Skip Phase 1 if requirements already clear. 0-2 clarifying Qs. Short sections. |
| **Standard** | Normal feature or bounded refactor with real decisions. 3-10 files. | Full phases, 2-5 clarifying Qs, 2-3 options. |
| **Deep** | Cross-cutting, strategic, ambiguous, or irreversible trade-offs. 10+ files or new architectural pattern. | Full phases + mandatory Mermaid diagram + council-style option debate in Phase 4. |

Announce the classification: `prd: scope = <TIER>`. If unclear, ask ONE disambiguating question.

For detailed rubric, see `references/scope-tiers.md`.

### Phase 1 — Understand the request (≤5 Qs)

Cover only what is NOT already clear from the user's description:

1. **WHY** — what problem does this solve?
2. **WHO** — who is affected?
3. **CURRENT STATE** — how does it work today?
4. **SUCCESS** — how will you know it's working?
5. **CONSTRAINTS** — timeline, tech stack, compatibility, deployment?

Rules:
- ONE question per turn. Never batch.
- Prefer `AskUserQuestion` with multiple-choice options when natural. Mark the best pick with `[Recommended]` only when you genuinely have one.
- Maximum 5 questions total. State remaining unknowns as assumptions to validate.
- STOP early when WHY/WHO/SUCCESS/CONSTRAINTS are clear from context.

For Lightweight scope, 0-2 questions are usually enough. For Deep, lean into the full 5.

### Phase 2 — Codebase pre-scan

Scan the repo to ground the PRD. Match depth to scope:

**Lightweight:** single Grep for the topic; check if something similar already exists.

**Standard:**
1. *Constraint check.* Read `AGENTS.md`, `CLAUDE.md`, or `README.md` for workflow / product / scope constraints that affect this PRD.
2. *Topic scan.* Grep for relevant terms. Read the most relevant existing artifact (prior PRD, spec, feature doc) if one exists. Skim one or two adjacent examples covering similar behavior.

**Deep:** Standard + map the module boundaries likely to be touched and note the interfaces. Look for prior-art PRDs in `docs/` to follow the same voice/format.

After scanning, produce a short **Grounding summary** (3-8 bullets max). When a claim is not verified from code, label it `(unverified)`.

### Phase 3 — Frame the problem

Produce these blocks (the PRD will carry them forward):

**Problem Statement** — one paragraph. Specific, not vague. *"Checkout takes 3 clicks when it should take 1"* not *"users are frustrated."*

**Goals** — numbered list. Each MUST be measurable or verifiable (you must be able to write an acceptance criterion).

**Non-Goals** — numbered list. What this effort explicitly will NOT address. Prevents scope creep. Non-Goals MUST be genuine exclusions, not goals in disguise.

**Constraints** — timeline, tech stack, backwards-compat, deployment, regulatory.

**Assumptions** — numbered list with verification status:
```
1. <assumption> — verified by reading code: YES / NO
2. <assumption> — verified: YES / NO
```

### Phase 4 — Generate options

Produce 2-3 genuinely different approaches when multiple credible paths exist. *Different* means meaningfully different trade-offs, not variations of the same idea.

If only one credible path exists, DO NOT manufacture fake alternatives. Document the single credible approach, name the discarded pseudo-alternatives briefly, and explain why they are not credible.

For each option:
- **Name, Description** (2-3 sentences)
- **Pros** (specific, not generic)
- **Cons** (every option MUST have cons)
- **Complexity:** XS / S / M / L / XL
- **Risk:** Low / Med / High (one-line explanation)

**Quality checks:**
- If all options have the same pros/cons pattern → they are not different enough.
- If one option is obviously superior → find its downside.
- Options MUST consider existing codebase patterns from Phase 2.

For **Deep** scope, also run a brief council-style debate: imagine a staff engineer, a product manager, and a skeptic each critiquing each option in one sentence. Incorporate dissent into the recommendation.

### Phase 5 — Recommend a direction

State:
1. Which option and why.
2. The key trade-off being consciously accepted.
3. Why the alternatives were not chosen (specific reasons, not "feels right").

The recommendation MUST follow logically from goals and constraints.

### Phase 6 — Complete the supporting sections

**Complexity tier** — restate the Phase 0.2 classification (LOW = Lightweight, MEDIUM = Standard, HIGH = Deep) with one-sentence justification.

**User Stories** — numbered list in the format:
> As a `<actor>`, I want `<feature>`, so that `<benefit>`.

Aim for 3-10 stories covering the main user flows. Each story should correspond to a verifiable acceptance criterion.

**Not-Doing list** — explicit trade-offs. One line per item: `<what> — <reason>`. This is the most valuable section; it makes scope decisions durable.

**Open Questions** — things that need answering before or during build. Mark each as `now` (blocks PRD) or `during-build` (acceptable to defer).

### Phase 7 — Visual companion offer (conditional)

If the topic has spatial or systemic structure (UI layouts, data flows, component hierarchies, state machines), ASK once:
```
This topic has <spatial/systemic> structure that might benefit from a diagram.
Include an inline Mermaid diagram in the PRD?
  A) Yes — <type> diagram [Recommended if structure is non-trivial]
  B) No — text is sufficient
```

For **Deep** scope, a diagram is mandatory (do not ask, just include one).

If yes, draft a Mermaid block that will live inside the PRD under a `## Diagram` section.

### Phase 8 — Self-review checklist (MANDATORY, emit to user)

Before writing the file, emit this checklist to the user with `✓` or `✗` on each line. A missing checklist is a protocol violation.

- [ ] Problem statement is specific, not vague or aspirational
- [ ] Goals are measurable or verifiable — each has a way to confirm it's met
- [ ] Non-goals are genuine exclusions — not goals in disguise
- [ ] Constraints are stated (timeline, tech stack, compatibility)
- [ ] Codebase claims are verified by reading files; unverified items labeled (unverified)
- [ ] Options are genuinely distinct — not three variations of the same approach
- [ ] Every option has cons
- [ ] Recommended direction references specific pros/cons
- [ ] User stories are in As-a / I-want / So-that form and each maps to a goal
- [ ] Not-Doing list is non-empty and names real trade-offs
- [ ] Open questions are tagged `now` vs `during-build`
- [ ] No placeholder text or `[TBD]` markers
- [ ] Complexity tier matches scope classification
- [ ] **Compactness check** — every empty / trivial / off-scope section is cut (not kept with a placeholder). Remaining sections pass the caps in `_shared/artifact-compactness.md`. Cutting more would remove information, not just words.

If any check fails → STOP, fix, re-emit. Do not proceed with any `✗` unresolved.

**Deep scope only — 100-point scoring rubric (MANDATORY for Deep, skipped for Lightweight / Standard).**

After the checklist clears, score the draft against the four-dimension rubric in `references/scoring-rubric.md`:

- **Functional completeness** — 30 pts (user stories × flows, error paths named, measurable success metrics)
- **Technical soundness** — 25 pts (codebase claims verified, options genuinely different, recommendation follows from goals)
- **Implementation readiness** — 25 pts (constraints explicit, stories map to acceptance hooks, not-doing list real)
- **Business impact** — 20 pts (specific problem statement, trade-off named, open questions tagged)

Emit the score block to the user before Phase 9:

```
PRD score: <total>/100  (target ≥ 90)
  Functional completeness  : <n>/30   — <1-line note on weakest criterion>
  Technical soundness      : <n>/25   — <...>
  Implementation readiness : <n>/25   — <...>
  Business impact          : <n>/20   — <...>
```

If `total < 90`, loop back to the phase that produced the weakest dimension (Phase 3 for framing, Phase 4 for options, Phase 6 for stories / not-doing), fix, re-emit the Phase 8 checklist AND a fresh score block. Maximum three iterations before surfacing a stuck score to the user. Do NOT proceed to the HARD GATE below 90 unless the user explicitly accepts the gap.

### Phase 9 — HARD GATE: approve direction + slug

Propose the directory slug: kebab-case, 2-4 words, derived from the problem statement. Example: *"faster mobile checkout"* → `faster-mobile-checkout`.

Present the gate via `AskUserQuestion` (or plain text if unavailable):

```
Ready to write the PRD.

Summary:
- Problem: <one sentence>
- Recommended direction: <one sentence>
- Complexity: <LOW | MEDIUM | HIGH>
- Proposed path: docs/<DATE>-<slug>/PRD.md

Approve and write, or describe changes?
  A) Approve — write the PRD at the path above [Recommended]
  B) Change the slug
  C) Change the direction
```

**STOP HERE.** Do not create the directory. Do not write PRD.md. Do not invoke other skills. WAIT for user approval.

If the user requests changes, loop back to the relevant phase, then re-emit the Phase 8 checklist before re-gating.

### Phase 10 — Write the PRD

Only after explicit approval:

1. `mkdir -p docs/<DATE>-<slug>/`
2. Write `docs/<DATE>-<slug>/PRD.md` using `templates/PRD.md`. Replace every `{{placeholder}}` with actual content. Set `status: approved` in frontmatter.
3. If a diagram was agreed (Phase 7), include it under `## Diagram`.
4. Confirm to the user:
   ```
   PRD written: docs/<DATE>-<slug>/PRD.md
   ```

The PRD MUST be complete — no placeholders, no TODOs. It MUST be standalone: readable without this conversation.

### Transition

After writing, INFORM the user in one sentence: *"PRD written and approved. You can now hand this off to an implementation plan, open a GitHub issue with its contents, or iterate on it directly."*

Do NOT invoke any implementation skill. Do NOT start coding. The skill is complete.

## Anti-patterns

- **"This is too simple for a PRD."** If it touches more than one file or involves a decision between approaches, write at least a Lightweight PRD. Skip Phase 1 if requirements are already clear; don't skip the document.
- **"I already know the right approach."** Surface it as an assumption. The PRD validates assumptions.
- **"The user seems impatient."** A 10-minute PRD that prevents 2 hours of rework is a bargain.
- **"Let me just start coding."** Code without a PRD is gambling. Refuse.
- **Yes-machining a weak idea.** Push back with specificity and kindness when the framing is off. Be a sharp thinking partner, not a facilitator reading from a script.

## Red flags — self-check

- You manufactured fake options in Phase 4 to satisfy the template
- Non-Goals is empty
- An option has no cons listed
- All options have the same pros/cons pattern
- You skipped the Phase 8 checklist
- You wrote PRD.md before the user confirmed in Phase 9
- You skipped clarification even though WHY / WHO / SUCCESS / CONSTRAINTS were unclear
- Your problem statement is vague or aspirational rather than specific
- You cited a file, table, endpoint, or dependency without reading it
- Deep scope, and you proceeded to Phase 9 with a rubric score below 90 without explicit user acceptance of the gap
- You rounded rubric scores up to clear the 90 threshold — score as the next reader would, not as the session wants to finish

## References

- `templates/PRD.md` — the output template. Read only at Phase 10.
- `references/scope-tiers.md` — detailed Lightweight / Standard / Deep rubric.
- `references/section-guides.md` — how-to for each PRD section (what makes a good Problem Statement, how to spot fake Non-Goals, etc.). Load when a section is underperforming the self-review checklist.
- `references/scoring-rubric.md` — 100-point quality rubric used in Phase 8 for Deep-scope PRDs only. Load after the checklist clears.
