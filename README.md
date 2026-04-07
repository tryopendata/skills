# OpenData Skills

Agent skills for data, visualizations, and design. Built on the [Agent Skills spec](https://agentskills.io), distributed as a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces).

| Plugin | What it does | Skills |
|--------|-------------|--------|
| **opendata** | Query and analyze public datasets via the OpenData API | `opendata-api` |
| **openchart** | Generate charts, tables, graphs, and sankey diagrams with [OpenChart](https://github.com/tryopendata/openchart) | `openchart` |
| **opendesign** | Create SVG logos, icons, and graphics | `svg-design` |

## Installation

### Claude Code

Add the marketplace, then install whichever plugins you need:

```
/plugin marketplace add tryopendata/skills
```

```
/plugin install opendata@opendata-skills
/plugin install openchart@opendata-skills
/plugin install opendesign@opendata-skills
```

Or from the CLI directly:

```bash
claude plugin marketplace add tryopendata/skills
claude plugin install opendata@opendata-skills
```

You can scope the install with `--scope project` (shared via version control) or `--scope local` (gitignored). Default is `user` (available everywhere).

### OpenAI Codex, Gemini CLI, Cursor, VS Code/Copilot, etc.

Each skill is a standalone [Agent Skills spec](https://agentskills.io/specification) directory with a `SKILL.md` file. To use them on other platforms, copy the skill directory you want into your platform's skills location (typically `.agents/skills/`):

```
plugins/<plugin>/skills/<skill-name>/   # copy this directory
```

For example, to add `svg-design` to a Codex project:

```bash
cp -r plugins/opendesign/skills/svg-design .agents/skills/svg-design
```

Check your platform's [Agent Skills docs](https://agentskills.io) for where it discovers skills.

> **Migrating from pre-1.0?** `svg-design` moved from the `openchart` plugin to the new `opendesign` plugin in v1.0.0.

---

## Contributing

### Structure

```
.claude-plugin/
  marketplace.json           # Claude Code marketplace registry
plugins/
  opendata/                  # OpenData plugin
    skills/
      opendata-api/
  openchart/                 # OpenChart plugin
    skills/
      openchart/
  opendesign/                # OpenDesign plugin
    skills/
      svg-design/
```

Each skill is a directory with a `SKILL.md` file:

```
skills/<name>/
├── SKILL.md        # Instructions with YAML frontmatter (name + description)
├── evals/          # Test cases for output quality verification
├── scripts/        # Bundled executable code
├── references/     # Supplementary docs loaded on-demand
└── assets/         # Templates, schemas, static resources
```

### Adding a Skill

1. Create a directory under `plugins/<plugin>/skills/` with a `SKILL.md` file
2. Follow the [Agent Skills specification](https://agentskills.io/specification) for frontmatter format
3. Add [evals](https://agentskills.io/skill-creation/evaluating-skills) to verify output quality
4. See [best practices](https://agentskills.io/skill-creation/best-practices) for writing effective skills

## License

MIT
