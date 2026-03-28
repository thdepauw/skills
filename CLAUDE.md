# CLAUDE.md

## Project Overview

This repo is a Claude Code **plugin marketplace** containing reusable Agent Skills. Skills follow the [Agent Skills specification](https://github.com/agentskills/agentskills/blob/main/docs/specification.mdx) and the repo structure is modeled after [anthropics/skills](https://github.com/anthropics/skills).

## Repository Structure

```
skills/                           # All skills live here
  <skill-name>/
    SKILL.md                      # Required: frontmatter + instructions
    scripts/                      # Optional: executable code
    references/                   # Optional: supplementary docs
    assets/                       # Optional: templates, resources
    LICENSE.txt                   # Optional: per-skill license
.claude-plugin/
  marketplace.json                # Marketplace catalog (plugin registry)
```

## Skill Format (Agent Skills Spec)

### SKILL.md Frontmatter

```yaml
---
name: <skill-name> # 1-64 chars, lowercase alphanumeric + hyphens, must match directory name
description: <text> # 1-1024 chars, what the skill does and when to use it
---
```

Optional fields: `license`, `compatibility` (max 500 chars), `metadata` (key-value pairs), `allowed-tools` (space-delimited, experimental).

### Naming Rules

- Lowercase letters, numbers, and hyphens only
- No leading/trailing hyphens, no consecutive hyphens
- Directory name must match the `name` field exactly

### Size Guidelines

- Keep `SKILL.md` under 500 lines / ~5000 tokens
- Move detailed material to `references/`, `scripts/`, or `assets/`
- Progressive disclosure: metadata loads first, full instructions on activation, resources on demand

## Plugin & Marketplace

This repo doubles as a **plugin marketplace**. The marketplace catalog lives at `.claude-plugin/marketplace.json`.

### marketplace.json Schema

```json
{
  "name": "ineth-skills",
  "owner": { "name": "...", "email": "..." },
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./path/to/plugin",
      "description": "...",
      "version": "1.0.0"
    }
  ]
}
```

Each plugin entry can reference skills via relative paths (`./`-prefixed, relative to marketplace root).

### plugin.json (per-plugin manifest, optional)

Lives at `<plugin-dir>/.claude-plugin/plugin.json`. Required field: `name` (kebab-case). Optional: `version`, `description`, `author`, `homepage`, `repository`, `license`, `keywords`, plus component paths (`commands`, `agents`, `skills`, `hooks`, `mcpServers`, `lspServers`).

### Testing Locally

```bash
# Load the entire marketplace as a plugin directory
claude --plugin-dir .

# Reload after changes (inside Claude Code)
/reload-plugins
```

### Validation

```bash
# Validate plugin structure
claude plugin validate .

# Validate individual skills (Agent Skills spec)
npx skills-ref validate skills/<skill-name>
```

### Distribution

Users install via:

```
/plugin marketplace add <github-owner>/<repo>
/plugin install <plugin-name>@<marketplace-name>
```

## Key References

- [Agent Skills specification](https://github.com/agentskills/agentskills/blob/main/docs/specification.mdx)
- [Claude Code plugins docs](https://code.claude.com/docs/en/plugins)
- [Plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces)
- [Plugins reference (schemas)](https://code.claude.com/docs/en/plugins-reference)
- [anthropics/skills (reference repo)](https://github.com/anthropics/skills)
