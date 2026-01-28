# Verification Protocol (Phases 6-7)

> **Write access is earned by passing review.** This phase only runs on APPROVED.

## Phase 6: Stamp pass.csv

When all checks pass, record test verification in the work item's `pass.csv` ledger.

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

### 6.3 Update pass.csv

Record each passing test in the work item's `pass.csv`:

```bash
# Get blob SHA for test file
git hash-object spx/.../tests/feature.unit.test.ts

# Append to pass.csv
echo "$(date -u +%Y-%m-%dT%H:%M:%SZ),feature.unit.test.ts,{blob_sha},PASS" >> spx/.../pass.csv
```

Format: `timestamp,test_file,blob_sha,result`

## Phase 7: Verification Summary

Create verification summary in the work item's test output.

### 7.1 Verify pass.csv

Ensure pass.csv contains entries for all required tests:

```csv
timestamp,test_file,blob_sha,result
2024-01-15T10:30:00Z,feature.unit.test.ts,a1b2c3d,PASS
2024-01-15T10:30:05Z,edge-cases.unit.test.ts,e4f5g6h,PASS
```

### 7.2 Final Output

After updating pass.csv, report completion:

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

### Tests Verified (pass.csv)

| Requirement | Test File                 | Blob SHA |
| ----------- | ------------------------- | -------- |
| {FR1}       | `feature.unit.test.ts`    | a1b2c3d  |
| {FR2}       | `edge-cases.unit.test.ts` | e4f5g6h  |

### Work Item Status

This work item is now DONE.
```

**Continue to**: `workflows/commit-protocol.md`
