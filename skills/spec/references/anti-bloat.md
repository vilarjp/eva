# Anti-bloat — SPEC-specific rewrite patterns

The universal compactness discipline (section caps, cut rules, durability
guardrails, the three rewrite patterns) lives in
[`../../_shared/artifact-compactness.md`](../../_shared/artifact-compactness.md).
Load that first when the Phase 10 compactness check fails. This file covers
only the SPEC-specific application of the shared rewrite patterns.

A SPEC is a contract for an implementer to **interpret**, not a draft of the
code to paste. Every line of pseudo-implementation inside a SPEC steals
durability from it — the SPEC survives refactors only when it describes *what
must be true*, not *how to type it out*. An implementer copying from the SPEC
has turned the SPEC into lock-in; an implementer interpreting from the SPEC is
doing their job.

## The spec-vs-implementation test

Ask this of any code block, function signature, or long data-shape literal in
the draft SPEC:

> *"If the implementer copies this verbatim rather than interpreting it, does
> the SPEC still describe the decision correctly?"*

- **YES** → implementation disguised as specification. Replace with a
  responsibility table + invariants, or cut.
- **NO** → the block is the only way to convey the contract (canonical error
  envelope, wire-protocol sketch, migration DDL). Keep, but trim to ≤ 15 lines.

Shorter version at glance-depth: **code blocks longer than ~15 lines inside a
SPEC are almost always bloat.**

## Three common SPEC violators

### 1. Function signatures posing as interfaces

**Before (bloat):**

```ts
interface PaymentProcessor {
  capture(orderId: string, amount: Money, idempotencyKey: string): Promise<CaptureResult>;
  refund(chargeId: string, amount: Money, reason: RefundReason, idempotencyKey: string): Promise<RefundResult>;
  void(chargeId: string, idempotencyKey: string): Promise<VoidResult>;
  query(chargeId: string): Promise<ChargeStatus>;
  webhook(signature: string, body: string): Promise<WebhookOutcome>;
}
```

**After (interpretable) — responsibility table:**

| Operation | Responsibility                         | Inputs (semantic)                      | Outputs (semantic)                       | Error modes                                                         |
|-----------|----------------------------------------|-----------------------------------------|-------------------------------------------|---------------------------------------------------------------------|
| capture   | Charge an authorised order             | order id, amount, idempotency key       | captured-charge reference or typed failure | insufficient funds, expired auth, duplicate idempotency key        |
| refund    | Reverse a captured charge              | charge id, amount, reason, idempotency  | refund reference or typed failure          | exceeds capture, not capturable, duplicate idempotency key          |
| void      | Cancel an uncaptured authorisation     | charge id, idempotency                  | void reference or typed failure            | already captured, duplicate idempotency key                         |
| query     | Read current status of a charge        | charge id                               | charge status                              | charge unknown                                                      |
| webhook   | Verify and dispatch provider callback  | signature, body                         | dispatch outcome                           | signature mismatch, body malformed, unknown event type              |

All operations idempotent on the key; all failures are discriminated-union —
no exceptions cross the module boundary.

### 2. Data-model literals instead of invariants

**Before (bloat):**

```ts
type Order = {
  id: string;
  customerId: string;
  lineItems: Array<{ sku: string; quantity: number; unitPrice: Money; discounts?: Array<...> }>;
  shippingAddress: Address;
  billingAddress: Address;
  status: 'draft' | 'submitted' | 'paid' | 'shipped' | 'cancelled';
  submittedAt?: string;
  paidAt?: string;
  shippedAt?: string;
  cancelledAt?: string;
  cancelledReason?: string;
  total: Money;
  createdAt: string;
  updatedAt: string;
};
```

**After (interpretable) — entity paragraph + invariants:**

*Order* represents a customer's intent to purchase zero or more line items at
captured prices. It moves through a five-state lifecycle
(`draft → submitted → paid → shipped → cancelled`) with a monotonic timestamp
per transition.

Invariants:

- Status transitions are one-way except `submitted → cancelled` and
  `paid → cancelled`; `shipped` is terminal.
- Every timestamp mirrors its status — `paid` implies non-null `paidAt` and
  non-null `submittedAt`.
- `total` is computed and stored for auditability; it MUST equal
  `sum(lineItems.unitPrice × quantity) − sum(discounts)` at submission and
  MUST NOT change after `status = submitted`.
- `customerId` is a foreign key to `Customer`.
- `cancelledReason` is required when `status = cancelled` and is surfaced
  verbatim to the customer.
- `currency` is the same for every line item; mixed-currency orders are
  rejected at submission.

The schema file is the source of truth for every field.

### 3. Worked examples instead of acceptance criteria

**Before (bloat):**

```
User flow: repeat 1-tap checkout
1. User opens checkout.
2. Page calls GET /api/checkout/session.
3. Response: { sessionId, defaultCard, defaultAddress }.
4. Page renders "Pay with ****1234".
5. User clicks.
6. Page calls POST /api/checkout/capture.
7. On 200 → /checkout/thanks. On 409 → "already placed".
```

**After (interpretable) — acceptance criteria:**

- A returning customer with a default card and address lands on checkout with
  a single `Pay with …1234` action and no required input.
- The action captures against the existing authorisation without a second
  payment-form render.
- A double-click (or stale-tab retry) produces exactly one charge — the second
  call receives a `409` equivalent and the UI shows an "already placed" state
  without a new row in the orders table.
- A missing default card or address degrades to the full checkout form, not an
  error page.
- Every capture call carries an idempotency key derived from the session, not
  the click event.

### When a code block earns its place in a SPEC

- **Wire-protocol errors** — the exact JSON a client parses. Show the
  discriminant and one field per branch; never a full OpenAPI dump.
- **A canonical error envelope** — one example, not a catalogue.
- **A migration stub** — the minimum SQL/DDL that conveys the shape (additive
  vs destructive, default vs backfill). One statement, not a full script.
- **A state-machine literal** — when prose would duplicate a diagram. Usually
  prefer the diagram.

In every case, ≤ 15 lines, and the prose around the block names **why** this
shape and **what** invariant it encodes.

## Applying the test during drafting

During Phase 4 (architecture), Phase 5 (decisions), Phase 6 (data model + API
contracts), scan every block crossing ~15 lines. Replace with a responsibility
table + invariants, or acceptance criteria — then pass once more: *"Did I cut
an ambiguity the implementer can't resolve without asking?"* If yes, restore
one line (an invariant, not the code). If no, done.
