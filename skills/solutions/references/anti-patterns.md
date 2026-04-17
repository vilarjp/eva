# Anti-Patterns — Rationalisations, Shortcuts, and Failure Modes

SOLUTIONS.md is a reference document future sessions read cold. The antipatterns below are the ways the synthesis degrades into something unreadable, unreliable, or obsolete. Load this reference when you catch yourself about to cut a corner.

## The Iron-Law violations

### 1. Anchoring learnings to file:line

**Symptom:** Gotchas, Key Decisions, or Root Cause entries that name `src/foo.ts:42` as the substance.

**Why it fails:** The moment `foo.ts` is renamed or the line shifts (next refactor, next month, next quarter), the learning points at nothing. File:line anchors are the fastest-rotting data in a long-lived repo — SOLUTIONS.md is meant to outlast them.

**Fix:** Rephrase as a behaviour / invariant / mechanism.

- Bad: *"See `src/cache.ts:42` for the race."*
- Good: *"The cache must be invalidated before the write commits; invalidating after leaves a window where concurrent readers see stale values."*

File:line is allowed only in the `## References` section (pointing at upstream artifacts, not source) and as illustrative *examples* inside a prose clause. Never as the substance.

### 2. Copy-pasting from source artifacts verbatim

**Symptom:** The Root Cause paragraph is a copy of DIAGNOSIS.md's root cause. The Gotchas are copy-pasted Concerns. The Key Decisions are copy-pasted SPEC mini-ADRs.

**Why it fails:** SOLUTIONS adds no value over the sources — the reader could have just opened DIAGNOSIS.md. The whole point is synthesis: rephrased for durability, compressed across artifacts, deduped.

**Fix:** Every paragraph in SOLUTIONS is **this pipeline's synthesis of this source's content**, not a quote. If you're copy-pasting, you're not synthesising.

### 3. "The commit messages already say this"

**Symptom:** Approach section that reads like a CHANGELOG. Key Decisions that restate commit subjects.

**Why it fails:** Commit messages say *what* changed in a single diff. SOLUTIONS says *why* the path was chosen over alternatives and *what to watch afterward*. These are different documents.

**Fix:** If a sentence in SOLUTIONS could be dropped verbatim into a commit message, rewrite it. Name the trade-off, the invariant, or the alternative considered — things that don't fit in a 50-character commit subject.

## Padding anti-patterns

### 4. Synthetic entries to hit section quotas

**Symptom:** Three Gotchas listed because "a Standard pipeline should have three gotchas." Two Key Decisions because "every doc needs some decisions." A fake alternative so the mini-ADR feels real.

**Why it fails:** Future readers rely on SOLUTIONS as a filter — "here are the things that actually matter for this area." Padding dilutes the signal until readers stop trusting the doc.

**Fix:** Honest empty sections beat padded ones. When a section has nothing to say:

- *"No gotchas surfaced during this pipeline. This section is intentionally empty."*
- *"No post-plan decisions emerged — see SPEC.md's embedded mini-ADRs for planning-time decisions."*
- *"No material dead ends during this pipeline."*

A Standard pipeline with no What-Didn't-Work entries is rare — look again before declaring it empty. A Lightweight pipeline with no Gotchas is plausible.

### 5. "Nothing surprising happened"

**Symptom:** The user (or you) argues the pipeline was routine, so there's nothing to capture.

**Why it fails:** Every pipeline produces at least a few decisions worth recording — SPEC mini-ADRs that proved their weight, trade-offs accepted mid-execution, risks that materialised or were dodged. The *feeling* that nothing surprising happened is usually a memory bias — the artifacts themselves have specifics.

**Fix:** Open the artifacts. Scan EXECUTION's Concerns section. Scan CODE-REVIEW's Residual risks. Something is there.

### 6. "The gotchas section is empty because there were none"

**Symptom:** Deep pipeline with multi-slice EXECUTION, distributed concerns, or a migration — and zero gotchas in SOLUTIONS.

**Why it fails:** That pipeline didn't finish without invariants that mattered. You didn't look hard enough.

**Fix:** Re-read EXECUTION's NOTICED BUT NOT TOUCHING list, Concerns, and the per-slice GREEN proofs (what did the test end up asserting?). Re-read CODE-REVIEW's Testing gaps. Re-read DIAGNOSIS's Concerns (if applicable). The gotchas are there.

## Process anti-patterns

### 7. Skipping artifacts because "EXECUTION has everything"

**Symptom:** You draft SOLUTIONS from EXECUTION alone, skipping PRD / SPEC / DIAGNOSIS / CODE-REVIEW.

**Why it fails:** Each artifact holds different signal:
- PRD: problem framing and non-goals (useful for Summary).
- SPEC: mini-ADRs and alternatives considered at planning time (input to Key Decisions).
- DIAGNOSIS: REJECTED hypotheses and suggested defense-in-depth (input to What Didn't Work and Prevention).
- REVISION: patches that shifted direction (input to Key Decisions — these are late decisions).
- CODE-REVIEW: suppressed findings and testing gaps (input to What Didn't Work and Prevention).

EXECUTION is a log of doing; SOLUTIONS is a synthesis of deciding + doing + reviewing. You need all three.

**Fix:** Phase 1 mandates reading every present artifact. If you find yourself jumping to Phase 2 early, return to Phase 1.

### 8. Asking the user "what else did we learn?"

**Symptom:** The skill interviews the user mid-synthesis for gotchas or decisions not in the artifacts.

**Why it fails:** Tacit knowledge extraction was explicitly out of scope for this skill. If the learning wasn't in the artifacts, it was not verified — and unverified learnings are not durable (the next engineer can't check them).

**Fix:** If you notice a gap in the artifacts, log it as a Concern in SOLUTIONS (*"Unverified: did the production retry logic ever fire? Worth a follow-up check."*) — do not ask the user to fill the gap at synthesis time.

Disambiguation questions (which folder, resume policy) are allowed — they're not content extraction, they're logistics.

### 9. Writing before the checklist + gate

**Symptom:** `SOLUTIONS.md` appears on disk before Phase 4's checklist and Phase 5's HARD GATE completed.

**Why it fails:** The gate is the contract with the user that synthesis quality passed a self-review. Skipping it erodes trust the whole plugin depends on.

**Fix:** Never write before the gate. The cost of one approval round is trivial compared to the cost of a bad artifact landing and having to be rewritten from memory.

## Boundary anti-patterns

### 10. Editing upstream artifacts "while I'm here"

**Symptom:** You fix a typo in SPEC.md, add a missing link in EXECUTION.md, or update DIAGNOSIS's frontmatter during Phase 1 reading.

**Why it fails:** Solutions is **read-only** outside its write target. Upstream artifacts are history — they record what the pipeline looked like at each phase. Editing them post-hoc erases that history and breaks the audit trail the other skills rely on.

**Fix:** If you spot an error in an upstream artifact, note it as a Gotcha in SOLUTIONS (*"SPEC.md's tracer-bullet Phase 2 acceptance criterion was ambiguous — interpreted as X during execution; future SPECs should be more explicit about Y."*). Or raise a follow-up. Do not edit.

### 11. Editing CLAUDE.md / AGENTS.md "to make solutions discoverable"

**Symptom:** You add a line to CLAUDE.md pointing at `docs/*/SOLUTIONS.md`.

**Why it fails:** Instruction-file edits change behaviour for every future session. The user explicitly opted out of this for the solutions skill. A "small helpful edit" by the skill is a breach of that contract.

**Fix:** The transition text mentions the opportunity once, plainly, with no gate. The user decides to make the edit or not. Solutions does not touch the file.

### 12. Chaining to another skill after writing

**Symptom:** After writing SOLUTIONS.md, the skill invokes `/code-review`, `/execute`, or re-runs itself.

**Why it fails:** Solutions is the **end** of the pipeline. Its transition is a pointer back to the user, not a handoff to another skill. Chaining blurs the boundaries eva depends on.

**Fix:** Emit the transition line ("Solutions written. Future sessions working on this area can now open <path>."). Stop.

## Destructive anti-patterns

### 13. Overwriting prior SOLUTIONS.md silently

**Symptom:** The folder had an existing SOLUTIONS.md; the skill replaced it without explicit user consent.

**Why it fails:** Prior synthesis is history. Overwriting destroys the record of what was captured at a previous point in time. A future reader needs to see the evolution.

**Fix:** Default behaviour is **append** — `## Post-Release Bug Fix <DATE>` (if DIAGNOSIS is the trigger) or `## Re-run <DATE>` (otherwise). Overwrite requires the user to type "overwrite" verbatim at Phase 0.4. No shortcuts.

### 14. Auto-chaining from /code-review or /execute

**Symptom:** A transition line in code-review or execute says "now invoking /solutions" and solutions starts without the user typing anything.

**Why it fails:** The user chose manual invocation for solutions (Phase 5 design decision). Auto-chaining removes agency and violates the contract.

**Fix:** Solutions is **manually invoked**. Code-review's and execute's transition lines may *suggest* running /solutions; they do not invoke it.

## Scope anti-patterns

### 15. Lightweight SOLUTIONS with the Deep section set

**Symptom:** A Lightweight pipeline produced a SOLUTIONS.md with every section populated, including What Didn't Work (empty), Prevention (empty), and a Mermaid diagram (missing).

**Why it fails:** The tier discipline exists to match ceremony to pipeline weight. A Lightweight synthesis stuffed with empty sections signals "I didn't trust my tier classification."

**Fix:** Lightweight means compact. Sections not in the Lightweight set should be **omitted**, not included as placeholders. If the pipeline actually has a dead end or a gotcha, promote to Standard first (with an announcement), then include the sections honestly.

### 16. Deep scope without a mandatory diagram

**Symptom:** Deep classification in frontmatter but no Mermaid diagram section.

**Why it fails:** Deep pipelines have concurrency, cross-module, or architectural shapes that a diagram captures more durably than prose. Omitting the diagram loses information density at the exact point where it matters most.

**Fix:** Diagram is **mandatory** for Deep. The Phase 4 self-review checklist has this item — don't skip it. If the synthesis truly has no diagrammable structure, the classification was probably wrong — consider demoting to Standard.

## Style anti-patterns

### 17. Narrating instead of synthesising

**Symptom:** Approach section reads as a story ("First we tried X, then we noticed Y, then we had a discussion and decided Z…").

**Why it fails:** SOLUTIONS is a reference, not a memoir. Narrative structure wastes word budget and buries the invariants.

**Fix:** State the current shape and the key trade-off. The story of how you got there belongs in EXECUTION's slice log, not in SOLUTIONS.

### 18. Editorial commentary

**Symptom:** Entries like *"this was a tricky bug"*, *"the team did a great job"*, *"in retrospect this was the obvious choice."*

**Why it fails:** Future readers want invariants and trade-offs, not sentiment.

**Fix:** Cut. A SOLUTIONS.md has no voice — it has facts phrased as durable behaviour.

### 19. "We" as the subject

**Symptom:** *"We decided to X."* *"We chose Y over Z."*

**Why it fails:** SOLUTIONS is read by agents and engineers who were not there. "We" has no referent. The tone across eva artifacts is third-person / impersonal.

**Fix:** *"The decision was X because Y; the alternative Z was rejected because W."* Tighter, referent-free, durable.

## Quick rationalisations table

| Rationalisation | Reality |
|---|---|
| "Nothing surprising happened." | Every pipeline has decisions in SPEC and Concerns in EXECUTION. Write it. |
| "The commit messages say this." | Commits say *what*; SOLUTIONS says *why*, *trade-off*, *what to watch*. |
| "I'll write it later with more perspective." | Later means never. You have maximum context now — synthesis is cheap today, expensive in two weeks. |
| "Gotchas section is empty because there were none." | You didn't look. Re-read Concerns, NOTICED-BUT-NOT-TOUCHING, Residual risks, Testing gaps. |
| "File:line makes it concrete for the reader." | File:line rots on the next rename. Invariants are concrete AND durable. |
| "I'll edit the CLAUDE.md while I'm here." | No. Mention in transition; user edits. |
| "Skip the gate — it's just a small doc." | The gate is the trust contract. Keep it. |
| "EXECUTION has everything, skip the rest." | Key Decisions come from SPEC; What Didn't Work comes from DIAGNOSIS/CODE-REVIEW; Prevention comes from DIAGNOSIS hotspots. Read all of them. |
| "This section should have 3 items, let me pad." | Empty honest beats padded fake. |
| "A diagram would be nice but I'll skip it." | If Deep scope: not optional. If Standard and structure warrants: offer once. |

## The honesty principle

Above every rule: **if SOLUTIONS is not useful to someone who has never seen the pipeline, it has failed.** Test the draft by reading it as if you opened it for the first time. Would you trust it? Would you know what to do next time you touched this area? If no, rewrite. If yes, ship.
