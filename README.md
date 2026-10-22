# model-router

An agent skill that maps **work roles** (planner, builder, reviewer, vision, cheap subagents) to the **free models currently available on OpenCode Zen**, flags which ones are **open source / open weights**, and generates ready-to-use OpenCode agent config with fallbacks.

> Unofficial community project. Not affiliated with or endorsed by OpenCode, Anomaly, or any model vendor.

## Why

OpenCode's free model list changes constantly. Many models are "limited time", get swapped out, or are stealth aliases. This skill does not hardcode a lineup. It makes the agent:

1. Check which free models are live right now.
2. Ask what the project needs (context size, task types, code sensitivity, open-source requirement).
3. Assign models to roles, using a different maker for the reviewer than for the builder.
4. Write `opencode.json` or `.opencode/agents/*.md`, with a fallback chain.

## Install

Copy the skill folder into your OpenCode skills directory.

```bash
# Global
mkdir -p ~/.config/opencode/skills
cp -r skills/model-router ~/.config/opencode/skills/

# Or per project
mkdir -p .opencode/skills
cp -r skills/model-router .opencode/skills/
```

Skill discovery paths can differ between OpenCode versions. If the skill is not picked up, check the skills section of the current OpenCode docs.

## Usage

Invoke it by name (`model-router`, e.g. `/model-router` where slash commands are supported) or just ask your agent, for example:

- "Set up free models for this project and tell me which are open source."
- "Pick a builder and a reviewer from the free OpenCode models."
- "Laguna is rate-limited, re-map my agents."

For a step-by-step walkthrough, see [docs/USAGE.md](docs/USAGE.md).

## Compatibility (works outside OpenCode?)

Partly. The skill file uses the common `SKILL.md` layout (YAML frontmatter with `name` and `description`, then Markdown instructions), so other agents that load skills in that format can read it. What carries over depends on the step:

| Part of the skill | Outside OpenCode |
|---|---|
| Steps 2 and 4 (understand the project, role-to-model map) | Works anywhere. It is plain guidance. |
| Step 3 (catalog: makers, open-weights status, licenses) | Works anywhere. Facts about the models do not depend on the tool. |
| Step 1 (live check) | The `curl` command to the Zen `/models` endpoint works from any shell with network access. The `opencode models` command needs the OpenCode CLI. |
| Step 5 (generate config) | OpenCode-specific: it writes `opencode.json` and `.opencode/agents/*.md`. Other tools need their own config. |
| Model IDs (`opencode/<id>`) | OpenCode-specific prefix. The Zen gateway also exposes an OpenAI-compatible endpoint (`https://opencode.ai/zen/v1/chat/completions`), so other tools can point at it with a Zen API key. Some of these models are also listed as free variants on OpenRouter; check availability there, since free listings change. |

Install location for other tools varies. For example, Claude Code reads skills from `~/.claude/skills/<name>/SKILL.md`. Check your tool's documentation for its path. If a tool has no skill loader, paste the contents of `SKILL.md` into its system prompt or project instructions.

The skill is tested only against the OpenCode docs, not run end to end in other tools. If you adapt Step 5 for another tool, a pull request is welcome.

## What is inside

| Path | Purpose |
|---|---|
| `skills/model-router/SKILL.md` | The skill (English) |
| `docs/USAGE.md` | Usage manual |
| `docs/PUBLISHING.md` | Maintainer notes for publishing on GitHub |
| `CONTRIBUTING.md` | How to update the model catalog |
| `CHANGELOG.md` | Version history |

## Model catalog accuracy

The catalog in `SKILL.md` is a **snapshot (October 2026)**. Treat it as a starting point and rely on the live check in Step 1 of the skill.

- Benchmark figures are vendor claims, not independent measurements.
- "Open" means open weights with the license shown. It does not imply open training data unless stated.
- Models marked ❓ are not verified. Pull requests that verify them with a primary source are welcome.

### Sources used for the snapshot

- OpenCode Zen docs: https://opencode.ai/docs/zen
- OpenCode agents docs: https://opencode.ai/docs/agents/
- Poolside Laguna S 2.1 release: https://aimagazine.com/globenewswire/3330818
- Cohere North Mini Code: https://cohere.com/north/mini-code
- NVIDIA Nemotron 3 Ultra: https://www.deeplearning.ai/the-batch/nvidias-nemotron-goes-big
- Xiaomi MiMo-V2.5: https://www.opensourceforu.com/?p=98035
- DeepSeek V4 Flash: https://dev.co/ai/llms/deepseek-v4-flash
- Meituan LongCat-2.0: https://www.opensourceforu.com/2026/06/meituan-open-sources-longcat-2-0-under-mit-license/

## Privacy note

Free models may use prompts and outputs for training. The skill reminds the agent to avoid them for client code, credentials, and personal data.

## License

MIT. See [LICENSE](LICENSE).
