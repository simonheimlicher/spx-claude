# Story: [Story Name]

## Purpose

[What this story delivers and why it matters.]

## Outcomes

### 1. [Outcome name]

```gherkin
GIVEN [precondition]
WHEN [action]
THEN [expected result]
AND [additional assertion]
```

#### Test Files

| File                                     | Level | Harness |
| ---------------------------------------- | ----- | ------- |
| [{slug}.unit](tests/{slug}.unit.test.ts) | 1     | -       |

#### Analysis

*Implementation may diverge as understanding deepens.*

| File               | Intent         |
| ------------------ | -------------- |
| `src/path/file.ts` | [What changes] |
| `src/path/new.ts`  | [What it does] |

| Constant                            | Intent                    |
| ----------------------------------- | ------------------------- |
| `src/path/constants.ts::CONST_NAME` | [Why using existing]      |
| `src/path/constants.ts::NEW_CONST`  | [Why new constant needed] |

| Config Parameter | Test Values      | Expected Behavior   |
| ---------------- | ---------------- | ------------------- |
| `ENV_VAR`        | `unset`, `value` | [Behavior for each] |

---

### 2. [Second outcome if needed]

```gherkin
GIVEN [precondition]
WHEN [action]
THEN [expected result]
```

#### Test Files

| File                                                   | Level | Harness                                                                       |
| ------------------------------------------------------ | ----- | ----------------------------------------------------------------------------- |
| [{slug}.integration](tests/{slug}.integration.test.ts) | 2     | [harness-name](specs/work/doing/capability-NN/feature-NN_test-infrastructure) |

#### Analysis

*Implementation may diverge as understanding deepens.*

| File               | Intent         |
| ------------------ | -------------- |
| `src/path/file.ts` | [What changes] |

| Constant                          | Intent |
| --------------------------------- | ------ |
| `src/path/constants.ts::EXISTING` | [Why]  |

*No configuration for this outcome.*

---

## Architectural Constraints

| ADR                                     | Constraint                               |
| --------------------------------------- | ---------------------------------------- |
| [adr-NN_name](decisions/adr-NN_name.md) | [What constraint this ADR imposes]       |
| [pdr-NN_name](decisions/pdr-NN_name.md) | [What product behavior this PDR governs] |
