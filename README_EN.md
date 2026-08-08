# Ziwei-Liuyao Skills

Chart casting, hexagram generation, and strength calculation all run locally and deterministically — the AI only reads the results and interprets.

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-standard-1f6feb)](https://agentskills.org) [![Node](https://img.shields.io/badge/Node-18%2B-339933)](https://nodejs.org) [![零依赖](https://img.shields.io/badge/zero--dep-pure%20Node-3fb950)](https://nodejs.org) [![License](https://img.shields.io/badge/License-NonCommercial(original)-red)](LICENSE) [![Engine](https://img.shields.io/badge/Chart%20Engine-MIT-9747ff)](https://github.com/Renhuai123/ziwei-doushu)

[Intro](#intro) · [Docs](#docs) · [Features](#features) · [Quick Start](#quick-start) · [Install](INSTALL.md) · [Usage](#usage) · [Knowledge Base](#knowledge-base) · [FAQ](#faq) · [Contributing](#contributing) · [License](#license) · [中文](README.md)

## Intro

Two self-contained agent skills following the Agent Skills open standard, usable by 30+ mainstream agents out of the box:

- **ziwei-agent** (Zi Wei Dou Shu): true-solar-time calibrated chart casting, hour inference from 15 candidates when the birth hour is unknown, formal readings, and verification of year/month/decade/annual progressions (the Zi Wei chart engine is a secondary development based on [Wang Duoyu AI's open-source chart engine](https://github.com/Renhuai123/ziwei-doushu))
- **liuyao-agent** (Liu Yao / Six Lines): hexagram casting via random/manual/number methods (three-coin method — old yin/young yang/young yin/old yang at 1/8, 3/8, 3/8, 1/8), strength calculation, timing (应期) judgment, and perspective switching over the six relatives

The core idea in one sentence: casting, hexagram generation, and strength calculations are computed locally and deterministically — no AI guessing; the AI only reads and interprets the computed facts. Each skill has three parts — a `SKILL.md` entry index (auto-discovered and read by the agent), load-on-demand methodology documents (`references/`), and a single-file zero-dependency calculation script (runs directly on Node 18+, nothing to install).

The author has practiced metaphysics for eight years with intensive hands-on experience. Liu Yao is at about 70-80% of the author's own level; Zi Wei has been repeatedly tested and shows a reasonable accuracy rate.

## Docs

| Doc | What's inside |
|---|---|
| [Zi Wei Dou Shu Skill Intro](docs/ziwei-intro.md) | What it does, strengths, limitations, and roadmap |
| [Liu Yao Skill Intro](docs/liuyao-intro.md) | What it does, strengths, limitations, and roadmap |
| [Personal Insights](docs/insights.md) | The author's personal notes on studying and practicing the arts |

## Features

**Others' AI "calculates" fortunes; here the chart is computed.** Chart casting, the four transformations, strength, and timing — 100% deterministic local computation. The same birth data always yields the same chart, with built-in tools to fetch precise charts for any time — free from hallucination errors, the model focuses purely on inference. This dramatically raises the reasoning ceiling.

**Can't recall the birth hour? The 15-candidate inference is unique.** Elders usually only remember "it was getting dark" or "around when the roosters crowed." Fifteen candidate hours are generated and checked against life events you do remember: vague memories are confirmed before exclusion, strong contradictions are excluded outright, exclusion is final, and locking requires the subject's own confirmation. Other charting software just makes you guess.

**True-solar-time calibration — most charting apps skip this.** Metaphysics speaks of the local time of the birthplace; clocks run on standard time. Correcting by birthplace longitude plus the equation of time — a few degrees of longitude can shift an entire hour. Skip this step, and everything after is wasted.

**Clear school lineage.** Liu Yao casting follows 《增删卜易》 and 《卜筮正宗》 as its core, with partial compatibility with 《易隐》; where schools conflict, the author's years of hands-on practice decide.

**AI dares to fabricate; this project does not.** Interpretation lookups always go through knowledge-base retrieval; what is not found is stated plainly. Evidence is layered, unretrieved time layers get no conclusions, tool failures are not extrapolated — the discipline is written into the protocol, not left to model goodwill.

## Quick Start

```bash
# 1. Copy the skill directories into your agent (global install for Claude Code shown)
mkdir -p ~/.claude/skills
cp -r skills/ziwei-agent skills/liuyao-agent ~/.claude/skills/

# 2. Verify the knowledge-base mode (run from the release package root)
node skills/ziwei-agent/scripts/ziwei-calc.mjs retrieve --query "命宫紫微贪狼"
```

`retrieve` reports its mode: `mode: "complete"` means the full knowledge base is available; `mode: "basic"` falls back to the simple knowledge in `references/`. Note: the Liuyao retrieval knowledge base is not distributed with this package, so Liuyao `retrieve` always reports `basic` (expected — see Knowledge Base); Ziwei reports `complete`. See [INSTALL.md](INSTALL.md) for per-agent install, upgrade, and uninstall instructions.

## Usage

### ziwei-agent (Zi Wei Dou Shu)

| Scenario | Command |
|---|---|
| Chart with known hour | `init --gender <male\|female> --date <YYYY-MM-DD> --time HH:MM`, then take the returned `recommendedCandidateKey` and `chart --candidate-key <key>` |
| Chart with unknown hour | `init` without `--time` → 15 candidates, verify event by event per `references/03-inference-flow.md` |
| Year/month/decade checks | `year-facts --candidate-key <key> --year <YYYY> [--summary]` |
| Interpretation lookup | `retrieve --query <topic>` (e.g. "命宫紫微贪狼") |

### liuyao-agent (Liu Yao)

| Scenario | Command |
|---|---|
| Cast hexagram | `cast --question "..."` (random) · `--lines <6 digits 0-3>` (manual) · `--name <hexagram> [--changed-name <hexagram>] --timestamp` (user-specified) |
| Strength | `strength` (annual / monthly / five_day / intra_day time units) |
| Six-relatives perspective | `perspective` (local relationships relative to a chosen line) |
| Reading | identify the use god (用神) → check strength → examine moves and changes → judge fortune and timing |

Full subcommand parameters, output formats, and the state protocol (`--session` isolation, `history`, `visualize`) are documented in each skill's `references/*-tool-protocol.md`.

## Knowledge Base

- Ships with a public knowledge-base subset (`assets-private/knowledge/`): **Ziwei full retrieval rules** + Liuyao base injection rules
- **The Liuyao retrieval knowledge base (206 retrieval rules) is a private asset and is not distributed with this package** — Liuyao `retrieve` reporting `mode: basic` is expected; base interpretation rules live in `references/06-knowledge-injection.md` (two-stage injection guide) and `05-basic-knowledge.md` (simple knowledge). Ziwei `retrieve` reports `mode: complete`
- Delete the `assets-private/` directory to fall back to basic mode entirely — the feature pipeline stays intact, only interpretation depth is reduced
- To use a full knowledge base: place it in any directory (structure in [docs/knowledge-retrieval.md](docs/knowledge-retrieval.md)) and point to it with the `STARS_KNOWLEDGE_DIR` environment variable or `retrieve --knowledge-dir`. Detection order: `--knowledge-dir` > `STARS_KNOWLEDGE_DIR` > `cwd/assets-private` > script-relative paths
- Knowledge-base changes take effect immediately: scripts read from `manifest.json` at runtime — no need to rebuild anything

## FAQ

| Question | Answer |
|---|---|
| `retrieve` reports `basic`? | The knowledge directory was not found. Check that `assets-private/knowledge/manifest.json` exists, or set `STARS_KNOWLEDGE_DIR` to the knowledge root |
| Reproduce a cast? | Pass `--seed <integer>` — the same seed always yields the identical hexagram; without a seed, casting uses true three-coin randomness |
| Where is the state stored? | `.stars-state/` inside each skill directory (sessions, archives, visualizations) — safe to delete anytime |
| Multiple readings mixed up? | Use `--session <name>` to isolate, or `reset` to clear |
| How to upgrade? | Re-copy the skill directories, overwriting the previous ones |

## Contributing

- Bugs and suggestions: open an issue
- Docs and knowledge base: edit `skills/`, `docs/`, `assets-private/knowledge/` directly; after changing knowledge files, update the `files` list in `manifest.json` (see docs/knowledge-retrieval.md)
- Calculation scripts: the scripts in this package are build artifacts; source code and the build pipeline live in the upstream repository

## License

**Dual license**:

- **Original work (by linjiang)** (Liu Yao engine, agent skills, knowledge base, docs): [PolyForm Noncommercial License 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0) — free for personal use; **any form of commercial use is strictly prohibited**. Commercial use requires separate written authorization from the author
- **Zi Wei chart engine**: a secondary development based on [Wang Duoyu AI's open-source chart engine](https://github.com/Renhuai123/ziwei-doushu), under its MIT license

See [LICENSE](LICENSE) for details.
