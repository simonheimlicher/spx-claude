# Customer Outcome Driven Engineering (CODE)

## Rationale

CODE treats specs as durable, version-controlled product structure—not "work to implement." The `spx/` tree is the always-current map of the product. Each container holds its spec, tests, and pass ledger together, so agents can discover, validate, and evolve the system with minimal guesswork. Outcomes are proven by `outcomes.yaml` files validated at precommit and insured by CI.

**Important**: Use the `spx` CLI for all structural operations (like `gh` for GitHub).

---

## Principles

1. **Durable map**
   The spec tree is the always-on system map. Nothing moves because work is "done."

2. **Co-location**
   Each container holds its spec file, tests directory, and pass ledger together. No parallel trees to synchronize.

3. **Type in names, order in directories**
   Container type is in both directory and file names. Ordering (BSP) is in directory names only, so rebalancing never requires file renames.

4. **Tests are the executable proof**
   The spec describes intent, constraints, and strategy. Tests prove the implementation works. `outcomes.yaml` is the machine-verifiable contract.

5. **Incomplete is valid**
   A container with 2 of 5 tests passing is in progress, not broken. `outcomes.yaml` lists only passing tests.

6. **Blob-based staleness**
   Staleness is detected by comparing git blob SHAs, not timestamps. This is deterministic across rebases, checkouts, and touched files.

7. **Precommit is primary, CI is insurance**
   Agents get feedback at precommit. CI validates that precommit wasn't bypassed.

8. **Decisions are durable and updated in-place**
   No "superseded" decisions. When a decision changes, update it in place.

9. **Tool-mediated refactors**
   Moves/renames are performed via `spx` CLI to keep structure consistent.

10. **Test infrastructure is first-class**
    Harnesses are production code requiring their own specs and test coverage.

11. **Higher levels unaware of lower level breakdown**
    Features don't list story outcomes. Capabilities don't list feature outcomes. Completion bubbles up through `outcomes.yaml`, not spec references. This prevents drift between levels.

12. **Analysis sections are not contracts**
    Story specs include analysis of files, constants, and configuration to prove the agent examined the codebase. Implementation may diverge as understanding deepens—this is expected, not a failure.

13. **No stale references in specs**
    Specs contain only durable information. Dependencies are encoded in the tree (BSP ordering), not listed in spec files. Harness references appear in test tables, not separate sections. Nothing that can go stale belongs in a spec.

---

## Structure

### Container hierarchy

```
spx/
  {product-slug}.prd.md              # Product requirements (one per product)
  NN-{slug}.adr.md                   # Product-wide decisions
  13-test-infrastructure.capability/ # Harnesses are first-class
    {slug}.capability.md
    NN-{slug}.feature/
      {slug}.feature.md
      outcomes.yaml
      tests/
  NN-{slug}.capability/
    {slug}.capability.md
    outcomes.yaml                         # Pass ledger (may be incomplete)
    tests/
    NN-{slug}.adr.md
    NN-{slug}.feature/
      {slug}.feature.md
      outcomes.yaml
      tests/
      NN-{slug}.adr.md
      NN-{slug}.story/
        {slug}.story.md
        outcomes.yaml
        tests/
```

### Directory naming

```
{BSP}-{slug}.{type}/
```

Examples:

- `13-test-infrastructure.capability/`
- `15-validation.capability/`
- `21-testable-validation.feature/`
- `47-validation-commands.story/`

See [Appendix: Recursive Decimal BSP](#appendix-recursive-decimal-bsp) for insertion algorithms.

### Spec file naming

Inside each container, the canonical spec file:

```
{slug}.{type}.md
```

Examples:

- `15-validation.capability/validation.capability.md`
- `21-testable-validation.feature/testable-validation.feature.md`
- `47-validation-commands.story/validation-commands.story.md`

BSP is in directory only—renumbering never touches spec files.

### Decisions (ADR)

- Location: in the container where the decision applies
- Naming: `NN-{slug}.adr.md`
- Updated in-place; no "superseded" workflow

---

## Capability and Feature Spec Format

Capabilities and features describe **intent, constraints, and strategy**. Tests are the **executable proof**.

### Structure

```markdown
# {Capability|Feature}: {Name}

## Purpose

What this container delivers and why it matters.

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

## Outcomes

### 1. [Scenario name]

\`\`\`gherkin
GIVEN [precondition]
WHEN [action]
THEN [expected result]
\`\`\`

| File                                   | Level | Harness                                                                     |
| -------------------------------------- | ----- | --------------------------------------------------------------------------- |
| [{slug}.e2e](tests/{slug}.e2e.test.ts) | 3     | [e2e-harness](spx/NN-test-infrastructure.capability/NN-e2e-harness.feature) |

## Architectural Constraints

| ADR                              | Constraint                         |
| -------------------------------- | ---------------------------------- |
| [adr-NN_name](../adr-NN_name.md) | [What constraint this ADR imposes] |
```

### Rationale

- **No drift** between spec prose and test ledger—`outcomes.yaml` is the sole contract
- **Test strategy** documents approach without duplicating test logic
- **Harness references** point to the harness spec (the durable contract), not the implementation code
- **Prose requirements** capture intent that tests alone cannot express
- **Outcomes** use the same numbered Gherkin pattern as stories for consistency
- **Architectural Constraints** reference ADRs that impose requirements on this container

---

## Story Spec Format

Stories describe **specific outcomes with implementation details**. Unlike capabilities and features, stories document the exact implementation path as proof of analysis.

### Structure

```markdown
# Story: {Name}

## Purpose

What this story delivers and why it matters.

## Outcomes

### 1. {Outcome name}

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

### 2. {Second outcome if needed}

{Same structure: gherkin + 4 tables}

---

## Architectural Constraints

| ADR                              | Constraint                         |
| -------------------------------- | ---------------------------------- |
| [NN-name.adr](../NN-name.adr.md) | {What constraint this ADR imposes} |
```

### Rationale

- **Gherkin is source of truth**—tests implement it, spec doesn't contain code
- **Test Files table** is the contract—harness references here, not in separate sections
- **Analysis section proves examination**—agent looked before coding, but implementation may diverge
- **No completion criteria**—stories are atomic; outcomes.yaml is the contract
- **No inline code**—code in specs drifts from actual implementation

---

## Tests

Each container has a `tests/` subdirectory containing all tests for that outcome.

### Test naming

Test level is in the filename suffix:

- `*.unit.test.ts`
- `*.integration.test.ts`
- `*.e2e.test.ts`

### Path derivation

For each `test_file` in `outcomes.yaml`, SPX derives the runnable path as:

```
<container>/tests/<test_file>
```

The `tests/` prefix is never stored in `outcomes.yaml`.

### Test runner configuration

```ts
// vitest.config.ts
export default {
  include: ["spx/**/tests/*.test.ts"],
};
```

---

## Outcome Ledger (outcomes.yaml)

Each container MAY have an `outcomes.yaml` listing tests that currently pass. This is the **machine-verifiable proof** for the container.

### Format

```yaml
spec_blob: a3f2b7c...
committed_at: 2026-01-28T14:15:00Z
tests:
  - file: parsing.unit.test.ts
    blob: 1f2e...
    passed_at: 2026-01-27T10:30:00Z
  - file: cli.integration.test.ts
    blob: 9ac4...
    passed_at: 2026-01-28T14:15:00Z
```

| Field          | Description                                                    |
| -------------- | -------------------------------------------------------------- |
| `spec_blob`    | Git blob SHA of container spec file when outcome was committed |
| `committed_at` | ISO 8601 timestamp of last commit operation                    |
| `file`         | Filename relative to container's `tests/` directory            |
| `blob`         | Git blob SHA of test file content when it last passed          |
| `passed_at`    | ISO 8601 timestamp of when that test last passed               |

### State derivation

| Condition                              | State     | Required Action                 |
| -------------------------------------- | --------- | ------------------------------- |
| Test Files links don't resolve         | Unknown   | Write tests                     |
| Tests exist, not all passing           | Pending   | Fix code or fix tests           |
| Spec or test blob changed since commit | Stale     | Re-commit with `spx spx commit` |
| All tests pass, blobs unchanged        | Passing   | None—potential realized         |
| Was passing, now fails, blobs same     | Regressed | Investigate and fix             |

### Failure classification

When a test in `outcomes.yaml` fails:

| `spec_blob` | `test_blob`            | Diagnosis                                                     |
| ----------- | ---------------------- | ------------------------------------------------------------- |
| Unchanged   | Unchanged              | **Regression** - implementation, dependency, or harness broke |
| Unchanged   | Changed                | **Stale test** - test was modified, needs re-commit           |
| Changed     | Unchanged              | **Stale spec** - spec was modified, needs re-commit           |
| Changed     | Changed                | **Stale both** - spec and test modified, needs re-commit      |
| N/A         | Not in `outcomes.yaml` | **In progress** - never passed, not a regression              |

### Rationale

- **Blob-based comparison** is deterministic across rebases, checkouts, touched files
- **Per-test tracking** enables precise diagnosis: "this test passed 3 days ago with blob X"
- **Incomplete ledgers are valid** - partial progress is normal, not an error state

### Relationship to Git

Git is a Merkle tree: every blob is content-addressed by SHA, every tree hashes its children, every commit hashes its root tree. This provides cryptographic integrity of *content*.

`outcomes.yaml` extends Git with *test verification state*:

```text
Git Merkle Tree                    outcomes.yaml (verification)
─────────────────                  ─────────────────────────────
spx/
├── auth.capability.md  ←────────  spec_blob: a3f2b7c
├── outcomes.yaml
└── tests/
    └── auth.unit.test.ts  ←─────  blob: 9ac4..., passed_at: ...
```

Git answers "what is the content?" `outcomes.yaml` answers "did this content pass tests, and when?"

### Generating outcomes.yaml

`outcomes.yaml` MUST be generated by `spx spx commit`, never hand-edited.

```bash
spx spx commit <container>
```

The command:

1. Runs all tests in `tests/`
2. Writes `outcomes.yaml` with current `spec_blob` and `committed_at` timestamp
3. Adds each passing test with its `blob` and `passed_at`
4. May be incomplete (only passing tests are listed)

---

## Precommit Validation

Precommit is the primary feedback loop. CI is insurance.

For each container that has `tests/`:

### 1. Phantom check

Every `test_file` in `outcomes.yaml` must exist at runtime path `<container>/tests/<test_file>`.

- Missing file → **error** (phantom entry)

### 2. Regression check

Run exactly the tests listed in `outcomes.yaml` (derive paths as above).

If a listed test fails:

- Compute current blob of that test file
- If unchanged from `test_blob` → **Regression** (error, blocks commit)
- If changed → **Stale** (re-commit required)

### 3. Spec staleness check

Compute `spec_blob_now` from current spec file.

- If `spec_blob_now != spec_blob` in header → **Stale** (re-commit required)

### 4. Progress tests rule

Tests in `tests/` but not in `outcomes.yaml` are **in progress** (not an error).

### What this catches

| Scenario                                         | Result                     |
| ------------------------------------------------ | -------------------------- |
| Test in `outcomes.yaml` fails, nothing changed   | Regression - blocks commit |
| Test in `outcomes.yaml` fails, test file changed | Stale - re-commit required |
| Spec changed since last commit                   | Stale - re-commit required |
| Test file deleted but still in `outcomes.yaml`   | Phantom - blocks commit    |
| New test file not yet in `outcomes.yaml`         | In progress - OK           |

### Flow

1. **Write spec**: define intent, constraints, and test strategy
2. **Implement**: write code and tests
3. **Commit outcome**: run `spx spx commit <container>` to generate `outcomes.yaml`
4. **Commit**: spec + implementation + tests + outcomes.yaml together
5. **Precommit**: validates no regressions, no phantoms, no staleness
6. **CI**: re-runs validation as insurance

---

## Test Infrastructure

Test harnesses are production code requiring their own specifications and test coverage.

### Location

Harness **specs** live in `spx/`; harness **code** lives in `tests/harness/` (same level as `src/`):

```
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
        │   ├── outcomes.yaml
        │   └── tests/                    # Tests for the harness itself
        │       ├── setup.unit.test.ts
        │       └── isolation.integration.test.ts
        └── 20-e2e-harness.feature/
            ├── e2e-harness.feature.md
            ├── outcomes.yaml
            └── tests/
```

**Key distinction**: Harness specs in `spx/` define *what the harness must do*. Harness code in `tests/harness/` is *the implementation*. Tests in `spx/.../tests/` verify the harness works correctly.

**Harness references in Test Files tables point to the spec**, not the implementation. The spec is the durable contract; code organization can change.

### Why harnesses need specs

1. **Harness bugs break all dependent tests** - high impact requires tracking
2. **Harness refactoring needs regression protection** - `outcomes.yaml` detects breakage
3. **Harness capabilities need documentation** - what can tests rely on?
4. **Harness dependencies need BSP ordering** - tests depend on harness (lower BSP)

### Harness spec format

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

## Reference Layout

```text
spx/
  myproduct.prd.md
  21-testing-strategy.adr.md
  13-test-infrastructure.capability/
    test-infrastructure.capability.md
    10-cli-harness.feature/
      cli-harness.feature.md
      outcomes.yaml
      tests/
        setup.unit.test.ts
  15-validation.capability/
    validation.capability.md
    outcomes.yaml
    tests/
      validation.e2e.test.ts
    21-isolated-test-fixtures.adr.md
    21-testable-validation.feature/
      testable-validation.feature.md
      outcomes.yaml
      tests/
        validation.integration.test.ts
      04-level-2-integration-only.adr.md
      47-validation-commands.story/
        validation-commands.story.md
        outcomes.yaml
        tests/
          commands.unit.test.ts
```

---

## Appendix: Recursive Decimal BSP

**Binary Space Partitioning (BSP)** encodes dependency order: lower BSP items are dependencies that higher-BSP items may rely on; same BSP means independent. The "binary" refers to insertion by halving available space.

### Syntax

```text
{BSP}-{slug}.{type}
```

- **BSP**: Two digits (10-99), with `@` for recursive insertion
- **`-`**: Slug separator (hyphen, ASCII 45)
- **slug**: Human-readable name
- **type**: `.capability`, `.feature`, `.story`, or `.adr`

### Why Hyphen

Filesystems sort by ASCII code. Hyphen (45) sorts before `@` (64), ensuring parents appear before children:

| Character | ASCII | Sort Position |
| --------- | ----- | ------------- |
| `-`       | 45    | Parent        |
| `@`       | 64    | Children      |

```text
20-auth.capability/      ← Parent (- sorts first)
20@50-mfa.feature/       ← Child (@ sorts after -)
21-billing.capability/   ← Sibling (2 > @)
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

**Recursive insert** (between 20 and 21, no integer space):

```text
20@floor((10 + 99) / 2) = 20@54 → 20@54-audit
```

**Deep recursion** (between 20@54 and 20@55):

```text
20@54@50-detailed-trace
```

### Rules

1. **Hyphen only**: Always `-` between BSP and slug, never `_`
2. **No renaming**: `20-auth` stays `20-auth` forever; use `@` to insert
3. **Rebalance at depth 3**: If you reach `20@50@50@...`, consider rebalancing
4. **Same BSP = parallel**: `20-auth.capability/` and `20-billing.capability/` can be built concurrently
