# Hydration Pipeline — From minified bytes to readable code

The goal is to maximize the amount of teardown that's anchored to real, verifiable identifiers — and to honestly label what couldn't be recovered. Every tool in this file is optional; the pipeline degrades cleanly.

## The ladder (JS/TS only)

Run top to bottom. Stop at the first rung that yields readable code. Record what you used in the TEARDOWN frontmatter so readers can calibrate trust.

### Rung 1 — Source-map discovery

Source maps are the biggest win: they restore original module paths, function names, and line numbers. Many production bundles publish them (intentionally or by default).

**Check, in order:**
1. Trailing comment in the target file: `//# sourceMappingURL=<relative-or-absolute-URL>`.
2. Sibling file at the same URL / filesystem path: `<target>.map`, `<target-without-extension>.map`, `<basename>.map`.
3. For URLs, common dev-server paths: `<origin>/.well-known/source-maps/<path>`, `<origin>/<hashed-bundle>.map`.

**When found:**
- For URL-fetched maps: fetch the `.map`, validate it is a valid Source Map v3 JSON (`"version": 3`, `"sources"`, `"mappings"`).
- Use `shuji` to reconstruct the original source tree:
  ```
  npx --yes shuji@2 <target>.map -o teardown-sources/hydrated/
  ```
- If `shuji` is unavailable: the map's `sources` + `sourcesContent` arrays already contain the original bodies — unpack them manually to `teardown-sources/hydrated/` with `jq` + file-per-source.

**When NOT found:** proceed to Rung 2. Do not invent a map.

### Rung 2 — Beautification

Turns `function a(b,c){return b+c}` into indented, multi-line code that both a human and an LLM can read.

**Preferred order:**
1. `npx --yes prettier@3 --parser babel --print-width 100 <target> > teardown-sources/beautified/<basename>.js`
2. If Prettier errors (old syntax, JSX fragments with TS quirks, unrecognized features): `npx --yes js-beautify@1 <target> > teardown-sources/beautified/<basename>.js`
3. If `npx` is unavailable entirely: fall back to LLM internal reasoning. Set `unverified_formatting: true` in frontmatter so readers know line references are ephemeral.

**Do not:** swap semicolons for no-semicolons, reorder imports, or apply ESLint fixes. Beautification must be byte-equivalent to the input — only whitespace changes. Any transformation beyond formatting is a Rung 3 step, which is surfaced to the user.

### Rung 3 — Decompilation / identifier renaming (Deep scope, optional)

Only for obfuscated or heavily-minified bundles where Rungs 1-2 leave you with code still dominated by single-letter identifiers.

**Tools worth trying (cheapest first):**

- **`webcrack`** — fast; targets Webpack unpacking and obfuscator.io reversal.
  ```
  npx --yes webcrack <target> -o teardown-sources/renamed/
  ```
- **`wakaru`** — modern JS decompiler; unpacks bundled output from Webpack / Browserify; un-transpiles Babel / SWC / TS / Terser transformations.
  ```
  npx --yes @wakaru/cli <target> -o teardown-sources/renamed/
  ```
  For small files, the [Wakaru Playground](https://wakaru.vercel.app/) is a fast validator before automating.
- **`humanify`** — LLM-driven identifier renaming. Does not change structure, only provides rename hints. Use with a local model if the code is private. Slow; best on a critical slice, not a full bundle.

**Tool-selection heuristic:**
- Webpack signature present → `webcrack` first, then `wakaru`.
- Rollup IIFE / ESM / esbuild → `wakaru`.
- Heavy obfuscator.io features (string-array, control-flow flattening) → `webcrack`; if it can't reverse the specific obfuscation, fall back to `humanify` on just the region you care about.

**Always:**
- Surface the tool choice to the user before running — these commands install packages via `npx`.
- Save renamed output under `teardown-sources/renamed/` alongside `beautified/` so the reader can compare.
- Never treat an identifier rename from `humanify` or `wakaru` as ground truth without cross-referencing call sites.

### Rung 4 — Fallback: read as-is

If nothing above produces readable code, narrate what you can — and flag every internal-identifier claim `(unverified)`. Prefer fewer strong claims over many weak ones.

## Non-JS/TS targets

- **HTML.** Parse structure (head / body), enumerate scripts and stylesheets, walk inline event handlers. No beautification step — HTML formatting rarely helps.
- **CSS.** If minified, `npx --yes prettier@3 --parser css`. Otherwise read as-is. Focus on selectors, custom properties, and pseudo-classes that imply behavior.
- **WASM.** Disassemble with `wasm-tools print` or `wasm2wat` when available (both are installable via `cargo install wasm-tools` or the `wabt` package). If unavailable, read the module's JS wrapper and narrate the exported function signatures; do not narrate the binary directly.
- **Python / Go / Ruby / shell.** No hydration step. Read and narrate. If deliberately obfuscated (encoded strings, base64 blobs), document the obfuscation; do NOT decode without user confirmation — decoding can trigger intended side effects in malicious payloads.

## When to stop

- **Beautified output is still 90%+ single-letter identifiers and Rung 3 tools don't apply or install cleanly** → stop, flag `(unverified)` heavily, document what the pipeline actually produced.
- **Source map lies about paths** (e.g. all modules map to `webpack:///src/index.js:1`, or `sourcesContent` is all `null`) → treat the map as unreliable, fall back to Rung 2, note the map's failure mode in frontmatter.
- **Tool output differs from the input semantically** (e.g. `webcrack` rewrote control flow in a way that changed branching behavior) → reject the rewritten output, stick with beautified + hand-narrated, note the rejection.

## Recording in frontmatter

```yaml
hydration: "source-map"              # or: beautify | read-as-is | none
hydration_tools: "shuji + prettier"  # or: "prettier only" | "webcrack + prettier" | "none (LLM reasoning)"
unverified_formatting: false         # true only when LLM-only beautification was used
```

The reader uses these fields to calibrate trust. Be precise.

## A note on tool availability

The skill does not assume `npx`, `node`, `wasm-tools`, or any specific tool is available. Every rung is best-effort. If a tool is missing, surface it to the user and proceed to the next rung. Do NOT install global tools without explicit user approval — `npx --yes` is acceptable for transient use, anything more persistent is not.
