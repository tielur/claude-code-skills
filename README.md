# Claude Code Skills

A community collection of plugins that extend Claude Code with specialized capabilities — diagram generation, document conversion, visualization, and more. Built for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Getting Started

### Claude Code

```bash
# Add the marketplace
claude plugin marketplace add tielur/claude-code-skills

# Install a specific plugin
claude plugin install svg-diagram@claude-code-skills
```

Once installed, skills activate automatically when relevant. You can also invoke them directly with slash commands (e.g., `/svg-diagram:svg-diagram`).

## Plugins

| Plugin | How it helps | Output |
|--------|-------------|--------|
| **[svg-diagram](./svg-diagram)** | Generate technical diagrams, architecture visuals, flowcharts, and layered illustrations from descriptions or screenshots. | `.svg` + `.html` (PNG export) |

## How Plugins Work

Every plugin follows the same structure:

```
plugin-name/
├── .claude-plugin/plugin.json   # Manifest
├── skills/                      # Domain knowledge Claude draws on automatically
│   └── skill-name/
│       └── SKILL.md
└── commands/                    # Slash commands you invoke explicitly (optional)
```

- **Skills** encode domain expertise, design guidelines, and step-by-step workflows. Claude draws on them automatically when the task matches.
- **Commands** are explicit actions you trigger (e.g., `/svg-diagram:svg-diagram`).

Everything is file-based — markdown and JSON, no code, no infrastructure, no build steps.

## Contributing

Plugins are just markdown files. Fork the repo, add your plugin, and submit a PR.

### Adding a plugin

1. Create a directory for your plugin at the repo root
2. Add `.claude-plugin/plugin.json` with name, version, description, and author
3. Add skills in `skills/<skill-name>/SKILL.md`
4. Add an entry to `.claude-plugin/marketplace.json`
5. Update the plugin table in this README
6. Submit a PR

## License

MIT
