# Outcome Engineering: Reference

Detailed spec formats, naming conventions, test infrastructure, and filesystem reference for the [Outcome Engineering framework](outcome-engineering-framework.md).

---

## Capability and Feature Spec Format

Capabilities and features describe **outcome hypotheses and the outputs required to validate them**. Tests validate the outputs.

### Structure

```markdown
# {Capability|Feature}: {Name}

## Purpose

{What outcome this contributes to — locally validated through the outputs below}

## Requirements

Prose description of functional and quality requirements.
Constraints and invariants that tests must verify.

## Test Strategy

| Component         | Level | Harness     | Rationale                       |
| ----------------- | ----- | ----------- | ------------------------------- |
| Argument parsing  | 1     | -           | Pure function, standard dev env |
| Config validation | 1     | -           | Pure function with temp dirs    |
| CLI integration   | 2     | cli-harness | Needs real spx binary           |
| Full workflow     | 3     | e2e-harness | Needs credentials               |

### Escalation Rationale

- **1 → 2**: [What confidence does Level 2 add that Level 1 cannot provide?]
- **2 → 3**: [What confidence does Level 3 add that Level 2 cannot provide?]

## Assertions

### 1. [Scenario name] (Scenario)

\`\`\`gherkin
GIVEN [precondition]
WHEN [action]
THEN [expected result]
\`\`\`

| File                                   | Level | Harness                                                                     |
| -------------------------------------- | ----- | --------------------------------------------------------------------------- |
| [{slug}.e2e](tests/{slug}.e2e.test.ts) | 3     | [e2e-harness](spx/NN-test-infrastructure.capability/NN-e2e-harness.feature) |

### 2. [Mapping name] (Mapping)

> [What universal claim this makes over the finite set]

| Input      | Expected Output |
| ---------- | --------------- |
| [member 1] | [result 1]      |
| [member 2] | [result 2]      |
| [member 3] | [result 3]      |

| File                                     | Level | Harness |
| ---------------------------------------- | ----- | ------- |
| [{slug}.unit](tests/{slug}.unit.test.ts) | 1     | -       |

### 3. [Conformance name] (Conformance)

**Conforms to:** [standard or tool reference]

**Predicate:** [what the oracle checks — e.g., zero warnings, zero errors]

| File                                                   | Level | Harness     |
| ------------------------------------------------------ | ----- | ----------- |
| [{slug}.integration](tests/{slug}.integration.test.ts) | 2     | cli-harness |

## Architectural Constraints

| ADR                              | Constraint                         |
| -------------------------------- | ---------------------------------- |
| [adr-NN_name](../adr-NN_name.md) | [What constraint this ADR imposes] |
```

### Rationale

- **Purpose** states the outcome hypothesis — what we believe this contributes to (WHY)
- **Requirements** describe the outputs that must be true for the hypothesis to hold (WHAT)
- **Assertions** are structured output specifications (Scenario, Mapping, Conformance, Property) — locally testable prerequisites
- **Test strategy** documents approach without duplicating test logic
- **Harness references** point to the harness spec (the durable contract), not the implementation code
- **Architectural Constraints** reference ADRs/PDRs that constrain how outputs are produced (HOW)

---

## Story Spec Format

Stories describe **specific output specifications with implementation details**. Unlike capabilities and features, stories document the exact implementation path as proof of analysis.

### Structure

```markdown
# Story: {Name}

## Purpose

{What outcome this story contributes to — locally validated through the outputs below}

## Assertions

### 1. {Scenario name} (Scenario)

\`\`\`gherkin
GIVEN {precondition}
WHEN {action}
THEN {expected result}
AND {additional assertion}
\`\`\`

#### Test Files

| File                                     | Level | Harness |
| ---------------------------------------- | ----- | ------- |
| [{slug}.unit](tests/{slug}.unit.test.ts) | 1     | -       |

#### Analysis

*Implementation may diverge as understanding deepens.*

| File               | Intent         |
| ------------------ | -------------- |
| `src/path/file.ts` | {What changes} |
| `src/path/new.ts`  | {What it does} |

| Constant                       | Intent                    |
| ------------------------------ | ------------------------- |
| `src/constants.ts::CONST_NAME` | {Why using existing}      |
| `src/constants.ts::NEW_CONST`  | {Why new constant needed} |

| Config Parameter | Test Values      | Expected Behavior   |
| ---------------- | ---------------- | ------------------- |
| `ENV_VAR`        | `unset`, `value` | {Behavior for each} |

---

### 2. {Mapping name} (Mapping)

> {What universal claim this makes over the finite set}

| Input      | Expected Output |
| ---------- | --------------- |
| {member 1} | {result 1}      |
| {member 2} | {result 2}      |

#### Test Files

| File                                     | Level | Harness |
| ---------------------------------------- | ----- | ------- |
| [{slug}.unit](tests/{slug}.unit.test.ts) | 1     | -       |

#### Analysis

*Implementation may diverge as understanding deepens.*

| File               | Intent         |
| ------------------ | -------------- |
| `src/path/file.ts` | {What changes} |

---

## Architectural Constraints

| ADR                              | Constraint                         |
| -------------------------------- | ---------------------------------- |
| [NN-name.adr](../NN-name.adr.md) | {What constraint this ADR imposes} |
```

### Rationale

- **Purpose** states the outcome hypothesis at the atomic level (WHY)
- **Structured output specifications are source of truth**—tests implement them, spec doesn't contain code
- **Test Files table** is the contract—harness references here, not in separate sections
- **Analysis section proves examination**—agent looked before coding, but implementation may diverge
- **No completion criteria**—stories are atomic; the status file tracks which outputs are validated
- **No inline code**—code in specs drifts from actual implementation

---

## Filesystem Conventions

### Directory Naming

```text
{index}-{slug}.{type}/
```

Examples:

- `13-test-infrastructure.capability/`
- `15-validation.capability/`
- `21-testable-validation.feature/`
- `47-validation-commands.story/`

### Spec File Naming

Inside each node, the canonical spec file:

```text
{slug}.{type}.md
```

Examples:

- `15-validation.capability/validation.capability.md`
- `21-testable-validation.feature/testable-validation.feature.md`
- `47-validation-commands.story/validation-commands.story.md`

The index is in directory names only—renumbering never touches spec files.

### Decisions (ADR)

- Location: in the node where the decision applies
- Naming: `NN-{slug}.adr.md`
- Updated in-place; no "superseded" workflow

### Test Naming

Test level is in the filename suffix:

- `*.unit.test.ts`
- `*.integration.test.ts`
- `*.e2e.test.ts`

### Path Derivation

For each `test_file` in `status.yaml`, SPX derives the runnable path as:

```text
<node>/tests/<test_file>
```

The `tests/` prefix is never stored in `status.yaml`.

### Test Runner Configuration

```ts
// vitest.config.ts
export default {
  include: ["spx/**/tests/*.test.ts"],
};
```

---

## Test Infrastructure

Test harnesses are production code requiring their own specifications and test coverage.

### Location

Harness **specs** live in `spx/`; harness **code** lives in `tests/harness/` (same level as `src/`):

```text
project/
├── src/                                  # Implementation code
├── tests/
│   └── harness/                          # Harness implementation code
│       ├── cli/                          # CLI harness code
│       └── e2e/                          # E2E harness code
└── spx/
    └── 13-test-infrastructure.capability/  # Harness specs
        ├── test-infrastructure.capability.md
        ├── 10-cli-harness.feature/
        │   ├── cli-harness.feature.md
        │   ├── status.yaml
        │   └── tests/                    # Tests for the harness itself
        │       ├── setup.unit.test.ts
        │       └── isolation.integration.test.ts
        └── 20-e2e-harness.feature/
            ├── e2e-harness.feature.md
            ├── status.yaml
            └── tests/
```

**Key distinction**: Harness specs in `spx/` define *what the harness must do*. Harness code in `tests/harness/` is *the implementation*. Tests in `spx/.../tests/` verify the harness works correctly.

**Harness references in Test Files tables point to the spec**, not the implementation. The spec is the durable contract; code organization can change.

### Why Harnesses Need Specs

1. **Harness bugs break all dependent tests** - high impact requires tracking
2. **Harness refactoring needs regression protection** - the status file detects breakage
3. **Harness capabilities need documentation** - what can tests rely on?
4. **Harness dependencies need index ordering** - tests depend on harness (lower index)

### Harness Spec Format

````markdown
# Feature: CLI Harness

## Purpose

Provides isolated CLI execution environment for Level 2 tests.

## Capabilities

- Spawns spx binary in temp directory
- Captures stdout/stderr
- Manages fixture files
- Cleans up after test

## Test Strategy

| Component           | Level | Rationale                        |
| ------------------- | ----- | -------------------------------- |
| Temp dir management | 1     | Pure filesystem with os.tmpdir() |
| Process spawning    | 2     | Needs real spx binary            |

## Usage

```typescript
import { withCliHarness } from "tests/harness/cli";

it("runs spx spec next", async () => {
  await withCliHarness(async (harness) => {
    const result = await harness.run("spx", ["spec", "next"]);
    expect(result.exitCode).toBe(0);
  });
});
```

## Dependencies

- spx binary must be built
- Node.js >= 18
````

---

## Precommit Validation

### What Precommit Catches

| Scenario                                   | Result                     |
| ------------------------------------------ | -------------------------- |
| Recorded test fails                        | Regression - blocks commit |
| Descendant status file changed             | Stale - re-record required |
| Test file deleted but still in status file | Phantom - blocks commit    |
| New test file not yet in status file       | In progress - OK           |

For detailed validation rules, see [status.md](status.md)

---

## Reference Layout

```text
spx/
  myproduct.prd.md
  21-testing-strategy.adr.md
  13-test-infrastructure.capability/
    test-infrastructure.capability.md
    10-cli-harness.feature/
      cli-harness.feature.md
      status.yaml
      tests/
        setup.unit.test.ts
  15-validation.capability/
    validation.capability.md
    status.yaml
    tests/
      validation.e2e.test.ts
    21-isolated-test-fixtures.adr.md
    21-testable-validation.feature/
      testable-validation.feature.md
      status.yaml
      tests/
        validation.integration.test.ts
      04-level-2-integration-only.adr.md
      47-validation-commands.story/
        validation-commands.story.md
        status.yaml
        tests/
          commands.unit.test.ts
```

---

## Fractional Indexing

Directory ordering uses **fractional indexing** — a well-known technique for order-preserving insertion (used in CRDTs, Figma layer ordering, database B-trees). Lower indices encode dependencies; higher indices may rely on lower ones; same index means independent.

### Syntax

```text
{index}-{slug}.{type}
```

- **index**: Two digits (10–99), with dot-separated fractions for insertion (e.g., `20.54`)
- **`-`**: Separator between index and slug (hyphen, ASCII 45)
- **slug**: Human-readable name
- **type**: `.capability`, `.feature`, `.story`, or `.adr`

### Sort Order

Filesystems sort by ASCII code. Hyphen (45) sorts before dot (46), so integer entries appear before their fractional insertions:

| Character | ASCII | Role                |
| --------- | ----- | ------------------- |
| `-`       | 45    | Index–slug boundary |
| `.`       | 46    | Fractional level    |

```text
20-auth.capability/       ← Integer index (- sorts first)
20.54-audit.capability/   ← Fractional insert (. sorts after -)
21-billing.capability/    ← Next integer (2 > .)
```

### Insertion Algorithms

**Append** (new sibling after 20):

```text
floor((20 + 99) / 2) = 59 → 59-billing
```

**Insert** (between 20 and 30):

```text
floor((20 + 30) / 2) = 25 → 25-subscriptions
```

**Fractional insert** (between 20 and 21, no integer space):

```text
20.floor((10 + 99) / 2) = 20.54 → 20.54-audit
```

**Deep fraction** (between 20.54 and 20.55):

```text
20.54.50-detailed-trace
```

### Rules

1. **Hyphen separator**: Always `-` between index and slug, never `_`
2. **No renaming**: `20-auth` stays `20-auth` forever; use fractional insertion instead
3. **Rebalance at depth 3**: If you reach `20.50.50.…`, consider rebalancing the level
4. **Same index = parallel**: `20-auth.capability/` and `20-billing.capability/` can be built concurrently
