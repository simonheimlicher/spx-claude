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

## Outcomes

### 1. [Primary integration scenario]

```gherkin
GIVEN [real environment with tools]
WHEN [feature action]
THEN [integrated behavior verified]
```

| File                                                   | Level | Harness                                                                       |
| ------------------------------------------------------ | ----- | ----------------------------------------------------------------------------- |
| [{slug}.integration](tests/{slug}.integration.test.ts) | 2     | [harness-name](specs/work/doing/capability-NN/feature-NN_test-infrastructure) |

---

### 2. [Error handling scenario]

```gherkin
GIVEN [error condition]
WHEN [feature action]
THEN [graceful failure with clear message]
```

| File                                                                 | Level | Harness                                                                       |
| -------------------------------------------------------------------- | ----- | ----------------------------------------------------------------------------- |
| [{slug}-errors.integration](tests/{slug}-errors.integration.test.ts) | 2     | [harness-name](specs/work/doing/capability-NN/feature-NN_test-infrastructure) |

---

## Architectural Constraints

| ADR                                     | Constraint                               |
| --------------------------------------- | ---------------------------------------- |
| [adr-NN_name](decisions/adr-NN_name.md) | [What constraint this ADR imposes]       |
| [pdr-NN_name](decisions/pdr-NN_name.md) | [What product behavior this PDR governs] |
