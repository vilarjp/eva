---
name: teardown
date: "{{DATE}}"
status: "{{draft | approved}}"
topic: "{{short topic — what is being torn down}}"
scope: "{{LIGHTWEIGHT | STANDARD | DEEP}}"
focus: "{{none | <slug>}}"
target_type: "{{local | url}}"
target: "{{local path OR URL}}"
target_sha256: "{{omit for local files, else sha256 of the fetched bytes}}"
target_size_bytes: {{byte count}}
detected_type: "{{JS module | IIFE bundle | ESM | UMD | HTML | CSS | WASM | other}}"
bundler_signature: "{{Webpack 5 | Rollup IIFE | Vite dev | esbuild | Parcel | esm.sh | none detected | N/A}}"
minified: "{{yes | no | partial}}"
hydration: "{{source-map | beautify | read-as-is | none}}"
hydration_tools: "{{e.g. shuji + prettier, prettier only, webcrack + prettier, none (LLM reasoning)}}"
unverified_formatting: {{false | true}}
sidecars_included: {{true | false}}
origin_acknowledged: "{{true | false | N/A}}"
origin_acknowledgment_note: "{{omit if N/A, else one-line rationale given by the user}}"
spec_directory_reuse: {{false | true}}
original_spec_dir: "{{omit unless spec_directory_reuse is true, else: docs/YYYY-MM-DD-<slug>/}}"
---

# Teardown: {{topic}}

## Summary

{{5-8 lines. What this file is, what it does, how it does it, what's surprising, what depends on it. A reader who stops here should have an accurate mental model. No adjectives like "clean", "robust", "elegant" — only what the code does.}}

## Inputs & Hydration

| Dimension | Value |
|-----------|-------|
| Target | `{{local path | URL}}` |
| Size | {{n}} bytes / {{n}} lines |
| Detected type | {{JS module / IIFE bundle / ESM / UMD / HTML / CSS / WASM / other}} |
| Bundler signature | {{Webpack 5 / Rollup IIFE / Vite dev / esbuild / Parcel / none detected}} |
| Minified? | {{yes | no | partial}} |
| Source map | {{none | declared via `//# sourceMappingURL=` | sibling .map | N/A}} |
| Hydration applied | {{source-map → beautify | beautify only | read-as-is | none}} |
| Hydration tools | {{e.g. shuji + prettier, prettier only, wakaru (Deep), none}} |
| Sidecars | {{list of files saved under `teardown-sources/`, or "none"}} |

{{Optional paragraph: noteworthy quirks detected during input resolution — e.g. "obfuscator.io signatures present", "source map references paths that no longer exist", "HTML page had 5 external scripts, user selected the 312 KB main bundle".}}

## Structural Map

### Entry points

- {{what runs on load — IIFE invocation, top-level export, module.exports, AMD define, window assignment, SW install, etc. — with `file:line`}}

### Module layout

_Omit if not bundled._

| Module id / name | Hydrated path (if any) | Role |
|------------------|------------------------|------|
| {{e.g. `a` / `src/auth/session.ts`}} | `teardown-sources/hydrated/src/auth/session.js` | {{one-sentence role}} |
| ... | ... | ... |

### Function inventory

| Function | Signature | Location | Role |
|----------|-----------|----------|------|
| `{{name \| anonymous+fingerprint}}` | `{{(args) → return}}` | `{{file:line \| beautified:line}}` | {{one-sentence role}} |
| ... | ... | ... | ... |

_Elide trivial internal helpers; note the elision — e.g. "Omitted 12 single-use inline lambdas from the renderer path."_

### State inventory

| Binding | Scope | Location | Lifecycle |
|---------|-------|----------|-----------|
| `{{name}}` | {{module / closure / global}} | `{{file:line}}` | {{created when / mutated by / disposed by}} |

## Behavior Walkthrough

### Happy path

1. {{step — name called function with location}}
2. {{step}}
3. ...

### Branches

- {{non-happy conditional arm — semantics, not syntax, with location}}
- {{...}}

### Error paths

- {{try/catch or rejection handler: where it is, what it catches, what it does — log / swallow / rethrow}}

### Event surfaces

- {{DOM listener, emitter, lifecycle hook — when it fires, what it triggers}}

## Data Flow

### Inputs

- {{function args, fetch responses, DOM reads, storage reads, URL params, etc. — one line each, with location}}

### Transformations

- {{shape / validate / merge / normalize / serialize / encrypt / cache steps, in order, with locations}}

### Outputs

- {{network request (method, URL template, payload shape), DOM write, storage write, event emit, return value}}

### Coupling / fan-out

- {{depends on — globals, imported modules, external APIs}}
- {{depended on by — if inferable from the bundle; otherwise omit}}

### Diagram

_Required for Standard/Deep. Prefer `flowchart` for data paths, `sequenceDiagram` for multi-actor interaction, `stateDiagram-v2` for state machines._

```mermaid
{{mermaid block — flowchart / sequenceDiagram / stateDiagram-v2}}
```

_Caption: {{one-line description of what the diagram shows}}._

## Edge Cases & Error Paths

| Scenario | Trigger | Code location | Behavior |
|----------|---------|---------------|----------|
| {{e.g. empty cart during coupon apply}} | {{e.g. `applyCoupon` called before cart resolves}} | `{{file:line}}` | {{silent bail / error log / user-facing message / retry / rethrow}} |
| ... | ... | ... | ... |

## External Surface

### Network

| Method | URL (template) | Auth | Payload shape (inferred) |
|--------|----------------|------|--------------------------|
| POST | `https://api.example.com/v1/auth/challenge` | Bearer header | `{ challengeId, nonce }` |
| ... | ... | ... | ... |

### DOM

- {{selectors queried, elements created, attributes mutated, events dispatched — with location}}

### Storage

- {{localStorage / sessionStorage / IndexedDB / cookies — keys, read/write sites, TTL assumptions}}

### Workers / offloaded work

- {{web workers, service workers, offloaded Promises, timers, etc. — if present}}

## Focus Slice

_Omit unless focus mode is active._

**Focus:** `{{focus slug}}`

{{Narrowed analysis. Re-runs of Phases 3-5 scoped to the focus region. Cross-links to full behavior where relevant, with `file:line` pointers for readers who want to expand.}}

## Unknowns & Residual Risk

### Unknowns

| Region | Location | Why unverified |
|--------|----------|----------------|
| {{identifier / function / path}} | `{{file:line}}` | {{minified past extraction / obfuscated control flow / external state not reachable / no source map / unrecognized bundler runtime / …}} |

### Residual risk

- **If a reader acts on {{unknown 1}}, {{consequence}}.** {{One-paragraph elaboration — what could go wrong, what the reader should verify independently before acting.}}
- **If a reader acts on {{unknown 2}}, {{consequence}}.**

### Questions a reader might still have

- {{question}}
- {{question}}

## References

- `teardown-sources/beautified/{{basename}}.js` — the beautified target (only if sidecars saved)
- `teardown-sources/hydrated/` — source-map-hydrated modules (only if map was found and sidecars saved)
- `teardown-sources/renamed/` — decompiled / identifier-renamed output (Deep scope, only if Rung 3 ran)
- {{other teardowns, PRD, SPEC, or DIAGNOSIS in sibling folders, with file links}}

## Relationship to Original Spec

_Include ONLY when `spec_directory_reuse: true`. Otherwise delete this section._

**Related spec directory:** {{original_spec_dir}}

**How this teardown informs that feature:**

- {{e.g. "the competitor's auth flow confirms our planned retry strategy is within the same latency envelope"}}
- {{e.g. "the torn-down bundle is the reference implementation we're porting — Option 2 in SPEC.md is a direct adaptation"}}

**Forward-reference added to:** {{SPEC.md | PRD.md}} — `## Teardown <DATE>` section.
