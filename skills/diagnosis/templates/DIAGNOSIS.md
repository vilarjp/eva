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

## Bug Description

**What was reported:** {{plain-language account of the observed behavior}}

**Expected:** {{what should have happened}}

**Actual:** {{what actually happened}}

**Who is affected:** {{all users / users on X platform / one customer / internal only / etc.}}

**Impact:** {{outage severity, data correctness, blocked workflow, cosmetic, etc.}}

## Environment

| Dimension | Value |
|-----------|-------|
| Runtime | {{Node 20.11, Bun 1.1, Ruby 3.3, Python 3.12, …}} |
| OS / platform | {{macOS 14, Linux (Debian 12), iOS 17, Android 14, …}} |
| Release / commit | {{v1.4.3 / git SHA}} |
| Environment | {{local / staging / production}} |
| Reproducibility | {{every time / intermittent (~X%) / one-off / unable to reproduce here}} |

## Investigation Trail

Record every search, read, and command that informed the diagnosis.

| Step | What I searched / ran | What I found |
|------|----------------------|--------------|
| 1 | {{e.g. `git log --oneline -20 src/checkout/`}} | {{e.g. recent refactor of CouponApply on 2026-04-10}} |
| 2 | {{e.g. `Grep "applyCoupon" --type ts`}} | {{e.g. 4 call sites; only one passes the cart context}} |
| 3 | {{e.g. `Read src/checkout/coupon.ts:40-90`}} | {{e.g. null dereference path when cart.items is empty}} |

## Pattern Analysis

**Working example(s) found:** {{paths to analogous code that works, or "no analogue found — <reason>"}}

**Key differences between working and broken paths:**

- {{difference 1 — be specific about lines/functions}}
- {{difference 2}}
- {{difference 3}}

**Assumptions the working path makes that the broken path violates:**

- {{assumption 1}}
- {{assumption 2}}

## Hypotheses

Minimum 3, **structurally different** (missing null check ≠ race condition ≠ wrong API contract). Exception: TRIVIAL-severity bugs may document 1-2 with a `why fewer` justification.

| # | Hypothesis (what & where) | Evidence FOR | Evidence AGAINST | Verdict |
|---|--------------------------|--------------|------------------|---------|
| 1 | {{e.g. `applyCoupon` dereferences `cart.items` before the promise resolves — src/checkout/coupon.ts:62}} | {{e.g. stack trace top frame; reproduces when cart is empty}} | {{e.g. same code path is exercised in tests successfully}} | {{CONFIRMED / REJECTED / INCONCLUSIVE}} |
| 2 | {{structurally different hypothesis}} | {{…}} | {{…}} | {{…}} |
| 3 | {{structurally different hypothesis}} | {{…}} | {{…}} | {{…}} |

**Why fewer than 3 (only if TRIVIAL):** {{justification}}

## Causal Chain

Full chain from original trigger → observed symptom, each step named with specific code (file:line) or a specific event. **No gaps.** "Somehow X leads to Y" is a gap — keep going.

1. **Trigger:** {{e.g. User clicks "Apply coupon" while cart is still loading (StartLoading state)}}
2. **Step:** {{e.g. `CouponInput.onSubmit` dispatches `applyCoupon(code)` — src/components/CouponInput.tsx:48}}
3. **Step:** {{e.g. `applyCoupon` reads `cart.items` from the store synchronously — src/checkout/coupon.ts:62}}
   - **Prediction (uncertain link):** {{e.g. "If this is the cause, applying a coupon before cart-load should fail in the integration test too" — test confirmed: integration test at tests/checkout/coupon.integration.ts:24 reproduces the failure}}
4. **Step:** {{…}}
5. **Symptom:** {{observed failure}}

Mark any link without a prediction as OBVIOUS and explain in one sentence why (e.g. "explicit null dereference on a typed field").

## Root Cause

{{1-3 sentences. Cite file:line. Name the violated invariant. The explanation must trace back to one CONFIRMED hypothesis and a specific link in the causal chain.}}

## Reproduction Test

- **File:** `{{path/to/test-file}}`
- **Test name:** `{{descriptive — include the bug reference, e.g. "applyCoupon throws when cart is still loading (GH#1234)"}}`
- **Status:** {{RED (fails as expected) | UNTESTABLE — see justification below}}

**RED proof (REQUIRED when status is RED).** Literal command and literal output. No summaries.

```
$ {{exact command, e.g. pnpm test tests/checkout/coupon.test.ts}}
{{stdout + stderr showing the failing assertion with file:line and expected-vs-actual}}
```

**UNTESTABLE justification (REQUIRED when status is UNTESTABLE).** {{One paragraph explaining exactly why the bug cannot be captured in an automated test — e.g. requires external infrastructure, timing at a scale we can't reproduce, visual regression — plus manual reproduction steps.}}

## Hotspots

Where the implementer should focus.

| File : Function / Line | Why focus here |
|------------------------|----------------|
| {{src/checkout/coupon.ts:applyCoupon}} | {{primary site of root cause}} |
| {{src/store/cart.ts:setCartItems}} | {{sets the state that is read too early — related invariant}} |
| {{src/components/CouponInput.tsx:onSubmit}} | {{trigger path — may want guard here too}} |

## Suggested Fix

**Approach:** {{brief paragraph describing the minimal fix — addresses root cause, not symptom. Name the specific change.}}

**Files touched:** {{file list — keep minimal}}

**Defense-in-depth (optional):** {{if invalid data flows through multiple layers, recommend layered validation — entry, business logic, environment guards, debug instrumentation. See references/investigation-techniques.md.}}

**Architectural concern (if COMPLEX):** {{name the pattern if the same gap exists in N other files, or if each fix reveals a new symptom. Do not paper over it — surface it for the user to decide.}}

## Concerns

- **Risks of the suggested fix:** {{what could regress, what could be subtly wrong}}
- **Low-confidence areas:** {{where evidence was thin or inconclusive}}
- **What to watch after the fix ships:** {{error rates, user reports, metrics}}

## Diagram

_Required for Deep scope. Optional for Standard when structure would clarify. Prefer sequence diagrams for cross-component bugs, flowcharts for call-chain backward-traces, state diagrams for state-machine bugs._

```mermaid
{{mermaid block — e.g. sequenceDiagram / flowchart / stateDiagram-v2}}
```

## Relationship to Original Spec

_Include ONLY when `spec_directory_reuse: true`. Otherwise delete this section._

**Related spec directory:** {{original_spec_dir}}

**How the bug relates to the original feature:**

- {{e.g. the bug is a gap in the original plan — the cart-loading state wasn't considered at checkout}}
- {{e.g. an edge case below the "covered" bar in the original Test Strategy}}
- {{e.g. a regression introduced by a later refactor}}

**Forward-reference added to:** {{SPEC.md | PRD.md}} — `## Post-Release Bug Fix <DATE>` section.
