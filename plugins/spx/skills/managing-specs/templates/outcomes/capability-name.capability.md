# Capability: [Capability Name]

## Purpose

[What this capability delivers and why it matters to users.]

## Success Metric

- **Baseline**: [Current measurable state]
- **Target**: [Expected improvement]
- **Measurement**: [How progress will be tracked]

## Requirements

[Prose description of functional and quality requirements. Constraints and invariants that tests must verify.]

## Test Strategy

| Component       | Level | Harness       | Rationale                              |
| --------------- | ----- | ------------- | -------------------------------------- |
| [Component 1]   | 1     | -             | [Why this level is minimum sufficient] |
| [Component 2]   | 2     | [harness-ref] | [Why this level is minimum sufficient] |
| [Full workflow] | 3     | e2e-harness   | [Why E2E verification is needed]       |

### Escalation Rationale

- **1 → 2**: [What confidence does Level 2 add that Level 1 cannot provide?]
- **2 → 3**: [What confidence does Level 3 add that Level 2 cannot provide?]

## Capability Tests (Level 3)

### E2E1: [Primary user journey]

```gherkin
GIVEN [full environment with real services]
WHEN [complete user workflow]
THEN [value delivered to user]
```

| File                                   | Level | Harness                                                         |
| -------------------------------------- | ----- | --------------------------------------------------------------- |
| [{slug}.e2e](tests/{slug}.e2e.test.ts) | 3     | [e2e-harness](spx/capability-13_test-infrastructure/feature-NN) |

### E2E2: [Alternative scenario]

```gherkin
GIVEN [alternative preconditions]
WHEN [user action]
THEN [expected behavior]
```

| File                                           | Level | Harness                                                         |
| ---------------------------------------------- | ----- | --------------------------------------------------------------- |
| [{slug}-alt.e2e](tests/{slug}-alt.e2e.test.ts) | 3     | [e2e-harness](spx/capability-13_test-infrastructure/feature-NN) |

## Completion Criteria

- [ ] All Level 1 tests pass (via feature/story completion)
- [ ] All Level 2 tests pass (via feature completion)
- [ ] All Level 3 E2E tests pass
- [ ] Success metric achieved
