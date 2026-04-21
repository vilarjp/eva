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

{{5-8 lines. What this file is, what it does, how it does it, what's
surprising, what depends on it. A reader who stops here has an accurate mental
model. No adjectives — only what the code does.}}

## Inputs & hydration

| Dimension         | Value                                                        |
|-------------------|--------------------------------------------------------------|
| Target            | `{{local path / URL}}`                                       |
| Size              | {{n}} bytes / {{n}} lines                                    |
| Detected type     | {{JS module / IIFE bundle / ESM / UMD / HTML / CSS / WASM}}  |
| Bundler signature | {{Webpack 5 / Rollup IIFE / Vite dev / esbuild / none}}      |
| Minified          | {{yes / no / partial}}                                       |
| Source map        | {{none / declared / sibling .map / N/A}}                      |
| Hydration         | {{source-map → beautify / beautify only / read-as-is}}       |
| Tools             | {{shuji + prettier / prettier / wakaru / none}}              |
| Sidecars          | {{list under `teardown-sources/` or "none"}}                  |

{{Optional line on noteworthy input quirks — e.g. "obfuscator.io signatures",
"source map references missing paths". Cut if nothing surprising.}}

## Structural map

**Entry points:** {{what runs on load — IIFE invocation / top-level export /
module.exports / AMD define / window assignment — with `file:line`}}.

### Module layout

_Omit if not bundled._

| Module id          | Hydrated path                       | Role                          |
|--------------------|-------------------------------------|-------------------------------|
| `{{a / src/auth/session.ts}}` | `teardown-sources/hydrated/...` | {{one-line role}}      |

### Function inventory

| Function         | Signature            | Location              | Role                       |
|------------------|----------------------|-----------------------|----------------------------|
| `{{name}}`       | `{{(args) → ret}}`   | `{{file:line}}`       | {{one-line role}}          |

Elide trivial helpers; note the elision in a single line if ≥ 5 cut.

### State inventory

| Binding    | Scope                    | Location         | Lifecycle                             |
|------------|--------------------------|------------------|---------------------------------------|
| `{{name}}` | {{module/closure/global}}| `{{file:line}}`  | {{created / mutated by / disposed}}   |

## Behaviour walkthrough

Prose is fine here — this is the part humans read cold.

**Happy path:**

1. {{step — named function + location}}
2. {{step}}

**Branches, error paths, event surfaces:** {{one line each with location.
Group as bullets; skip the sub-heading if fewer than 3 items total.}}

## Data flow

| Direction       | What                                      | Location         |
|-----------------|-------------------------------------------|------------------|
| In              | {{args / fetch / DOM read / storage / URL param}} | `{{file:line}}`  |
| Transform       | {{shape / validate / normalise / encrypt / cache}} | `{{file:line}}`  |
| Out             | {{network / DOM write / storage / event / return}} | `{{file:line}}`  |
| Coupling        | {{globals / imports / external APIs — depends on / depended on by}} | `{{file:line}}`  |

### Diagram

_Mandatory on Deep; offered on Standard when flow is non-linear._

```mermaid
{{flowchart / sequenceDiagram / stateDiagram-v2}}
```

## External surface

### Network

| Method | URL (template)                          | Auth          | Payload (inferred)       |
|--------|-----------------------------------------|---------------|--------------------------|
| POST   | `https://api.example.com/v1/...`        | Bearer header | `{ field, field }`       |

### DOM / storage / workers

- **DOM:** {{selectors queried, elements created, events dispatched — with location}}
- **Storage:** {{local/session/IndexedDB/cookies — keys, R/W sites, TTL}}
- **Workers:** {{web / service workers, offloaded work — if present}}

## Edge cases & error paths

| Scenario                | Trigger                     | Location        | Behaviour                          |
|-------------------------|-----------------------------|-----------------|------------------------------------|
| {{e.g. empty cart}}     | {{`applyCoupon` before cart resolves}} | `{{file:line}}` | {{silent bail / log / user msg / retry / rethrow}} |

## Unknowns & residual risk

| Region                    | Location       | Why unverified                                        | If a reader acts on this… |
|---------------------------|----------------|-------------------------------------------------------|---------------------------|
| `{{identifier / function}}` | `{{file:line}}` | {{past extraction / obfuscated / no source map}}      | {{consequence — what could go wrong}} |

Open questions a reader might still have: {{one line each or "none"}}.

<!--
Optional sections — include ONLY when non-empty:

### Focus slice (focus mode only)

**Focus:** `{{focus slug}}`

{{Narrowed analysis scoped to the focus region, cross-linked to full behaviour
with `file:line` pointers.}}

### References

- `teardown-sources/beautified/{{basename}}.js` — beautified target (if saved)
- `teardown-sources/hydrated/` — source-map-hydrated modules (if map found)
- `teardown-sources/renamed/` — decompiled / renamed output (Deep only)
- {{sibling teardowns / PRD / SPEC / DIAGNOSIS links}}

### Relationship to original spec (only when spec_directory_reuse: true)

**Related spec directory:** `{{original_spec_dir}}`.

- {{how this teardown informs that feature — one line each}}

**Forward-reference added to:** `{{SPEC.md | PRD.md}}` — `## Teardown {{DATE}}`.

Cut any section above if empty. See _shared/artifact-compactness.md.
-->
