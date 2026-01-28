# Feature: [Feature Name]

## Purpose

[What this feature delivers and why it matters to the capability.]

## Requirements

[Prose description of functional and quality requirements. Constraints and invariants that tests must verify.]

## Test Strategy

| Component        | Level | Harness       | Rationale                         |
| ---------------- | ----- | ------------- | --------------------------------- |
| [Logic/parsing]  | 1     | -             | Pure function, can verify with DI |
| [Tool execution] | 2     | [harness-ref] | Needs real binary                 |

### Escalation Rationale

- **1 → 2**: [What confidence does Level 2 add that Level 1 cannot provide?]

## Feature Tests

### FI1: [Primary integration scenario]

```gherkin
GIVEN [real environment with tools]
WHEN [feature action]
THEN [integrated behavior verified]
```

| File                                                   | Level | Harness                                      |
| ------------------------------------------------------ | ----- | -------------------------------------------- |
| [{slug}.integration](tests/{slug}.integration.test.ts) | 2     | [harness-name](spx/capability-NN/feature-NN) |

### FI2: [Error handling scenario]

```gherkin
GIVEN [error condition]
WHEN [feature action]
THEN [graceful failure with clear message]
```

| File                                                                 | Level | Harness                                      |
| -------------------------------------------------------------------- | ----- | -------------------------------------------- |
| [{slug}-errors.integration](tests/{slug}-errors.integration.test.ts) | 2     | [harness-name](spx/capability-NN/feature-NN) |

## Completion Criteria

- [ ] All Level 1 tests pass (via story completion)
- [ ] All Level 2 integration tests pass
- [ ] Escalation rationale documented
