# Skills

A Claude Code plugin marketplace with reusable [Agent Skills](https://github.com/agentskills/agentskills).

## Installation

Add the marketplace and install a plugin:

```
/plugin marketplace add thdepauw/skills
/plugin install <plugin-name>@skills
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
