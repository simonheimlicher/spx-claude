# Graduation Protocol (Phases 6-7)

> **Write access is earned by passing review.** This phase only runs on APPROVED.

## Phase 6: Graduation

When all checks pass, graduate tests from the work item to the production test suite.

### 6.1 Identify Graduation Targets

Locate tests in the work item directory:

```bash
find specs/ -path "*/tests/*.test.ts" -name "*.test.ts"
```

Map test types to destinations:

| Test Type         | Source Pattern               | Destination         |
| ----------------- | ---------------------------- | ------------------- |
| Unit tests        | `*.test.ts` (no fixtures)    | `test/unit/`        |
| Integration tests | Uses binary/service fixtures | `test/integration/` |
| E2E tests         | Full system tests            | `test/e2e/`         |

### 6.2 Move Tests

```bash
# Example: Graduate story tests to production suite
cp specs/doing/.../story-XX/tests/feature.test.ts test/unit/feature.test.ts

# Update imports if needed (relative paths may change)
```

**Important**: If import paths need updating, edit the moved files to fix them.

### 6.3 Verify Graduated Tests Pass

Run the graduated tests in their new location:

```bash
npx vitest run test/unit/feature.test.ts
```

**If graduated tests fail**: The verdict becomes REJECTED with reason "Graduation failed - tests don't pass in new location."

### 6.4 Clean Up (Optional)

After successful graduation, the original tests in `specs/.../tests/` can be removed or left as documentation.

## Phase 7: Create DONE.md

Create completion evidence in the work item directory.

### 7.1 Write DONE.md

Create `specs/doing/.../story-XX/tests/DONE.md`:

```markdown
# Completion Evidence: {story-name}

## Review Summary

**Verdict**: APPROVED
**Date**: {YYYY-MM-DD}
**Reviewer**: reviewing-typescript

## Verification Results

| Tool    | Status | Details                      |
| ------- | ------ | ---------------------------- |
| tsc     | PASS   | 0 errors                     |
| eslint  | PASS   | 0 violations                 |
| Semgrep | PASS   | 0 findings                   |
| vitest  | PASS   | {X}/{X} tests, {Y}% coverage |

## Graduated Tests

| Requirement | Test Location                             |
| ----------- | ----------------------------------------- |
| {FR1}       | `test/unit/xxx.test.ts::test_name`        |
| {FR2}       | `test/integration/yyy.test.ts::test_name` |

## Verification Command

\`\`\`bash
npx vitest run --coverage
\`\`\`
```

### 7.2 Final Output

After creating DONE.md, report completion:

```markdown
## Review Complete: {work-item}

### Verdict: APPROVED

### Graduation

| Action          | Details                                  |
| --------------- | ---------------------------------------- |
| Tests graduated | {list of test files moved}               |
| DONE.md created | `specs/doing/.../story-XX/tests/DONE.md` |

### Verification

All graduated tests pass in their new location.

### Work Item Status

This work item is now DONE.
```

**Continue to**: `workflows/commit-protocol.md`
