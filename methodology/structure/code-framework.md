# Customer Outcome Driven Engineering (CODE)

## Rationale

CODE treats specs as durable, version-controlled product structure—not "work to implement." The `spx/` tree is the always-current map of the product. Each container holds its spec, tests, and pass ledger together, so agents can discover, validate, and evolve the system with minimal guesswork. Outcomes are proven by `pass.csv` files validated at precommit and insured by CI.

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
   The spec describes intent, constraints, and strategy. Tests prove the implementation works. `pass.csv` is the machine-verifiable contract.

5. **Incomplete is valid**
   A container with 2 of 5 tests passing is in progress, not broken. `pass.csv` lists only passing tests.

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
    Features don't list story outcomes. Capabilities don't list feature outcomes. Completion bubbles up through `pass.csv`, not spec references. This prevents drift between levels.

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
  adr-NN_{slug}.md                   # Product-wide decisions
  capability-13_test-infrastructure/ # Harnesses are first-class
    {slug}.capability.md
    feature-NN_{slug}/
      {slug}.feature.md
      pass.csv
      tests/
  capability-NN_{slug}/
    {slug}.capability.md
    pass.csv                         # Pass ledger (may be incomplete)
    tests/
    adr-NN_{slug}.md
    feature-NN_{slug}/
      {slug}.feature.md
      pass.csv
      tests/
      adr-NN_{slug}.md
      story-NN_{slug}/
        {slug}.story.md
        pass.csv
        tests/
```

### Directory naming

```
{container-type}-{BSP}_{slug}/
```

Examples:

- `capability-13_test-infrastructure/`
- `capability-15_validation/`
- `feature-21_testable-validation/`
- `story-47_validation-commands/`

### Spec file naming

Inside each container, the canonical spec file:

```
{slug}.{container-type}.md
```

Examples:

- `capability-15_validation/validation.capability.md`
- `feature-21_testable-validation/testable-validation.feature.md`
- `story-47_validation-commands/validation-commands.story.md`

BSP is in directory only—renumbering never touches spec files.

### Decisions (ADR)

- Location: in the container where the decision applies
- Naming: `adr-NN_{slug}.md`
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

## Completion Criteria

- [ ] All Level 1 tests pass
- [ ] Level 2 tests pass with documented harness
```

### Rationale

- **No drift** between spec prose and test ledger—`pass.csv` is the sole contract
- **Test strategy** documents approach without duplicating test logic
- **Harness references** make infrastructure dependencies explicit
- **Prose requirements** capture intent that tests alone cannot express
- **Completion criteria** at capability/feature level only—stories have none

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
| [adr-NN_name](../adr-NN_name.md) | {What constraint this ADR imposes} |
```

### Rationale

- **Gherkin is source of truth**—tests implement it, spec doesn't contain code
- **Test Files table** is the contract—harness references here, not in separate sections
- **Analysis section proves examination**—agent looked before coding, but implementation may diverge
- **No completion criteria**—stories are atomic; pass.csv is the contract
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

For each `test_file` in `pass.csv`, SPX derives the runnable path as:

```
<container>/tests/<test_file>
```

The `tests/` prefix is never stored in `pass.csv`.

### Test runner configuration

```ts
// vitest.config.ts
export default {
  include: ["spx/**/tests/**/*.test.ts"],
};
```

---

## pass.csv: Test Ledger

Each container MAY have a `pass.csv` listing tests that currently pass. This is the **machine-verifiable proof** for the container.

### Format

```csv
# spec_blob,a3f2b7c...
# run,2026-01-28T14:15:00Z
test_file,test_blob,pass_time
parsing.unit.test.ts,1f2e...,2026-01-27T10:30:00Z
cli.integration.test.ts,9ac4...,2026-01-28T14:15:00Z
```

| Field       | Description                                                 |
| ----------- | ----------------------------------------------------------- |
| `spec_blob` | Git blob SHA of container spec file when ledger was stamped |
| `run`       | ISO 8601 timestamp of last stamp operation                  |
| `test_file` | Filename relative to container's `tests/` directory         |
| `test_blob` | Git blob SHA of test file content when it last passed       |
| `pass_time` | ISO 8601 timestamp of when that test last passed            |

### State derivation

| Condition                              | State       | Meaning                |
| -------------------------------------- | ----------- | ---------------------- |
| No `tests/` directory                  | Spec only   | No tests written yet   |
| `tests/` exists, no `pass.csv`         | Blocked     | Tests exist, none pass |
| `pass.csv` has 2 of 5 tests            | In progress | 2 pass, 3 don't        |
| `pass.csv` lists all tests in `tests/` | Validated   | All tests pass         |

### Failure classification

When a test in `pass.csv` fails:

| `spec_blob` | `test_blob`       | Diagnosis                                                     |
| ----------- | ----------------- | ------------------------------------------------------------- |
| Unchanged   | Unchanged         | **Regression** - implementation, dependency, or harness broke |
| Unchanged   | Changed           | **Stale test** - test was modified, needs re-stamp            |
| Changed     | Unchanged         | **Stale spec** - spec was modified, needs re-stamp            |
| Changed     | Changed           | **Stale both** - spec and test modified, needs re-stamp       |
| N/A         | Not in `pass.csv` | **In progress** - never passed, not a regression              |

### Rationale

- **Blob-based comparison** is deterministic across rebases, checkouts, touched files
- **Per-test tracking** enables precise diagnosis: "this test passed 3 days ago with blob X"
- **Incomplete ledgers are valid** - partial progress is normal, not an error state

### Generating pass.csv

`pass.csv` MUST be generated by `spx test --stamp`, never hand-edited.

```bash
spx test --stamp <container>
```

The command:

1. Runs all tests in `tests/`
2. Writes `pass.csv` header with current `spec_blob` and `run` timestamp
3. Writes one row per passing test with its `test_blob` and `pass_time`
4. May be incomplete (only passing tests are listed)

---

## Precommit Validation

Precommit is the primary feedback loop. CI is insurance.

For each container that has `tests/`:

### 1. Phantom check

Every `test_file` in `pass.csv` must exist at runtime path `<container>/tests/<test_file>`.

- Missing file → **error** (phantom entry)

### 2. Regression check

Run exactly the tests listed in `pass.csv` (derive paths as above).

If a listed test fails:

- Compute current blob of that test file
- If unchanged from `test_blob` → **Regression** (error, blocks commit)
- If changed → **Stale** (re-stamp required)

### 3. Spec staleness check

Compute `spec_blob_now` from current spec file.

- If `spec_blob_now != spec_blob` in header → **Stale** (re-stamp required)

### 4. Progress tests rule

Tests in `tests/` but not in `pass.csv` are **in progress** (not an error).

### What this catches

| Scenario                                    | Result                     |
| ------------------------------------------- | -------------------------- |
| Test in `pass.csv` fails, nothing changed   | Regression - blocks commit |
| Test in `pass.csv` fails, test file changed | Stale - re-stamp required  |
| Spec changed since last stamp               | Stale - re-stamp required  |
| Test file deleted but still in `pass.csv`   | Phantom - blocks commit    |
| New test file not yet in `pass.csv`         | In progress - OK           |

### Flow

1. **Write spec**: define intent, constraints, and test strategy
2. **Implement**: write code and tests
3. **Stamp**: run `spx test --stamp <container>` to generate `pass.csv`
4. **Commit**: spec + implementation + tests + pass.csv together
5. **Precommit**: validates no regressions, no phantoms, no staleness
6. **CI**: re-runs validation as insurance

---

## Test Infrastructure

Test harnesses are production code requiring their own specifications and test coverage.

### Location

```
spx/
  capability-13_test-infrastructure/
    test-infrastructure.capability.md
    feature-10_cli-harness/
      cli-harness.feature.md
      pass.csv
      tests/
        setup.unit.test.ts
        isolation.integration.test.ts
    feature-20_e2e-harness/
      e2e-harness.feature.md
      pass.csv
      tests/
```

### Why harnesses need specs

1. **Harness bugs break all dependent tests** - high impact requires tracking
2. **Harness refactoring needs regression protection** - `pass.csv` detects breakage
3. **Harness capabilities need documentation** - what can tests rely on?
4. **Harness dependencies need BSP ordering** - build harness before tests that use it

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
  adr-21_testing-strategy.md
  capability-13_test-infrastructure/
    test-infrastructure.capability.md
    feature-10_cli-harness/
      cli-harness.feature.md
      pass.csv
      tests/
        setup.unit.test.ts
  capability-15_validation/
    validation.capability.md
    pass.csv
    tests/
      validation.e2e.test.ts
    adr-21_isolated-test-fixtures.md
    feature-21_testable-validation/
      testable-validation.feature.md
      pass.csv
      tests/
        validation.integration.test.ts
      adr-04_level-2-integration-only.md
      story-47_validation-commands/
        validation-commands.story.md
        pass.csv
        tests/
          commands.unit.test.ts
```
