# Test Record

The test record tracks validation state for the Spec Tree. Each node MAY have a `test-record.yaml` file that records which tests pass.

## Format

```yaml
tests:
  - file: login.unit.test.ts
    passed_at: 2026-01-28T14:15:00Z
  - file: login.integration.test.ts
    passed_at: 2026-01-28T14:15:00Z
descendants:
  - path: 10-parse-credentials.story/
    record_blob: a3f2b7c
  - path: 22-validate-token.story/
    record_blob: 9bc4e1d
```

| Field                       | Description                               |
| --------------------------- | ----------------------------------------- |
| `tests[].file`              | Test filename relative to node's `tests/` |
| `tests[].passed_at`         | ISO 8601 timestamp when test passed       |
| `descendants[].path`        | Child node directory name                 |
| `descendants[].record_blob` | Git blob SHA of child's test-record.yaml  |

## Node States

| State         | Condition                       | Required Action     |
| ------------- | ------------------------------- | ------------------- |
| **Unknown**   | No tests exist                  | Write tests         |
| **Pending**   | Tests exist, not all recorded   | Fix code or record  |
| **Stale**     | Descendant record_blob mismatch | Re-record           |
| **Passing**   | All tests pass, blobs match     | None                |
| **Regressed** | Recorded test fails             | Investigate and fix |

States are mutually exclusive. Every node is in exactly one state.

## Tree Coupling

Parent test-record.yaml stores `record_blob` for each child. When a child's test-record.yaml changes:

1. Child's Git blob changes
2. Parent's stored `record_blob` no longer matches
3. Parent becomes **Stale**
4. Parent must re-record to update references

This creates a Merkle tree of validation state, separate from Git's content Merkle tree.

## Commands

### `spx test [node [--tree] | --all]`

Run tests without changing anything.

```bash
spx test spx/21-auth.capability/22-login.feature/
# Run login's tests only, show results

spx test spx/21-auth.capability/ --tree
# Run capability + all descendants

spx test --all
# Run all tests in spx/
```

### `spx record [node [--tree] | --all]`

Record passing tests in test-record.yaml.

```bash
spx record spx/21-auth.capability/22-login.feature/10-parse.story/
# 1. Run story's tests
# 2. All pass? Update test-record.yaml
# 3. Any fail? Error, cannot record

spx record spx/21-auth.capability/ --tree
# Record all descendants bottom-up, then record capability
```

**Recording requires bottom-up order.** If a child changed, parent cannot record until child is re-recorded.

### `spx check [node | --all]`

Check that recorded results hold by running recorded tests.

```bash
spx check --all
# Run all tests listed in test-record.yaml files
# Report: Passing or Regressed
```

### `spx status [node | --all]`

Show current states without running tests.

```bash
spx status --all
# Show tree with states: Unknown, Pending, Stale, Recorded
```

Note: `status` shows "Recorded" for nodes with test-record.yaml. Use `check` to confirm Passing vs Regressed.

## State Computation

### Leaf Nodes (Stories)

Story state IS its aggregate state (no descendants).

### Non-Leaf Nodes (Features, Capabilities)

Aggregate state = worst of (local state, descendant states).

**State ordering (worst to best):**

```text
Regressed > Stale > Pending > Unknown > Passing
```

### Examples

```text
Capability: Stale (child changed)
  Feature-1: Passing
    Story-1: Passing
  Feature-2: Stale (child changed)
    Story-2: Passing
    Story-3: Passing ← just re-recorded, blob changed
```

```text
Capability: Regressed (worst of all)
  Feature-1: Passing
  Feature-2: Regressed
    Story-2: Regressed ← recorded test now fails
```

## Workflow

```bash
# 1. Development: run tests, iterate
spx test spx/path/to/node/

# 2. Ready: record passing tests
spx record spx/path/to/node/ --tree

# 3. Commit: save the record
git add spx/ && git commit -m "feat: implement X"

# 4. CI: check records hold
spx check --all
```

## Test Reduction

Only tests in test-record.yaml are run during `check`. Nodes without test-record.yaml (Unknown/Pending) are not checked — there's nothing recorded to check.

This provides test reduction: `check` runs recorded tests, not all possible tests.
