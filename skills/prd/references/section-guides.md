# Section guides — how to write each PRD section well

Load this file when a section is failing the Phase 8 self-review checklist, or when you need a concrete example.

## Problem Statement

**Good:** *"Our mobile checkout takes 3 taps and 2 network round-trips. Users on slow connections abandon at 38% after tap 1, vs. 12% on web. The current flow predates the single-tap patterns introduced in the 2025 redesign."*

**Bad:** *"Users are frustrated with checkout."*

The good version names: the observed behavior (3 taps, 2 round-trips), the affected user (mobile, slow connection), the cost (38% abandonment), and the context (predates 2025 patterns). The bad version could be about any product at any time.

**Test:** Could this paragraph be copy-pasted into any other product's PRD without edits? If yes, it's too vague.

## Goals

Each goal must be **measurable or verifiable**. A goal you cannot write an acceptance criterion for is not a goal — it's a wish.

**Good:**
1. Reduce mobile checkout from 3 taps to 1 tap on the default path.
2. Reduce mobile checkout abandonment rate from 38% to ≤ 20% within 30 days of launch.
3. Maintain ≤ 800ms p95 checkout completion latency.

**Bad:**
1. Make checkout better.
2. Improve user experience.
3. Be performant.

**Test:** Can you write a test, a dashboard query, or a shippable demo that proves each goal is met?

## Non-Goals

Non-Goals are **explicit exclusions that prevent scope creep**. They must be plausible things a reader might expect this work to cover, and you are explicitly saying "not here."

**Good:**
1. Desktop checkout flow is unchanged in this PRD.
2. New payment methods are out of scope — we only optimize the existing ones.
3. No changes to the cart / pre-checkout step.

**Bad (these are goals in disguise, not exclusions):**
1. We will not build a bad product.
2. We will not ignore user feedback.

**Test:** If a teammate read only the Goals and not this list, would they reasonably assume the excluded item was included? If yes, it's a real Non-Goal. If no, delete it.

## Options

Options must be **genuinely distinct trade-offs**. Three flavors of the same approach is not three options.

**Good (distinct axes):**
- **A. Client-side single-tap** — embed payment SDK, reduce round-trips. Fast, but ties us to vendor.
- **B. Server-side optimistic checkout** — server speculates intent, reconciles async. Vendor-agnostic, but adds reconciliation complexity.
- **C. Hybrid** — single tap for repeat customers (client-side), two-step for new (server-side). Best UX, most code.

**Bad (all variants of the same thing):**
- A. Refactor checkout.
- B. Rewrite checkout.
- C. Redo checkout.

**Test:** Can you write one sentence per option describing its unique trade-off that would NOT apply to the others? If not, collapse them.

## User Stories

Use the canonical form: **As a `<actor>`, I want `<feature>`, so that `<benefit>`.**

Each story must:
- Name a specific actor (not "the user" — be concrete: "repeat mobile customer", "first-time visitor", "admin")
- Describe a feature in user-observable terms, not implementation
- Explain a benefit that connects to a Goal

**Good:** *As a repeat mobile customer with a saved card, I want to complete checkout in one tap, so that I can finish purchases before my attention drifts.*

**Bad:** *As a user, I want fast checkout, so that checkout is fast.*

## Not-Doing list

The Not-Doing list is **the most valuable section** of the PRD. It records decisions you're consciously accepting.

Format: `<omitted thing> — <why it's omitted>`

**Good:**
- Apple Pay / Google Pay support — out of scope for MVP; re-evaluate after 30-day data.
- A/B test of button placement — we pick one layout on design-team recommendation; revisit if abandonment doesn't move.
- Localization of new copy — ship English first, queue translations for next sprint.

**Bad:**
- Bugs — we will not ship bugs.
- Features — we will not build features that users don't want.

**Test:** Does each item name a specific thing a reasonable reader might have expected, and a reason for omitting it? If not, delete.

## Open Questions

Tag each as `now` (must resolve before finalizing the PRD or before build) or `during-build` (can be settled with a later decision).

**Good:**
- [ ] Is the existing `CheckoutSessionService` reusable or does this require a new service? — `now`
- [ ] Do we need a feature flag or can we ship directly? — `during-build`
- [ ] Which analytics events fire on the new single-tap path? — `during-build`

**Bad:**
- [ ] How do we build this? — too vague to resolve
- [ ] Will it work? — not a question, an anxiety

## Assumptions

Each assumption must have a **verification status**. "I think X is true" becomes either "verified by reading `src/foo.ts`: YES" or "unverified — needs investigation."

**Good:**
1. `CheckoutSessionService` already exposes `completeOrder(userId, cardId)` — verified by reading `src/checkout/session.ts`: YES.
2. Mobile traffic uses the same API as web — verified: NO, needs confirmation from platform team.
3. Payment SDK is approved for client-side use — verified: NO, needs security review.

**Bad:**
1. Everything will be fine.
2. The codebase probably supports this.

Unverified assumptions must surface in the **Open Questions** section if they block the direction.
