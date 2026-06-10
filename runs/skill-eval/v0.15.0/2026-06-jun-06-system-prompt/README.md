# v0.15.0 — June 6, 2026 — system-prompt arm

Run ID: `skill-eval-v0.15.0-2026-06-jun-06-system-prompt`. Machine-readable manifest: [`manifest.json`](manifest.json).

## TL;DR

- First full sweep of the skill eval harness across 20 tasks × {gemini-2.5-pro, gemini-2.5-flash} × {with-skill, without-skill}, judged by gemini-2.5-pro.
- With-skill `api_match` is 0.93-1.00, without-skill is 0.03-0.07. The `SKILL.md` content is what makes the difference between idiomatic xrblocks code and made-up APIs.
- Without the skill, flash produces a `major` hallucination on every task and pro on 17/20. The patterns are consistent (jsx-like custom elements, A-Frame-style names, npm packages that don't exist).
- Surfaced a few concrete observations about which skills still leak hallucinations even when loaded; see Findings.

## Findings

- **xb-modelviewer skill misses some `xb.user` boundaries.** Even with the skill loaded, both models still hallucinate `xb.user.height` / `xb.user.objectDistance` / a generic `xb.user` positioning helper. The skill correctly directs them to `xb.ModelViewer`, but the rationale field on both `modelviewer-gltf-with-skill` judge runs cites invented `xb.user.*` calls for placement. Skill could close the loop with an explicit example of where to put the model.
- **Flash misses adjacent setup on `xb-hands` even with the skill.** `hands-pinch-spawn` lands at composite `0.78` (api_match `0.33`) because flash calls the headline API but skips the rest of the recipe. `canvas-pet-rock` (uses `xb-ai`) similarly drops to `0.89`. Both look like cases for a "minimum viable setup" snippet near the top of the affected skills.
- **`xb-netblocks` skill shipped a broken import path.** Directory imports like `xrblocks/addons/netblocks/src` 404 on `@build` because browser importmaps don't do node-style index resolution. The eval's smoke runner caught this during harness development (not this sweep specifically); fixed in [google/xrblocks#349](https://github.com/google/xrblocks/pull/349). [#350](https://github.com/google/xrblocks/pull/350) applied the same fix to the lipsync skill, which doesn't yet have an eval task.
- **Canvas-faithful prompts widen the skill gap, not narrow it.** Vague user-style prompts force the model to lean on system-prompt priors, which is exactly what the skill content is. The harness was originally going to drop the canvas-faithful arm; this run argues for keeping it.

## Numbers

### api_match (fraction of `expected_apis` called)

| model | prompt style | with skill | without skill | Δ |
| --- | --- | ---: | ---: | ---: |
| pro | engineer-spec | 1.00 | 0.07 | **+0.93** |
| pro | canvas-faithful | 1.00 | 0.03 | **+0.97** |
| flash | engineer-spec | 0.93 | 0.07 | +0.87 |
| flash | canvas-faithful | 0.97 | 0.03 | +0.93 |

### composite (mean of import_match, api_match, forbidden_clean, parse_ok)

| model | prompt style | with skill | without skill | Δ |
| --- | --- | ---: | ---: | ---: |
| pro | engineer-spec | 1.00 | 0.65 | +0.35 |
| pro | canvas-faithful | 1.00 | 0.65 | +0.36 |
| flash | engineer-spec | 0.98 | 0.65 | +0.33 |
| flash | canvas-faithful | 0.99 | 0.61 | **+0.38** |

### hallucination severity (judge: gemini-2.5-pro)

| model | with skill (none / minor / major) | without skill (none / minor / major) |
| --- | :---: | :---: |
| pro | 15 / 1 / 4 | 3 / 0 / **17** |
| flash | 13 / 2 / 5 | 0 / 0 / **20** |

## Compared to previous runs

N/A — this is the first run for this suite.

## Reproduce

The harness at `cef1fe1` did not yet have a `requirements.txt`, and `plot.py` at that commit wrote PNGs. The recipe below reflects what the actual sweep used, with the PDF-rendering step added afterwards.

```bash
git clone https://github.com/google/xrblocks
cd xrblocks
git checkout cef1fe1
pip install google-genai matplotlib numpy playwright
export GEMINI_API_KEY=...

# Pro sweep + judge.
GEMINI_MODEL=gemini-2.5-pro ./evals/run_all.sh --judge

# Flash sweep + judge.
GEMINI_MODEL=gemini-2.5-flash ./evals/run_all.sh --judge

# Charts. cef1fe1's plot.py emits PNGs; for PDFs apply this one-line tweak first:
#   sed -i.bak 's|\.png|.pdf|g' evals/plot.py
python3 evals/plot.py

# Summary table.
python3 evals/summarize_proto.py
```

Outputs land under `evals/results/<model>/{with-skill,without-skill,judge}/` and `evals/charts/`.

## Config

- **SDK npm version at run time**: `xrblocks@0.15.0` (published 2026-05-22). Skills used were at xrblocks commit `cef1fe1`, which had unpublished improvements on top of the v0.15.0 tag.
- **Tasks**: 20, split 10 engineer-spec / 10 canvas-faithful.
- **Trials per cell**: 1. The harness README warns ±0.1 swings between samples on flash.
- **Token usage and raw model outputs**: not captured in this run; future runs should archive both.
- Full machine-readable config: [`manifest.json`](manifest.json).

## Charts + raw

- [`charts/`](charts/) — six vectorized PDFs
- [`results/_summary.md`](results/_summary.md) — markdown summary table
- [`results/gemini-2.5-pro/`](results/gemini-2.5-pro/) and [`results/gemini-2.5-flash/`](results/gemini-2.5-flash/) — per-task scorer + judge JSONs
