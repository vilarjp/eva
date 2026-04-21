# Artifact compactness — the shared discipline

Eva's artifacts are durable, cold-readable handoffs. They are **not** padded
forms. Every empty section, every placeholder sentence, every structural block
that isn't doing work makes the artifact slower to write, harder to read, and
weaker as a handoff. This file is the rule every skill's Phase 10 self-review
and Phase 11 HARD GATE enforces before an artifact ships.

## The rule

> Shorter is better when the information survives. If a cold reader cannot
> answer the artifact's purpose from the trimmed version, restore **one line**
> — an invariant, an alternative, or a file:line — never a paragraph.

Compactness is a durability property, not a terseness aesthetic. A 200-line
artifact that a future reader skims is strictly worse than a 50-line artifact
they read. Token cost is the secondary gain; comprehension is the primary one.

## Section caps

| Element                          | Cap                                     |
|----------------------------------|-----------------------------------------|
| Any prose section                | ≤ 5 lines                               |
| Any table                        | ≤ 10 rows (escalate to a sidecar above) |
| Any code / output block          | ≤ 15 lines (paraphrase above that)      |
| Frontmatter                      | unchanged — the contract                |
| Append-only re-run sections      | same caps per block                     |

Exceeding a cap is allowed only when the block **is** the contract (a canonical
error envelope, the literal RED output of a test, a migration DDL). In every
other case, the cap means "rewrite as a table or prose summary."

## Cut rules

- **Empty section → cut the heading.** No `_none_`, no `N/A`, no placeholder
  sentence. The section is gone.
- **Trivial section → cut the heading, keep one line.** A single invariant or
  file:line belongs in the neighbouring section.
- **Optional section → include only when the tier asks for it.** Diagrams on
  Lightweight are cut unless the user explicitly requested one.
- **Repeating shape → collapse to a table.** Options A/B/C, mini-ADRs,
  hypotheses, findings, phases, API endpoints, hotspots — one row each.

## Durability guardrails (never cut)

These are the reasons a cold reader trusts the artifact. Cutting any of them
defeats the refactor.

- Frontmatter block: `date`, `status`, `slug`, `topic`, `scope`, and the
  skill's own required fields (`severity`, `mode`, `pipeline`, etc.).
- **Alternatives column** for every real decision. A decision without a named
  alternative is a wish.
- **Evidence column** (file:line + verbatim quote) for every claim about code,
  a diff, a source document, or a PR comment.
- Append-only re-run sections: `## Re-review`, `## Re-audit`, `## Re-triage`,
  `## Re-run`, `## Fixes applied`, `## Refactors applied`,
  `## Post-Release Bug Fix`. Structure must survive the append.

## The three rewrite patterns

These generalise the SPEC-specific patterns in
`skills/spec/references/anti-bloat.md` to every other artifact. When a section
looks bloated, try them in order.

### 1 · Responsibility table

Replace multi-method interfaces, long signature lists, or repeating role
bullets with a single table: **Name · Responsibility · Inputs · Outputs ·
Error modes** (or the skill's equivalent columns). The shape survives
refactors; the specific signature lives in code.

Used in: `SPEC.md` (API Contracts), `TEARDOWN.md` (Function Inventory, State
Inventory), `AUDIT.md` (finding refactor shape), `EXECUTION.md` (slices).

### 2 · Entity + invariants

Replace schema literals or type-definition dumps with one paragraph naming
the entity + its relationships, and a short bullet list of invariants. Trivia
lives in the schema file.

Used in: `SPEC.md` (Data Model), `DIAGNOSIS.md` (Root Cause invariant),
`SOLUTIONS.md` (behaviour-as-invariant), `PRD.md` (Assumptions).

### 3 · Acceptance criteria

Replace worked examples — "step 1, user does X; step 2, server returns Y" —
with 2-5 post-conditions an implementer must make true. The criteria are
testable; the example was implementation in disguise.

Used in: `SPEC.md` (Tracer-bullet phases), `EXECUTION.md` (acceptance trace),
`PRD.md` (user stories), `DIAGNOSIS.md` (reproduction test assertions).

## The self-review check

The last bullet of every artifact-producing skill's Phase 10 checklist:

> **Compactness check.** Every empty / trivial / off-scope section is cut
> (not kept with a placeholder). Remaining sections pass the caps above.
> Cutting more would remove information, not just words.

If the check fails, trim — then re-run it. The HARD GATE does not fire until
it passes.

## Anti-patterns

- **"I'll leave the placeholder for clarity."** Placeholders are skeleton, not
  clarity. The filled artifact speaks for itself.
- **"The section is empty but I want to show we considered it."** The
  frontmatter's `scope` already shows what was considered. An empty section is
  noise.
- **"The reader will want the full signature."** The reader wants the
  semantics. The signature is how the implementer will encode them.
- **"A complex feature deserves a long artifact."** Most long artifacts are
  long because they copy-pasted through complexity. A complex feature with a
  short artifact is almost always a better-designed feature.
