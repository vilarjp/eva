# Anti-bloat — keep the SPEC interpretable, not copy-pasteable

A SPEC is a contract for an implementer to **interpret**, not a draft of the code for them to paste. This reference gives a single sharp test for bloat, the fix patterns for the three most common violators, and a small set of before/after examples.

The rule exists because every line of pseudo-implementation inside a SPEC steals durability from it — the SPEC survives refactors only when it describes *what must be true*, not *how to type it out*. An implementer copying from the SPEC has turned the SPEC into lock-in; an implementer interpreting from the SPEC is doing their job.

## The spec-vs-implementation test

Ask this question of any code block, function signature, or long data-shape literal in the draft SPEC:

> *"If the implementer copies this verbatim rather than interpreting it, does the SPEC still describe the decision correctly?"*

- If the answer is **YES**, the content is implementation disguised as specification. Replace it with a behavioural description + a table, or cut it.
- If the answer is **NO** — the block is the only way to convey the contract (e.g. a canonical error-response JSON shape, a wire-protocol sketch) — keep it, but trim to the minimum that captures the contract.

A shorter version of the test, useful at glance-depth: **code blocks longer than ~15 lines inside a SPEC are almost always bloat.** The exceptions are rare and named (see "When a code block earns its place" below).

## The three common violators

### 1. Function signatures posing as interfaces

**Symptom.** A TypeScript interface with five methods, each with full parameter lists and return types, 20+ lines long.

**Why it bloats.** The SPEC is now coupled to the exact parameter order, exact types, and exact method names. A refactor that renames one method invalidates the SPEC.

**Fix.** Replace with an **interface responsibility table**: Name · Responsibility · Inputs (at semantic level) · Outputs (at semantic level) · Error modes. The contract survives the refactor; the specific signature lives in the code.

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

**After (interpretable):**

| Operation | Responsibility | Inputs (semantic) | Outputs (semantic) | Error modes |
|-----------|----------------|-------------------|--------------------|-----------| 
| capture | Charge an authorised order | order id, amount, idempotency key | captured-charge reference or typed failure | insufficient funds, expired auth, duplicate idempotency key |
| refund | Reverse a captured charge in whole or part | charge id, amount, reason, idempotency key | refund reference or typed failure | amount exceeds capture, charge not capturable, duplicate idempotency key |
| void | Cancel an uncaptured authorisation | charge id, idempotency key | void reference or typed failure | charge already captured, duplicate idempotency key |
| query | Read current status of a charge | charge id | charge status | charge unknown |
| webhook | Verify and dispatch provider callback | signature, body | dispatch outcome | signature mismatch, body malformed, unknown event type |

Every operation is idempotent on its `idempotencyKey`. All error modes are discriminated-union failures; no exceptions cross the module boundary.

### 2. Data-model literals instead of invariants

**Symptom.** A 30-line `Order` type with every field, every subtype, every enum member, every nullable marker.

**Why it bloats.** The schema belongs in the code. The SPEC should name the **entities**, their **relationships**, and their **invariants** — the shape survives.

**Fix.** Describe the entity in one paragraph, list the invariants as bullets, and show only the fields that carry non-obvious semantics (discriminators, foreign keys, immutability, computed). Let field-level trivia live in the migration or the schema file.

**Before (bloat):**
```ts
type Order = {
  id: string;
  customerId: string;
  lineItems: Array<{
    sku: string;
    quantity: number;
    unitPrice: { amount: number; currency: string };
    discounts?: Array<{ code: string; amount: { amount: number; currency: string } }>;
  }>;
  shippingAddress: Address;
  billingAddress: Address;
  status: 'draft' | 'submitted' | 'paid' | 'shipped' | 'cancelled';
  submittedAt?: string;
  paidAt?: string;
  shippedAt?: string;
  cancelledAt?: string;
  cancelledReason?: string;
  total: { amount: number; currency: string };
  createdAt: string;
  updatedAt: string;
};
```

**After (interpretable):**

*Order* represents a customer's intent to purchase zero or more line items at captured prices. It moves through a five-state lifecycle (`draft → submitted → paid → shipped → cancelled`) with a strictly monotonic timestamp per transition.

Invariants:
- `status` transitions are one-way except `submitted → cancelled` and `paid → cancelled`; `shipped` is terminal.
- Every timestamp field mirrors its status: an `Order` with `status = paid` MUST have a non-null `paidAt` and a non-null `submittedAt`.
- `total` is a computed field stored for auditability; it MUST equal `sum(lineItems.unitPrice × quantity) − sum(discounts)` at submission time and MUST NOT change after `status = submitted`.
- `customerId` is a foreign key to `Customer`; an `Order` cannot be created for a non-existent customer.

Non-obvious fields: `cancelledReason` is required when `status = cancelled` and is surfaced to the customer verbatim. `currency` is the same for every line item in an order; mixed-currency orders are rejected at submission.

The schema file is the source of truth for every field.

### 3. Worked examples instead of acceptance criteria

**Symptom.** A 40-line "example flow" that walks through a user's happy path with literal values, function calls, and log output.

**Why it bloats.** A worked example reads like a scenario but is actually a plausible implementation sketch. Implementers copy it, then the SPEC tracks the implementation instead of driving it.

**Fix.** Replace with two to five **acceptance criteria** per flow, each phrased as a post-condition an implementer must make true.

**Before (bloat):**
```
User flow: repeat 1-tap checkout

1. User opens the checkout page.
2. The page calls GET /api/checkout/session with the customer's session cookie.
3. Response is { sessionId: "cs_abc", defaultCard: "card_xyz", defaultAddress: {...} }.
4. The page renders a "Pay with ****1234" button.
5. User clicks the button.
6. The page calls POST /api/checkout/capture with { sessionId: "cs_abc" }.
7. On 200, the page navigates to /checkout/thanks.
8. On 409, the page shows "This order was already placed."
```

**After (interpretable):**

*Acceptance criteria — repeat 1-tap checkout:*
- A returning customer with a default card and default address lands on checkout with a single `Pay with …1234` action and no required input.
- The action captures against the customer's existing authorisation without a second payment-form render.
- A double-click (or a stale-tab retry) produces exactly one charge — the second call receives a `409` equivalent and the UI shows an "already placed" state without a new row in the orders table.
- A missing default card or missing default address degrades to the full checkout form, not an error page.
- Every capture call carries an idempotency key derived from the session, not the click event.

### When a code block earns its place

A code block belongs in a SPEC only when **the shape itself IS the contract** — the reader cannot interpret it from prose. The canonical cases:

- **Wire-protocol errors** — the exact JSON shape a client parses, because every byte matters. Keep it, but show only the discriminant and one field per branch — never a full OpenAPI dump.
- **A canonical error envelope** — the one shape every error in the system conforms to. One example, not a catalogue.
- **A data-migration stub** — the minimum SQL or DDL that conveys the migration shape (additive vs destructive, default vs backfill). One statement, not the full script.
- **A state-machine literal** — when prose would duplicate a diagram. Usually prefer the diagram.

In every case, the block should be ≤15 lines and the prose around it should say **why** this shape and **what** invariant it encodes.

## How to apply the test during drafting

While drafting Phase 4 (architecture), Phase 5 (decisions), and Phase 6 (data model + API contracts), scan every block that crosses ~15 lines. For each, run the spec-vs-implementation test. One of three outcomes:

1. **Keep as-is** — the block is the contract, it cannot be paraphrased, and it is already minimal.
2. **Replace with a table + invariants** — the most common outcome; almost every 30-line interface or 40-line data type collapses to 6-10 lines of table + invariants.
3. **Replace with acceptance criteria** — the right move when the block was really a worked example in disguise.

Then pass over the trimmed draft once more with one question: *"Has anything I cut left an ambiguity the implementer cannot resolve without a new question?"* If yes, restore the one line that disambiguates — usually an invariant, not the code. If no, you are done.

## Anti-patterns

- **"I'll just leave the signature in for clarity."** Clarity is the disease pretending to be the cure. The signature is clear; the contract is not.
- **"The implementer wants to see the types."** The implementer wants the semantics. Types are how they will encode the semantics.
- **"A worked example makes it concrete."** A worked example makes it lock-in. Acceptance criteria make it concrete AND durable.
- **"The SPEC is long because the feature is complex."** Most long SPECs are long because they copy-pasted their way through complexity. A complex feature with a short SPEC is almost always a better-designed feature.
