# GitHub API cheatsheet — pr-feedback

The skill uses `gh` CLI exclusively. Auth is whatever `gh auth status` reports. All calls are read-only.

## PR reference parsing

Accepted input forms:

| Form | Regex | Parse to |
|------|-------|----------|
| Full URL | `^https://github\.com/([^/]+)/([^/]+)/pull/(\d+)` | `{owner, repo, number}` |
| Qualified | `^([^/]+)/([^/]+)#(\d+)$` | `{owner, repo, number}` |
| Bare number | `^#?(\d+)$` | `{number}`; resolve `owner/repo` via `gh repo view --json nameWithOwner -q .nameWithOwner` in the cwd |

If the bare form is used and the cwd has no GitHub remote, refuse. If the parsed `owner/repo` differs from the cwd's remote, refuse — do not attempt cross-repo triage.

## PR metadata (Phase 0.1, Phase 1.1)

One call, used for both folder strategy and the state gate:

```bash
gh pr view <N> --repo <owner>/<repo> \
  --json number,title,state,isDraft,headRefName,headRefOid,headRepositoryOwner,baseRefName,author,body,url,reviewDecision,labels,mergeable
```

Fields used:

| Field | Used for |
|-------|----------|
| `state` | `OPEN` / `MERGED` / `CLOSED`. Refuse merged/closed unless `--force-state`. |
| `isDraft` | Warn + confirm. |
| `headRefName` | Branch name for the checkout gate. |
| `headRefOid` | SHA at PR head — the value `git rev-parse HEAD` must match after checkout. |
| `headRepositoryOwner` | Detect forks (`!= owner` → use `gh pr checkout`). |
| `baseRefName` | Default base for `gh pr diff`. |
| `reviewDecision` | Informs scope tier (`CHANGES_REQUESTED` forces Standard+). |
| `author` | PR author (for the artifact summary). |
| `title` / `body` | Summary + slug derivation fallback. |
| `labels` | Escalation hints (e.g. `needs-tests`, `blocker`, `security`). |

## Branch state (Phase 1.2)

Inspect only — never mutate without user consent:

```bash
git branch --show-current
git rev-parse HEAD
git status --porcelain
```

On user approval (Phase 1.2 Path 3 Option A — includes forks cleanly):

```bash
gh pr checkout <N> --repo <owner>/<repo>
```

On user approval (Phase 1.2 Path 2 Option A — already on branch, need to fast-forward):

```bash
git fetch origin <headRefName>
git pull --ff-only origin <headRefName>
```

After either, verify:

```bash
test "$(git rev-parse HEAD)" = "<headRefOid>"
```

If mismatch, abort and fall back to refuse-until-correct — print the exact manual recovery commands and stop.

## Fetching comments (Phase 2)

### 2.1 Inline review-thread comments — GraphQL (single call)

Why GraphQL: REST's `/pulls/{n}/comments` does NOT expose `isResolved` or `isOutdated` as first-class fields. GraphQL returns thread state + replies + author + body + diff-hunk in one round-trip.

```bash
gh api graphql \
  -f owner=<owner> -f repo=<repo> -F number=<N> \
  -f query='
query($owner:String!, $repo:String!, $number:Int!) {
  repository(owner:$owner, name:$repo) {
    pullRequest(number:$number) {
      reviewThreads(first:100) {
        pageInfo { hasNextPage endCursor }
        nodes {
          id
          isResolved
          isOutdated
          isCollapsed
          viewerCanResolve
          path
          line
          originalLine
          startLine
          diffSide
          comments(first:50) {
            nodes {
              id
              databaseId
              author { login }
              body
              url
              createdAt
              outdated
              minimizedReason
              originalCommit { oid }
              commit { oid }
              replyTo { id }
              diffHunk
              pullRequestReview { id state submittedAt }
            }
          }
        }
      }
    }
  }
}'
```

Pagination: if `pageInfo.hasNextPage == true`, re-query with `first:100 after:"<endCursor>"`. Same for the inner `comments(first:50)` — rare but possible on very hot threads.

Per-comment handling:
- If `pullRequestReview.state == "DISMISSED"` → drop, log skip.
- If `pullRequestReview.state == "PENDING"` → drop (not yet published).
- If the thread node has `isResolved: true` → include, render inside `<details>`.
- If `outdated: true` OR `line == null` → flag `anchor_stale: true`, attempt re-anchor via `diffHunk`.

### 2.2 Timeline issue comments — REST

```bash
gh api repos/<owner>/<repo>/issues/<N>/comments --paginate
```

Each element:

| Field | Notes |
|-------|-------|
| `id` | For dedup. |
| `body` | Raw markdown. |
| `user.login` | Author. |
| `created_at` | Ordering. |
| `html_url` | Permalink (for the artifact). |

Not anchored to files. Treat as `source: "timeline"`, `path: null`, `line: null`.

### 2.3 Review bodies — REST

```bash
gh api repos/<owner>/<repo>/pulls/<N>/reviews --paginate
```

Each element:

| Field | Notes |
|-------|-------|
| `id` | For dedup. |
| `body` | Top-level review summary. May be empty string — skip those. |
| `state` | `APPROVED` / `CHANGES_REQUESTED` / `COMMENTED` / `DISMISSED` / `PENDING`. |
| `user.login` | Reviewer. |
| `submitted_at` | Ordering. |
| `html_url` | Permalink. |
| `commit_id` | SHA the review was submitted against. |

Filter: drop `state == "DISMISSED"` or `state == "PENDING"`. Log the skip.

### 2.4 Diff (Phase 3.1)

```bash
gh pr diff <N> --repo <owner>/<repo>
```

Used for:
- Verifying `already-done` (was the line changed since the comment?).
- Finding handoff `file:line` for must-fix verdicts.
- Re-anchoring stale comments (text-search the comment's `diffHunk` snippet in the current diff output).

For a per-file machine-readable list, supplement with:

```bash
gh pr diff <N> --repo <owner>/<repo> --name-only
```

## Field semantics — gotchas

### `line` vs `originalLine`

- `line` — current line number in the PR's head. `null` when the comment is outdated (line no longer exists at that location).
- `originalLine` — line number at the time the comment was posted. Always populated.
- Prefer `line` when non-null. Fall back to `originalLine` with a "stale" badge.

### `position` vs `line` (REST)

GitHub's REST `/pulls/<N>/comments` returns both `position` (counted from the `@@` hunk header, deprecated for line-level precision) and `line` (absolute file line). Use `line`. Ignore `position`. The GraphQL query above avoids this entirely.

### `commit_id` vs `original_commit_id`

- `original_commit_id` (REST) / `originalCommit.oid` (GraphQL) — SHA at which the comment was posted.
- `commit_id` (REST) / `commit.oid` (GraphQL) — SHA the comment currently anchors to (PR head at fetch time).
- If these differ AND `line == null`, the comment is outdated.

### Thread reply structure

- Top-level comment in a thread: `replyTo: null`.
- Reply comment: `replyTo.id` is the id of the top-level comment.
- Walk up to `replyTo == null` to find the thread root; nest replies under it in the artifact.

### Suggestion blocks

Comment body syntax:

    ```suggestion
    <literal replacement for the commented lines>
    ```

Parse via fenced-code detection with language tag `suggestion`. The block replaces exactly the range the comment anchors to (single-line for `line`, `startLine..line` for multi-line threads).

Store parsed blocks as `suggestion_blocks: [{ body: "<content>", is_multiline: <bool> }]`. In the handoff for `must-fix` verdicts, include the block verbatim so `/execute` can apply it as-is.

### Resolved vs outdated — they're independent

- `isResolved: true`, `isOutdated: false` — reviewer (or author) marked the thread resolved. Comment line still exists. Common for "addressed" threads.
- `isResolved: false`, `isOutdated: true` — line moved under the comment; thread is still open. The reviewer may not yet know.
- `isResolved: true`, `isOutdated: true` — both. Typically means a rebase removed the line and someone resolved the thread.
- `isResolved: false`, `isOutdated: false` — the normal "active" state.

Triage all four states. Collapse only `isResolved: true`.

## Rate limits

A single PR triage is usually 4-6 API calls: 1 metadata + 1 GraphQL threads + 1 timeline + 1 reviews + 1 diff + optional name-only diff. Well within authenticated limits (5000 req/hr REST, 5000 points/hr GraphQL).

Paginated calls on large PRs (300+ comments) may add 2-3 more calls. Still safe.

If `gh api` returns a 403 with `X-RateLimit-Remaining: 0`:
- Stop immediately.
- Report the reset time (from `X-RateLimit-Reset`, epoch seconds) to the user.
- Do not retry in a loop.

## Authentication edge cases

- `gh auth status` fails → tell the user to run `gh auth login`. Stop.
- PR is in a private repo the user lacks access to → `gh pr view` 404s. Stop with a clear message.
- `gh pr checkout` fails on a fork due to missing fork remote → fall back to refuse-until-correct with the manual `git fetch <fork-url> <branch>:<local-branch> && git checkout <local-branch>` commands.
