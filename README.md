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

| Skill              | Purpose                         |
| ------------------ | ------------------------------- |
| `/test`            | Foundational testing principles |
| `/typescript-test` | TypeScript-specific patterns    |

**Core principles:** No mocking, dependency injection, reality as the oracle.

### python

Complete Python development workflow.

| Skill                           | Purpose                            |
| ------------------------------- | ---------------------------------- |
| `/python-auto`                  | Autonomous implementation          |
| `/python-coder`                 | Implementation with remediation    |
| `/python-reviewer`              | Strict code review                 |
| `/python-architect`             | ADR producer with testing strategy |
| `/python-architecture-reviewer` | ADR validator                      |
| `/python-test`                  | Python-specific testing patterns   |

### claude

Claude Code productivity skills.

| Skill            | Purpose                              |
| ---------------- | ------------------------------------ |
| `/skill-creator` | Create maintainable skills           |
| `/ci`            | Git commit with Conventional Commits |

## Documentation

- [Claude Code Plugins](https://code.claude.com/docs/en/plugins)
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [SPX Framework](https://spx.sh)

## License

MIT
