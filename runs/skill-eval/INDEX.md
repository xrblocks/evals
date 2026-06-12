# skill-eval runs

Does the `xb-*` skill content move the needle on idiomatic xrblocks code generation? Each row is one full sweep. The harness lives at [`evals/`](https://github.com/google/xrblocks/tree/main/evals) in [google/xrblocks](https://github.com/google/xrblocks); each run pins the exact harness commit.

| run_id | SDK | date | arm | models | Δ api_match (pro / flash) | Δ composite (pro / flash) | source |
| --- | --- | --- | --- | --- | ---: | ---: | --- |
| [skill-eval-v0.15.0-2026-06-jun-06-system-prompt](v0.15.0/2026-06-jun-06-system-prompt/) | v0.15.0 | 2026-06-06 (Jun 6) | system-prompt | pro, flash | +0.95 / +0.90 | +0.35 / +0.36 | [#351](https://github.com/google/xrblocks/pull/351) |

## What the metrics mean

The scorer (`evals/prototypes/score_proto.py` in the harness) produces a 0-1 score per dimension and a composite mean:

| metric              | meaning                                                                                                             |
| ------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `import_match`      | fraction of `expected_imports` the generated file referenced (skipped, not zeroed, when `expected_imports` is empty) |
| `api_match`         | fraction of `expected_apis` substrings present anywhere in the generated file                                       |
| `forbidden_clean`   | `1` if no `forbidden_patterns` regex matched, else `0`                                                              |
| `parse_ok`          | `1` if `node --check` parses the generated file                                                                     |
| `composite`         | mean of the four above                                                                                              |

The judge (`evals/prototypes/judge.py`) reads the generated file with the full `SKILL.md` as ground truth:

| judge field              | scale                                                                  |
| ------------------------ | ---------------------------------------------------------------------- |
| `accomplishes_task`      | 1-5: does the output actually solve the user's task?                   |
| `idiomatic_xrblocks`     | 1-5: does it use the SDK the way the skill content recommends?         |
| `hallucination_severity` | `none` / `minor` / `major`: did the model invent APIs that don't exist? |

Run-level rollups live in each run's `derived_summary` block; raw JSONs under `results/` are the source of truth.

## Arms planned

- `system-prompt` ← this one, mirrors the Canvas Gem deployment.
- `agentic-listskills` — give the agent `listSkills` / `getSkill` tools and let it pick (per [dli7319 on #351](https://github.com/google/xrblocks/pull/351#issuecomment-4651537773)).
- `full-baseline` — use `https://xrblocks.github.io/prompts/` as the baseline prompt (per [dli7319 on #351](https://github.com/google/xrblocks/pull/351#issuecomment-4651537773)).
