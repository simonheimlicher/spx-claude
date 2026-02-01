# Verification Protocol (Phases 6-7)

> **Write access is earned by passing review.** This phase only runs on APPROVED.

## Phase 6: Commit Outcome

When all checks pass, record test verification in the work item's `outcomes.yaml` ledger.

### 6.1 Identify Test Files

Locate tests in the work item directory:

```bash
find spx/ -path "*/{work-item}/tests/*.test.ts" -name "*.test.ts"
```

Test files are already co-located with their specs:

| Test Level  | Location                           | Filename Suffix         |
| ----------- | ---------------------------------- | ----------------------- |
| Unit        | `spx/.../NN-{slug}.story/tests/`   | `*.unit.test.ts`        |
| Integration | `spx/.../NN-{slug}.feature/tests/` | `*.integration.test.ts` |
| E2E         | `spx/NN-{slug}.capability/tests/`  | `*.e2e.test.ts`         |

### 6.2 Verify Tests Pass

Run all tests for the work item:

```bash
npx vitest run spx/.../NN-{slug}.story/tests/
```

**If tests fail**: The verdict becomes REJECTED with reason "Tests don't pass."

### 6.3 Commit Outcomes

Commit the test outcomes to the work item's outcome ledger:

```bash
spx spx commit spx/.../NN-{slug}.story
```

This generates `outcomes.yaml` with all passing tests.

## Phase 7: Verification Summary

Create verification summary in the work item's test output.

### 7.1 Verify outcomes.yaml

Ensure outcomes.yaml contains entries for all required tests:

```yaml
spec_blob: a1b2c3d...
committed_at: 2024-01-15T10:30:00Z
tests:
  - file: feature.unit.test.ts
    blob: a1b2c3d
    passed_at: 2024-01-15T10:30:00Z
  - file: edge-cases.unit.test.ts
    blob: e4f5g6h
    passed_at: 2024-01-15T10:30:05Z
```

### 7.2 Final Output

After updating outcomes.yaml, report completion:

```markdown
## Review Complete: {work-item}

### Verdict: APPROVED

### Verification

| Tool    | Status | Details                      |
| ------- | ------ | ---------------------------- |
| tsc     | PASS   | 0 errors                     |
| eslint  | PASS   | 0 violations                 |
| Semgrep | PASS   | 0 findings                   |
| vitest  | PASS   | {X}/{X} tests, {Y}% coverage |

### Tests Verified (outcomes.yaml)

| Requirement | Test File                 | Blob SHA |
| ----------- | ------------------------- | -------- |
| {FR1}       | `feature.unit.test.ts`    | a1b2c3d  |
| {FR2}       | `edge-cases.unit.test.ts` | e4f5g6h  |

### Work Item Status

This work item is now DONE.
```

**Continue to**: `workflows/commit-protocol.md`
