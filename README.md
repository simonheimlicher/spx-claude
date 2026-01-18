# SPX-Claude Plugin Marketplace

A Claude Code plugin marketplace with skills and commands for testing, Python and TypeScript development, specifications, and productivity.

## Philosophy

1. **RTFM:** Follow state-of-the-art (SOTA) model prompting guidance, such as [structured prompts based on XML tags](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/use-xml-tags#tagging-best-practices)
2. **KILO:**: *Keep It Local and Observable,* to facilitate discovery by agents by keeping the golden source for all specifications locally within the project's Git repository

## Quick Install

Add this marketplace and install plugins directly from GitHub:

```bash
# Add the marketplace
claude plugin marketplace add simonheimlicher/spx-claude

# Install plugins you want
claude plugin install core@spx-claude
claude plugin install test@spx-claude
claude plugin install typescript@spx-claude
claude plugin install python@spx-claude
claude plugin install specs@spx-claude
```

Now slash commands like `/commit`, `/handoff`, `/pickup` and skills like `/testing-typescript` are available in all your projects.

### Update Plugins

```bash
# Update this marketplace
claude plugin marketplace update spx-claude

# Or update all marketplaces
claude plugin marketplace update
```

For automatic updates, run `claude`, navigate to `/plugin marketplace`, select this marketplace, and enable `Enable auto-update`.

## Available Plugins

### core

Productivity commands and skills.

| Type    | Name                  | Purpose                                 |
| ------- | --------------------- | --------------------------------------- |
| Skill   | `/committing-changes` | Commit message guidance                 |
| Command | `/commit`             | Git commit with Conventional Commits    |
| Command | `/handoff`            | Create timestamped context handoff      |
| Command | `/pickup`             | Load and continue from previous handoff |

Credit: `/handoff` is inspired by [TÂCHES Claude Code Resources](https://github.com/glittercowboy/taches-cc-resources/tree/main?tab=readme-ov-file#context-handoff).

### test

BDD testing methodology with three-tier testing (Unit, Integration, E2E).

| Type  | Name       | Purpose                         |
| ----- | ---------- | ------------------------------- |
| Skill | `/testing` | Foundational testing principles |

### typescript

Complete TypeScript development workflow.

| Type  | Name                                 | Purpose                            |
| ----- | ------------------------------------ | ---------------------------------- |
| Agent | `typescript-simplifier`              | Simplify code for maintainability  |
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

### specs

Requirements documentation and specification skills.

| Type  | Name                   | Purpose                            |
| ----- | ---------------------- | ---------------------------------- |
| Skill | `/writing-prd`         | Write product requirements         |
| Skill | `/writing-trd`         | Write technical requirements       |
| Skill | `/managing-specs`      | Set up specs directory structure   |
| Skill | `/understanding-specs` | Load context before implementation |

### frontend

Frontend design and styling.

| Type  | Name                  | Purpose                                |
| ----- | --------------------- | -------------------------------------- |
| Skill | `/designing-frontend` | Create distinctive frontend interfaces |

### code

Autonomous coding orchestration.

| Type  | Name                   | Purpose                            |
| ----- | ---------------------- | ---------------------------------- |
| Skill | `/coding-autonomously` | Autonomous implementation patterns |

### claude

Meta-skills for Claude Code plugin development.

| Type  | Name               | Purpose                    |
| ----- | ------------------ | -------------------------- |
| Skill | `/creating-skills` | Create maintainable skills |

Credit: `/creating-skills` is inspired by [TÂCHES Claude Code Resources](https://github.com/glittercowboy/taches-cc-resources?tab=readme-ov-file#skills).

---

## Build Your Own Marketplace

Want to create your own plugin marketplace? Fork this repo as a starting point.

### Clone and Set Up

```bash
# Clone as your own marketplace
git clone https://github.com/simonheimlicher/spx-claude.git ~/Code/my-claude-plugins
cd ~/Code/my-claude-plugins

# Remove origin and set up your own remote
git remote remove origin
git remote add origin git@github.com:yourusername/my-claude-plugins.git
```

### Add as a Local Marketplace

During development, add your local clone as a marketplace:

```bash
claude plugin marketplace add ~/Code/my-claude-plugins
```

This lets you edit skills and commands locally with changes available immediately.

### Repository Structure

```text
my-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog
└── plugins/
    └── my-plugin/
        ├── .claude-plugin/
        │   └── plugin.json       # Plugin metadata and version
        ├── commands/
        │   └── my-command.md     # Slash commands
        └── skills/
            └── my-skill/
                └── SKILL.md      # Agent skills
```

| Concept         | What it is                                 |
| --------------- | ------------------------------------------ |
| **Skill**       | Agent guidance (SKILL.md files)            |
| **Command**     | Slash command (`/build` → `build.md`)      |
| **Plugin**      | Namespace grouping related skills/commands |
| **Marketplace** | Index pointing to plugins                  |

### Publish Your Marketplace

Once your marketplace is on GitHub, others can add it:

```bash
claude plugin marketplace add yourusername/my-claude-plugins
claude plugin install my-plugin@my-claude-plugins
```

## Documentation

- [Claude Code Plugins](https://code.claude.com/docs/en/plugins)
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference)

## License

MIT
