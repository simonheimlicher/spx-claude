# Your Claude Code Skills and Commands — Everywhere, Always Current

A single repo for all your skills and commands, instantly available across every project.

## How It Works

You write a skill once. You want it in every project. You want edits to propagate immediately.

The solution: keep all your skills and commands in one local repo. Add it once. Available everywhere.

```
# After setup, in any project:
/testing           # Your testing methodology
/ci                # Your commit workflow
/coding-python     # Your Python patterns
```

Edit the source files, run one command, every project has the latest.

## Quick Start

### 1. Create a repo for your skills and commands

```bash
mkdir -p ~/Code/my-skills
```

### 2. Add your first skill

```bash
mkdir -p ~/Code/my-skills/plugins/dev/.claude-plugin
mkdir -p ~/Code/my-skills/plugins/dev/skills/my-skill
```

Create the skill file:

```bash
cat > ~/Code/my-skills/plugins/dev/skills/my-skill/SKILL.md << 'EOF'
---
description: My custom skill
---
# My Skill

Instructions for Claude...
EOF
```

Skills are grouped into plugins (think: namespaces). Create the plugin manifest:

```bash
cat > ~/Code/my-skills/plugins/dev/.claude-plugin/plugin.json << 'EOF'
{
  "name": "dev",
  "version": "1.0.0",
  "description": "My dev tools"
}
EOF
```

### 3. Create the marketplace catalog

The marketplace is just an index pointing to your plugins:

```bash
mkdir -p ~/Code/my-skills/.claude-plugin

cat > ~/Code/my-skills/.claude-plugin/marketplace.json << 'EOF'
{
  "name": "my-skills",
  "owner": {"name": "Your Name"},
  "plugins": [{"name": "dev", "source": "./plugins/dev"}]
}
EOF
```

### 4. Add to Claude Code (once)

```
/plugin marketplace add ~/Code/my-skills
```

### 5. Install

```
/plugin install dev@my-skills
```

Now `/my-skill` is available in all projects.

## Repository Structure

```
my-skills/
├── .claude-plugin/
│   └── marketplace.json      # Index of all plugins
└── plugins/
    └── dev/                   # A plugin (namespace)
        ├── .claude-plugin/
        │   └── plugin.json    # Plugin metadata
        ├── skills/            # Your skills live here
        │   └── my-skill/
        │       └── SKILL.md
        └── commands/          # Your commands live here
            └── build.md
```

| Concept         | What it is                                 |
| --------------- | ------------------------------------------ |
| **Skill**       | Agent guidance (SKILL.md files)            |
| **Command**     | Slash command (`/build` → `build.md`)      |
| **Plugin**      | Namespace grouping related skills/commands |
| **Marketplace** | Index pointing to plugins                  |

## Updating After Edits

```bash
# Edit any skill or command in ~/Code/my-skills/...

# Refresh (picks up all changes)
/plugin marketplace update
```

## Sharing

Push to GitHub. Others add your marketplace:

```
/plugin marketplace add owner/repo
```

Same repo works locally and as public distribution.

## What's in This Repo

This repo contains skills and commands for testing, Python development, and Claude Code productivity.

### test

BDD testing methodology with three-tier testing (Unit, Integration, E2E).

| Skill                 | Purpose                         |
| --------------------- | ------------------------------- |
| `/testing`            | Foundational testing principles |
| `/testing-typescript` | TypeScript-specific patterns    |

### python

Complete Python development workflow.

| Type    | Name                             | Purpose                            |
| ------- | -------------------------------- | ---------------------------------- |
| Command | `/autopython`                    | Autonomous implementation          |
| Skill   | `/testing-python`                | Python-specific testing patterns   |
| Skill   | `/coding-python`                 | Implementation with remediation    |
| Skill   | `/reviewing-python`              | Strict code review                 |
| Skill   | `/architecting-python`           | ADR producer with testing strategy |
| Skill   | `/reviewing-python-architecture` | ADR validator                      |

### claude

Claude Code productivity skills.

| Type    | Name                  | Purpose                              |
| ------- | --------------------- | ------------------------------------ |
| Skill   | `/creating-skills`    | Create maintainable skills           |
| Skill   | `/committing-changes` | Commit message guidance              |
| Command | `/ci`                 | Git commit with Conventional Commits |

## Quick Install

If you just want to use these skills without forking:

```bash
/plugin marketplace add simonheimlicher/spx-claude
/plugin install test@spx-claude
/plugin install python@spx-claude
/plugin install claude@spx-claude
```

## Documentation

- [Claude Code Plugins](https://code.claude.com/docs/en/plugins)
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)

## License

MIT
