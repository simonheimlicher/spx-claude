# SPX-Claude Plugin Marketplace

Simon Heimlicher's Claude Code plugin marketplace, based on the spec-driven development framework [SPX](https://spx.sh).

## Documentation

### Official Anthropic Resources

**Core Documentation:**

- [Create plugins](https://code.claude.com/docs/en/plugins) - How to create and structure plugins
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) - How to create and distribute marketplaces
- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference) - Complete technical specifications, schemas, and CLI commands
- [Discover Plugins](https://code.claude.com/docs/en/discover-plugins) - How users find and install plugins
- [Agent Skills](https://code.claude.com/docs/en/skills) - Creating and using Skills

**Announcements:**

- [Claude Code Plugins Announcement](https://www.anthropic.com/news/claude-code-plugins) - Official plugin system launch
- [Agent Skills Introduction](https://www.anthropic.com/news/skills) - Skills feature announcement

**Best Practices:**

- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices) - Agentic coding patterns

## Version Management

### Versioning Rules (Conservative Approach)

All plugins follow semantic versioning: `MAJOR.MINOR.PATCH`

**MAJOR version (0.x.x → 1.x.x):**

- ⛔ **NEVER bump unless user explicitly requests it**
- All plugins remain at major version `0` until stable release
- Reserved for future stable release when all features are production-ready

**MINOR version (0.3.x → 0.4.x):**

- ✅ Adding new commands (e.g., new `/pickup` command)
- ✅ Adding new skills (e.g., new `/designing-frontend` skill)
- ✅ Major functional changes (e.g., atomic claim mechanism in `/pickup`)
- ✅ Significant user experience improvements
- 🎯 **Use sparingly** - only for substantial additions or changes

**PATCH version (0.3.1 → 0.3.2):**

- ✅ **Most common** - default for most changes
- ✅ Bug fixes
- ✅ Refactoring existing code
- ✅ Documentation improvements
- ✅ Small enhancements to existing features
- ✅ Performance optimizations
- ✅ Internal implementation changes
- 🎯 **Use liberally** - when in doubt, use PATCH

### Files to Update When Bumping Version

**Plugin version** (always update):

```bash
plugins/{plugin-name}/.claude-plugin/plugin.json
```

```json
{
  "name": "claude",
  "version": "0.4.0" // ← Update this
}
```

**Marketplace catalog** (optional, only if description changes):

```bash
.claude-plugin/marketplace.json
```

```json
{
  "plugins": [
    {
      "name": "claude",
      "source": "./plugins/claude",
      "description": "..." // ← Only update if description changes
    }
  ]
}
```

### Version Bump Examples

| Change                      | Old   | New   | Reason                          |
| --------------------------- | ----- | ----- | ------------------------------- |
| Add `/handoff` command      | 0.2.0 | 0.3.0 | New command = MINOR             |
| Add self-organizing handoff | 0.3.0 | 0.4.0 | Major functional change = MINOR |
| Fix typo in handoff.md      | 0.4.0 | 0.4.1 | Documentation fix = PATCH       |
| Refactor pickup logic       | 0.4.1 | 0.4.2 | Refactoring = PATCH             |
| Improve error messages      | 0.4.2 | 0.4.3 | Small enhancement = PATCH       |
| Add `/designing-frontend`   | 0.4.3 | 0.5.0 | New skill = MINOR               |

## Skill Organization Principles

### Foundational + Language-Specific Pattern

Skills follow a **reference pattern** to avoid duplication:

1. **Foundational skill** (`/testing`) - Contains core principles, methodology, and language-agnostic patterns
2. **Language-specific skills** (`/testing-python`, `/testing-typescript`) - Reference the foundational skill, provide only language-specific implementations

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

## Test Plugin (`/testing`)

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

## TypeScript Plugin

Complete TypeScript development workflow with testing, implementation, and review.

### Skills

| Skill                                | Purpose                                                    |
| ------------------------------------ | ---------------------------------------------------------- |
| `/testing-typescript`                | TypeScript-specific testing patterns (requires `/testing`) |
| `/coding-typescript`                 | Implementation workhorse with remediation loop             |
| `/reviewing-typescript`              | Strict code review with zero-tolerance                     |
| `/architecting-typescript`           | ADR producer with testing strategy                         |
| `/reviewing-typescript-architecture` | ADR validator against testing principles                   |

### Core Principles

- No mocking - dependency injection only
- Reality is the oracle
- Behavior testing, not implementation testing
- Tests at appropriate levels (Unit/Integration/E2E)
- Pure XML structure in all skills
- Self-contained skills (no inter-skill invocations)

## Python Plugin

Complete Python development workflow with testing, implementation, and review.

### Skills

| Skill                            | Purpose                                                |
| -------------------------------- | ------------------------------------------------------ |
| `/testing-python`                | Python-specific testing patterns (requires `/testing`) |
| `/coding-python`                 | Implementation workhorse with remediation loop         |
| `/reviewing-python`              | Strict code review with zero-tolerance                 |
| `/architecting-python`           | ADR producer with testing strategy                     |
| `/reviewing-python-architecture` | ADR validator against testing principles               |

### Commands

| Command       | Purpose                                |
| ------------- | -------------------------------------- |
| `/autopython` | Autonomous implementation orchestrator |

### Core Principles

- No mocking - dependency injection only
- Reality is the oracle
- Behavior testing, not implementation testing
- Tests at appropriate levels (Unit/Integration/E2E)

## Claude Plugin

Productivity skills and commands for Claude Code.

### Skills

| Skill                 | Purpose                                   |
| --------------------- | ----------------------------------------- |
| `/creating-skills`    | Create production-grade, reusable skills  |
| `/committing-changes` | Comprehensive git commit message guidance |

### Commands

| Command    | Purpose                                             |
| ---------- | --------------------------------------------------- |
| `/ci`      | Git commit with Conventional Commits (auto-context) |
| `/handoff` | Create timestamped context handoff                  |
| `/pickup`  | Load and continue from previous handoff             |

## Specs Plugin

Requirements documentation and specification skills.

### Skills

| Skill                             | Purpose                                                                                                 |
| --------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `/writing-technical-requirements` | Systematic TRD creation with testing methodology, validation strategy, and infrastructure documentation |
| `/writing-product-requirements`   | Systematic PRD creation with user value proposition, measurable outcomes, and acceptance criteria       |

### Core Principles

- Testing is first-class: Validation strategy documented before implementation
- Three-tier testing: Level 1 (Unit) → Level 2 (Integration) → Level 3 (E2E)
- No mocking: Dependency injection + real infrastructure
- Infrastructure explicit: Test harnesses and credentials documented or tracked as gaps
- User confirmation required: Problem understanding and measurable outcomes
- Quantified value: Measurable outcomes with X% improvement targets

## Discovering Other Installed Skills

Search for `SKILL.md` in `.claude/plugins/cache/{marketplace-name}/{plugin-name}/`

---

## For AI Agents Modifying This Marketplace

### Before Making Changes

1. **Read the context**: Check [CLAUDE.md](CLAUDE.md:1) (this file) for current structure and versioning rules
2. **Check existing commands**: Use Glob to find existing `.md` files in `plugins/*/commands/`
3. **Review plugin structure**: Each plugin has its own `plugin.json` in `.claude-plugin/`

### After Adding/Modifying Commands or Skills

**Determine version bump type** (see [Version Management](#version-management) above):

- **MAJOR** (0.x.x → 1.x.x): ⛔ NEVER unless user explicitly requests
- **MINOR** (0.3.x → 0.4.x): New command/skill OR major functional change
- **PATCH** (0.3.x → 0.3.1): Bug fixes, refactoring, small changes (MOST COMMON)

**Update plugin.json**:

```bash
# Location: plugins/{plugin-name}/.claude-plugin/plugin.json
# Update "version" field according to rules above
```

**Update marketplace description** (only if needed):

```bash
# Location: .claude-plugin/marketplace.json
# Update description for the modified plugin (only if description changes)
```

**Document changes**: Update this [CLAUDE.md](CLAUDE.md:1) file if adding new commands/skills to the plugin tables

### Quick Reference: File Locations

```
spx-claude/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog
├── plugins/
│   ├── claude/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Version: 0.4.0
│   │   ├── commands/
│   │   │   ├── ci.md
│   │   │   ├── handoff.md
│   │   │   └── pickup.md
│   │   └── skills/
│   │       ├── committing-changes/
│   │       └── creating-skills/
│   ├── python/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Version: 0.x.x
│   │   └── commands/
│   │       └── autopython.md
│   ├── typescript/
│   │   └── .claude-plugin/
│   │       └── plugin.json       # Version: 0.x.x
│   └── test/
│       └── .claude-plugin/
│           └── plugin.json       # Version: 0.x.x
└── CLAUDE.md                      # This file
```

### Versioning Reminder

**When in doubt:**

- Most changes = PATCH version bump
- New items or major changes = MINOR version bump
- Major version stays at 0.x.x unless user requests otherwise
