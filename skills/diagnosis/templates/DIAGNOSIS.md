---
name: diagnosis
date: "{{DATE}}"
status: "{{draft | approved}}"
topic: "{{short topic — what is broken}}"
severity: "{{TRIVIAL | STANDARD | COMPLEX}}"
scope: "{{LIGHTWEIGHT | STANDARD | DEEP}}"
spec_directory_reuse: {{false | true}}
original_spec_dir: "{{omit unless spec_directory_reuse is true, else: docs/YYYY-MM-DD-<slug>/}}"
issue_ref: "{{omit if none, else: GH#123 | org/repo#123 | ABC-456}}"
---

# Diagnosis: {{topic}}

## Bug

- **Reported:** {{plain-language account}}
- **Expected vs actual:** {{what should happen → what happens}}
- **Affected:** {{who — all users / platform X / one customer / internal}}
- **Impact:** {{outage / data / workflow / cosmetic}}

## Environment

| Runtime | OS / platform | Release | Env | Reproducibility |
|---------|---------------|---------|-----|-----------------|
| {{...}} | {{...}}       | {{...}} | {{local/staging/prod}} | {{every time / ~X% / once / unable}} |

## Investigation trail

| # | What I searched / ran                   | What I found                                     |
|---|-----------------------------------------|--------------------------------------------------|
| 1 | {{e.g. `git log src/checkout/`}}        | {{e.g. recent refactor on 2026-04-10}}           |
| 2 | {{e.g. `Grep "applyCoupon"`}}           | {{e.g. 4 call sites; one doesn't pass context}}  |
| 3 | {{e.g. `Read src/checkout/coupon.ts:40-90`}} | {{e.g. null path when cart.items is empty}}  |

## Hypotheses

Minimum 3 **structurally different** (missing null check ≠ race ≠ wrong
contract). TRIVIAL severity may have 1-2 with a `why fewer` note.

| # | Hypothesis (what & where)                     | Evidence FOR     | Evidence AGAINST | Verdict                |
|---|-----------------------------------------------|------------------|------------------|------------------------|
| 1 | {{e.g. `coupon.ts:62` dereferences `cart.items` before promise resolves}} | {{stack trace frame; empty-cart repro}} | {{same path green in tests}} | CONFIRMED / REJECTED / INCONCLUSIVE |
| 2 | {{structurally different}}                    | {{...}}          | {{...}}          | {{...}}                |
| 3 | {{structurally different}}                    | {{...}}          | {{...}}          | {{...}}                |

**Why fewer than 3 (only if TRIVIAL):** {{justification}}

## Causal chain

From trigger → symptom. No gaps. "Somehow X leads to Y" is a gap — keep going.
Mark uncertain links with a prediction that a test / read confirmed.

1. **Trigger:** {{e.g. user taps "Apply coupon" while cart is still loading}}
2. {{step — code location or event}}
3. {{step — code location or event}}
   - *Prediction (uncertain):* {{"If this is the cause, X should reproduce" —
     confirmed by {{test|read}} at {{path:line}}}}
4. **Symptom:** {{observed failure}}

## Root cause

{{1-3 sentences. Cite file:line. Name the violated invariant. Trace back to
one CONFIRMED hypothesis and a specific causal-chain link.}}

## Reproduction test

- **File:** `{{path}}`
- **Test:** `{{descriptive name with bug ref, e.g. "...(GH#1234)"}}`
- **Status:** {{RED (fails as expected) | UNTESTABLE — see below}}

```
$ {{exact command}}
{{literal failing output with file:line and expected-vs-actual}}
```

**UNTESTABLE justification (only when status is UNTESTABLE):** {{why no
automated test is possible + manual reproduction steps.}}

## Hotspots & fix

| File : line                         | Why focus here                      |
|-------------------------------------|-------------------------------------|
| `{{path:line}}`                     | {{primary root-cause site}}         |
| `{{path:line}}`                     | {{related invariant}}               |

**Approach:** {{one paragraph — minimal fix addressing root cause, not
symptom. Name the specific change and the files it touches.}}

**Defense-in-depth (optional):** {{if invalid data flows through multiple
layers — see references/investigation-techniques.md.}}

## Concerns

- **Risk of the fix:** {{what could regress}}
- **Low-confidence areas:** {{where evidence was thin}}
- **Watch after ship:** {{error rates, user reports, metrics}}

<!--
Optional sections — include ONLY when they add signal:
- Pattern analysis — working examples vs broken, violated assumptions
- Diagram (mermaid) — mandatory on Deep; sequence / flowchart / state
- Architectural concern — only when same gap exists in N other files (COMPLEX)
- Relationship to Original Spec — only when spec_directory_reuse: true

Cut any section above if empty. See _shared/artifact-compactness.md.
-->
