# SPX-Claude

Claude Code plugin marketplace with skills for testing, Python development, and productivity.

## Installation

Add this marketplace to your Claude Code configuration:

```bash
claude plugins add spx-claude --marketplace https://github.com/simonheimlicher/spx-claude
```

Or add individual plugins:

```bash
claude plugins add test --marketplace https://github.com/simonheimlicher/spx-claude
claude plugins add python --marketplace https://github.com/simonheimlicher/spx-claude
claude plugins add claude --marketplace https://github.com/simonheimlicher/spx-claude
```

## Plugins

### test

BDD testing methodology with three-tier testing (Unit, Integration, E2E).

| Skill                 | Purpose                         |
| --------------------- | ------------------------------- |
| `/testing`            | Foundational testing principles |
| `/testing-typescript` | TypeScript-specific patterns    |

**Core principles:** No mocking, dependency injection, reality as the oracle.

### python

Complete Python development workflow.

| Type    | Name                             | Purpose                            |
| ------- | -------------------------------- | ---------------------------------- |
| Command | `/autopython`                    | Autonomous implementation          |
| Skill   | `/coding-python`                 | Implementation with remediation    |
| Skill   | `/reviewing-python`              | Strict code review                 |
| Skill   | `/architecting-python`           | ADR producer with testing strategy |
| Skill   | `/reviewing-python-architecture` | ADR validator                      |
| Skill   | `/testing-python`                | Python-specific testing patterns   |

### claude

Claude Code productivity skills.

| Type    | Name                 | Purpose                              |
| ------- | -------------------- | ------------------------------------ |
| Skill   | `/creating-skills`   | Create maintainable skills           |
| Skill   | `/committing-changes`| Comprehensive commit message guide   |
| Command | `/ci`                | Git commit with Conventional Commits |

## Documentation

- [Claude Code Plugins](https://code.claude.com/docs/en/plugins)
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [SPX Framework](https://spx.sh)

## License

MIT
