# XR Blocks evals

Evaluation runs, charts, and raw scores for [google/xrblocks](https://github.com/google/xrblocks).

Runs are organized by **suite → SDK version → run** so cross-version comparison within a suite is just walking the tree, and new eval suites (demo-quality, agentic-loop, runtime-correctness, ...) can land alongside the existing ones without restructuring.

## Layout

```
runs/
└── <suite>/                          # e.g. skill-eval
    ├── INDEX.md                      # table of all runs in this suite
    └── v<sdk-version>/
        └── <YYYY-MM-mon-DD>-<arm>/   # e.g. 2026-06-jun-06-system-prompt
            ├── manifest.json         # machine-readable run metadata (run_id, commits, models, derived_summary)
            ├── README.md             # human narrative (TL;DR, findings, numbers, reproduce)
            ├── charts/               # vectorized PDFs
            └── results/              # raw scorer + judge JSONs the charts came from
```

## Current suites

- [**`skill-eval/`**](runs/skill-eval/) — does the `xb-*` skill content move the needle on idiomatic xrblocks code generation? Companion to [google/xrblocks#351](https://github.com/google/xrblocks/pull/351).

## Adding a run

1. Run the harness against the SDK version you want to measure.
2. Create `runs/<suite>/v<sdk>/<YYYY-MM-mon-DD>-<arm>/` and drop in `charts/` + `results/`. Example: `2026-06-jun-06-system-prompt/`. The numeric `MM` keeps `ls` sorted chronologically; the `mon` (lowercase 3-letter month) makes the date unambiguous for any reader. Arms are lowercase kebab-case. If you re-run the same arm on the same date, append a discriminator (e.g. `-n3` for an n=3 trial sweep, `-rerun-1`).
3. Write `manifest.json` with at least: `schema_version`, `run_id`, `suite`, `sdk_npm_version`, `harness_commit`, `run_date`, `run_arm`, `agent_models`, `task_suite` (with `count` and `source_path`), `source_pr`. Suite-specific fields (e.g. judge config) live alongside.
4. Write a `README.md` using the template from a recent run (TL;DR → findings → numbers → compared-to-previous → reproduce → config → charts/raw).
5. Append a row to `runs/<suite>/INDEX.md`. Each suite's INDEX defines its own columns because suites measure different things.

## Why PDFs, not PNGs

Vectorized so they stay legible at any zoom, text is selectable, and they're 4-5× smaller than the equivalent PNGs. The repo deliberately doesn't accept binary chart PNGs.

## Conventions

- **Dates in directory names**: `YYYY-MM-mon-DD` (e.g. `2026-06-jun-06`). Numeric `MM` first so `ls` and `git log` stay sorted chronologically; lowercase 3-letter month so the date is unambiguous across UK / US / non-engineer audiences without needing to know ISO 8601.
- **Dates in `manifest.json`**: ISO 8601 `YYYY-MM-DD` in the `run_date` field — the data field is for scripts to parse cleanly.
- **SDK versions**: prefix with `v` (`v0.15.0`). Matches npm tag, sorts cleanly per SDK release.
- **Arm slugs**: lowercase kebab-case (`system-prompt`, `agentic-listskills`, `full-baseline`).
- **`manifest.json`** is the source of truth for run config; per-run `README.md` is the human narrative; `results/` JSONs are the raw data. If a `derived_summary` block in the manifest disagrees with the raw JSONs, the JSONs win.

For what specific metrics mean, see the README of the suite you're reading (e.g. [`runs/skill-eval/INDEX.md`](runs/skill-eval/INDEX.md)).
