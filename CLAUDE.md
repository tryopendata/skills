# OpenData Skills

Cross-platform agent skills repo following the [Agent Skills specification](https://agentskills.io/specification).

- **Repo:** https://github.com/tryopendata/skills
- **Spec:** https://agentskills.io

## Plugins

This repo contains multiple independently installable plugins:

| Plugin | Namespace | Source | Skills |
|--------|-----------|--------|--------|
| opendata | `opendata` | `plugins/opendata/` | `opendata-api` |
| openchart | `openchart` | `plugins/openchart/` | `openchart`, `svg-design` |

Each plugin has its own `plugins/<name>/.claude-plugin/plugin.json` for metadata. The root `.claude-plugin/marketplace.json` registers all plugins.

## Directory Structure

```
plugins/
  opendata/
    .claude-plugin/plugin.json
    skills/
      opendata-api/
    commands/          # when needed
    agents/            # when needed
  openchart/
    .claude-plugin/plugin.json
    skills/
      openchart/
      svg-design/
.claude-plugin/
  marketplace.json     # root marketplace registry (all plugins)
```

## Adding a Skill

1. Create `plugins/<plugin>/skills/<name>/SKILL.md` with YAML frontmatter:

```yaml
---
name: my-skill-name
description: >
  What this skill does and when to use it. Use imperative phrasing
  like "Use when..." Max 1024 chars.
---
```

2. **`name` field:** lowercase letters, numbers, hyphens only. Max 64 chars. Must match the parent directory name. No leading/trailing/consecutive hyphens.

3. **`description` field:** Describe what and when. Use imperative phrasing ("Use when..."). Focus on user intent, not implementation. Max 1024 chars.

4. **Optional frontmatter:** `license`, `compatibility`, `metadata` (key-value map), `allowed-tools` (space-delimited)

5. **Body:** Instructions for the agent. Keep under 500 lines / 5000 tokens. Split large content into `references/` files and reference them conditionally ("Read `references/api-errors.md` if the API returns a non-200 status").

6. **Registration:** Add the skill directory path to the plugin's entry in `.claude-plugin/marketplace.json` under the `skills` array (e.g., `"./skills/my-skill"`). Bump the version in both `marketplace.json` (for that plugin) and the plugin's `plugin.json`.

### Skill Directory Structure

```
skills/<name>/
├── SKILL.md              # Required: frontmatter + instructions
├── evals/                # Recommended: eval test cases
│   ├── evals.json        # Test prompts, expected outputs, assertions
│   └── files/            # Input files for test cases
├── scripts/              # Optional: executable code
├── references/           # Optional: docs loaded on-demand
└── assets/               # Optional: templates, schemas, resources
```

### Skill Evals

Each skill should include `evals/evals.json` to verify output quality. Test cases have a prompt, expected output, optional input files, and assertions:

```json
{
  "skill_name": "my-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "A realistic user message",
      "expected_output": "What success looks like",
      "files": ["evals/files/input.csv"],
      "assertions": [
        "The output includes X",
        "Y is formatted as JSON"
      ]
    }
  ]
}
```

Run each test case with and without the skill to measure the delta. Grade assertions as PASS/FAIL with evidence. Aggregate into benchmark.json with pass rates, token usage, and timing. See https://agentskills.io/skill-creation/evaluating-skills for the full workflow.

### Script Conventions

Scripts in `scripts/` should be:
- **Non-interactive** (no TTY prompts, accept input via flags/env/stdin)
- **Self-documenting** (include `--help` output)
- **Structured output** (JSON/CSV to stdout, diagnostics to stderr)
- **Idempotent** where possible
- **Self-contained deps** (PEP 723 for Python, npm: imports for Deno, bundler/inline for Ruby)

## Adding a Command

Create `plugins/<plugin>/commands/<name>.md` with YAML frontmatter:

```yaml
---
description: What the command does
allowed-tools: Bash, Read, Grep, Glob
---

Command instructions here. Use $ARGUMENTS for user input.
```

Add the path to the plugin's `commands` array in `.claude-plugin/marketplace.json`.

## Adding an Agent

Create `plugins/<plugin>/agents/<name>.md` with YAML frontmatter:

```yaml
---
name: agent-name
description: What expertise this agent provides
tools: Read, Grep, Glob, Bash
---

Agent personality and workflow instructions.
```

Add the path to the plugin's `agents` array in `.claude-plugin/marketplace.json`.

## Version Management

Versions are bumped automatically via GitHub Action when plugin files change on main. Each plugin's version is tracked in both its `plugin.json` and the corresponding entry in `marketplace.json`.
