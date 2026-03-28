# Skills

A Claude Code plugin marketplace with reusable [Agent Skills](https://github.com/agentskills/agentskills).

## Installation

Add the marketplace and install a plugin:

```
/plugin marketplace add https://github.com/thdepauw/skills
/plugin install <plugin-name>@thdepauw-personal
```

## Local Development

```bash
# Test the marketplace locally
claude --plugin-dir .

# Reload after changes (inside Claude Code)
/reload-plugins

# Validate plugin structure
claude plugin validate .
```

## License

[MIT](LICENSE)

## Resources

- [anthropics/skills](https://github.com/anthropics/skills) — Anthropic's official example skills repository
- [claude-plugins-official](https://github.com/anthropics/claude-plugins-official) — Official Anthropic plugin marketplace
- [Creating plugins](https://code.claude.com/docs/en/plugins) — Claude Code plugin authoring guide
- [Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces#walkthrough-create-a-local-marketplace) — How to create and distribute a marketplace

## My Learnings

### General

#### Plugins

- Allow for bundling related skills together.
- Provides a namespace for skills, which can help avoid naming conflicts and organize skills by theme or functionality.

#### Marketplaces

- Are a way to distribute plugins to others.
- Can contain local plugins (from the same repo) or remote plugins (from other repos).

### Gotchas

- Naming a skill specifically overrules the default name derived from the filename/plugin (e.g. `/sync-claude-permissions`, without name field, this is `/thdepauw:sync-claude-permissions`).
