# Outcome Ledger

The outcome ledger records verification state for the Product Tree. Each container MAY have an `outcomes.yaml` file that claims its tests pass.

## Format

```yaml
tests:
  - file: login.unit.test.ts
    passed_at: 2026-01-28T14:15:00Z
  - file: login.integration.test.ts
    passed_at: 2026-01-28T14:15:00Z
descendants:
  - path: 10-parse-credentials.story/
    outcomes_blob: a3f2b7c
  - path: 22-validate-token.story/
    outcomes_blob: 9bc4e1d
```

| Field                         | Description                                    |
| ----------------------------- | ---------------------------------------------- |
| `tests[].file`                | Test filename relative to container's `tests/` |
| `tests[].passed_at`           | ISO 8601 timestamp when test passed            |
| `descendants[].path`          | Child container directory name                 |
| `descendants[].outcomes_blob` | Git blob SHA of child's outcomes.yaml          |

## Container States

| State         | Condition                         | Required Action     |
| ------------- | --------------------------------- | ------------------- |
| **Unknown**   | No tests exist                    | Write tests         |
| **Pending**   | Tests exist, not all claimed      | Fix code or claim   |
| **Stale**     | Descendant outcomes_blob mismatch | Re-claim            |
| **Passing**   | All tests pass, blobs match       | None                |
| **Regressed** | Claimed test fails                | Investigate and fix |

States are mutually exclusive. Every container is in exactly one state.

## Tree Coupling

Parent outcomes.yaml stores `outcomes_blob` for each child. When a child's outcomes.yaml changes:

1. Child's Git blob changes
2. Parent's stored `outcomes_blob` no longer matches
3. Parent becomes **Stale**
4. Parent must re-claim to update references

This creates a Merkle tree of verification state, separate from Git's content Merkle tree.

## Commands

### `spx spx test [container [--tree] | --all]`

Run tests without changing anything.

```bash
spx spx test spx/21-auth.capability/22-login.feature/
# Run login's tests only, show results

spx spx test spx/21-auth.capability/ --tree
# Run capability + all descendants

spx spx test --all
# Run all tests in spx/
```

### `spx spx claim [container [--tree] | --all]`

Assert tests pass and update outcomes.yaml.

```bash
spx spx claim spx/21-auth.capability/22-login.feature/10-parse.story/
# 1. Run story's tests
# 2. All pass? Update outcomes.yaml
# 3. Any fail? Error, cannot claim

spx spx claim spx/21-auth.capability/ --tree
# Claim all descendants bottom-up, then claim capability
```

**Claim requires bottom-up order.** If a child changed, parent cannot claim until child is re-claimed.

### `spx spx verify [container | --all]`

Check that claims hold by running claimed tests.

```bash
spx spx verify --all
# Run all tests listed in outcomes.yaml files
# Report: Passing or Regressed
```

### `spx spx status [container | --all]`

Show current states without running tests.

```bash
spx spx status --all
# Show tree with states: Unknown, Pending, Stale, Claimed
```

Note: `status` shows "Claimed" for containers with outcomes.yaml. Use `verify` to confirm Passing vs Regressed.

## State Computation

### Leaf Containers (Stories)

Story state IS its aggregate state (no descendants).

### Non-Leaf Containers (Features, Capabilities)

Aggregate state = worst of (local state, descendant states).

**State ordering (worst to best):**

```
Regressed > Stale > Pending > Unknown > Passing
```

### Examples

```
Capability: Stale (child changed)
  Feature-1: Passing
    Story-1: Passing
  Feature-2: Stale (child changed)
    Story-2: Passing
    Story-3: Passing ← just re-claimed, blob changed
```

```
Capability: Regressed (worst of all)
  Feature-1: Passing
  Feature-2: Regressed
    Story-2: Regressed ← claimed test now fails
```

## Workflow

```bash
# 1. Development: run tests, iterate
spx spx test spx/path/to/container/

# 2. Ready: claim tests pass
spx spx claim spx/path/to/container/ --tree

# 3. Commit: record the claim
git add spx/ && git commit -m "feat: implement X"

# 4. CI: verify claims hold
spx spx verify --all
```

## Verification Reduction

Only tests in outcomes.yaml are run during `verify`. Containers without outcomes.yaml (Unknown/Pending) are not verified—there's nothing claimed to check.

This provides test reduction: verify runs claimed tests, not all possible tests.
