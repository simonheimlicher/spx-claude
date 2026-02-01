# PRD: SPX-Claude Plugin Marketplace

> **Purpose**: Provide a collection of Claude Code plugins for spec-driven development using the SPX framework.

## Product Vision

### User Problem

**Who** are we building for? Solo developers using Claude Code who want structured, spec-driven development workflows.

**What** problem do they face?

```
As a solo developer using Claude Code, I am frustrated by inconsistent AI-assisted development
because Claude lacks structured workflows for spec-driven development,
which prevents me from getting reliable, repeatable results that follow requirements and decisions.
```

### Customer Solution

```
Provide a plugin marketplace that enables Claude Code users to adopt spec-driven development
through skills, commands, and agents, resulting in consistent implementations that honor
specifications, tests, and architectural decisions.
```

## Plugins

| Plugin     | Purpose                                                |
| ---------- | ------------------------------------------------------ |
| core       | Productivity skills: commits, handoffs, skill creation |
| specs      | Spec management: PRDs, TRDs, ADRs, work items          |
| test       | Testing methodology: three-level BDD testing           |
| python     | Python development: coding, reviewing, testing         |
| typescript | TypeScript development: coding, reviewing, testing     |
| frontend   | Frontend design: UI/UX patterns                        |

## Success Criteria

- All plugins follow consistent patterns
- Skills work independently AND via orchestrators
- Tests co-located with specs, not graduated
- Status tracked via `outcomes.yaml`, not directory location
