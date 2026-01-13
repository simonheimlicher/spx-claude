# Capability: Plugin Asset Refactoring

## Purpose

Recurring maintenance and improvement of Claude Code plugin assets (skills, commands, agents, templates) to ensure consistency, correctness, and adherence to evolving best practices.

## Success Metric

- **Baseline**: Plugin assets drift from documented patterns over time
- **Target**: All plugin assets comply with current ADRs and documented patterns
- **Measurement**: Manual audit or automated linting of skill/command/agent files

## Scope

This capability covers refactoring work that:

- Updates existing assets to follow new ADRs
- Improves documentation patterns across skills
- Standardizes structure across plugins
- Does NOT add new functionality (that's a feature, not refactoring)

## Completion Criteria

- [ ] All identified refactoring stories complete
- [ ] Assets comply with referenced ADRs
