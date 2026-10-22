# Usage manual

This manual shows how to install and use the `model-router` skill, and how to keep it working when the free model list changes.

## 1. Requirements

- An agent that can load `SKILL.md` skills (OpenCode is the primary target).
- Shell access with network, for the live model check.
- Optional: the OpenCode CLI (`opencode`) and a Zen API key if the endpoint asks for one.

## 2. Install

### OpenCode

```bash
git clone https://github.com/rndz1618/model-router.git
cd model-router

# Global (all projects)
mkdir -p ~/.config/opencode/skills
cp -r skills/model-router ~/.config/opencode/skills/

# Or only for the current project
mkdir -p .opencode/skills
cp -r skills/model-router .opencode/skills/
```

Restart OpenCode so it rescans skills.

### Other agents

- Claude Code: copy the folder to `~/.claude/skills/model-router/` (global) or `.claude/skills/model-router/` (project).
- Any tool without a skill loader: paste the contents of `skills/model-router/SKILL.md` into its system prompt or project instructions file.

Check your tool's docs for the exact path. Paths differ between tools and versions.

## 3. Verify the install

Ask the agent: "What skills do you have?" and confirm `model-router` is listed. If it is not, see Troubleshooting.

## 4. Use it

Invoke the skill by name (`/model-router` where slash commands are supported), or just describe the goal.

Example prompts:

- "Use model-router: set up free models for this repo."
- "I need an open-source-only setup. Which free models qualify and under what license?"
- "My builder model is rate-limited. Re-map the agents."
- "Pick a reviewer that is from a different maker than my builder."

## 5. What happens

1. **Live check.** The agent lists the free models that exist right now (`opencode models` or the Zen `/models` endpoint).
2. **Project questions.** It asks or infers: context size, main task types, code sensitivity, whether open source is required.
3. **Role map.** It assigns a model to each role (planner, builder, explorer, quick, reviewer, vision, docs) with fallbacks.
4. **Config.** It writes `opencode.json` and/or `.opencode/agents/*.md`.
5. **Report.** It tells you the live models, the role table, which models are open source and their licenses, the files written, and any privacy warning.

Check the generated files before relying on them. The shape of the output looks like this:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode/<builder-model>",
  "small_model": "opencode/<fast-model>",
  "agent": {
    "plan": { "model": "opencode/<planner-model>" },
    "build": { "model": "opencode/<builder-model>" }
  }
}
```

The actual model IDs depend on what is live when you run it.

## 6. Reading the open-source column

- ✅ Open weights with the license shown. You can download and self-host.
- ❓ Not verified. Do not assume it is open source.
- ❌ Closed or the model behind the name is undisclosed.

Open weights does not mean open training data unless the report says so.

## 7. Privacy

Free models may use your prompts and outputs for training. Do not use them for client code, credentials, or personal data. Ask the skill for a "sensitive project" setup and it will tell you to use a paid provider or a self-hosted model.

## 8. Keeping it current

- The catalog in `SKILL.md` is a snapshot. The live check in Step 1 is the source of truth.
- If a model is missing from the live check, treat it as removed and use the next fallback.
- To update the catalog, see [CONTRIBUTING.md](../CONTRIBUTING.md).
- To update your copy: `git pull`, then copy the skill folder again.

## 9. Troubleshooting

| Problem | What to try |
|---|---|
| Skill is not listed | Confirm the folder is `model-router/` and contains `SKILL.md` with `name` and `description` frontmatter. Restart the agent. Check your tool's skill path. |
| `opencode models` shows no free models | Update OpenCode, check your network, and try the `curl` command in Step 1. |
| `curl` to `/zen/v1/models` fails or returns 401 | Add an `Authorization: Bearer <key>` header with your Zen key, or use `opencode models` instead. |
| "model not found" at run time | The free model was removed. Re-run the skill. |
| Rate limited | Free usage limits can apply per IP. Switch to the next fallback or wait. |
| Config key not recognized | OpenCode docs for v2 use `agents` (plural) and v1 uses `agent`. Run `opencode --version` and match the docs. |

## 10. Uninstall

```bash
rm -rf ~/.config/opencode/skills/model-router
# or, for a project install
rm -rf .opencode/skills/model-router
```
