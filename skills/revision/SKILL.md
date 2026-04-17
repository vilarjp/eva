---
name: revision
description: Cross-review an already-drafted PRD.md and SPEC.md pair to catch gaps, inconsistencies, and mismatches before implementation. Use when the user wants to revise, audit, cross-check, or sanity-check paired PRD and SPEC documents, verify alignment between product framing and technical design, resolve contradictions between a PRD and a SPEC, or invokes /revision. Produces docs/YYYY-MM-DD-slug/REVISION.md in the same folder as the reviewed documents and, on explicit user approval, patches both the PRD and the SPEC with a dated Revision section that summarizes the changes. Composite pass covering cross-doc alignment, internal coherence, adversarial premise challenge, and scope creep / feasibility. Scope-adaptive (Lightweight / Standard / Deep). Strictly gated: no edits to PRD.md or SPEC.md without explicit user approval.
argument-hint: "[path to docs folder, or blank to auto-detect]"
---

# REVISION — Cross-review PRD.md and SPEC.md before implementation

Cross-document review of a drafted PRD.md ↔ SPEC.md pair. Catches goal/spec gaps, assumption drift, contradictions, terminology divergence, scope creep, and shaky premises before any code is written. The durable output is `docs/YYYY-MM-DD-<slug>/REVISION.md`; on approval, both source documents gain a dated `## Revision` section summarizing the fixes.

This skill does NOT write code. It reads both documents in full, scores them against a composite rubric, produces findings, and — only on explicit user approval — patches the source documents.

## When to use

Trigger this skill when the user:
- says "revise", "review", "audit", "cross-check", "sanity-check", "find the gaps", "are the PRD and SPEC aligned", "is there anything missing", "red-team this pair"
- invokes `/revision` (optionally with a path)
- has drafted PRD.md + SPEC.md (or at least one of them) and wants a pre-implementation checkpoint
- reports suspected mismatch, drift, or contradiction between a PRD and a SPEC

Do NOT trigger for: authoring a new PRD (that's `prd`), authoring a new SPEC (that's `spec`), or implementing code.

## Core invariants

1. **No edits to PRD.md or SPEC.md until the human says "go".** The skill reads, flags, writes REVISION.md, and waits. Patches are never implicit.
2. **No new features, no scope expansion.** Revision fixes gaps and inconsistencies only. If the review surfaces a missing capability, record it as a finding — do NOT invent or extend.
3. **Ask at most 3 questions, one at a time, only when genuinely ambiguous.** Most revisions need zero questions. Target the auto-detected context first.
4. **Read both documents fully before flagging anything.** No skimming. Findings based on partial reads are a protocol violation.
5. **Every finding must quote evidence.** At least one direct quote (5–20 words) from PRD or SPEC, with section reference. Findings without evidence are suppressed.
6. **Synthetic findings are forbidden.** If a lens genuinely has nothing to add, say so. A clean lens is valid; a padded one is harmful.
7. **Emit the self-review checklist to the user** before writing REVISION.md. Skipping it is a protocol violation.
8. **HARD GATE 1: do not write REVISION.md until the user approves the finding set and proposed path.**
9. **HARD GATE 2: do not patch PRD.md or SPEC.md until the user explicitly approves patching.** Writing REVISION.md does NOT imply patch approval.

## Pre-flight (MANDATORY)

Before anything else:

1. Get today's date: `date +%Y-%m-%d` via Bash. Store as `<DATE>`.
2. If the user's invocation had no path (e.g., bare `/revision`), auto-detect (Phase 0.2) before asking. Only ask if auto-detect fails or is ambiguous.
3. Announce the skill is active with a single line: `revision: starting (target TBD)`.

## Process

Execute phases in order. Do not skip ahead.

### Phase 0 — Resume, target detection, scope

**0.1 Target detection.** Identify the docs folder to review:

- If the user passed a path (folder, PRD.md, or SPEC.md), use it.
- Otherwise, glob `docs/*/PRD.md` and `docs/*/SPEC.md`. Prefer the folder containing BOTH. Prefer the most recent dated folder. If exactly one candidate exists, use it silently.
- If multiple candidate folders contain PRD+SPEC pairs, ASK ONE question:
  ```
  Found multiple PRD+SPEC pairs. Which one should I revise?
    A) docs/<folder-1>/ — <topic from frontmatter>
    B) docs/<folder-2>/ — <topic from frontmatter>
    C) docs/<folder-3>/ — <topic from frontmatter>
  ```

Announce the target: `revision: target = docs/<folder>/`.

**0.2 Single-doc mode.** If only ONE of `PRD.md` / `SPEC.md` exists in the target folder:

Announce and offer a degraded review:
```
revision: only <PRD.md | SPEC.md> found. Full cross-review requires both.

Degrade to single-doc review (internal coherence + adversarial only, no cross-check)?
  A) Yes — review the one that exists [Recommended if drafting is in progress]
  B) No — abort; I'll author the missing doc first
```

In single-doc mode, Phase 3.1 (cross-doc alignment) is skipped. REVISION.md frontmatter records `mode: single-doc` and names the missing sibling.

**0.3 Resume check.** If `REVISION.md` already exists in the target folder:

```
Found an existing REVISION.md at <path>. What would you like to do?
  A) Resume and update — overwrite with a fresh review [Recommended if docs changed]
  B) Create REVISION-2.md — preserve history, new findings file
  C) Start fresh — REVISION-2.md and archive the old one as REVISION-1.md
```

Subsequent revisions follow the same numbering convention. The patched `## Revision YYYY-MM-DD` section in PRD/SPEC always cross-references the specific REVISION file.

**0.4 Scope classification.** Inherit complexity from the docs first:

- Read PRD.md frontmatter → `complexity: LOW | MEDIUM | HIGH`.
- Read SPEC.md frontmatter → `complexity: LOW | MEDIUM | HIGH`.
- The revision tier is the MAX of the two (HIGH wins; MEDIUM wins over LOW).
- If either doc lacks `complexity` frontmatter, fall back to word-count + section-count heuristics in `references/scope-tiers.md`.

| Tier | Criteria | Ceremony |
|------|----------|----------|
| **Lightweight** | Both docs are LOW complexity; 1–3 sections each; obvious 1:1 mapping likely. | Compact pass. 0 clarifying Qs. 4 lenses run short. 1–4 findings typical. Skip adversarial when nothing jumps out. |
| **Standard** | At least one MEDIUM; normal PRD↔SPEC pair with real decisions to cross-check. | Full pass. ≤3 Qs. All 4 lenses. 3–10 findings typical. Include adversarial pass. |
| **Deep** | At least one HIGH; cross-cutting or irreversible work. Multiple mini-ADRs, multiple tracer-bullet phases, public API surface. | Full pass + explicit adversarial integration + mandatory suppression-table disclosure. 5–20 findings typical. Every `P0`/`P1` must have a proposed fix. |

Announce: `revision: scope = <TIER> (PRD=<LOW|MEDIUM|HIGH>, SPEC=<LOW|MEDIUM|HIGH>)`.

For detailed rubric and re-classification rules, see `references/scope-tiers.md`.

### Phase 1 — Clarifying questions (≤3, only if ambiguous)

Most revisions need zero questions. Only ask when the auto-detected context is genuinely ambiguous:

1. **Target folder** — multiple PRD+SPEC pairs detected (Phase 0.1).
2. **Resume mode** — existing REVISION.md, user intent unclear (Phase 0.3).
3. **Single-doc mode acceptance** — one doc missing, user's intent unclear (Phase 0.2).
4. **Scope override** — frontmatter complexity seems wrong given doc content.

Rules:
- ONE question per turn. Prefer `AskUserQuestion` with multiple-choice.
- STOP asking as soon as the context is resolved. Do not manufacture questions to fill the budget.
- Do NOT ask product-framing questions here — that's the PRD's job. Do NOT ask engineering-design questions — that's the SPEC's job. Revision only asks about *which* artifacts to review and *how deep*.

### Phase 2 — Load both documents FULLY

Read the entire PRD.md and SPEC.md (or the single doc in single-doc mode). Do NOT summarize or skim. Capture:

- Frontmatter (date, slug, complexity, status, origin)
- Goals + Non-goals (count, text)
- Assumptions + verification status
- Constraints
- For SPEC: mini-ADRs (IDs, titles, alternatives presence), data model, API contracts, tracer-bullet phases, risks
- For PRD: options, recommended direction, user stories, not-doing list
- Open questions in both (`now` / `during-build`)

Emit a short **Grounding summary** (3–8 bullets) to the user before running the rubric, so they can correct any misreading before findings are produced.

### Phase 3 — Composite review pass (4 lenses)

Run all 4 lenses in a single sequential pass. Each lens yields zero or more findings. When a lens has no material concern, say so explicitly — do not pad. The full rubric lives in `references/review-checklist.md`; load it if any lens feels underspecified.

**3.1 Cross-doc alignment** *(skipped in single-doc mode)*

- Every PRD goal → ≥1 SPEC section addresses it (goal coverage).
- Every SPEC commitment (mini-ADR, tracer-bullet phase, API contract) → traces back to a PRD goal, a PRD non-goal exclusion, or an explicit SPEC-only technical concern.
- PRD non-goals respected in SPEC (SPEC does not absorb them).
- PRD assumptions preserved in SPEC (same wording or explicit supersession with rationale).
- PRD constraints preserved in SPEC.
- PRD user stories mapped to tracer-bullet phases (each story acceptable at a specific phase).
- New SPEC assumptions not contradicted by PRD.
- Complexity tier consistency (PRD tier should generally align with SPEC tier; deviations require explicit rationale in the SPEC's Phase 0 preamble).

**3.2 Internal coherence**

- Contradictions within each doc (claim X in one section, ¬X in another).
- Contradictions across the pair (same term used differently; same count/number/name mismatched).
- Terminology drift (PRD says "session token", SPEC says "auth credential"; PRD says "customer", SPEC says "user").
- Broken internal references (section/decision IDs that don't exist).
- Ambiguous wording where two readers would reasonably diverge.
- Empty sections, `[TBD]` markers, or placeholder text.
- Orphaned mini-ADRs (decisions not referenced by any other section).

**3.3 Adversarial premise challenge** — *dispatch the `revision-adversarial-review` agent*

Stance: *rigorous, not hostile* (defined in `references/review-checklist.md#adversarial-stance`). The lens runs in a fresh-context subagent so its read is not shaped by the findings you've already produced in 3.1 and 3.2.

Dispatch via the `Agent` tool with:

- `subagent_type: "revision-adversarial-review"`
- Inputs passed inline in the prompt:
  1. Absolute paths to `PRD.md` and/or `SPEC.md` (both in full cross-review; one in single-doc mode).
  2. The scope tier (`Lightweight` / `Standard` / `Deep`) — calibrates the agent's depth.
  3. A one-line note listing the other lenses already run (cross-doc, coherence, scope/feasibility) so the agent does not repeat their work.

The agent runs five techniques (premise / unstated assumption / decision stress / simplification pressure / alternative blindness) and returns a markdown document with `AF-1`, `AF-2`, … findings, each carrying a verbatim evidence quote, suggested severity, proposed fix, and confidence. Clean-pass lines for techniques that produced nothing are expected and valid.

**Tier calibration:**

- **Lightweight** — skip the dispatch. Do a one-line self-critique inline using the same five techniques; accept a clean pass.
- **Standard** — dispatch by default. Accept a clean pass on any technique.
- **Deep** — dispatch mandatory. Every `P0`/`P1` adversarial finding must be addressed in the Patch Plan or surfaced as an Open Question before Gate 1.

**Integration rules:**

- Renumber the agent's `AF-N` IDs into the skill's `F-N` sequence as you merge its findings with the other lenses in Phase 4. Preserve the verbatim quotes and the technique tag in the findings table's `lens` column (value: `adversarial:premise` / `adversarial:unstated` / `adversarial:decision-stress` / `adversarial:simplification` / `adversarial:alternative`).
- Do NOT paraphrase agent output as your own observation — the technique tag preserves attribution.
- Route any `confidence: low` finding through the Suppressed table (Phase 4), same rule as the other lenses.
- Copy the agent's full returned document (with `AF-N` IDs intact) into the REVISION.md **Adversarial Appendix** so the original read stays auditable.
- Do NOT dispatch twice; one adversarial pass per revision cycle. If the user requests material changes that loop back to Phase 3, re-dispatch at the same tier after the new draft lands.

**3.4 Scope creep + feasibility**

- SPEC scope proportional to PRD recommended direction (no hidden expansion).
- No unjustified abstractions or modules (each must trace to a PRD goal, constraint, or explicit SPEC trade-off).
- Proposed architecture survives happy / nil / empty / error paths (shadow-path tracing).
- Migration shape realistic (additive, online, reversible when possible).
- Risks table covers real failure modes; "bugs" is not a risk.
- Data model invariants named (uniqueness, foreign keys, cascade rules).
- API contracts specify idempotency, backwards-compat, and error semantics.
- Tracer-bullet phases are durable — no file/function names.

### Phase 4 — Classify findings

For every finding produced in Phase 3:

- **ID** — `F-1`, `F-2`, …
- **Lens** — `cross-doc` / `coherence` / `adversarial` / `scope-feasibility`
- **Title** — one line, descriptive (no severity prefix — severity is separate).
- **Evidence** — ≥1 quote from PRD or SPEC with section reference: `PRD §Goals`, `SPEC §D-2`, etc.
- **Why it matters** — one sentence explaining the risk or cost if unfixed.
- **Severity** — `P0 | P1 | P2 | P3` (see `references/review-checklist.md#severity-rubric`).
  - **P0 (Blocker)** — contradicts user's explicit ask, or PRD goal has zero SPEC coverage, or SPEC violates a PRD non-goal, or creates a security/data-integrity gap, or acceptance criterion is untestable.
  - **P1 (Major)** — key assumption unverified and blocks direction, mini-ADR missing alternatives, scope creep with real cost, risk table misses a real failure mode.
  - **P2 (Minor)** — ambiguous wording, terminology drift, broken refs, missing prior-art.
  - **P3 (Nit)** — typo, capitalization, optional field missing.
- **Finding type** — `error` (something is wrong) or `omission` (something is missing).
- **Proposed fix** — one or two sentences describing the minimal fix. Fix must NOT introduce new scope.
- **Target** — which document(s) the fix touches: `PRD`, `SPEC`, or `PRD+SPEC`.

**Invariant:** Each lens's findings must be genuinely distinct. If two findings have the same evidence quote + same proposed fix, merge them.

**Confidence suppression:** If a finding's evidence is indirect or requires speculation, mark it `(low confidence)` and move it to the **Suppressed findings** table in the REVISION artifact. Never silently drop a finding; visibility is the point of review.

### Phase 5 — Self-review checklist (MANDATORY, emit to user)

Before writing REVISION.md, emit this checklist with `✓` or `✗` on each line. A missing checklist is a protocol violation.

- [ ] Both documents were read in full (or the one, in single-doc mode)
- [ ] Grounding summary was emitted to the user and corrected if needed
- [ ] Every finding quotes ≥1 piece of evidence with section reference
- [ ] No finding proposes new features or scope expansion
- [ ] No lens is padded with synthetic findings — a clean lens is valid
- [ ] Adversarial lens ran (unless Lightweight with nothing to flag — then explicitly stated)
- [ ] Every finding has a severity, a finding_type, and a proposed fix
- [ ] Suppressed findings (low confidence) are listed separately, not silently dropped
- [ ] Cross-doc lens skipped correctly in single-doc mode (and not otherwise)
- [ ] Complexity tier matches the MAX of PRD/SPEC frontmatter
- [ ] No placeholder text or `[TBD]` markers in REVISION.md content
- [ ] Clean pass (zero findings) is explicitly stated if that is the result

If any check fails → STOP, fix, re-emit. Do not proceed with any `✗` unresolved.

### Phase 6 — HARD GATE 1: approve REVISION.md content and path

Propose the output path: **same folder as the docs under review**, filename per the Phase 0.3 resume decision (`REVISION.md` or `REVISION-N.md`).

Present the gate via `AskUserQuestion` (or plain text if unavailable):

```
Ready to write REVISION.md.

Summary:
- Target: docs/<DATE>-<slug>/ (PRD + SPEC | single-doc: <PRD | SPEC>)
- Complexity: <LOW | MEDIUM | HIGH>
- Findings: P0=<count> · P1=<count> · P2=<count> · P3=<count> · Suppressed=<count>
- Result: <material findings | clean pass>
- Proposed path: docs/<DATE>-<slug>/<REVISION.md | REVISION-N.md>

Approve and write, or describe changes?
  A) Approve — write REVISION.md at the path above [Recommended]
  B) Change a specific finding (re-open Phase 3 for one lens)
  C) Abort — do not write
```

**STOP HERE.** Do NOT create the file. Do NOT patch PRD or SPEC. Do NOT invoke other skills. WAIT for user approval.

If the user requests changes, loop back to Phase 3 for the relevant lens, then re-emit Phase 5 before re-gating.

### Phase 7 — Write REVISION.md

Only after explicit approval at Gate 1:

1. Write `docs/<DATE>-<slug>/<filename>` using `templates/REVISION.md`. Replace every `{{placeholder}}` with actual content. Set `status: approved` in frontmatter.
2. The file MUST be self-contained — no placeholders, no TODOs, readable without this conversation.
3. Confirm to the user:
   ```
   REVISION.md written: docs/<DATE>-<slug>/<filename>
   ```

If the result is a **clean pass** (zero findings), write REVISION.md anyway — it records that the review happened. Skip directly to the closing line below; do NOT proceed to Gate 2 because there is nothing to patch.

For clean pass, close with: *"Clean pass recorded. PRD and SPEC are untouched. You can now proceed to implementation."* The skill ends here.

For non-clean pass, proceed to Phase 8.

### Phase 8 — HARD GATE 2: approve patching PRD / SPEC

Present the patch plan:

```
REVISION.md is written. Now decide: should I patch PRD.md and SPEC.md with a dated ## Revision section reflecting these fixes?

Proposed patches:
- PRD.md: <count> changes across <sections> — adds a ## Revision <DATE> section referencing F-<ids>
- SPEC.md: <count> changes across <sections> — adds a ## Revision <DATE> section referencing F-<ids>

Patch how?
  A) Apply all patches and add ## Revision sections [Recommended when findings are material]
  B) Apply a subset — I'll specify which finding IDs to skip
  C) Don't patch — REVISION.md stands as an audit-only record
  D) Change the proposed fixes first (re-open Phase 3 on specific findings)
```

**STOP HERE.** Do NOT edit PRD.md or SPEC.md. WAIT for explicit approval.

**If the user picks C** (audit only), close with: *"REVISION.md stands as an audit record. PRD and SPEC are untouched. Findings remain available for future work."*

**If the user picks B** (subset), ask which finding IDs to skip, then proceed to Phase 9 with the reduced set.

**If the user picks D**, loop back to Phase 3 for the finding(s) in question, re-emit Phase 5 + Gate 1 (or a narrow re-gate for the specific finding), then re-present Gate 2.

### Phase 9 — Patch PRD.md and SPEC.md

Only after explicit approval at Gate 2:

For each finding targeted at `PRD`, `SPEC`, or `PRD+SPEC`:
1. Apply the minimal fix described in the finding's Proposed fix. Do NOT refactor adjacent content. Do NOT introduce new sections beyond what the fix requires.
2. Preserve the document's voice and formatting; match the template style for any new sub-elements.

After all fixes are applied, append to EACH patched document:

```markdown
## Revision {{DATE}}

Cross-reference: `REVISION.md` (or `REVISION-N.md`).

- [section] <what changed> — <why, finding F-<id>, severity>
- [section] <what changed> — <why, finding F-<id>, severity>
- [section] <what changed> — <why, finding F-<id>, severity>
```

If a prior `## Revision <earlier date>` section already exists (from a previous revision cycle), append the new section BELOW it. Each revision is preserved in order.

Confirm to the user:
```
Patches applied:
- PRD.md: <count> findings fixed, ## Revision <DATE> appended
- SPEC.md: <count> findings fixed, ## Revision <DATE> appended
```

### Transition

After patching (or deciding not to patch), INFORM the user in one sentence: *"Revision complete. PRD and SPEC are now consistent; you can proceed to implementation or iterate further."*

Do NOT invoke any implementation skill. Do NOT start coding. The skill is complete.

## Anti-patterns

- **"The docs look fine, nothing to revise."** Run the full pass before declaring clean. A clean pass is only valid after the 4 lenses actually ran. Premature "clean" is the most common failure mode.
- **"I'll just fix the small stuff as I go."** No. Every change to PRD or SPEC must be gated. Silent edits defeat the review's auditability.
- **"Let me suggest a better architecture."** Revision is not design. Surface the gap as a finding and stop. Authoring belongs to `spec`.
- **"Let me add a feature the SPEC missed."** No. Surface the omission as a finding. New scope is a new PRD, not a revision.
- **Padding the adversarial lens to look thorough.** A synthetic premise challenge is worse than none. If the premise holds, say so and move on.
- **Yes-machining the docs.** If a mini-ADR is a rationalization with no real alternative, flag it. Kindly, specifically, once.
- **Skipping the hard gates "to save time".** The skill's value is that PRD/SPEC only change with explicit consent. Skipping a gate corrupts that guarantee.
- **Batching questions in Phase 1.** One question per turn. Revisions rarely need any; never need more than 3.

## Red flags — self-check

- You flagged a finding without quoting evidence from the source documents
- You proposed a fix that introduces new capability rather than repairing an existing gap
- You wrote REVISION.md before the user confirmed Gate 1
- You edited PRD.md or SPEC.md before the user confirmed Gate 2
- You declared a clean pass without running the adversarial lens (unless Lightweight + explicitly stated)
- A "finding" has no proposed fix, or the fix is "consider X"
- Two findings share the same evidence and fix (should have been merged)
- You asked a product-framing or engineering-design question (out of scope for revision)
- Your severity distribution is all-P0 or all-P3 (calibration likely off — recheck the severity rubric)
- You ran cross-doc alignment in single-doc mode (impossible) or skipped it with both docs present (invalid)

## References

- `templates/REVISION.md` — the output template. Read only at Phase 7.
- `references/scope-tiers.md` — detailed Lightweight / Standard / Deep rubric with inheritance rules from PRD/SPEC frontmatter.
- `references/review-checklist.md` — full rubric for each of the 4 lenses, severity calibration, adversarial stance, and good/bad examples. Load when a lens is under-producing findings, when the self-review checklist flags a lens as weak, or when the `revision-adversarial-review` agent fails to dispatch and Phase 3.3 must fall back to inline.
- `agents/revision-adversarial-review.md` *(plugin root)* — the cold-context sub-agent dispatched by Phase 3.3. Returns structured findings with verbatim evidence quotes. Read-only.
