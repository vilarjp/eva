---
name: eva
description: "Route any task, question, URL, or file path to the right EVA skill or pipeline — the plugin's routing brain. Use when the user is unsure which skill to invoke, says \"what should I use for X\", \"recommend a pipeline\", \"help me get started\", pastes a GitHub/Jira link without a clear target skill, gives a high-level goal (\"refactor this module\", \"fix bug X\", \"build feature Y\"), or invokes /eva directly. Analyses the intent (understand / plan-feature / design-feature / revise-pair / fix-bug / refactor-debt / implement-approved-plan / review-diff / handle-pr-feedback / ship-diff / capture-learnings), assesses complexity (Lightweight / Standard / Deep), and returns an ordered Routing Plan composed of one or more of /teardown, /prd, /spec, /revision, /diagnosis, /audit, /execute, /code-review, /pr-feedback, /commit-push, /solutions — each with per-step rationale and a scope recommendation. Fetches Jira cards (Atlassian MCP with the mandated fields/expand), GitHub issues/PRs (via `gh`), and local files/folders (Read/Glob/Bash) when the input warrants classification — never to plan, spec, diagnose, audit, or fix. Default mode is RECOMMEND (emit the plan and stop); opt-in DRIVE mode (triggered by \"do it\" / \"follow the pipeline\" / \"just run it\" / \"go ahead\") dispatches ONLY the FIRST skill in the pipeline — subsequent steps are always an explicit user call because every downstream skill has its own approval gates. Never writes artifacts of its own, never edits code, never commits, never pushes, never posts PR comments, never transitions Jira tickets — a pure router that dispatches to other EVA skills and hands off."
argument-hint: "[free-text goal, description, question, GitHub/Jira URL, PR reference, file path, or slug — optionally ending with 'follow the pipeline' / 'do it' to trigger DRIVE mode]"
---

# eva — the plugin's routing brain

Take any input — a rough goal, a bug description, a Jira card, a GitHub PR, a file path, a pasted stack trace, or just *"what should I do?"* — and hand back a **Routing Plan**: the specific EVA skill(s) to run, in order, with a complexity tier and a one-line rationale per step.

This skill does NOT write artifacts, edit code, run tests, commit, push, post PR comments, or transition Jira issues. It reads, classifies, and recommends. The actual work happens in the skill it routes to — each downstream skill carries its own approval gates, invariants, and artifact.

## When to use

Trigger this skill when the user:
- asks a routing question: *"which skill should I use for X?"*, *"recommend a pipeline for Y"*, *"what should I do with this?"*, *"where do I start?"*, *"help me get started"*
- pastes a URL or reference without naming a skill: a GitHub PR, a GitHub issue, a Jira card or key, a bare URL, a `docs/<DATE>-<slug>/` path
- describes a high-level goal without naming a skill: *"I want to refactor the checkout module"*, *"build a feature for invoice exports"*, *"fix the bug where customers see the wrong currency"*
- invokes `/eva` (optionally followed by free-text, a URL, a reference, or a path)

Do NOT trigger when:
- The user already named a skill (*"run /prd for X"*, *"diagnose this bug"*, *"audit src/payments"*) — dispatch to that skill directly.
- Another EVA skill is already running and the user asks a sub-question — the hosting skill handles it.
- The user asks a generic coding question with no routing signal (*"how do I write a Promise in TypeScript?"*) — answer normally, don't invoke `/eva`.

## The Iron Law

**Route, do not substitute.**

`/eva` recommends which skill to run. It does NOT do the work of that skill inline. A recommendation like *"I'd start with /diagnosis and here's the root cause I found"* is a protocol violation — the root cause belongs in `DIAGNOSIS.md`, produced by `/diagnosis`, gated on its own approval. `/eva`'s output is a **plan**, not a deliverable.

The only inline reading `/eva` does is **classification reading**: enough to pick the right skill. Not enough to plan the feature, spec the design, diagnose the bug, audit the debt, or triage the review. If the reading is slipping into substance, stop and hand off.

## Core invariants

1. **Read-only.** Writes no file. Edits no code. Makes no commit, push, or PR comment. Never transitions a Jira issue state.
2. **Classification reading only.** Read enough to route — file metadata, issue title/body/labels, URL type, folder structure, `git status`. Never enough to draft a PRD, a SPEC, a diagnosis, an audit finding, or a code-review finding.
3. **Routing Plan is the output.** No `docs/<DATE>-<slug>/` folder, no `.md` file. The Routing Plan is rendered inline as the skill's response.
4. **One plan, one recommendation.** The plan names exactly one ordered pipeline (1-6 skills). Alternatives live in a separate **Alternatives considered** section, never as a second plan.
5. **DRIVE mode is opt-in.** Default is RECOMMEND (emit the plan and stop). DRIVE only fires when the user explicitly says *"do it"* / *"follow the pipeline"* / *"just run it"* / *"go ahead"*.
6. **DRIVE dispatches only the FIRST skill.** Never auto-chain multiple skills. The plan itself is the sequence; running it end-to-end is a series of explicit user decisions because every downstream skill has its own HARD GATE.
7. **Honour downstream skill gates.** `/eva`'s recommendation does not pre-approve any downstream skill's artifact. The user still approves each `docs/<DATE>-<slug>/<ARTIFACT>.md` before it's written.
8. **Scope recommendation, not scope override.** The plan suggests a scope (Lightweight / Standard / Deep) per skill. Each skill owns its final scope classification.
9. **Ask ONE clarifying question at a time. Max 2 total.** Classification needs intent + complexity; usually both can be inferred. When they can't, ask ONE question, wait, then decide.
10. **Refuse to substitute for a named skill.** If the user clearly named a skill (*"/audit src/X"*, *"run /spec"*), do not re-route. Dispatch to that skill.
11. **No external side effects.** May read a Jira card (Atlassian MCP with the mandated `fields: ["*all"]` and `expand: "renderedFields,names,schema,changelog,versionedRepresentations"`), read a GitHub issue/PR (via `gh`), or read local files — but never edits, comments, reassigns, or transitions.
12. **No speculative pipelines.** Every step in the plan must be justified by a specific fact observed during Phase 1 classification reading — a URL type, a file count, an issue label, an existing artifact. Padding the pipeline with *"optional"* steps the user did not ask for is a protocol violation.
13. **No artifact content.** Never draft a PRD body, a SPEC section, a diagnosis hypothesis, an audit finding, a reply to a PR comment, or a commit message. That content belongs in the downstream skill's artifact.

## Two modes

`/eva` has exactly two modes, picked from the args:

**RECOMMEND (default).** Produce a Routing Plan and stop. The user reads it and decides. This mode fires for *"what should I use for X?"*, *"recommend a pipeline"*, or a bare goal / URL / path.

**DRIVE.** Triggered when the args contain a drive phrase: *"do it"*, *"follow the pipeline"*, *"just run it"*, *"go ahead"*, *"start it"*, *"kick it off"*, or when the user explicitly asks `/eva` to also act (*"investigate and fix bug X"*, *"refactor module Y and ship"*). Emits the Routing Plan, then dispatches the FIRST skill via the Skill tool. Subsequent steps are the user's explicit call — `/eva` does NOT invoke a chain.

If the mode is ambiguous (the input could go either way), default to RECOMMEND and include in the output: *"Say 'do it' to dispatch the first step."*

## Pre-flight (MANDATORY)

Before anything else:

1. Get today's date: `date +%Y-%m-%d` via Bash. Store as `<DATE>`.
2. Parse the args:
   - **Empty** → ask once: *"What should I route? A goal, a bug, a URL, a file path, or a question?"* Wait for a reply before proceeding.
   - **URL** → note the URL type (GitHub PR / issue / repo / Jira / other).
   - **Path** → resolve it and note its shape (file / directory / `docs/<DATE>-<slug>/`).
   - **Drive phrase** → flag DRIVE mode.
   - **Otherwise** → treat as free-text.
3. Announce: `eva: routing (mode: RECOMMEND | DRIVE, input: <short summary>)`.

## Process

Execute phases in order. Skipping phases is a protocol violation.

### Phase 1 — Classification reading

Do the **minimum** reading needed to classify intent. Route by input type:

**GitHub PR URL:**
```
gh pr view <url> --json title,body,state,isDraft,reviewDecision,labels,files
gh pr view <url> --json reviewThreads --jq '.reviewThreads | length'
```
- Review threads present AND `reviewDecision ∈ {CHANGES_REQUESTED, COMMENTED}` → lean `handle-pr-feedback`.
- No review threads, large diff, open → lean `review-diff`.
- State is `MERGED` / `CLOSED` → ask the user what they want to do.

**GitHub issue URL:**
```
gh issue view <url> --json title,body,labels,state
```
- Label set includes `bug` / `regression` / `broken` → lean `fix-bug`.
- Label set includes `feature` / `enhancement` / `feature-request` → lean `plan-feature`.
- No label → read the body; a reproduction section → `fix-bug`, otherwise `plan-feature`.

**Jira card (URL or key):**
Use Atlassian MCP `getJiraIssue` with `fields: ["*all"]` and `expand: "renderedFields,names,schema,changelog,versionedRepresentations"` (user CLAUDE.md mandates these).  
The standard `description` is often empty — content lives in custom fields like *"Passo a Passo"*, *"Lojas afetadas"*, *"Emails dos consumidores"*. Read those before classifying.
- Issue type is `Bug` / `Incident` → `fix-bug`.
- Issue type is `Story` / `Task` / `Feature` → `plan-feature` (or `design-feature` if the card links to an existing PRD).

**Local file path:**
```
ls -la <path>
git status --short <path>
```
Plus a head read of the file.
- Minified (single long line, no whitespace), unknown bundle, third-party script → `understand`.
- Inside the working tree, no uncommitted diff → ask once: *"Do you want to understand it, audit it, or review a change?"*
- Inside the working tree, uncommitted diff → `review-diff` if the user wants a pre-commit read, or `ship-diff` if they want it out the door.

**Local directory path:**
```
find <path> -type f | wc -l
find <path> -type f \( -name '*.ts' -o -name '*.tsx' -o -name '*.js' -o -name '*.jsx' -o -name '*.py' -o -name '*.go' -o -name '*.rb' -o -name '*.java' -o -name '*.rs' \) -print0 | xargs -0 wc -l 2>/dev/null | tail -1
```
- `docs/<DATE>-<slug>/` → list artifacts (`ls docs/<DATE>-<slug>/`). Route based on which EVA artifacts already exist (see Phase 4 trimming).
- Source directory, no bug/feature signal → default to `refactor-debt` (`/audit`).

**Free-text only:**
No reading. Intent comes from keywords (see Phase 2 table). Complexity is inferred from scope keywords.

Keep the reading **shallow**. `/eva` classifies; it does not investigate.

### Phase 2 — Intent classification

Map the classified input to exactly one of these intents:

| Intent | Signal |
|--------|--------|
| `understand` | Unfamiliar file/bundle/third-party script, URL to a raw `.js` file, *"how does X work"*, *"explain this file"*, *"reverse-engineer"* |
| `plan-feature` | *"new feature"*, *"I want to build"*, *"brainstorm"*, rough idea, GH issue labelled `feature`/`enhancement`, Jira Story/Task without a PRD |
| `design-feature` | Approved PRD in folder, *"spec this out"*, *"design the architecture"*, *"tech spec"* |
| `revise-pair` | Folder has `PRD.md` + `SPEC.md` but no `REVISION.md`, *"cross-check"*, *"sanity check"*, *"revise the pair"* |
| `fix-bug` | *"bug"*, *"broken"*, *"error"*, stack trace, regression, GH issue labelled `bug`, Jira Bug/Incident, reproduction steps |
| `refactor-debt` | *"refactor module X"*, *"survey tech debt"*, *"code smells"*, *"clean up"*, no bug, stable code |
| `implement-approved-plan` | `DIAGNOSIS.md` / `SPEC.md` / `REVISION.md` / `PRD.md` in folder at `status: approved` — planning done, only coding remains |
| `review-diff` | *"review my diff"*, *"check these changes"*, uncommitted working tree |
| `handle-pr-feedback` | GitHub PR URL with review comments, *"handle PR feedback"*, *"triage the review"*, *"address the review"* |
| `ship-diff` | *"commit and push"*, *"ship this"*, *"save my changes"*, uncommitted diff + user wants it out |
| `capture-learnings` | `docs/<DATE>-<slug>/` with `EXECUTION.md` and/or `CODE-REVIEW.md` + *"wrap up"*, *"what did we learn"*, *"learnings"*, *"retrospective"* |

If the input is ambiguous between two intents, pick the one with the STRONGER signal and surface the alternative in the Routing Plan's **Alternatives considered** section. If genuinely tied, ask ONE clarifying question.

### Phase 3 — Complexity assessment

Classify the work into Lightweight / Standard / Deep:

| Tier | Criteria |
|------|----------|
| **Lightweight** | 1-3 files, < 1 day of work, low blast radius, no new module, no schema change, no security surface, no cross-service contract. |
| **Standard** | 4-15 files, 1-3 days, moderate blast radius, one cohesive module, may touch a schema or an internal API but not a trust boundary. |
| **Deep** | 15+ files, architectural, > 3 days, high blast radius — migration, security-sensitive, cross-service contract, data model change, multi-team coordination. |

Signals:
- File count from Phase 1 reading.
- Issue labels (`architecture`, `security`, `migration`, `breaking-change` → Deep).
- Scope keywords in free-text (*"migration"*, *"schema change"*, *"refactor everything"*, *"cross-service"* → Deep).
- Default to Standard when signals are mixed.

This tier sets the **expectation** for the work. Individual skills may run at their own scope — but the recommendation sets the starting point.

### Phase 4 — Pipeline composition

Map the intent → pipeline using this table:

| Intent | Pipeline |
|--------|----------|
| `understand` | `/teardown` |
| `plan-feature` | `/prd` → `/spec` → `/execute` → `/code-review` → `/commit-push` [→ `/solutions` on Standard/Deep] |
| `design-feature` | `/spec` → `/execute` → `/code-review` → `/commit-push` [→ `/solutions`] |
| `revise-pair` | `/revision` → `/execute` → `/code-review` → `/commit-push` [→ `/solutions`] |
| `fix-bug` | `/diagnosis` → `/execute` → `/code-review` → `/commit-push` [→ `/solutions` on Standard/Deep] |
| `refactor-debt` | `/audit` → `/execute` (REFACTOR mode) → `/code-review` → `/commit-push` |
| `implement-approved-plan` | `/execute` → `/code-review` → `/commit-push` [→ `/solutions`] |
| `review-diff` | `/code-review` [→ `/execute` FIX mode if findings are actionable] |
| `handle-pr-feedback` | `/pr-feedback` → `/execute` FIX mode → `/code-review` → `/commit-push` |
| `ship-diff` | `/commit-push` |
| `capture-learnings` | `/solutions` |

**Trim the pipeline** based on signals already present in the target folder:
- `PRD.md` at `status: approved` → skip `/prd`, start at `/spec`.
- `PRD.md` + `SPEC.md` without `REVISION.md` → insert `/revision` on Standard/Deep, or skip to `/execute` on Lightweight.
- `DIAGNOSIS.md` at `status: approved` → skip `/diagnosis`, start at `/execute`.
- `CODE-REVIEW.md` exists → recommend `/execute` FIX mode instead of a fresh `/code-review`.
- `EXECUTION.md` at `status: approved` + user wants closure → recommend `/solutions`.

`/solutions` is **optional** on Lightweight pipelines and **recommended** on Standard/Deep. Always mark it as optional in the output.

### Phase 5 — Self-review checklist

Before emitting, run this inline:

```
Routing self-review:
- [ ] Intent is a single tag from the Phase 2 table (not "could be either").
- [ ] Every step in the pipeline is justified by a specific Phase 1 observation.
- [ ] Complexity tier matches the signals (file count, labels, keywords).
- [ ] /solutions is marked _optional_ on Lightweight pipelines.
- [ ] /eva did NOT substitute for a skill the user already named.
- [ ] No artifact content was drafted (no PRD body, no SPEC sections, no diagnosis hypotheses, no audit findings, no review comments).
- [ ] The "Alternatives considered" section names a real alternative, not a straw one.
- [ ] Mode is correct (RECOMMEND default; DRIVE only if the user asked).
- [ ] No external side effect fired (no PR comment, no Jira transition, no commit).
```

If any box is unchecked, fix before emitting.

### Phase 6 — Emit the Routing Plan

Use this exact shape:

```markdown
## EVA Routing Plan

### Understood
**Goal:** <one-sentence paraphrase of the user's ask>
**Input type:** <free-text | GitHub PR | GitHub issue | Jira card | local file | local directory | docs folder>
**Evidence:** <1-2 lines — what I actually read to classify; cite file:line, issue label, URL type, or folder contents>

### Recommendation
**Complexity:** Lightweight | Standard | Deep
**Pipeline:**
1. `/<skill1> <args>` — <one-line rationale>
2. `/<skill2> <args>` — <one-line rationale>
3. `/<skill3> <args>` — <one-line rationale>
4. `/<skill4> <args>` — <one-line rationale>
5. `/<skill5>` — _optional_ — <one-line rationale>

### Skipped steps
- `/<skill>` — <why this step is not in the pipeline>

### Alternatives considered
- <alternative pipeline>: <why not>

### Next step
Say **"do it"** to dispatch the first step (`/<skill1>`) — each subsequent step is an explicit call.
Or invoke the first one yourself: `/<skill1> <args>`.
```

Keep each rationale to ONE line. Keep the full plan under 30 lines.

### Phase 7 — DRIVE dispatch (only in DRIVE mode)

If and only if the user asked for DRIVE:

1. Emit the Routing Plan (Phase 6).
2. Confirm once, inline: *"Dispatching `/<skill1>` now. Each subsequent step will need an explicit call — every EVA skill has its own approval gate."*
3. Invoke the Skill tool with `skill: "<skill1-name>"` and `args: "<args>"`. Use the plugin-qualified form (`eva:<skill1-name>`) only if there's ambiguity with another plugin.
4. **Stop.** Do NOT chain to the next step. The downstream skill owns its own flow; when it completes, the user decides the next step.

## Input type handling — cheat sheet

| Input | First reads | Typical routing |
|-------|------------|-----------------|
| Free-text ("refactor X") | none | intent from keywords |
| GitHub PR URL | `gh pr view --json …` | `handle-pr-feedback` or `review-diff` |
| GitHub issue URL | `gh issue view --json …` | `fix-bug` or `plan-feature` (by label) |
| Jira key / URL | Atlassian MCP `getJiraIssue` with mandated fields/expand | `fix-bug` or `plan-feature` (by type) |
| File path | `ls -la` + head + `git status` | `understand` / `review-diff` / `ship-diff` / ask |
| Directory path | `find … | wc -l` | `refactor-debt` (default) |
| `docs/<DATE>-<slug>/` | `ls` + artifact frontmatter scan | resume the pipeline at the next missing stage |

## Out of scope

These are NOT `/eva`'s job — redirect rather than route:

- **Writing the PRD / SPEC / DIAGNOSIS / AUDIT / CODE-REVIEW / EXECUTION / SOLUTIONS itself.** `/eva` recommends the skill that writes it.
- **Listing every skill.** The README is the catalogue.
- **Deciding the user's roadmap.** `/eva` routes one task, not a quarter.
- **Bypassing approval gates.** Even in DRIVE mode, the downstream skill's HARD GATE still fires.
- **Commenting on a PR or transitioning a Jira issue.** Read-only on external state.

## Anti-patterns

- **Routing into multiple intents at once** (*"this is both a bug and a feature"*). Pick the stronger signal; if genuinely dual, ask ONE clarifying question.
- **Drafting artifact content** (*"here's the PRD I'd write..."*). That's the downstream skill's job.
- **Padding the pipeline with `/solutions` on Lightweight work.** Mark it optional or skip it.
- **Auto-DRIVE because the input "seems like a go"** — DRIVE is opt-in; the user must explicitly say so.
- **Substituting for a named skill** — if the user typed `/spec`, do not intercept through `/eva`.
- **Deep investigation reading.** `/eva` reads enough to classify. If you're reading source code line-by-line, you're doing `/audit`'s or `/diagnosis`'s job.
- **Yes-machining a missing intent.** If you cannot classify, ask. Do not guess.
- **Two plans in one response.** One recommended plan. Alternatives live in their own section.

## Red flags — self-check

- You wrote content that belongs in PRD.md / SPEC.md / DIAGNOSIS.md / AUDIT.md / CODE-REVIEW.md / EXECUTION.md / SOLUTIONS.md.
- You dispatched more than one skill in DRIVE mode.
- You skipped Phase 1 reading on an input that demanded it (URL, path, folder).
- You recommended `/solutions` on Lightweight work without marking it optional.
- You took the first intent that fit without checking the alternatives.
- You overrode a named skill (*"you said /audit but I think /diagnosis fits better"*) without explicit user confirmation.
- Your Routing Plan exceeds 30 lines.
- You fetched a Jira card without `fields: ["*all"]` and `expand: "renderedFields,names,schema,changelog,versionedRepresentations"`.
- You transitioned a Jira ticket, posted a PR comment, pushed a branch, or made a commit. Any of those is a protocol violation — `/eva` is strictly read-only on external state.

## Examples

### Example 1 — Free-text refactor goal

**Input:** `/eva I'd like to refactor module src/payments/processor.ts, what do you recommend?`

**Output:**
```
## EVA Routing Plan

### Understood
**Goal:** Refactor src/payments/processor.ts to clean up structural debt.
**Input type:** free-text + local file path
**Evidence:** `ls -la src/payments/processor.ts` → 820 LOC, no uncommitted diff.

### Recommendation
**Complexity:** Standard
**Pipeline:**
1. `/audit src/payments/processor.ts` — catalogue smells with severity + named category before touching code.
2. `/execute <audit-folder>` (REFACTOR mode) — apply selected refactors under behaviour-preservation.
3. `/code-review` — pre-merge read on the refactor diff.
4. `/commit-push` — ship the refactor commit.

### Skipped steps
- `/diagnosis` — no bug signal in the ask.
- `/prd` / `/spec` — behaviour-preserving refactor, no new feature.
- `/solutions` — optional, Standard work; recommend after /code-review closes.

### Alternatives considered
- Direct `/execute` (RAW): would skip the audit and risk scope drift — refactors without a smell inventory tend to grow.

### Next step
Say **"do it"** to dispatch `/audit` now.
Or invoke it yourself: `/audit src/payments/processor.ts`.
```

### Example 2 — Bug with auto-drive

**Input:** `/eva investigate and follow the recommended pipeline to fix bug PROJ-1234`

**Output:** Emit the Routing Plan (Jira card fetched, classified as `fix-bug`, complexity inferred from the card), then **dispatch `/diagnosis`** (first step, DRIVE mode) — stop after dispatch. Do NOT chain `/execute` afterward; the user decides when `/diagnosis` completes.

### Example 3 — URL-only

**Input:** `/eva https://github.com/acme/app/pull/742`

**Output:** Fetch the PR via `gh`, check review threads + `reviewDecision`. If CHANGES_REQUESTED with comments → recommend `/pr-feedback` pipeline. If no reviews yet → recommend `/code-review` pipeline. Evidence cites the PR fields observed.

### Example 4 — Empty input

**Input:** `/eva`

**Output:** Ask once: *"What should I route? A goal, a bug, a URL, a file path, or a question?"* Wait.
