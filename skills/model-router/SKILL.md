---
name: model-router
description: Choose and configure free OpenCode (Zen) models per role and project need (planning, coding, review, multimodal, cheap subagents), with open-source status and fallbacks. Use when starting a new project in OpenCode, when writing opencode.json or .opencode/agents/, or when a free model is rate-limited or removed.
---

# Model Router

Goal: map **work roles** to **the free models available right now**, then produce a ready-to-use OpenCode agent configuration. OpenCode's free model list changes fast (many are "limited time"), so **do not trust the catalog in this file until you have re-checked it** (Step 1).

## Step 1 — Check which free models are live NOW

Run one of:

```bash
# Via the OpenCode CLI
opencode models | grep '^opencode/'

# Or via the Zen endpoint (add an Authorization header if required)
curl -s https://opencode.ai/zen/v1/models | jq -r '.data[].id' | grep -E -- '-free$|^big-pickle$'
```

Rule: a free model has an ID ending in `-free`, or is `big-pickle`. In config, IDs are written `opencode/<model-id>`.
A model in the table below that is missing from the check result has been removed. A model in the result but not in the table is new: look up its model card (Hugging Face) to determine its license before labeling it open source.

## Step 2 — Understand the project

Ask, or infer from the repo:
1. Context size needed (large repo, many files, long documents?)
2. Dominant task types: architecture/planning, implementation, refactoring, terminal/CLI debugging, review, documentation, image/UI input.
3. Code sensitivity. **Free models may use prompts and outputs to train models** (OpenCode states this for some free models, e.g. Big Pickle). For client code, credentials, or personal data, use a paid provider or a self-hosted model.
4. Is open source / open weights required (self-hosting, auditing, license compliance)?

## Step 3 — Free model catalog (snapshot October 2026, verify first)

"Open" column: ✅ = open weights with the license shown; ❓ = not verified; ❌ = closed or undisclosed identity. "Open weights" does not mean open training data unless stated.

| ID (`opencode/…`) | Maker | Open | License | Best for | Notes |
|---|---|---|---|---|---|
| `nemotron-3-ultra-free` | NVIDIA | ✅ | OpenMDW-1.1 | Planner, architecture, agentic reasoning | ~550B MoE; weights, data, and recipes released; fast; top US open-weight model on Artificial Analysis, behind Kimi K2.6 |
| `deepseek-v4-flash-free` | DeepSeek | ✅ | MIT | Fast generalist, long context | ~284–304B MoE (~13B active), 1M context |
| `longcat-2.0-free` | Meituan | ✅ | MIT | Agentic coding, large repos | 1.6T MoE, 1M context; SWE-bench Pro 59.5 per Meituan (vendor claim) |
| `laguna-s-2.1-free` | Poolside | ✅ | OpenMDW-1.1 | Implementation, terminal tasks | 118B MoE (8B active); Terminal-Bench 2.1 70.2% per Poolside (vendor claim); 262K context on OpenRouter |
| `north-mini-code-free` | Cohere | ✅ | Apache 2.0 | Cheap fast coding subagent, small tasks | 30B MoE (3B active), 256K context; very fast but verbose, weak on non-coding agentic tasks |
| `mimo-v2.5-free` | Xiaomi | ✅ | MIT | Multimodal input (image/video/audio), generalist | 310B omnimodal MoE; up to 1M context |
| `ling-3.0-flash-free` / `ling-3.0-tiny-free` | (Ling) | ❓ | ❓ | Light tasks: summarize, format, rename | Check the model card before claiming open source |
| `big-pickle` | Undisclosed | ❌ | — | General coding fallback | Stealth model; the model behind the alias may change; 200K context, 32K output; data may be used for training |
| `exo-free`, `space-bunny-free`, `step-5-preview-free`, `muse-spark-*-free`, `jev-*-free` | various | ❓ | ❓ | Experiments only | Appear in some doc snapshots; temporary |

Warnings:
- Benchmark numbers above are vendor claims, not independent results. Test on your own project.
- "Free" models are often removed without notice. Always keep a fallback.
- Free usage limits are IP-based and you can be rate-limited under heavy load.

## Step 4 — Role-to-model map (order = priority, the rest are fallbacks)

| Agent role | Main need | Choices |
|---|---|---|
| `planner` / architect | Reasoning, wide context | `nemotron-3-ultra-free` → `longcat-2.0-free` → `deepseek-v4-flash-free` |
| `builder` (implementation) | Code quality, tool use | `laguna-s-2.1-free` → `longcat-2.0-free` → `big-pickle` |
| `explorer` (read repo, find files) | Cheap, fast, large context | `deepseek-v4-flash-free` → `north-mini-code-free` |
| `quick` (small edits, renames, boilerplate) | Low latency | `north-mini-code-free` → `ling-3.0-flash-free` |
| `reviewer` | A different viewpoint from the builder | A model **from a different maker** than the builder: builder is Laguna → reviewer `nemotron-3-ultra-free` or `deepseek-v4-flash-free` |
| `vision` (UI screenshots, diagrams) | Multimodal | `mimo-v2.5-free` |
| `docs` | Writing and summarizing | `deepseek-v4-flash-free` → `mimo-v2.5-free` |

Selection rules:
1. The reviewer must not use the same model as the builder. Different models catch different mistakes.
2. Context > 200K → use only 1M-context models (`deepseek-v4-flash-free`, `longcat-2.0-free`, `mimo-v2.5-free`) or split the task.
3. Open source required → filter on the "Open" column = ✅ and tell the user the license.
4. Start with one model per role; add more only when there is evidence one model fails in that role.
5. Never put a ❓ or ❌ model in a critical role without a ✅ fallback.

## Step 5 — Generate the configuration

ID format: `provider/model-id`. A subagent without `model` uses its parent agent's model; a per-agent `model` overrides that.

**Option A — `opencode.json` (project) or `~/.config/opencode/opencode.json` (global):**

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode/laguna-s-2.1-free",
  "small_model": "opencode/north-mini-code-free",
  "agent": {
    "plan": { "model": "opencode/nemotron-3-ultra-free" },
    "build": { "model": "opencode/laguna-s-2.1-free" }
  }
}
```

**Option B — Markdown agent (`.opencode/agents/<name>.md` or `~/.config/opencode/agents/<name>.md`):**

```markdown
---
description: Reviews code for bugs, security, and consistency. Use after the builder finishes.
mode: subagent
model: opencode/deepseek-v4-flash-free
temperature: 0.1
permission:
  edit: deny
  bash: deny
---
You are a reviewer. Read the diff and report real bugs, security risks, and inconsistencies.
Do not rewrite code. Give file:line and a short reason.
```

Version note: the OpenCode v2 docs use the plural key `agents` and support `provider/model#variant`; the v1 docs use `agent`. Check `opencode --version` and the matching docs before writing config.

Optional: the `opencode-subagent-models` plugin overrides the model of all subagents globally or per session without editing agent files. The `opencode-free-models` plugin picks free models dynamically with a fallback.

## Step 6 — Fallback and operations

- If a model fails or is rate-limited: move to the next choice in the role map. Do not retry the same model repeatedly.
- If all free models fail: tell the user and offer a paid provider or OpenCode Go.
- After important work from a free model, verify the result by running tests/lint/build, not just by reading the model's answer.
- Review this map monthly, or whenever a "model not found" error appears.

## Report format to the user

1. Step 1 result: the free models live today.
2. Role → model table (with fallbacks).
3. Which models are open source and their licenses, and which are not or unclear.
4. The generated config (files written and their locations).
5. A data/privacy warning if the project is sensitive.
