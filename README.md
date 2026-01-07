# Your Claude Code Skills and Commands — Everywhere, Always Current

Fork this repo and you will have a single place for all your Claude Code skills, commands, agents... instantly available across every project.

## How It Works

You write a skill once but want to use it in all projects and your improvements must propagate immediately.
Fork this repo, which matches Claude Code's expectations for a [plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces), and keep all your skills and commands in a repo that you can access locally during development and via Github URL when on other machines.

### Start by cloning an existing plugin marketplace

For example, to clone this marketplace as a template into `~/Code/claude-repo`:

```zsh
git clone https://github.com/simonheimlicher/spx-claude.git ~/Code/claude-repo
```

### 1. Add the marketplace to Claude Code at `user` level (once)

By passing a filesystem path like `~/Code/claude-repo`, Claude Code treats this as a local marketplace.

```zsh
claude plugin marketplace add ~/Code/claude-repo
```

### 2. Add a first plugin

```zsh
claude plugin install claude@spx-claude
```

Now the slash command `/claude:ci` and the skill `creating-skills` are available in all projects on your machine.

## Repository Structure

Refer to the [official plugin marketplace documentation](https://code.claude.com/docs/en/plugin-marketplaces) for the latest version.

```
~/Code/claude-repo
├── .claude-plugin
│   └── marketplace.json
├── plugins
│   ├── claude
│   │   ├── .claude-plugin
│   │   │   └── plugin.json
│   │   ├── commands
│   │   │   └── ci.md
│   │   └── skills
│   │       ├── committing-changes
│   │       │   └── SKILL.md
[...]
```

| Concept         | What it is                                 |
| --------------- | ------------------------------------------ |
| **Skill**       | Agent guidance (SKILL.md files)            |
| **Command**     | Slash command (`/build` → `build.md`)      |
| **Plugin**      | Namespace grouping related skills/commands |
| **Marketplace** | Index pointing to plugins                  |

## Updating After Edits

Enable `auto-update` by running `Claude Code` in a terminal:

```bash
# Start Claude Code
claude

# Navigate to plugin marketplace
/plugin marketplace

# Select your marketplace and turn on `Enable auto-update`
```

## What's in This Repo

This repo contains skills and commands for testing, Python and Typescript development, and Claude Code productivity.

### test

BDD testing methodology with three-tier testing (Unit, Integration, E2E).

| Skill      | Purpose                         |
| ---------- | ------------------------------- |
| `/testing` | Foundational testing principles |

### typescript

Complete TypeScript development workflow.

| Type  | Name                                 | Purpose                            |
| ----- | ------------------------------------ | ---------------------------------- |
| Skill | `/testing-typescript`                | TypeScript-specific testing        |
| Skill | `/coding-typescript`                 | Implementation with remediation    |
| Skill | `/reviewing-typescript`              | Strict code review                 |
| Skill | `/architecting-typescript`           | ADR producer with testing strategy |
| Skill | `/reviewing-typescript-architecture` | ADR validator                      |

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
/plugin install typescript@spx-claude
/plugin install python@spx-claude
/plugin install claude@spx-claude
```

## Documentation

- [Claude Code Plugins](https://code.claude.com/docs/en/plugins)
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)

## License

MIT
