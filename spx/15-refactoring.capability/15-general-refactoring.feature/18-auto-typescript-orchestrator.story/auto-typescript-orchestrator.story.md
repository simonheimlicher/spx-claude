# Story: Refactor auto-typescript as Pure Orchestrator

## Relevant ADRs

- [ADR-24](../../../../adr-24_orchestrator-pattern.md) - Skills stay independent; orchestrators coordinate

## Problem

The current `auto-typescript` skill violates ADR-24 by duplicating content from execution skills rather than referencing them. The `<testing_emphasis>` section (40 lines) repeats rules already in `/testing-typescript`. The success criteria mixes orchestration concerns with implementation concerns.

## Target State

After this story:

1. **auto-typescript SKILL.md** is a pure orchestrator that:
   - Coordinates 4 skills: understanding-specs, testing-typescript, coding-typescript, reviewing-typescript
   - References skills by name, never duplicates their content
   - Uses `/skill-name` syntax (not `plugin:skill-name`)
   - Focuses success criteria on orchestration completeness

2. **auto-typescript command** stays minimal (invokes the skill)

## Functional Requirements

### FR1: Remove duplicate testing content

```gherkin
GIVEN auto-typescript SKILL.md has <testing_emphasis> section
WHEN applying ADR-24 orchestrator pattern
THEN <testing_emphasis> section is deleted
AND Step 2 (Design Tests) adds reference: "See /testing-typescript for complete methodology"
```

### FR2: Standardize skill invocation syntax

```gherkin
GIVEN skill invocations use `specs:understanding-specs` and `typescript:*` format
WHEN standardizing per Claude Code conventions
THEN all invocations use `/skill-name` format (no plugin prefix)
```

### FR3: Add error recovery guidance

```gherkin
GIVEN line 46 says "STOP. Create missing specs before proceeding."
WHEN improving error recovery
THEN line 46 references `/managing-specs` skill for creating missing specifications
```

### FR4: Simplify success criteria

```gherkin
GIVEN success criteria includes implementation concerns (mocking, constants pattern)
WHEN applying separation of concerns
THEN success criteria focuses only on orchestration:
  - All stories processed
  - Each story passed reviewer approval
  - All tests pass
AND implementation quality concerns are delegated to reviewing-typescript
```

## Tests

- [Unit: No testing_emphasis section](tests/no-testing-emphasis.unit.test.ts)
- [Unit: Uses /skill-name syntax](tests/skill-syntax.unit.test.ts)
- [Unit: Success criteria is orchestration-only](tests/success-criteria.unit.test.ts)

## Completion Criteria

- [ ] `<testing_emphasis>` section deleted
- [ ] All skill references use `/skill-name` format
- [ ] Error recovery references `/managing-specs`
- [ ] Success criteria focuses on orchestration completeness only
- [ ] `claude plugin validate plugins/typescript` passes
- [ ] Plugin version bumped (0.5.2 → 0.5.3)
