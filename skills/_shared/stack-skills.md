# Stack Skills — A shared pointer registry for eva skills

A cross-skill reference that names the external stack-specific skills eva defers to for idiom-level evaluation. Cited by `/audit` during smell inventory, by `/execute` before each slice that touches a registered stack, and by `/code-review` when dispatching Stage 2 reviewers.

This reference does NOT restate the registered skills' rules. It names where each skill lives and when to invoke it, so eva flows compose with externally-maintained stack knowledge instead of duplicating it.

## Why it exists

Eva's smell vocabulary (Fowler + `no-workarounds`) is stack-agnostic by design — it surfaces shape-level debt that holds across React, Node, Go, Rust, or anything else. But shape-level correctness isn't enough: a module can be Fowler-clean and still violate every React rendering rule, every Next.js data-fetching boundary, every Node async-error pattern. External stack skills carry that deeper vocabulary; eva's job is to know when to defer to them so findings name the specific idiom being violated instead of a generic smell.

## Iron Law — Defer, don't duplicate

When a target, diff, or slice falls within a registered stack skill's remit, consult the skill. Do NOT restate its rules inside eva, do NOT embed summaries, do NOT fork the vocabulary. Cite the skill by name in the finding and let the reader follow. If a stack skill is absent from the registry, Claude's general knowledge is the fallback — but a registered skill always takes precedence over general knowledge within its remit.

## Precedence

Repo conventions (`CLAUDE.md`, `AGENTS.md`, adjacent code, ADRs) override stack-skill idioms when they conflict. A repo that deliberately avoids compound components, or deliberately uses synchronous reads at startup, has a deviation the stack skill respects. Outside deliberate deviations, the registered skill is the bar.

## When to cite this reference

| Skill | Phase | Purpose |
|---|---|---|
| `/audit` | Phase 2 — smell inventory | Extend the Fowler + `no-workarounds` catalogues with stack-skill named patterns. A finding may cite `vercel-react: stale-closure in effect` the same way it cites `SWALLOW`. |
| `/execute` | Phase 6 — slice-by-slice | Before writing a slice whose files touch a registered stack, consult the relevant skill for patterns to apply. The TDD discipline is unchanged; the stack skill shapes *what* you write, not *when*. |
| `/code-review` | Phase 4 — Stage 2 parallel fan-out | Pass the registry to every dispatched reviewer so each lane evaluates stack-skill patterns against the diff in its own context. |

Other skills (`/spec`, `/diagnosis`) may cite this reference secondarily when an architecture decision or root-cause hypothesis falls within a registered skill's remit.

## The registry

Each entry names the skill, the triggers that place a target / diff / slice inside its remit, and the domain the skill is authoritative for. Triggers are verbatim from the skill's own description — eva does not invent new trigger logic.

---

### `vercel-react-best-practices`

**Triggers on.** React components, Next.js pages / app-router files, data fetching, bundle optimization, performance work on React or Next.js code.

**Authoritative for.** Hook rules, rendering correctness, effect discipline, data-fetching boundaries, Suspense + streaming, server vs client components, bundle + render cost, key stability in lists.

**Eva citation style.** Findings and slice notes name the skill in the smell column: `vercel-react: <idiom>` — e.g. `vercel-react: stale-closure in effect`, `vercel-react: inline object prop triggers re-render cascade`, `vercel-react: key={index} in dynamic list`.

---

### `vercel-composition-patterns`

**Triggers on.** Component APIs with boolean-prop proliferation, compound-component design, render-prop / context-provider refactors, React 19 API design.

**Authoritative for.** Compound components, render props, slots, children-as-function, context boundary choices, React 19 API changes and their effect on composition.

**Eva citation style.** `vercel-composition: <idiom>` — e.g. `vercel-composition: boolean prop explosion — reshape as compound`, `vercel-composition: context misused for prop drilling`, `vercel-composition: render prop where children-as-function fits`.

---

### `nodejs-backend-patterns`

**Triggers on.** Node.js servers, REST / GraphQL APIs, Express / Fastify middleware, auth flows, DB integration, error handling, microservice boundaries.

**Authoritative for.** Middleware ordering, error-propagation patterns with context, auth session handling, DB transaction + connection-pool patterns, API versioning, input validation at the edge, health / readiness contracts, async discipline on the request path.

**Eva citation style.** `nodejs-backend: <idiom>` — e.g. `nodejs-backend: unhandled promise rejection in handler`, `nodejs-backend: synchronous readFileSync on request path`, `nodejs-backend: missing error-propagation context`.

---

## Finding shape

A stack-skill finding follows the same rules as any other eva finding:

- **Named pattern in the smell column** — `<skill-key>: <idiom>`.
- **Verbatim evidence** — 5-30 word quote with `file:line`.
- **Stated cost** — what the idiom violation actually costs this codebase (incident shape, change-cost, migration pain). *"Violates React best practices"* is not a cost; *"re-renders the entire product grid on every scroll event"* is.
- **Proposed refactor shape** — one sentence, same as any other finding. No code blocks in audit / code-review; code lives in the eventual `/execute` slice.

Severity calibration (P0 / P1 / P2 / P3 for audits; the same bands for reviews) applies unchanged. A stack-skill violation is not automatically P1 — it earns its tier via stated cost, like anything else.

## Anti-patterns in usage

- **Citing without naming the idiom.** *"vercel-react issue here"* is not a finding. Name the specific pattern so the reader can follow to the skill's rule.
- **Duplicating the skill's rules inside eva.** If a finding needs three paragraphs to explain an idiom, the reader should go to the skill, not re-read eva's summary. Cite the skill, quote the violation, state the cost, stop.
- **Treating the registry as exhaustive.** A repo may use a stack outside the registry (Rust, Go, Python). Claude's general knowledge is still the fallback — a finding that names `python-async: unawaited coroutine` in a Python repo is valid even without a registry entry.
- **Firing a stack skill on code that merely mentions the stack.** A comment referencing React inside a Node backend file is not a React target. The trigger has to match the actual code shape.
- **Promoting a stack-skill finding to P0 / P1 by default.** A registered violation still earns its severity from a named cost. A cosmetic hook-rule nit on a cold path is P3.

## Evolving this reference

When a new stack skill is installed globally, add an entry to the registry with: skill name, triggers (verbatim from the skill's own description), authoritative domain, and eva citation style. When a stack skill is uninstalled, remove its row — a stale pointer is worse than no pointer.

The citing skills (`/audit`, `/execute`, `/code-review`) reference this file generically — one registry update propagates.

## References

This reference is cited from:

- `skills/audit/SKILL.md` — Phase 2, smell inventory.
- `skills/execute/SKILL.md` — Phase 6, slice-by-slice implementation.
- `skills/code-review/SKILL.md` — Phase 4, Stage 2 parallel fan-out dispatch.

A finding or slice note that cites a registered skill without naming the idiom, or that duplicates the skill's rules instead of pointing, should be sent back for revision.
