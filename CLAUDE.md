# SPX-Claude Plugin Marketplace

Simon Heimlicher's Claude Code plugin marketplace, based on the spec-driven development framework [SPX](https://spx.sh).

## Documentation

- [Claude Code Plugins](https://code.claude.com/docs/en/plugins) - How to create and use plugins
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) - How marketplaces work

## Skill Organization Principles

### Foundational + Language-Specific Pattern

Skills follow a **reference pattern** to avoid duplication:

1. **Foundational skill** (`/test`) - Contains core principles, methodology, and language-agnostic patterns
2. **Language-specific skills** (`/python-test`, `/typescript-test`) - Reference the foundational skill, provide only language-specific implementations

**Usage:** Always read the foundational skill first, then the language-specific skill for concrete patterns.

### Why This Pattern?

- **Single source of truth** - Core principles live in one place
- **No drift** - Changes to methodology propagate automatically
- **Focused content** - Each skill contains only what's unique to it
- **Maintainability** - Less duplication means less divergence over time

### Skill Invocation

Claude Code skills cannot automatically invoke other skills. However, skills can:

1. Instruct the AI to read another skill file first
2. Reference foundational concepts by skill name
3. Be invoked sequentially by the user/AI

## Test Plugin (`/test`)

The test plugin provides BDD testing methodology with three-tier testing:

| Level          | Question                               | Requirements                     |
| -------------- | -------------------------------------- | -------------------------------- |
| 1: Unit        | "Is our logic correct?"                | Dependency injection, NO mocking |
| 2: Integration | "Does it work with real dependencies?" | Documented test harnesses        |
| 3: E2E         | "Does it work for users?"              | Documented credentials           |

**Core rules:**

- No mocking - use dependency injection at Level 1, real dependencies at Level 2+
- Progress tests (may fail) go in `specs/.../tests/`
- Regression tests (must pass) go in `test/` or `tests/`

## Python Plugin (`/python-*`)

Complete Python development workflow with testing, implementation, and review.

### Skills

| Skill                           | Purpose                                             |
| ------------------------------- | --------------------------------------------------- |
| `/python-test`                  | Python-specific testing patterns (requires `/test`) |
| `/python-auto`                  | Autonomous implementation orchestrator              |
| `/python-coder`                 | Implementation workhorse with remediation loop      |
| `/python-reviewer`              | Strict code review with zero-tolerance              |
| `/python-architect`             | ADR producer with testing strategy                  |
| `/python-architecture-reviewer` | ADR validator against testing principles            |

### Workflow

```
/python-auto → /python-coder → /python-reviewer
                    ↓
            /python-architect (if ADRs needed)
                    ↓
    /python-architecture-reviewer (validates ADRs)
```

### Work Item Discovery

Use the `spx` CLI to find work items:

- `spx status` - Overview of work items and their status
- `spx next` - Get the next work item to work on

### Core Principles

- No mocking - dependency injection only
- Reality is the oracle
- Behavior testing, not implementation testing
- Tests at appropriate levels (Unit/Integration/E2E)
