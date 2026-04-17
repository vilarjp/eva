# Commit Message Style — commit-push

The draft-approve-commit loop lives or dies by the message. A great message answers the three questions a future reviewer cannot google: **why now**, **what is the invariant**, and **what would a regression look like**.

## Convention precedence

Pick the convention in this strict order — stop at the first hit:

1. **Declared convention in repo instructions.** Read (if present, already in session context): `CLAUDE.md`, `.claude/CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `.github/CONTRIBUTING.md`. If any declares a commit format, follow it verbatim. Project instructions outrank everything else.
2. **Commitlint / pre-commit config.** If `.commitlintrc*`, `commitlint.config.*`, or a `commitlint` section in `package.json` exists, its rules are the convention. Obey type whitelists, scope rules, subject-length limits, and case rules exactly.
3. **Last 10 commits.** `git log --oneline -10`. If ≥7 of 10 follow a clear pattern (Conventional Commits, `TICKET-1234: subject`, `[module] subject`, emoji prefix, etc.), match it. Do not mix styles in the same repo even if the conventional style would be "cleaner."
4. **Conventional Commits default.** `<type>(<scope>): <subject>`. Use when none of the above provides guidance.

## Conventional Commits — the default grammar

### Types

| Type | Meaning | When to use |
|------|---------|-------------|
| `feat` | A new user-visible capability | New endpoint, new UI feature, new CLI flag, new public function |
| `fix` | Corrects a defect in existing behavior | Bug fix, regression fix, incorrect validation |
| `refactor` | Restructuring without behavior change | Extract helper, rename internal, split file, simplify expression |
| `perf` | Performance improvement with no other observable change | Added index, memoization, algorithm swap |
| `test` | Adds or changes tests only | New unit tests, coverage fill, test infrastructure |
| `docs` | Documentation only | README, JSDoc, docstrings, markdown guides, inline `//`-comments without code changes |
| `chore` | Maintenance that doesn't fit elsewhere | Dependency bumps, tool config, housekeeping |
| `style` | Formatting / whitespace only | `gofmt`, `prettier` runs; never mix with behavior |
| `ci` | CI/CD configuration | GitHub Actions, CircleCI, GitLab CI workflows |
| `build` | Build system / package manifests | Webpack, Vite, Rollup, `pyproject.toml` build targets |

### Scopes

Scope is optional but recommended. Use the module / directory name the change lives in: `auth`, `billing`, `api`, `schema`, `ui/checkout`. Don't invent scopes — if the repo's recent commits use `(api)`, use `(api)`.

### Subject

- Imperative mood: "add X", "fix Y", "extract Z". Never "added X" / "adds X" / "adding X".
- Lowercase first character (unless repo convention differs).
- No trailing period.
- ≤72 characters (hard limit). GitHub truncates at ~50 in lists, so aim for ≤50 when natural.
- Describes the observable outcome, not the mechanism: `feat(auth): rotate refresh tokens on every access-token refresh` > `feat(auth): call rotateToken() in refresh handler`.

### Body

Required for `feat` and `fix` on Standard/Deep. Optional otherwise but encouraged.

Explain **why**, not **what**. The diff shows *what*. The body's job is the motivation, the invariant, the cost of not doing it, and the alternatives that were considered and rejected.

Line-wrap at 72 characters. Separate body from subject with one blank line. Separate body paragraphs with one blank line.

### Testing scenarios (mandatory for feat/fix on Standard/Deep)

A short before/after block that tells a reviewer how to verify the change behaviorally. Not code — prose.

```
Testing scenarios:

Valid rotation
Before: refresh token reused indefinitely after access-token refresh.
After:  refresh token is replaced; the old token is rejected with 401.

Rotation without upstream
Before: refresh failed silently when the IdP was unreachable.
After:  refresh surfaces a 503 with a user-safe message and a retry header.
```

Use 1–3 scenarios per feat/fix commit. If a scenario needs >3 lines each, it probably belongs in an ADR or a SPEC — reference that doc instead.

### References

Add these at the bottom of the body when applicable:

- `See docs/<DATE>-<slug>/SPEC.md#phase-2` — points at the SPEC tracer-bullet phase this commit implements.
- `Fixes root cause in DIAGNOSIS.md` — points at the diagnosis this commit resolves.
- `Closes #123` / `Fixes #123` — closes a GitHub issue on merge.
- `Refs ABC-456` — Jira ticket.
- `BREAKING CHANGE:` — one-line prefix then a paragraph; triggers a major-version bump in Conventional Commits tooling.

### Footers to AVOID

- `Signed-off-by:` — only if the repo uses DCO (Developer Certificate of Origin). Check `.github/CONTRIBUTING.md` first.
- `Co-Authored-By:` — do NOT inject the skill's name or the model's name. If the human pair-programmed with another human, they'll add the footer themselves.
- Promotional generator footers (`🤖 Generated with ...`) — do NOT add unless the repo's recent commits already use them. Most production repos prefer clean history.
- Emoji in subject (`:sparkles:`, `✨ `) — only if the last 10 commits use them.

## Full worked examples

### feat with testing scenarios

```
feat(auth): rotate refresh tokens on every access-token refresh

Refresh tokens previously stayed valid until their expiry — a leaked
token worked indefinitely. Rotate on every access-token refresh so a
leak is valuable for at most one round-trip. The old token is written
to a short-lived denylist so a concurrent refresh attempt is rejected
rather than silently racing.

Testing scenarios:

Valid rotation
Before: refresh token reused indefinitely after access-token refresh.
After:  refresh token is replaced; the old token is rejected with 401.

Concurrent refresh race
Before: two concurrent refreshes both succeed, leaving two valid tokens.
After:  the second refresh hits the denylist and returns 401.

See docs/2026-04-17-refresh-rotation/SPEC.md#phase-2
```

### fix with root-cause reference

```
fix(billing): prorate half-day renewals instead of charging a full day

A subscription renewed <24h after the previous bill was charged a full
day's rate because the prorate check compared wall-clock days instead
of elapsed seconds. Switch to elapsed-seconds and document the 86400
boundary in the helper comment.

Testing scenarios:

Half-day renewal
Before: $10/day plan charged $10 for a 12h renewal.
After:  $10/day plan charged $5 for a 12h renewal.

Exact-day renewal
Before: 24h renewal charged $10 (correct, coincidence).
After:  24h renewal still charges $10 (explicit).

Fixes root cause in docs/2026-04-14-prorate-bug/DIAGNOSIS.md.
Closes #847
```

### refactor (no testing scenarios required)

```
refactor(validators): extract shared postal-code normalizer

Three validators normalized postal codes inline with slightly different
rules (NL trimmed whitespace, UK uppercased, US did neither). Extract a
single `normalizePostal(country)` helper that encodes each country's
rule once so future validators don't drift.

No behavior change — each validator's output matches the previous
implementation for every input in the fixture set.
```

### chore with chore-exemption audit (Deep tier)

```
chore(deps): bump eslint and typescript

Routine patch bumps. No behavior change.

Chore exemptions:
  - .eslintrc.json — tooling config, no runtime behavior
  - package.json — dependency version bump only
  - pnpm-lock.yaml — regenerated from manifest
```

### Respecting an existing unusual convention

If `git log --oneline -10` shows:
```
TICKET-881: update retry policy
TICKET-877: new usage endpoint
TICKET-874: fix stale token race
```

Match it. Draft:
```
TICKET-902: rotate refresh tokens on every access-token refresh

<WHY body>

<Testing scenarios>
```

Do not "fix" the convention to Conventional Commits. The repo has already chosen.

## Subject-line anti-patterns

| Bad | Why | Fix |
|-----|-----|-----|
| `Update utils.ts` | Says "what" (file), not "why" or even "what happened" | `refactor(utils): extract path-join helper` |
| `Fix bug` | Zero information | `fix(cart): prevent duplicate coupon on re-apply` |
| `WIP` | Not a commit message — a mental state | Squash or rewrite before pushing; skill should refuse. |
| `more changes` / `final` / `oops` | Save-point noise | Rewrite with real subject before push. |
| `feat: adds user form` | Wrong mood (non-imperative) | `feat: add user form` |
| `feat: Add User Form.` | Wrong case + trailing period | `feat: add user form` |
| `[billing] fix the thing` | Doesn't match conventional grammar + vague | `fix(billing): prorate half-day renewals` |

## Body anti-patterns

- **Narrating the diff.** "This PR changes `foo.ts` by adding a new parameter and updating the tests." The reader already knows — they're reading the diff. The body should answer "why is this parameter the right shape?"
- **Mentioning the ticket and nothing else.** "Implements #123." The ticket can disappear. The commit is history. Write a WHY paragraph and *also* reference the ticket.
- **Implementation detail.** "The new helper uses a Map instead of an object because Map preserves insertion order." This is comment-in-code territory, not commit-body territory.
- **Apologies.** "Sorry this is so big." Not your past self's problem — write a clean message and the diff speaks for itself.

## Reader contract

A commit message is read cold, months later, by someone using `git log --grep` or `git bisect` or a blame walker. They have:
- The subject line (first).
- The body (if they scroll).
- Nothing else from this conversation.

Every line of the message must carry weight under cold reading. When a sentence feels conversational ("As discussed with João, we decided to..."), cut it — the commit doesn't know who João is.
