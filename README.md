# ERCore

**A reference-quality entity resolution engine to start your ER journey with real data.**

ERCore is a small, local-first entity resolution (ER) engine written in Go.  
It is designed to help you *understand*, *test*, and *iterate* on entity resolution strategies using your own datasets — without needing a platform, cloud service, or enterprise tooling.

This is **not** a hosted service or a full ER product.  
It is a practical starting point.

## Why ERCore exists

Entity resolution is rarely a “plug and play” problem.

Before choosing tools, platforms, or architectures, you often need to answer simpler questions:

- What fields actually behave like identifiers in *this* data?
- What happens if I change blocking keys or thresholds?
- Where do false positives and false negatives come from?
- How sensitive is matching to small configuration changes?

ERCore is built to make those questions **visible**.

It runs locally, logs its reasoning, and favors clarity over abstraction.

## What ERCore does

At a high level, ERCore:

1. Profiles your data (single or multiple CSV files)
2. Infers blocking keys when none are provided
3. Builds candidate record pairs via blocking
4. Compares records using simple matching strategies
5. Clusters records into entities

All of this happens locally, on your machine.

## Installation

### Download a binary (recommended)

Go to **Releases** and download the binary for your platform:

- Linux
- macOS (Intel / Apple Silicon)
- Windows

Unpack it and place `ercore` somewhere on your PATH.

Source code is not published in v0.1.  
ERCore is distributed as a binary-only reference engine for now.

## Quick start

### Single file

ercore --input data.csv

Outputs will be written to `./out/` by default.

### Multiple files

ercore --input part1.csv,part2.csv,part3.csv

ERCore profiles across all files and infers blocking keys globally.

### Override matching threshold

ercore --input data.csv --threshold 0.75

## External configuration

ERCore supports an optional external JSON config, activated explicitly:

ercore --input data.csv --config config.min.json

Config precedence:

defaults < external config < CLI flags

- Defaults are always applied
- External config overrides defaults
- CLI flags override both

Invalid or unreadable config files fail fast.

## Blocking key inference

If you do **not** specify blocking keys, ERCore will attempt to infer them based on profiling.

This includes:
- filtering out overly sparse or low-cardinality columns
- avoiding ID-like or near-unique fields as primary blockers
- falling back to safe composite keys when no strong primary exists

If blocking keys are provided explicitly, auto-inference is skipped.

## What ERCore is not

ERCore is intentionally **not**:

- a hosted service
- an enterprise ER platform
- a black box
- a replacement for production-grade ER systems

It is meant to help you **see the terrain** before committing to tooling or architecture.

## Status

ERCore v0.1 is a **reference implementation**.

Expect:
- clear behavior
- explicit trade-offs
- conservative defaults

Future versions may expose more tuning parameters, but v0.1 intentionally favors simplicity.

## License

See LICENSE.

## Feedback

If ERCore helped you think more clearly about your data, that’s success.

Issues and discussions are welcome — especially around:
- surprising inference behavior
- edge cases in real datasets
- ideas for keeping ERCore simple but useful
