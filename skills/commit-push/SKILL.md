---
name: commit-push
description: Commit the current working changes and push to a remote feature branch with safety gates — confirms the target branch with the human, runs scope-adaptive pre-commit verification (secrets, tests, lint, scope-vs-diff), and drafts commit messages that match the repo's existing convention. Use when the user says "commit this", "commit and push", "save my changes", "push my changes", "ship the commit", or invokes /commit-push. Refuses protected branches (`main`, `master`, `production`, etc.). Distinct from /pr-feedback (triages reviewer comments, doesn't commit). Never opens a PR, never `git add -A`, never `--force` (only `--force-with-lease` on explicit retry), never `--no-verify`, never commits secrets, never modifies source.
argument-hint: "[optional hint: target branch name, subject-line hint, or blank]"
---

# commit-push — Commit cleanly, push safely, never open the PR

Turn the current working tree (staged + unstaged changes) into one to three Conventional-Commit-style commits on a human-confirmed non-protected feature branch, then fetch-rebase-push to the remote. The durable output is the commits themselves — no new markdown artifact is written unless a `docs/<DATE>-<slug>/` folder is adjacent, in which case a compact `## Commits <DATE>` back-pointer is appended to the primary artifact (EXECUTION.md or DIAGNOSIS.md).

This skill COMMITS AND PUSHES. It does NOT open pull requests, does NOT run `gh pr create`, does NOT modify production source code, does NOT drive the pipeline forward. If the human wants a PR, they will ask in a separate, explicit turn.

## When to use

Trigger this skill when the user:
- says "commit this", "commit it", "commit and push", "save my changes", "save and push"
- says "push my changes", "push this branch", "ship the commit", "get this on the remote"
- invokes `/commit-push` (optionally with a branch-name hint or subject-line hint)
- has just finished work via `/execute` (or by hand) and wants the diff committed + pushed on a feature branch
- wants to stack another commit on the current non-protected feature branch

Do NOT trigger for:
- Opening / updating a pull request (this skill refuses — separate action).
- Writing code (`/execute` does that).
- Running a pre-merge review (`/code-review` does that).
- Rewriting history (`rebase -i`, `filter-branch`, force-pushes as a default) — out of scope; the user owns rewrites.

## The Iron Law

**No commit lands without explicit human confirmation of the branch name in the current conversation, and never on a protected branch.**

A branch name the skill "obviously" could have derived is still a branch name the human did not type. Auto-mode does not override this. `/yolo`, `/auto`, or durable instructions do not override this. The human names or confirms the branch in this turn, or the skill stops.

Protected branch names (refused unconditionally, both as current branch and as target): `main`, `master`, `production`, `prod`, `stable`, `live`, `trunk`, and any branch matching `^release[/_-].*$` or `^release$`.

## Core invariants

1. **Human-gated branch.** Branch is named or confirmed by the human in the current conversation. No silent derivations, no "the obvious feat/ prefix was obvious."
2. **Protected-branch refusal.** If current branch matches the protected list → stop and ask for a feature branch. If requested branch matches the protected list → refuse and ask for a different name. No carve-outs.
3. **No pull requests.** `gh pr create`, `gh pr edit`, web-UI PR creation, or any PR-opening primitive is out of scope. If you feel a PR coming on — STOP. The human will ask separately.
4. **Scope-adaptive pre-commit gate.** Secrets + protected-branch always. Tests + lint + debug-artifact on Standard/Deep. Scope-vs-diff + coverage-for-code-changes on Deep. A blocker at any tier stops the commit — no `--no-verify`, no "I'll fix the lint later."
5. **Explicit staging.** Files are staged by name. `git add -A`, `git add .`, `git add -u` are forbidden — they leak `.env`, build artifacts, IDE droppings. Stage specific paths the skill verified against the diff.
6. **Message draft → human approves.** The skill drafts title + body from the diff and matches the repo's existing convention (recent commits → commitlint config → Conventional Commits default). Human approves, edits, or rewrites before the commit is created. No silent commits.
7. **Up to 3 commits, human-approved split.** When the diff clusters into distinct concerns (feat + refactor, or feat + test-infra), the skill proposes the split and asks the human to approve or collapse. Max 3 per run. Group at the **file level only** — never `git add -p`.
8. **No history rewrites.** No `--force`, no `--force-with-lease` (unless the human asks explicitly in a separate turn after the skill reports a push rejection), no `commit --amend` to a pushed commit, no `rebase -i`.
9. **Rebase-then-push.** Before push: `git fetch origin` and `git pull --rebase origin <upstream-default>`. Abort on non-trivial conflicts and show them — the human resolves. Then `git push -u origin <branch>`. On rejection, fetch + rebase again — never force.
10. **No secrets.** Detected API-key / token / password / `.env` / private-key patterns stop the skill unconditionally, even if the human insists.
11. **Verify before claiming.** Every test-pass / lint-pass / push-success claim comes with literal command output pasted in a fenced block, from a fresh run in the same turn. No "tests passed earlier."
12. **One question per turn. Max 3–4 total.** Usually 2: branch decision + final HARD GATE. Splits and edits add at most one each.
13. **Never writes source code.** The skill stages, commits, pushes. If the pre-commit gate demands a code change (missing test, lint error), STOP and hand back to the human or `/execute` — do not fix the code yourself.
14. **Idempotent-ish resume.** If a previous commit already landed this session but the push failed, the skill resumes at rebase + push and does not re-stage/re-commit the same content.

## Pre-flight (MANDATORY)

Before anything else, in this order. One fenced output block per step, compact.

1. **Today's date.** `date +%Y-%m-%d`. Store as `<DATE>`.
2. **Git state snapshot.** In one Bash call:
   ```bash
   git status --short --branch
   git diff --stat HEAD
   git log --oneline -10
   git branch --show-current
   git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "__NO_UPSTREAM__"
   git remote show origin 2>/dev/null | grep "HEAD branch" | awk '{print $NF}' || echo "__NO_ORIGIN__"
   ```
3. **Clean-tree check.** If no staged, unstaged, or untracked files are pending → STOP. Tell the user: *"Working tree is clean — nothing to commit. If you meant to push already-committed work, tell me and I'll rebase + push without touching the commit step."* Do not proceed unless the user confirms the push-only path.
4. **Protected-branch refusal (current).** If `git branch --show-current` matches `^(main|master|production|prod|stable|live|trunk)$` or `^release([/_-].*)?$`:
   ```
   Refusing to commit on `<branch>`. Please create a feature branch first, e.g.:

     git checkout -b feat/<short-topic>

   Then re-invoke /commit-push.
   ```
   STOP. Do not continue until the user is on a safe branch.
5. **Detached HEAD.** If `git branch --show-current` is empty → STOP. Tell the user a branch is required and ask whether to create one now. Do not proceed until a named branch exists.
6. **Clarify on bare invocation.** If `/commit-push` was called with no free-text argument and the diff is large (>10 files or >300 lines), consider noting that Standard/Deep tiering is likely and the skill will ask a split question later. Do not block on this — continue.
7. **Announce.** Emit exactly one line:
   ```
   commit-push: starting (scope TBD, branch = <current-branch>)
   ```

## Process

Execute phases in order. Do not skip ahead.

### Phase 1 — Scope classification

Classify the diff with the table below. Details in `references/scope-tiers.md`.

| Tier | Signals | Pre-commit gate | Split proposal |
|------|---------|----------------|----------------|
| **Lightweight** | 1–2 files, <50 lines changed, no mixed concerns. Typo, one-liner, config flag, copy tweak, version bump. | Secrets + protected-branch + debug-artifact scan (token-cheap). | Single commit. |
| **Standard** | 3–10 files, <500 lines, single coherent concern. Feature increment, bug fix, refactor of one module. | Lightweight gate + full test suite + lint/formatter. | Single commit usually; propose a 2-commit split only if files cluster clearly (e.g., test infra independent from feature). |
| **Deep** | 10+ files OR >500 lines OR mixed concerns (feat + refactor, schema + feat) OR changes under trust boundaries / migrations / concurrency. | Standard gate + scope-vs-diff check + test-coverage-for-code-changes + chore-exemption audit. | Propose 2–3 commit split by concern; human approves the grouping. |

Announce: `commit-push: scope = <LIGHTWEIGHT | STANDARD | DEEP>`.

**Re-classification rule:** if Phase 3 uncovers mixed concerns in what looked like Lightweight, escalate to Standard and re-announce before continuing.

### Phase 2 — Branch decision [HARD HUMAN GATE]

The current branch was already verified non-protected at Pre-flight. Now confirm where the commits land.

Offer exactly one `AskUserQuestion` with two options:

```
You're on `<current-branch>`. Commit here, or create a new branch?

  A) commit on `<current-branch>` [Recommended when stacking work]
  B) create a new branch — you'll name it next
```

- **If A** → target branch = `<current-branch>`. Proceed to Phase 3. This answer counts as the explicit human confirmation.
- **If B** → ask one follow-up question:
  ```
  Name the new branch. Suggested: `<type>/<kebab-slug-from-diff-or-hint>`
  (type one of feat/fix/chore/refactor/docs/test; derive slug from argument-hint, adjacent SPEC/DIAGNOSIS folder `topic:` frontmatter, or top-level filename change).

  Reply with the branch name.
  ```
  WAIT for the human's reply.
  - If reply is empty or "use the suggestion" → use the suggested name.
  - If reply matches the protected list → refuse and re-ask.
  - Otherwise → `git checkout -b <name>`. If the branch already exists locally and is unrelated to the current work, refuse and ask for a different name.

A silent proceed without either A or an explicit branch name in a fresh human reply is a protocol violation. If that happens, stop the commit, `git checkout <previous-branch>` (and `git branch -D` the accidental branch if it was just created), and re-ask.

Announce: `commit-push: branch = <target-branch>`.

### Phase 3 — Pre-commit verification [HARD GATE]

Run checks per the scope tier. A single failure in any REQUIRED check halts the skill — no `--no-verify`, no "let me commit and fix in a follow-up."

**3.1 Secrets scan (Lightweight + Standard + Deep).**
Search the full diff (`git diff HEAD`) for:
- Hardcoded API keys (e.g., `sk-[A-Za-z0-9]{20,}`, `AKIA[0-9A-Z]{16}`, `ghp_[A-Za-z0-9]{36}`, `xoxb-`, `xoxp-`).
- Token / password / secret assignments (`password\s*=\s*["']`, `secret\s*=\s*["']`, `token\s*=\s*["']`).
- `.env` / `.env.local` / `.env.production` being added (not just edited — added files with those names are almost always a mistake).
- Private-key headers (`-----BEGIN (RSA |OPENSSH |EC |DSA )?PRIVATE KEY-----`).

If ANY match → STOP. Show the matches. Do not stage those files. Ask the user to unstage and remove before continuing. This is unconditional — do not proceed even if the user asks you to.

**3.2 Debug-artifact scan (Lightweight + Standard + Deep).**
Search changed non-test files for: `console.log`, `console.debug`, `debugger`, `print(` (in Python production code, not tests), `pp `, `binding.pry`, `puts` (in Ruby production code), `dd(`, `var_dump`, `TODO` without ticket reference, `FIXME`, `HACK`, `XXX`.

If matches → list them and ask the user:
```
Debug / TODO artifacts found in these changed files:

  - src/foo.ts:42  console.log("here")
  - src/bar.py:17  # TODO cleanup

Reply:
  A) keep — I intended these (add a note why in the commit body)
  B) remove — I'll clean these up now and re-invoke
  C) skip scan — treat these as not-a-blocker (recorded in commit body)
```
Respect the answer. If B, STOP and let the human clean up.

**3.3 Test suite (Standard + Deep).**
Detect test runner from the pre-scan:
- `package.json` scripts: `test`, `test:unit`, `test:ci` → `npm test` / `pnpm test` / `yarn test` (use the lockfile present).
- `pyproject.toml` / `requirements.txt` with pytest → `pytest`.
- `go.mod` → `go test ./...`.
- `Cargo.toml` → `cargo test`.
- `Gemfile` with rspec → `bundle exec rspec`.
- Else → ask the user for the test command. If the user says "no tests" and scope is Standard/Deep, pause — propose Lightweight with a one-line reason in the eventual commit body.

Run the full suite. Paste literal output in a fenced block. Any failure → STOP. The human fixes before re-invoking.

**3.4 Lint + formatter (Standard + Deep).**
Detect: ESLint (`.eslintrc*`, `eslint.config.*`), Prettier, Ruff / Black, Rubocop, gofmt, rustfmt, php-cs-fixer, etc. Run the project-specified command (from `package.json` scripts `lint`, `lint:fix`; `pre-commit` config; `Makefile` targets).

Errors → STOP. Warnings → show and ask whether to proceed. Formatter must pass — if format-on-save would have fixed it, run the formatter first and restage automatically (this is a deterministic, reviewable action — tell the user).

**3.5 Scope-vs-diff check (Deep only).**
Enumerate every modified file. For each, classify:
- **In scope** — matches an adjacent SPEC tracer-bullet phase, DIAGNOSIS hotspot, or the argument-hint topic.
- **Out of scope** — files not in any of the above.

If any files are out-of-scope, ask the user per file:
```
These files changed outside the detected scope:
  - <path>
For each, reply: keep (intentional) / drop (unstage) / defer (move to a separate later commit).
```

Respect the answer. "Drop" = unstage and let the human decide later. "Defer" = stage into a second commit group in Phase 4 with a `chore:` or `refactor:` prefix.

**3.6 Test-coverage-for-code-changes (Deep only).**
Partition changed files into **code files** (production source: `.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.rb`, `.go`, `.rs`, `.java`, `.kt`, `.swift`, `.php`, `.ex`, `.exs`, `.sh`, `.c`, `.cpp`, `.h`) and **chore files** (docs `.md`, config `.json`/`.yaml`/`.toml`, `.gitignore`, CI `.github/workflows/*`, dependency manifests like `package-lock.json`, type-only `.d.ts`, static assets).

For each code file, check whether a corresponding test file was also created or modified in this diff. If any code file has no corresponding test change:
```
Code files changed without a test update:

  - src/auth/session.ts
  - lib/billing/prorate.py

For each, reply:
  cover  — I'll add the test before committing (skill will STOP and wait)
  chore  — genuinely non-behavior (doc-only runtime impact, type-only, logging format) and needs justification
```

Every file marked `chore` must have a one-sentence justification from the human. Record those in the eventual commit body under `Chore exemptions:` (one line per file). A silent chore exemption is equivalent to silent coverage loss — refuse to proceed without justification.

**3.7 Announce gate result.**
Emit one line:
```
commit-push: pre-commit gate passed (tier = <TIER>, checks = <list>).
```

### Phase 4 — Split proposal + message draft [HUMAN GATE]

Cluster the changed files by concern. A cluster is "distinct" when ALL of these hold:
- Files in the cluster share a reason-for-change (same feature area, same refactor target).
- Moving one cluster to a separate commit would preserve a passing state in both commits.
- The clusters would carry different Conventional Commits `type` (feat + refactor, fix + test, chore + feat).

If ≤1 cluster (or the scope tier is Lightweight) → single commit. Skip the split question; draft one message.

If ≥2 distinct clusters → propose the split via `AskUserQuestion`:

```
The diff splits cleanly into <N> commits:

  C1) feat(<scope>): <subject>
      Files: <file1>, <file2>

  C2) refactor(<scope>): <subject>
      Files: <file3>

Reply:
  A) split into <N> commits as proposed [Recommended]
  B) keep as one commit
  C) re-group — tell me how
```

Max 3 commits per run. If the human asks for >3, explain the limit and ask them to collapse or run `/commit-push` twice.

**Message drafting rules** (load `references/commit-message-style.md` for the full version):
- **Convention precedence**: repo-declared in CLAUDE.md / AGENTS.md / commitlint config → pattern in the last 10 commits → Conventional Commits default (`<type>(<scope>): <subject>`).
- **Types**: `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `chore`, `style`, `ci`, `build`. For `feat` and `fix`, a body is mandatory.
- **Subject**: imperative mood, lowercase-first, no period, ≤72 chars. "add user form" not "added user form" or "Adds user form.".
- **Body**: explains **why**, not **what** — the diff already shows what. Reference SPEC phases (`See docs/2026-04-17-foo/SPEC.md#phase-2`), DIAGNOSIS root causes (`Fixes root cause in DIAGNOSIS.md`), or issue numbers (`Closes #123`) when applicable.
- **Testing scenarios** (mandatory for feat/fix when scope is Standard/Deep):
  ```
  Testing scenarios:

  <scenario name>
  Before: <what happened before>
  After: <what happens now>
  ```
- **Chore exemptions** section appended when Deep-tier chore-exemption was recorded in 3.6.
- **Footer** (never):
  - Do NOT add `Signed-off-by:` unless the repo requires DCO (check `.github/CONTRIBUTING.md` or `CODEOWNERS`).
  - Do NOT add `Co-Authored-By:` lines from the skill itself.
  - Do NOT add "🤖 Generated with..." footers unless the repo explicitly uses them.

Show the draft(s) to the human via `AskUserQuestion`:

```
Commit message draft:

  feat(auth): add refresh-token rotation

  Rotate the refresh token on every access-token refresh so a leaked
  refresh token loses value after one use. Closes the replay window
  described in DIAGNOSIS.md#root-cause.

  Testing scenarios:

  Valid rotation
  Before: refresh token reused indefinitely after access-token refresh.
  After:  refresh token is replaced; the old token is rejected with 401.

Reply:
  A) approve — commit with this message [Recommended]
  B) edit — give me a revised subject / body / both
  C) rewrite — I'll write the message myself
```

If B or C, incorporate the user's edits and re-show before committing. If C and the user provides a message, still validate it: refuse a non-imperative subject, a subject over 72 chars, or a `feat`/`fix` without a body on Standard/Deep — show the violation and ask the user to revise.

### Phase 5 — Self-review checklist + HARD GATE

Before any `git add` or `git commit`, emit this checklist with `✓` / `✗` (or `✓ (N/A)` with a one-word reason):

- [ ] Current branch is NOT protected; target branch is confirmed by the human in this conversation
- [ ] Pre-commit gate passed at the declared tier (secrets, debug, tests, lint, coverage where applicable)
- [ ] No file scheduled for staging was flagged as out-of-scope without a `keep` / `defer` decision
- [ ] No secret or `.env` file is in the planned stage set
- [ ] Commit message(s) follow the detected repo convention (recent commits / commitlint / Conventional default)
- [ ] feat/fix commits on Standard/Deep carry a WHY body and a Testing scenarios block (when applicable)
- [ ] Chore exemptions (if any) are recorded in the commit body with a justification per file
- [ ] Split proposal matches the human's answer (single / approved multi-commit grouping)
- [ ] Staging plan uses explicit file paths (no `git add -A`, no `git add .`)
- [ ] Back-pointer target determined (adjacent `EXECUTION.md`/`DIAGNOSIS.md` path, or "none — standalone")
- [ ] No `gh pr create` planned; no `--force`, `--force-with-lease`, `--amend`, or `--no-verify` planned
- [ ] Rebase plan: fetch origin, pull --rebase <default>, then push -u; abort on non-trivial conflict

If any `✗` → fix and re-emit. Do not advance.

Then present the HARD GATE via `AskUserQuestion`:

```
Ready to commit and push.

Branch:        <target-branch>
Scope:         <LIGHTWEIGHT | STANDARD | DEEP>
Commits:       <N>
  C1: <type(scope): subject>
  C2: <type(scope): subject>  (if any)
Staged files:  <count> (explicit list in the log)
Back-pointer:  <docs/<DATE>-<slug>/<artifact>.md | none>

Reply:
  A) approve — stage, commit, rebase, push [Recommended]
  B) change the split / message
  C) abort
```

**STOP HERE.** Do not run `git add`, `git commit`, `git push`, or any mutation. WAIT for explicit approval. A → continue to Phase 6. B → loop back to Phase 4. C → stop cleanly with a one-line summary of nothing-done.

### Phase 6 — Stage, commit, rebase, push

Run these steps in order. Each Bash call must paste its literal output in a fenced block.

**6.1 Stage and commit (per group).** For each commit group, run in a single call with a heredoc to preserve message formatting:

```bash
git add <file1> <file2> <file3> && git commit -m "$(cat <<'EOF'
<type>(<scope>): <subject>

<body lines, blank line separated>

Testing scenarios:

<scenario block>
EOF
)"
```

Never `git add -A`, never `git add .`, never `git add -u`. Stage only the files listed in the approved plan.

After each commit, paste the output (commit hash, file summary). If the commit fails (pre-commit hook rejects, etc.) — STOP. Do NOT `--amend`, do NOT `--no-verify`. Report the failure and hand back to the human. They fix the underlying issue and re-invoke.

**6.2 Determine upstream default.** Using the value captured at Pre-flight:
- If origin's HEAD branch was resolved → use it.
- Else try `git rev-parse --abbrev-ref origin/HEAD 2>/dev/null` and strip the `origin/` prefix.
- Else try `gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'`.
- Else fall back to `main` and tell the user you guessed.

**6.3 Fetch + rebase.**
```bash
git fetch origin
git pull --rebase origin <upstream-default>
```
Paste output. On non-trivial conflict:
- STOP.
- Show `git status` and the conflict markers.
- Ask the user to resolve. Do not attempt to resolve non-trivial conflicts yourself.
- On trivial conflicts (whitespace-only, clearly separable hunks), the skill may resolve and continue — but tell the user what was resolved, in one paragraph.

**6.4 Push.**
```bash
git push -u origin <target-branch>
```
Paste output. On rejection (`non-fast-forward`, stale ref):
- Do NOT `--force` or `--force-with-lease`.
- Re-run 6.3 (fetch + rebase).
- Re-run 6.4 once.
- If still rejected → STOP. Report the reason and hand back. The user decides whether to force-with-lease in a separate, explicit turn.

### Phase 7 — Back-pointer (optional) + report

**7.1 Back-pointer.** Scan for an adjacent `docs/*/` folder whose topic matches the committed work:
- Newest `docs/<DATE>-<slug>/` whose `EXECUTION.md` has `status: completed` or `status: in_progress` and was touched in this session or this commit range.
- Else newest `docs/<DATE>-<slug>/DIAGNOSIS.md` with `status: approved` whose hotspots include files from this diff.
- Else none — skip.

If a target is found, append to the bottom of the primary artifact:

```markdown
## Commits <DATE>

- `<SHA1>` <type(scope): subject line>
- `<SHA2>` <type(scope): subject line>

Branch: `<target-branch>`
```

Append only. Never rewrite existing sections. If a `## Commits <DATE>` section with today's date already exists on that artifact, append new bullets rather than creating a duplicate heading.

**7.2 Report.** Emit exactly:
```
commit-push complete.

Branch:    <target-branch>
Commits:
  <SHA1>  <type(scope): subject>
  <SHA2>  <type(scope): subject>  (if any)
Pushed to: origin/<target-branch>
Backpointer: <docs/.../EXECUTION.md | docs/.../DIAGNOSIS.md | none>

Next: open a PR when you're ready (this skill does not open PRs).
```

### Transition

After reporting, STOP. Do NOT:
- Run `gh pr create` or any PR-creation primitive.
- Invoke `/code-review` yourself. If the user wants a review, they will ask.
- Suggest "shall I open the PR now?" — the user owns that decision.
- Rewrite, amend, or force-push any commit from this run.

Tell the user in one sentence: *"Commits are on `<branch>`. You can open a PR, run `/code-review`, or iterate with `/execute`."*

## Anti-patterns

See `references/anti-patterns.md` for the full list. Headline rationalizations to resist:

- *"The suggested branch name is obvious — just use it."* The human names the branch in this turn. Even obvious suggestions need an explicit "yes, use the suggestion."
- *"Tests passed last turn, I won't re-run."* State changes between turns. Re-run for every commit.
- *"Lint warnings are fine."* Check the config. Some projects treat warnings as errors. Run the linter.
- *"`git add -A` is faster and I checked the diff."* It's still forbidden. `.env`, `.DS_Store`, build artifacts, and IDE droppings sneak in through the checks you skipped.
- *"This secret is a fake / in a test fixture."* Skill stops. Scope is unconditional. If the fixture needs a fake secret, use a documented sentinel value (`sk_test_...`) outside the patterns the scan looks for.
- *"I'll `git push --force` — it's my branch."* Not from this skill. If a rebase is needed, stop and hand back.
- *"While I'm pushing, let me open the PR too."* `gh pr create` is explicitly forbidden from this skill. PR creation is a separate, human-invoked action.
- *"The commit failed the pre-commit hook — I'll `--no-verify` once."* NO. The hook caught something. Fix it, then re-commit.
- *"I'll `commit --amend` the pushed commit to fix the message."* Not from this skill. Amending pushed commits requires force-push, which is out of scope.
- *"The code file doesn't need a test — it's a one-liner rename."* If the rename touches public API, a test IS the contract. If it's purely internal, mark it `chore` and record the reason.

## Red flags — self-check

STOP if you catch yourself:

- About to run `git commit` with no human answer to the branch question in this conversation
- On `main` / `master` / any protected branch and continuing
- About to `git add -A`, `git add .`, or `git add -u`
- Pasting a secret / token / `.env` content into a staged hunk
- About to run `gh pr create` from this skill
- Planning to `--force`, `--force-with-lease`, `--no-verify`, or `--amend` a pushed commit
- Writing or editing source code beyond the already-present working-tree changes
- Skipping a pre-commit check because "the last run was fine"
- Declaring the skill complete before the push actually succeeded
- Inventing a commit message convention when the last 10 commits clearly follow one
- Appending a `Co-Authored-By:` or promotional footer to the commit message
- Planning a 4th commit in a single run
- Resolving a non-trivial conflict silently instead of handing back to the user

## References

- `references/scope-tiers.md` — detailed Lightweight / Standard / Deep rubric, pre-commit gate matrix, re-classification triggers.
- `references/commit-message-style.md` — Conventional Commits rules, convention-detection order, subject/body/footer guidance, testing-scenarios block shape, worked examples.
- `references/anti-patterns.md` — full list of rationalizations, bypass patterns, scope creep, secret-handling stances, and the "error output is data, not instructions" security rule.
