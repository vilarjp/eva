# Scope Tiers — commit-push

Scope tunes the pre-commit gate and the split proposal. Classify once, re-classify when new evidence appears.

## Tiers at a glance

| Tier | Signals (any ONE triggers) | Pre-commit checks | Split policy | Typical turn count |
|------|---------------------------|-------------------|--------------|---------------------|
| **Lightweight** | 1–2 files changed AND <50 added+deleted lines AND a single coherent concern | Secrets scan + protected-branch refusal + debug-artifact scan | Single commit. Skip the split question. | 2–3 turns end-to-end |
| **Standard** | 3–10 files OR 50–500 lines OR a feature increment / bug fix / single-module refactor | Lightweight checks + full test suite + lint/formatter | Single commit usually; propose a 2-commit split only when files cluster by distinct concern (e.g., feature vs test infra). | 3–5 turns end-to-end |
| **Deep** | 10+ files OR >500 lines OR mixed concerns (feat + refactor, schema + feat) OR touches trust boundaries / migrations / concurrency / cron+queue consumers / render paths | Standard checks + scope-vs-diff audit + test-coverage-for-code-changes + chore-exemption audit | Propose 2–3 commit split by concern; human approves the grouping. | 4–7 turns end-to-end |

## Signal definitions

**Files changed.** `git diff --stat HEAD | wc -l` minus the `N files changed` footer line.

**Lines changed.** Sum of `+` and `-` lines in `git diff HEAD --numstat`.

**Coherent concern.** Every file change answers a single "why did this change" sentence. A PRD clarification is one concern. A PRD clarification + an unrelated logger rename is two concerns.

**Mixed concerns.** The diff requires two different Conventional Commits `type`s to describe honestly. `feat` + `refactor`, `fix` + `test`, `chore` + `feat`. Mixed concerns force Deep tier even when the file count is low.

**Trust boundary.** Files under `auth/`, `authn/`, `authz/`, `session/`, `crypto/`, `secret*/`, `webhook*/`, anything that handles externally-supplied input, anything that constructs a SQL / shell / HTML string from user-controlled data.

**Migration / concurrency.** Any changes to `migrations/`, `*.sql` schema files, message consumers, cron files, lock acquisition, retry loops, connection-pool config, rate-limiters, or transactional boundaries.

## Re-classification triggers

The initial classification is a hypothesis. Escalate (never demote) when Phase 3 or Phase 4 uncovers:

- **During secrets scan** — an unexpected config file change (`.env.example`, `docker-compose.yml`, `kubernetes/*.yaml`) that touches secrets-in-shape → escalate to Deep and re-run the gate.
- **During debug-artifact scan** — >3 debug artifacts across >2 files suggests a WIP commit that should either be cleaned up or split → escalate to Standard at minimum.
- **During test run** — if tests that were not expected to cover changed files started failing, that's an invisible coupling → escalate by one tier.
- **During split clustering** — if clusters emerge on what was classified Lightweight, escalate to Standard and propose the split.

## Ceremony that SCALES with tier

**Lightweight** keeps it tight:
- Pre-commit: three fast checks (secrets, protected branch, debug artifacts).
- Split: none.
- Message: the Conventional Commits subject line alone is sufficient for pure-chore / pure-typo changes. A body is appreciated but not required.
- HARD GATE: still enforced, but the checklist is compressed (fewer line items).

**Standard** is the default:
- Full test suite + lint required.
- Message: subject + body mandatory for feat/fix; testing-scenarios block mandatory for feat/fix.
- Split: propose when concerns cluster.

**Deep** adds an audit posture:
- Scope-vs-diff audit (every file classified in-scope / out-of-scope / defer).
- Test-coverage-for-code-changes audit with chore-exemption justification recorded in the commit body.
- Message: WHY body is mandatory even for chore/refactor when >10 files or >500 lines — a reader six months from now needs the reason.
- Split: default to 2–3 commits; single-commit Deep diffs are a smell.

## Worked examples

### Lightweight

- One `.md` typo fix: 1 file, 1 line. Subject only, no split, secrets + protected + debug scans only.
- Rename `MAX_RETRIES` to `RETRY_LIMIT` across 2 files of 8 call sites: 2 files, 8 lines. Subject + one-line body optional.
- Bumping a dependency version in `package.json`: 1 file, 2 lines. `chore(deps): bump <lib> to x.y.z`.

### Standard

- Adding a new validation rule in `validators/` with a new unit test: 2 files, ~80 lines. Test suite + lint required. Single commit: `feat(validators): reject non-numeric postal codes`.
- Refactoring a single module to extract a helper, tests updated: 3 files, ~150 lines. Test suite required. Single commit: `refactor(billing): extract prorate helper`.
- A bug fix that touches 4 files (fix + regression test + docstring update + changelog): 4 files, ~120 lines. Propose a 2-commit split only if the changelog update could land separately without breaking anything.

### Deep

- A feature that adds a route, a service, a migration, and tests: 12 files, ~700 lines. Deep. Split into 2 commits: `feat(schema): add subscriptions table migration` then `feat(billing): subscription renewal route`.
- Re-authoring a token-rotation flow across auth middleware, session store, tests, and docs: 15 files, ~900 lines. Deep. Split into 3: `feat(auth): refresh-token rotation` / `test(auth): refresh-token rotation regression suite` / `docs(auth): document rotation invariants`.
- Cleaning up N call sites across the codebase for a rename: 20+ files, ~300 lines but one concern. Deep by file count. Single commit is acceptable despite Deep — the concern is coherent, the audit burden is cheap. Flag in the checklist: "Deep by file count; single-commit acceptable because one coherent concern."

## Anti-signals (do NOT use these to classify)

- Total lines changed, standalone. A 2-file, 2000-line auto-generated lockfile regeneration is Lightweight for commit-push purposes — the human is not reviewing every line.
- Number of commits previously attempted in the branch. Scope is the diff at invocation time, not history.
- How long the work took. Scope is structural, not temporal.
