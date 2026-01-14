# External configuration (config.json)

ERCore supports an optional external JSON configuration file, enabled explicitly via the CLI.

This is intended to make ERCore easy to experiment with and repeat runs, while keeping a predictable override model.

---

## Enable external config

Use `--config` to load a JSON config file:

ercore --input data.csv --config examples/config.min.json

ERCore parses external config **strictly**:
- Unknown fields cause an error
- Invalid JSON causes an error
- Trailing non-whitespace data causes an error

This is intentional: it catches typos early and keeps configs reliable.

---

## Precedence and merge rules

Configuration precedence:

defaults < --config file < CLI flags

In other words:
- ERCore starts from internal defaults
- if `--config` is provided, those fields override defaults
- CLI flags override both

### “Unset” vs “explicitly set”
In the external JSON config:
- If a field is **omitted**, it does not override anything (defaults remain)
- If a field is **present**, it overrides — even if it’s an empty list

This matters for lists like `blockOn` and `matchOn`:
- `"blockOn": []` means “explicitly empty”
- omitting `blockOn` means “leave whatever was there”

---

## Minimal example

See: `examples/config.min.json`

A minimal config usually only includes:
- threshold tuning
- whether to allow auto inference
- optional output behaviour

---

## Full example

See: `examples/config.full.json`

The full example shows the commonly adjustable knobs without requiring you to know internal defaults.

---

## Top-level config fields

### Inputs / outputs

- `inputPaths` (array of strings)
  - Optional. If set, overrides inputs.
  - Most users should prefer `--input` on the CLI instead.

- `outputDir` (string)
  - Output directory (default: `out`)

- `baseName` (string)
  - Output basename (default: `clusters`)

- `outputType` (string)
  - `json` | `csv` | `both` (default: `both`)
  - NOTE: `writeJSON` and `writeCSV` are derived from `outputType` during normalize.

- `writeSummary` (bool)
  - Whether ERCore should emit a human-readable run summary.
  - Default is true in internal defaults.

### Matching

- `mode` (string)
  - `er` | `dedupe`

- `threshold` (number)
  - Match threshold in range [0,1]
  - Useful for quick experimentation and diagnostics

- `blockOn` (array of strings)
  - Explicit blocking fields to use (column names).
  - If provided and non-empty, ERCore treats this as a command (auto-inference is skipped).

- `matchOn` (array of strings)
  - Match fields to use (column names).
  - If omitted/empty, ERCore may infer match keys depending on the engine flow.

---

## Engine settings

Engine settings live under:

engine.runStrategy

### runStrategy fields

- `batchSize` (int)
- `maxRecords` (int64)
- `maxNullOrEmptyRatio` (float)
- `minDistinctRatioForKey` (float)
- `deprioritiseRandomNumeric` (bool)
- `allowAutoKeyInference` (bool)
- `trimAndClean` (object)

### trimAndClean fields

- `trimWhitespace` (bool)
- `collapseWhitespace` (bool)
- `toLower` (bool)
- `stripQuotes` (bool)

### columnOptions

Column options allow you to provide per-column hints.

Keys are column names, values are objects with the following fields:

- `exclude` (bool)
- `match_mode` (string)  
- `match_strategy` (string)
- `prefer_for_blocking` (bool)
- `weight` (number)

Note the field names: they are **snake_case** (e.g. `prefer_for_blocking`), and config parsing is strict.

---

## Blocking key inference behavior

Blocking keys are determined in this order:

1) If `blockOn` is provided and non-empty  
   → ERCore uses those fields as the blocking key(s) (treated as a command).

2) Else if `engine.runStrategy.allowAutoKeyInference` is true  
   → ERCore profiles the dataset and suggests blocking specs automatically.

3) Else  
   → ERCore errors:

no blocking keys provided and auto-inference is disabled (set engine.runStrategy.allowAutoKeyInference=true or provide blockOn)

This is intentional: it keeps runs explicit and predictable.

## Match key inference behavior

Match keys are determined in this order:

1) If `matchOn` is provided and non-empty  
   → ERCore uses those fields as match keys (treated as a command).

2) Else  
   → ERCore profiles the dataset and suggests match key specs automatically.

For small datasets, ERCore uses a more permissive “small profile” configuration:
- if total rows < 50 → `DefaultMatchKeyInferenceConfigSmall()`
- else → `DefaultMatchKeyInferenceConfig()`

Note: in v0.1, match key auto-inference currently runs whenever `matchOn` is empty.  
There is no “disable inference and error” branch for match keys yet (unlike blocking keys).

---

## Common pitfalls

### “My config field didn’t apply”
If a field is omitted, it won’t override defaults.  
If you intended to override, include the field explicitly.

### “Config parsing fails”
ERCore uses strict decoding. Common causes:
- wrong field name (e.g. `matchOn` vs `match_on`)
- wrong nesting (e.g. `runStrategy` not under `engine`)
- trailing commas or invalid JSON

---

## Recommended workflow

- Start with `config.min.json`
- Use `--threshold` on the CLI to iterate quickly
- When stable, move that value into config
- Only graduate to `config.full.json` when you need deeper tuning
