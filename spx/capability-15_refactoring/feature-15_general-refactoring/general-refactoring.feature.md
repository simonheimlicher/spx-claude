# Feature: General Refactoring

## Observable Outcome

Plugin assets (skills, commands, agents) are updated to comply with ADRs and documented patterns, improving consistency and reducing Claude errors when accessing skill resources.

## Scope

This feature covers refactoring stories that:

- Apply ADR decisions to existing assets
- Standardize documentation patterns
- Fix structural issues identified in audits

## Tests

- [Integration: Skills have accessing_skill_files section](tests/skill-files-section.integration.test.ts)
- [Integration: Orchestrators reference skills correctly](tests/orchestrator-references.integration.test.ts)

## Capability Contribution

Each completed story improves the overall quality and consistency of the plugin marketplace, reducing maintenance burden and improving Claude's reliability when using skills.

## Completion Criteria

- [ ] All stories in this feature complete
- [ ] Affected assets pass validation
