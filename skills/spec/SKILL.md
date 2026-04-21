---
name: spec
description: Turn a feature idea or approved PRD into a rigorous, codebase-grounded technical specification. Use when the user wants to write a tech spec, design the architecture for a feature, document engineering decisions before coding, plan how to build something approved in a PRD, or invokes /spec. Produces docs/YYYY-MM-DD-slug/SPEC.md with current + proposed architecture, mini-ADRs for each key decision, data model, API contracts, module boundaries, test strategy, risks, and open questions. Auto-detects an adjacent PRD.md and builds on it when present. Distinct from /prd (product framing, not engineering design).
argument-hint: "[feature idea to design or PRD to build on]"
---

# SPEC — Write a rigorous Technical Specification

Turn a feature idea (or an approved PRD) into a codebase-grounded technical specification through structured dialogue, decision capture, and an explicit approval gate. The durable output is a single file: `docs/YYYY-MM-DD-<slug>/SPEC.md`.

This skill does NOT write code. It designs, decides, stress-tests, and documents — then hands off.

## When to use

Trigger this skill when the user:
- says "write a tech spec", "design this", "what's the architecture", "draft the SPEC", "plan how to build this", "move from PRD to spec", "architectural design doc"
- invokes `/spec` (optionally with a description or PRD reference)
- presents an approved PRD and asks for the engineering design, or describes a feature that is concrete enough to design but not yet coded

Do NOT trigger for: vague problem framing (that's `prd`), granular task breakdown into 2–5 minute steps (that is a future `plan` skill), or actual implementation.

## Core invariants

1. **No code until the SPEC is approved.** No scaffolding, no edits to source files, no dependency installs.
2. **Ask ONE question at a time. Max 5 total** (fewer when a PRD is in context — see Phase 1).
3. **Verify before claiming.** Every claim about existing modules, schemas, interfaces, or endpoints must be confirmed by reading the file. Unverified claims MUST be labeled `(unverified)`.
4. **No synthetic decisions.** Only document decisions that actually required deliberation. If a path is obvious, name it without manufacturing alternatives.
5. **Every mini-ADR must list Alternatives Considered.** A Decision with no alternatives is a rationalization.
6. **Durability rule for tracer-bullet phases.** Phases describe *what capability* a slice delivers + acceptance criteria. Do NOT list file names or function names inside phases — those change during implementation.
7. **Run the adversarial red-team pass before the self-review checklist.** Skipping it is a protocol violation.
8. **Emit the self-review checklist to the user** before writing. Skipping it is a protocol violation.
9. **HARD GATE: do not write SPEC.md until the user approves the proposed direction AND the output path.**
10. **Spec-vs-implementation: the SPEC is a contract to interpret, not code to paste.** Code blocks longer than ~15 lines are almost always bloat — replace them with a behavioural description + table + invariants, or cut them. Detailed rule, fix patterns, and before/after in `references/anti-bloat.md`.

## Pre-flight (MANDATORY)

Before anything else:

1. Get today's date: `date +%Y-%m-%d` via Bash. Store as `<DATE>`.
2. If the user's invocation had no description (e.g., bare `/spec`), ask: *"What would you like to spec? Describe the feature, problem, or approved PRD you're designing for."* Do not proceed until you have a description.
3. Announce the skill is active with a single line: `spec: starting (scope TBD)`.

## Process

Execute phases in order. Do not skip ahead.

### Phase 0 — Resume, PRD detection, Scope

**0.1 Resume check.** Look in `docs/` for any directory matching `<DATE>-*` or a prior date whose topic obviously matches. If a matching `SPEC.md` exists, ask the user:
```
Found an existing SPEC at <path>. Resume from it or start fresh?
  A) Resume and update [Recommended if topic matches]
  B) Start fresh — create a new dated directory
```
If resuming, read the file, summarize current state, and continue from open questions. Update the existing file rather than creating a duplicate.

**0.2 PRD detection.** Glob `docs/*/PRD.md` (newest first). If a PRD matches the current topic (by slug or description), read it and announce:
```
spec: found PRD at <path>. I'll build on it.

PRD summary:
- Problem: <one sentence>
- Recommended direction: <one sentence>
- Complexity (PRD): <LOW | MEDIUM | HIGH>
- Goals: <count> · Non-goals: <count> · User stories: <count>

Proceed building the SPEC on this PRD, or start independent of it?
  A) Build on the PRD [Recommended]
  B) Start independent
```

When building on a PRD: the SPEC lives in the **same dated folder** as the PRD (e.g. PRD at `docs/2026-04-16-faster-checkout/PRD.md` → SPEC at `docs/2026-04-16-faster-checkout/SPEC.md`). No slug re-negotiation.

If no PRD is found: continue without one. The SPEC is standalone.

**0.3 Scope classification.** Use the feature description (plus PRD complexity if present) and a quick glob/read to classify:

| Tier | Criteria | Ceremony |
|------|----------|----------|
| **Lightweight** | Small, well-bounded, low ambiguity. 1-3 files likely affected. Pattern already in codebase. | Compact SPEC. Skip Phase 1 if requirements are clear from PRD. 0-2 Qs. Mini-ADRs only where a real decision was made (often 1). Tracer-bullet phases may be 1-2. |
| **Standard** | Normal feature or bounded design. 3-10 files. Multiple valid approaches on at least one axis (API shape, data model, or integration point). | Full phases, 2-5 Qs, 2-4 mini-ADRs, 2-4 tracer-bullet phases, offer a diagram. |
| **Deep** | Cross-cutting, strategic, or irreversible. 10+ files, new architectural pattern, or external-API surface change. | Full phases + mandatory Mermaid diagram + full adversarial red-team + explicit migration/rollout section. |

Announce the classification: `spec: scope = <TIER>`. If unclear, ask ONE disambiguating question.

For detailed rubric, see `references/scope-tiers.md`.

### Phase 1 — Understand what's NOT already clear (≤5 Qs)

When a PRD is in context, the WHY / WHO / SUCCESS / NON-GOALS are usually answered. Do NOT re-ask them. Spend the question budget on the architectural gaps:

1. **SCALE** — expected load, data volume, growth curve, p95 latency targets?
2. **CONSISTENCY** — strong vs eventual; transactional boundaries; concurrency model?
3. **INTEGRATION** — which existing modules, services, or third-parties must this touch?
4. **COMPATIBILITY** — backwards-compat required? Migration strategy? Feature flag?
5. **OPERABILITY** — who owns it on-call, what is observable, what alerts exist?

When running **standalone** (no PRD), you may need to combine these with product-level questions — prefer to suggest `prd` first if the problem framing itself is vague. If the user declines, continue with up to 5 Qs mixing product + technical.

Rules:
- ONE question per turn. Never batch.
- Prefer `AskUserQuestion` with multiple-choice when natural. Mark the best pick with `[Recommended]` only when you genuinely have one.
- STOP early when the architectural picture is clear. Remaining unknowns become `(unverified)` assumptions to be validated in Phase 2 or surfaced as Open Questions.

Fewer Qs when PRD is in context; lean into the full 5 for Deep standalone work.

### Phase 2 — Codebase pre-scan

Ground the SPEC. Match depth to scope:

**Lightweight:** Grep for the topic; read the one or two files the change will touch; confirm the pattern exists.

**Standard:**
1. *Constraint check.* Read `AGENTS.md`, `CLAUDE.md`, or `README.md` for workflow / architecture constraints.
2. *Topic + pattern scan.* Grep for relevant terms. Read the primary module(s) likely touched. Skim one or two adjacent examples covering similar behavior or a similar pattern.
3. *Interface map.* Identify the public interfaces, schemas, and error shapes at module boundaries that the SPEC will touch or extend.

**Deep:** Standard + map the full module boundaries likely to be affected, list the interfaces and their current shape, and locate prior-art SPECs or ADRs in `docs/` to match voice and format. Deep scope **fans out the pre-scan into 3–5 parallel `Explore` agent calls** in a single message — one for the target module(s), one for each distinct consumer surface the change is likely to touch, and one for shared utilities / schemas / middleware that the change will cross. Each call runs at `medium` thoroughness with a self-contained prompt (state the SPEC topic, the specific area to map, and what to report back). Sequential exploration on Deep is a protocol violation — the latency cost is real and the single-threaded pre-scan routinely misses a consumer that the fan-out catches. Merge the returned summaries into the Grounding summary below, attributing each bullet to the lane it came from.

After scanning, produce a short **Grounding summary** (3-10 bullets). Every factual claim that ends up in the SPEC must trace back to a file read here. Claims that cannot be verified must be labeled `(unverified)` in the SPEC.

### Phase 3 — Frame the technical problem

Produce these blocks. They carry forward into the SPEC.

**Context** — one paragraph. If a PRD exists, summarize its direction in one sentence and cite the path. If standalone, state the problem and the user impact in technical terms.

**Goals (technical)** — numbered. Each MUST be verifiable. *"Every task write is durable across a node restart"*, not *"make it reliable."*

**Non-Goals (technical)** — numbered. Real exclusions, not goals in disguise. *"Multi-region replication is out of scope"*, not *"we will not ignore reliability."*

**Constraints** — tech stack, language/runtime, deployment target, backwards-compat, infra, regulatory, timeline.

**Assumptions** — numbered with verification status:
```
1. <assumption> — verified by reading <path>: YES / NO
2. <assumption> — verified: YES / NO
```

### Phase 4 — Current → Proposed architecture

**Current state.** Describe the parts of the system this SPEC affects TODAY, grounded in files read in Phase 2. One short paragraph plus a bullet list of the relevant modules + their responsibilities.

**Proposed state.** Describe the system AFTER this work: new modules, renamed or absorbed responsibilities, new boundaries. Name module responsibilities in interface terms ("exposes X, depends on Y"), not implementation terms.

This is the "what does the shape of the system look like" section. Keep it at the module level. Data model and interfaces have their own sections.

### Phase 5 — Key Decisions (mini-ADRs)

For every decision that required real deliberation, capture a mini-ADR inline. Assign sequential IDs `D-1`, `D-2`, etc.

Each entry:

- **ID & Title** — `D-N: <short decision>`
- **Context** — what pressure forced this decision (constraint, goal, trade-off)
- **Decision** — one or two sentences, definitive
- **Alternatives Considered** — at least one; each with a one-line reason for rejection
- **Consequences** — what becomes easier, what becomes harder, what rollback costs

**Invariant:** A mini-ADR with no Alternatives Considered is a rationalization — delete it or find the alternative you dismissed in your head.

**Quality check:** If every decision recommends the status quo with no trade-off named, you did not deliberate; you documented the default.

### Phase 6 — Data Model, API Contracts, Module Boundaries

**Data Model.** Entities, fields, types, relationships. Call out invariants (uniqueness, foreign keys, cascade rules). For schema changes, note whether they are additive, require a migration, or require a rollout sequence.

**API Contracts / Interfaces.** For every new or modified public interface (HTTP endpoint, GraphQL resolver, RPC, exported function):
- Inputs (required / optional, with types)
- Outputs (success shape; each error shape consistent with the rest of the codebase)
- Idempotency / retry behavior (if applicable)
- Backwards-compatibility note

Use a discriminated-union-style shape for state variants when relevant. Validate at boundaries; trust internal contracts.

**Module Boundaries.** List the directories or files that will be **created**, **modified**, or **absorbed/deleted**. For each: a one-line responsibility. This is a map, not a task list — no granular steps.

For deeper principles (Hyrum's Law, One-Version Rule, validation-at-boundaries), see `references/section-guides.md#api-contracts`.

### Phase 7 — Tracer-bullet Phases, Test Strategy, Risks

**Tracer-bullet Phases.** Break the work into 1-4 thin vertical slices. Each slice cuts end-to-end (data → interface → consumer) and is independently verifiable. Each phase:

- **Name** — short, capability-oriented ("Repeat customer 1-tap path", "Persisted checkout sessions")
- **What it delivers** — one paragraph, user-observable or system-observable behavior
- **Acceptance criteria** — 2-5 bullets, behavior-focused, testable
- **Depends on** — previous phase IDs or "none"

**DO NOT** name specific file paths, function names, or class names inside phases. Those are implementation-volatile; the SPEC should survive a refactor.

**Test Strategy.** What to test, at what level. Prior art: cite one or two existing test files that illustrate the pattern we'll follow. Call out any risky paths that need extra coverage (concurrency, migrations, 3rd-party failure modes).

**Risks & Mitigations.** Table of risks with impact (LOW/MED/HIGH) and mitigation. Include at least one risk for Standard and three for Deep.

### Phase 8 — Visual companion (conditional)

Mirror the PRD policy:

- **Lightweight:** skip unless user asks.
- **Standard:** if the topic has structural richness (multiple components, data flow, state machine), ASK once:
  ```
  This SPEC has <structural/flow/stateful> complexity that would benefit from a diagram.
  Include an inline Mermaid diagram?
    A) Yes — <component | sequence | state | flowchart> diagram [Recommended]
    B) No — text is sufficient
  ```
- **Deep:** mandatory — do not ask, just include one. Prefer a component diagram for cross-module work, a sequence diagram for multi-actor flows, or a state diagram for lifecycle changes.

If yes, draft a Mermaid block that will live in the SPEC under `## Diagram`.

### Phase 9 — Adversarial red-team pass

Tier-conditional. The red-team pass happens BEFORE the self-review checklist. Material issues raised must be integrated into the SPEC (fix in-place) or logged as Open Questions / Risks.

**Dispatch policy by tier:**

| Tier | Mode |
|------|------|
| **Lightweight** | Inline one-liner per role. No agent dispatch. |
| **Standard** | Inline by default. Offer full dispatch via `AskUserQuestion` when the SPEC touches a public API surface, a schema / migration, or a trust boundary. |
| **Deep** | **Mandatory parallel dispatch** of all three reviewer agents via the `Agent` tool. |

**Inline mode (Lightweight, Standard default).** Critique the SPEC yourself from three roles. Keep it brief — one line per role. Do NOT manufacture weak critiques to fill slots; `no material concern` is a valid answer.

- **Staff engineer** — what goes wrong under load, at a partition, during deploy, during rollback?
- **Security reviewer** — trust boundaries, authN / authZ surface, input validation, data exposure, rate-limiting, secret handling.
- **Future maintainer** — is the *why* behind each decision clear to a cold reader six months later? Does any section feel copy-pasted or aspirational?

Inline output:
```
Red-team pass (inline):
- Staff engineer: <one-line critique>  → Addressed in <section> / Open Question Q-N / N/A
- Security reviewer: <one-line critique>  → Addressed in <section> / Open Question Q-N / N/A
- Future maintainer: <one-line critique>  → Addressed in <section> / Open Question Q-N / N/A
```

**Dispatch mode (Standard opt-in, Deep mandatory).** Send all three agents in parallel — a single message with three `Agent` tool calls, not sequential:

- `subagent_type: "spec-staff-engineer-reviewer"` — production risk, load, deploy / rollback, concurrency, operability
- `subagent_type: "spec-security-reviewer"` — trust boundaries, authN / authZ, input validation, data exposure, secrets, abuse
- `subagent_type: "spec-future-maintainer-reviewer"` — cold-read durability, unexplained *why*, rationalized decisions, untestable criteria

Each prompt MUST include (passed inline — the SPEC has not been written yet, so there is no path to reference):

1. The full draft SPEC content inside a `<DRAFT SPEC>` fenced block.
2. A one-sentence topic summary.
3. The repo root path (so agents can verify claims against the codebase).

Each agent returns 3-5 section-anchored critiques with a Posture line and a Bottom line. After all three return, present the consolidated output to the user in one block, then decide per critique: fix in SPEC, log as Open Question `Q-N`, log as risk `R-N`, or mark N/A with a one-line justification.

**Standard opt-in ask.** When a Standard SPEC touches a public API surface, a schema / migration, or a trust boundary, ask via `AskUserQuestion`:
```
This SPEC touches <public API surface | schema change | trust boundary>. Dispatch the red-team reviewers for a full pass, or inline one-liner only?
  A) Dispatch all three agents (staff engineer, security reviewer, future maintainer) in parallel [Recommended]
  B) Inline one-liner — faster, lower depth
```

If the Standard SPEC touches none of those surfaces, skip the ask and use inline mode.

**Invariants for the dispatch path:**
- Do NOT paraphrase agent output as your own observation — attribute critiques to the agent role.
- Do NOT dispatch twice; one red-team pass per draft.
- If the user requests substantial changes post-gate and loops back to Phase 4 / 5, re-run the red-team pass at the same tier after the revision lands.

### Phase 10 — Self-review checklist (MANDATORY, emit to user)

Before writing, emit this checklist with `✓` or `✗` on each line.

- [ ] Context is specific; PRD is cited if present
- [ ] Goals and non-goals are technical (verifiable behavior, not product claims)
- [ ] Constraints name tech stack, compatibility, timeline explicitly
- [ ] Every codebase claim is verified by reading files; unverified items labeled `(unverified)`
- [ ] Current architecture block reflects what the code actually is
- [ ] Every mini-ADR has Context, Decision, ≥1 Alternative, and Consequences
- [ ] No mini-ADR is a rationalization of the default with no real alternative
- [ ] Data model states invariants (uniqueness, relationships, migration shape)
- [ ] API contracts specify inputs, outputs, error shape, idempotency, and backwards-compat
- [ ] Module boundaries list created/modified/absorbed with responsibilities
- [ ] Tracer-bullet phases describe capabilities and acceptance criteria; **no** file or function names
- [ ] No code block exceeds ~15 lines; every interface / data-type expressed as responsibility table + invariants, not full signatures (spec-vs-implementation test — see `references/anti-bloat.md`)
- [ ] Worked-example paragraphs have been replaced with acceptance criteria an implementer must make true
- [ ] Test strategy names levels and cites prior-art tests
- [ ] Risks table exists and names real concerns (not "bugs")
- [ ] Adversarial red-team pass emitted; material issues addressed or logged as Open Questions
- [ ] Open Questions are tagged `now` vs `during-build`
- [ ] Not-Doing list is non-empty and names real trade-offs
- [ ] No placeholder text or `[TBD]` markers
- [ ] Complexity tier matches scope classification

If any check fails → STOP, fix, re-emit. Do not proceed with any `✗` unresolved.

### Phase 11 — HARD GATE: approve direction + path

If building on an existing PRD, the output path is the PRD's folder. Otherwise, propose a slug: kebab-case, 2-4 words, derived from the topic.

Present the gate via `AskUserQuestion` (or plain text if unavailable):

```
Ready to write the SPEC.

Summary:
- Context: <one sentence>
- Key direction: <one sentence>
- Complexity: <LOW | MEDIUM | HIGH>
- Phases: <count> · Mini-ADRs: <count> · Risks: <count>
- Proposed path: docs/<DATE>-<slug>/SPEC.md

Approve and write, or describe changes?
  A) Approve — write the SPEC at the path above [Recommended]
  B) Change the slug / path
  C) Change the direction (re-open Phase 5 or 4)
```

**STOP HERE.** Do not create the directory. Do not write SPEC.md. Do not invoke other skills. WAIT for user approval.

If the user requests changes, loop back to the relevant phase, then re-emit the Phase 9 red-team + Phase 10 checklist before re-gating.

### Phase 12 — Write the SPEC

Only after explicit approval:

1. `mkdir -p docs/<DATE>-<slug>/`
2. Write `docs/<DATE>-<slug>/SPEC.md` using `templates/SPEC.md`. Replace every `{{placeholder}}` with actual content. Set `status: approved` in frontmatter. If a PRD was the source, populate the `origin` field with its path.
3. If a diagram was agreed (Phase 8), include it under `## Diagram`.
4. Confirm to the user:
   ```
   SPEC written: docs/<DATE>-<slug>/SPEC.md
   ```

The SPEC MUST be complete — no placeholders, no TODOs. It MUST be standalone: readable without this conversation.

### Transition

After writing, INFORM the user in one sentence: *"SPEC written and approved. You can now hand this off to an implementation plan / task breakdown, open a GitHub issue with its contents, or iterate on it directly."*

Do NOT invoke any implementation skill. Do NOT start coding. The skill is complete.

## Anti-patterns

- **"This is just a small change."** If it crosses a module boundary, changes a schema, or touches a public interface, write at least a Lightweight SPEC. Skip Phase 1 if clear; don't skip the artifact.
- **"The PRD already covers this."** The PRD covers *what* and *why*. The SPEC covers *how*. Different reader, different decisions.
- **"I already know the architecture."** Surface it as an assumption. The SPEC validates assumptions.
- **"Let me list 10 decisions to look thorough."** A fake decision is worse than an omitted one. A decision without real alternatives is a rationalization.
- **"Let me put file paths in the phases so it's actionable."** No. Phases are durable; file paths are volatile. Paths belong in the Module Boundaries map, not the phases.
- **Yes-machining the obvious path.** If the user's proposed design has a real flaw, the red-team pass must surface it. Kindly, specifically, and once.
- **Letting the SPEC slide into a task list.** Granular 2–5 minute TDD steps are the job of a future `plan` skill. Stop at tracer-bullet phases + acceptance criteria.

## Red flags — self-check

- You cited a file, schema, endpoint, or function that you did not read
- You manufactured alternatives inside a mini-ADR to satisfy the template
- All mini-ADRs end "stick with current approach" and name no real loss
- A tracer-bullet phase names a file or function
- A code block longer than ~15 lines appears in Phase 4/5/6 (you documented implementation, not a contract)
- An interface is spelled out as a full method signature list instead of a responsibility table
- A worked-example paragraph substitutes for acceptance criteria
- Non-Goals is empty or tautological ("we will not build a bad product")
- A risk is "bugs" or "unexpected issues"
- You skipped the Phase 9 red-team
- You skipped the Phase 10 checklist
- You wrote SPEC.md before the user confirmed in Phase 11
- The SPEC cannot be understood without reading the conversation

## References

- `templates/SPEC.md` — the output template. Read only at Phase 12.
- `references/scope-tiers.md` — detailed Lightweight / Standard / Deep rubric with re-classification rules.
- `references/section-guides.md` — how-to for each SPEC section (good/bad examples for Goals, ADRs, Data Model, API Contracts, Tracer-bullet Phases, Risks, Not-Doing). Load when a section is underperforming the self-review checklist.
- `references/anti-bloat.md` — the spec-vs-implementation test, the three common violators (function signatures, data-model literals, worked examples), and before/after fixes. Load when any section drafts longer than ~15 lines of code or reads like pseudo-implementation.
