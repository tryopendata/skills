# OpenData Skills

Cross-platform agent skills for the OpenData ecosystem, built on the [Agent Skills specification](https://agentskills.io).

## Plugins

| Plugin | Description | Skills |
|--------|-------------|--------|
| **opendata** | Query and analyze public datasets via the OpenData API | `opendata-api` |
| **openchart** | Generate charts, tables, graphs, and SVG graphics with [OpenChart](https://github.com/tryopendata/openchart) | `openchart`, `svg-design` |

## Installation

### Claude Code

```
/plugin marketplace add tryopendata/skills
```

### Other Platforms

This repo follows the [Agent Skills spec](https://agentskills.io/specification), which is supported by 30+ agent platforms including Cursor, Gemini CLI, VS Code/Copilot, OpenAI Codex, OpenCode, Goose, Roo Code, and more. Check your platform's docs for how to install agent skills.

## Structure

```
plugins/
  opendata/                # OpenData plugin
    skills/
      opendata-api/        # Query the OpenData API
  openchart/               # OpenChart plugin
    skills/
      openchart/           # Chart/table/graph VizSpec generation
      svg-design/          # SVG logo, icon, and graphic creation
.claude-plugin/
  marketplace.json         # Root marketplace registry
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

## Contributing a Skill

1. Create a directory under the appropriate `plugins/<plugin>/skills/` with a `SKILL.md` file
2. Follow the [Agent Skills specification](https://agentskills.io/specification) for frontmatter format
3. Add [evals](https://agentskills.io/skill-creation/evaluating-skills) to verify your skill produces good output
4. See [best practices](https://agentskills.io/skill-creation/best-practices) for writing effective skills

## License

MIT
