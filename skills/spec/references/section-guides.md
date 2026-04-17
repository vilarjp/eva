# Section guides — how to write each SPEC section well

Load this file when a section is failing the Phase 10 self-review checklist, or when you need a concrete example to calibrate.

## Context

**Good:** *"Per `docs/2026-04-16-faster-checkout/PRD.md` (approved), we are replacing the 3-tap mobile checkout with a 1-tap path for repeat customers. This SPEC covers the backend and mobile client work for the single-tap path; desktop and first-time-buyer flows are unchanged."*

**Bad:** *"We are improving checkout."*

Good Context names: origin (PRD link), the boundary of this SPEC, and what is NOT covered. A reader landing on the SPEC with no other artifact should know what it is and isn't.

## Goals vs Non-Goals (technical)

A technical goal is a **behavior you can test**. A technical non-goal is a real exclusion — a thing a reader might reasonably expect to be covered but isn't.

**Good goals:**
1. `POST /checkout/tap` returns a completed order within p95 ≤ 600ms under 500 rps.
2. A repeat customer with a saved card can complete checkout in a single HTTP request.
3. All payment errors surface to the client with a machine-readable code (`CARD_DECLINED`, `PROCESSOR_TIMEOUT`, etc.), never as HTTP 500.

**Bad goals:**
1. Make checkout better.
2. Improve user experience.
3. Be performant.

**Good non-goals:**
1. Desktop checkout is out of scope — no changes to `src/web/checkout`.
2. New payment methods (Apple Pay, Google Pay) are out of scope — covered by a separate SPEC.
3. Fraud-detection rules are unchanged — we reuse the existing `RiskScorer` as-is.

**Bad non-goals (goals in disguise):**
1. We will not ship bugs.
2. We will not ignore reliability.

**Test:** Can you write a passing test, a production dashboard query, or an observable deploy check for each goal? If not, it's a wish. Can a reader tell from the non-goals exactly what's excluded? If not, they're filler.

## Assumptions

Each assumption must have a **verification status** after the SPEC is done. Assumptions that were verified by reading code say `verified by reading <path>: YES`. Assumptions that couldn't be verified say `NO` and MUST appear in Open Questions if they block the direction.

**Good:**
1. `CheckoutSessionService.completeOrder(userId, cardId)` exists and accepts a saved-card flow — verified by reading `src/checkout/session.ts`: YES.
2. Mobile uses the same `/api/v2` surface as web — verified by reading `src/mobile/api-client.ts`: NO, mobile routes through `/api/v2-mobile`. (Surfaced as Open Question Q-1.)

**Bad:**
1. Everything will be fine.
2. The codebase probably supports this.

## Current → Proposed Architecture

The two architecture blocks must be **parallel** — describe the same modules in their current and future shapes, so the reader can diff them at a glance.

**Good (Current):**
- `CheckoutController` — receives `/checkout/*` requests, coordinates session, payment, inventory, and order modules. Currently 3 sequential round-trips.
- `PaymentService` — tokenizes card, authorizes charge via Stripe. Synchronous.
- `OrderRepository` — persists orders; does NOT currently know about saved cards.

**Good (Proposed):**
- `CheckoutController` — gains a `tapComplete(userId)` path that coordinates the three modules in a single server-side flow. Existing `/checkout/*` endpoints unchanged.
- `PaymentService` — unchanged.
- `OrderRepository` — gains `createFromSavedCard(userId, cardId)` which composes save + order atomically. Schema additive (new optional `source_card_id` column).

Name responsibilities in interface terms, not implementation. "Coordinates the three modules" > "calls `paymentService.authorize()` then `orderRepo.insert()`."

## Mini-ADRs

Use the classic Nygard shape, compressed to fit inline. Sequential IDs (`D-1`, `D-2`).

**Good:**

```
### D-1: Single-request server-side composition over client orchestration

- Context: Mobile network conditions make 3 round-trips unreliable on 3G. Client-side orchestration exposes retry complexity to app code and would require mirror changes in web.
- Decision: Expose `POST /checkout/tap` that performs session lookup, payment authorization, and order creation within a single server-side transaction.
- Alternatives Considered:
  - Client-side orchestration with optimistic UI — rejected: duplicates retry logic in every client, and partial failure states are user-visible.
  - Server-Sent Events progress stream — rejected: doesn't reduce latency (still 3 serial operations), just masks it; extra infra burden.
- Consequences:
  - Easier: mobile + web share the same single-request flow; client code shrinks.
  - Harder: server-side transaction spans an external dependency (Stripe). Mitigation in R-2.
  - Rollback: remove `/checkout/tap`; legacy `/checkout/*` endpoints remain functional.
```

**Bad:**

```
### D-1: Keep using the current approach

- Context: The current approach works.
- Decision: We will keep using it.
- Consequences: It keeps working.
```

No Alternatives Considered = rationalization. "Keep current" is a legitimate decision only when a real alternative was in play and you name why you rejected it.

## Data Model

State the fields, types, invariants (uniqueness, foreign keys, cascade rules), and the **migration shape**.

**Good:**
```sql
ALTER TABLE orders
  ADD COLUMN source_card_id UUID NULL REFERENCES saved_cards(id);

-- Invariants:
-- - source_card_id is NULL for legacy orders; non-NULL for tap-complete orders
-- - saved_cards.id is not cascade-deleted (we need to retain order provenance)
```

*Migration shape:* additive, online, zero downtime. Rollout: migration first, then deploy API with feature flag `checkout.tap_path=false`, flip flag after canary.

**Bad:**

> "We'll add a column for the card ID."

No type, no nullability, no FK, no invariants, no migration shape.

## API Contracts

**Good:**

```
POST /checkout/tap

Input:
  { userId: UUID, cardId: UUID, cartId: UUID }
  all required.

Output (201):
  { orderId: UUID, total: Money, paidAt: ISO8601 }

Errors (consistent with /errors/README.md):
  - 400 INVALID_INPUT  — malformed body
  - 404 CARD_NOT_FOUND — card doesn't exist or doesn't belong to user
  - 409 CART_MUTATED   — cart changed after tap (retry required)
  - 422 CARD_DECLINED  — payment authorization failed
  - 429 RATE_LIMITED   — 5 taps/min per user
  - 502 PROCESSOR_TIMEOUT — Stripe upstream timeout

Idempotency: safe to retry on 502 with same (userId, cartId) within 60s — server dedupes by cartId.

Backwards-compat: additive (new endpoint). Legacy /checkout/* endpoints unchanged.
```

**Bad:**

> "Add a tap endpoint that completes checkout."

No input/output types, no error shape, no idempotency story, no compat note.

### Principles to apply

- **Validate at boundaries** (route handler, form, external-API response). Trust internal contracts.
- **Hyrum's Law** — every observable behavior of a public interface becomes a de facto contract. Be intentional about what you expose; don't leak implementation details (timing, error message text, response ordering).
- **One-Version Rule** — prefer additive evolution over maintaining multiple versions.
- **Input / Output separation** — different shapes for what the caller sends vs what the system returns (server-generated IDs, timestamps).
- **Third-party responses are untrusted data** — validate their shape before using them in any logic, rendering, or decision-making.

## Module Boundaries

This is a **map**, not a task list. One line per module: what it does, whether it's new/modified/absorbed.

**Good:**

```
Created:
- src/checkout/tap-handler.ts — routes POST /checkout/tap and coordinates the three services

Modified:
- src/checkout/controller.ts — delegates to tap-handler for the new path; unchanged for legacy
- src/db/order-repository.ts — adds createFromSavedCard(userId, cardId)

Absorbed / Removed:
- (none)
```

**Bad:**

> "Touch some files in checkout and order."

No paths, no responsibilities, no indication of create/modify/absorb.

## Tracer-bullet Phases

Each phase delivers a **capability** end-to-end. Acceptance criteria are **behavior-focused** and **testable**. **No file names, no function names** — those live in Module Boundaries and in the eventual implementation plan.

**Good:**

```
### Phase 1 — Saved-card reference on orders

Delivers: Existing checkout paths can record which saved card was used without changing behavior. Enables tap-path to reuse order provenance.

Acceptance criteria:
- Orders created via legacy /checkout/* optionally record source_card_id
- Reading an order surfaces source_card_id when present
- No existing test breaks; no client-facing change

Depends on: none
```

**Bad:**

```
### Phase 1 — Database work

Delivers: Add column and update repository.

Acceptance criteria:
- `ALTER TABLE orders ADD COLUMN source_card_id` runs
- `order-repository.ts` gains `createFromSavedCard` method

Depends on: none
```

The bad version describes *steps*, names files, and ties acceptance to implementation locations. It breaks on refactor. The good version describes observable system behavior.

## Test Strategy

State what to test at what level, cite prior art, and call out risky paths. Do not enumerate every test case — that's implementation.

**Good:**

- Unit: payment-error translation (`CARD_DECLINED`, `PROCESSOR_TIMEOUT`) in `tap-handler`, mirroring the pattern in `src/checkout/__tests__/errors.test.ts`.
- Integration: full `POST /checkout/tap` happy path + 4 error paths against a seeded DB + mocked Stripe, following `src/checkout/__tests__/legacy-flow.test.ts`.
- E2E: none at this layer (covered by mobile team's end-to-end suite).
- Coverage focus: 502 retry idempotency (same cartId twice within 60s must dedupe) and concurrent requests on the same cart.

**Bad:**

> "Write tests for the new endpoint."

No level, no prior art, no risky paths.

## Risks & Mitigations

Real risks, real mitigations. "Bugs" is not a risk.

**Good:**

| # | Risk | Impact | Mitigation |
|---|------|--------|------------|
| R-1 | Stripe 502 mid-transaction leaves orphaned `payment_intent` without a matching order | MED | Reconciliation job every 5 min matches `payment_intent` IDs to orders; alerts on orphans > 10 min |
| R-2 | Server-side transaction holds DB connection during Stripe call; pool starvation under load | HIGH | Use short-lived DB connection for session lookup, release before Stripe call, reacquire for insert; load-test at 2× projected rps |
| R-3 | Mobile client retry loops on 502 cause Stripe rate-limit | MED | Server returns `Retry-After` header; client honors it; server-side dedupe covers accidental retries |

**Bad:**

| # | Risk | Impact | Mitigation |
|---|------|--------|------------|
| R-1 | Bugs | ? | Write tests |
| R-2 | Things might break | ? | Be careful |

## Not-Doing

One line per item: `<omitted thing> — <reason>`. Each item must be a thing a reasonable reader might have expected the SPEC to cover.

**Good:**
- Apple Pay / Google Pay support — covered by a separate SPEC; do not lift from current scope.
- Server-Sent Events progress stream — alternative considered in D-1 and rejected; revisit only if client reports confusing latency.
- `/checkout/tap` for first-time-buyer flow — requires full KYC path; out of scope here.

**Bad:**
- Bugs — we will not ship them.
- Unnecessary features — we will not build them.

## Open Questions

Tag each as `now` (blocks SPEC finalization or start of implementation) or `during-build` (can be settled later).

**Good:**
- [ ] Does the mobile client already route through `/api/v2-mobile` for all checkout calls, or only some? — `now`
- [ ] Should `source_card_id` be indexed, or is the foreign-key constraint enough? — `during-build`
- [ ] What is the correct `Retry-After` value — 1s, 5s, or Stripe's recommended value? — `during-build`

**Bad:**
- [ ] How do we build this? — too vague to resolve
- [ ] Will it work? — not a question, an anxiety

## Diagram tips

- **Component diagram** — when the SPEC introduces or restructures modules. Keeps names stable; don't include function names.
- **Sequence diagram** — when the SPEC is about an end-to-end flow across multiple actors (client, server, third-party).
- **State diagram** — when the SPEC is about a lifecycle change (order states, session states, feature-flag phases).
- **Flowchart** — when decision logic dominates.

Keep to one diagram; the SPEC is text-first. A second diagram belongs in an appendix.
