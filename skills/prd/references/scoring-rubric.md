# PRD scoring rubric — 100-point self-check (Deep scope only)

A quantitative quality gate for Deep-scope PRDs. The per-item checklist in Phase 8 catches common defects; this rubric catches the subtler problem of a PRD that passes every individual check but still under-serves the next reader. Four dimensions, 100 total points, **target ≥ 90**. If the score lands below 90, iterate through the lowest-scoring dimension until it clears — do not proceed to the HARD GATE.

This rubric is **optional for Lightweight / Standard** tiers and **mandatory for Deep**. Deep PRDs tend to outlive the session that produced them; the extra rigour pays itself back the first time someone reads the file a month later.

## Scoring discipline

- Score honestly. Rounding up to hit 90 defeats the point.
- Each sub-criterion lands at `0` / partial / `full`. No half-credit for "almost." Partial is the midpoint of the band.
- Show the scoring to the user before Phase 9's HARD GATE — raw numbers, one-line justifications, total. The user sees the weakest dimension and can push back before the write.

## Dimension 1 — Functional completeness (30 points)

How well the PRD frames *what* the product should do.

| Criterion | Band | Points |
|---|---|---|
| User stories cover every primary actor × primary flow identified in Phase 1 | 0 / 5 / 10 | 10 |
| Error paths, edge cases, and failure modes are either named as goals / acceptance hooks or explicitly listed in Non-Goals | 0 / 5 / 10 | 10 |
| Success metrics are concrete and measurable — every goal has a way to know it is met | 0 / 5 / 10 | 10 |

**Red flag for this dimension:** a Goal that reads *"improve X"* with no measurement attached scores `0` on the third criterion, regardless of how well the story is written.

## Dimension 2 — Technical soundness (25 points)

How grounded the PRD is in the real codebase and real trade-offs.

| Criterion | Band | Points |
|---|---|---|
| Every factual claim about the codebase was verified by reading files; unverified items are labelled `(unverified)` | 0 / 4 / 8 | 8 |
| Options are genuinely different — not three variations of the same idea — and every option has concrete cons | 0 / 4 / 8 | 8 |
| Recommended direction follows logically from stated goals and constraints; the Phase 5 reasoning names the trade-off being accepted | 0 / 4 / 9 | 9 |

**Red flag for this dimension:** if the three options all have the same Cons pattern (e.g. "more complex," "more work"), the dimension drops to ≤ 10/25 and the options need redesign.

## Dimension 3 — Implementation readiness (25 points)

How useful the PRD is to the person who will pick it up and build from it.

| Criterion | Band | Points |
|---|---|---|
| Constraints are explicit — timeline, tech stack, backwards-compatibility, deployment, regulatory | 0 / 3 / 7 | 7 |
| Every user story maps to a verifiable acceptance hook (a thing you could write a test for, or observe in staging) | 0 / 4 / 9 | 9 |
| Not-Doing list is non-empty and names real trade-offs — not a placeholder like "performance optimisation later" | 0 / 4 / 9 | 9 |

**Red flag for this dimension:** an empty Not-Doing list caps the dimension at ≤ 16/25. Deep-scope PRDs always have scope decisions worth recording.

## Dimension 4 — Business impact (20 points)

Whether the PRD answers *why now* and *what changes for whom*.

| Criterion | Band | Points |
|---|---|---|
| Problem Statement is specific — names the actor, the current behaviour, and the cost — not vague or aspirational | 0 / 3 / 7 | 7 |
| Recommendation states the accepted trade-off in one sentence (*"we accept X to get Y"*) | 0 / 3 / 7 | 7 |
| Open Questions are tagged `now` (blocks the PRD) vs `during-build` (acceptable to defer) | 0 / 3 / 6 | 6 |

**Red flag for this dimension:** a Problem Statement that begins *"users are frustrated with …"* without a specific behaviour and cost scores `0` on the first criterion.

## How to iterate

Emit the score block in this shape before Phase 9's HARD GATE:

```
PRD score: <total>/100  (target ≥ 90)
  Functional completeness  : <n>/30   — <1-line note on weakest criterion>
  Technical soundness      : <n>/25   — <...>
  Implementation readiness : <n>/25   — <...>
  Business impact          : <n>/20   — <...>
```

If `total < 90`, do not proceed to the HARD GATE. Instead:

1. Name the lowest-scoring dimension and the specific criterion that dragged it down.
2. Loop back to the phase that produced it (Phase 3 for framing, Phase 4 for options, Phase 6 for stories / not-doing).
3. Re-emit the Phase 8 checklist AND a fresh score block.
4. Repeat until `total ≥ 90`, then gate.

Maximum three iterations. If the score still lands below 90 after three passes, STOP and surface the gap to the user — *"Score stuck at 84/100 on Functional completeness (no measurable metrics on Goal #2); need your input on how to measure this before I can proceed."* A stuck score is a signal, not a step to skip.

## Anti-patterns in scoring

- **Generous scoring to clear the bar.** Inflating numbers to hit 90 defeats the purpose. Score as the next reader would.
- **Scoring before the Phase 8 checklist.** The checklist catches structural defects; the rubric catches subtler quality gaps. Running the rubric first lets structural defects inflate the score.
- **Applying to Lightweight / Standard.** The rubric is calibrated for Deep-scope ambition. Forcing a small feature PRD through it produces false "failure" scores.
- **Treating the rubric as the only quality check.** It is a last-mile gate. The clarifying questions in Phase 1, the codebase pre-scan in Phase 2, and the council debate in Phase 4 do the real quality work; the rubric just confirms nothing slipped.
