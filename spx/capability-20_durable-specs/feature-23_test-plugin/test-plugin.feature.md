# Feature: Test Plugin Migration

## Observable Outcome

The foundational `/testing` skill documents co-located tests with suffix naming instead of test graduation.

## Skill to Update

| Skill   | Changes Required                                                                |
| ------- | ------------------------------------------------------------------------------- |
| testing | Remove test graduation, document co-located tests, add suffix naming convention |

## Key Changes

### Remove Test Graduation

**Before:**

```markdown
**Test Graduation**:

- Story tests: `specs/.../tests/` → `tests/unit/`
- Feature tests: `specs/.../tests/` → `tests/integration/`
- Capability tests: `specs/.../tests/` → `tests/e2e/`
```

**After:**

```markdown
**Test Location**:

- Story tests: `spx/.../story-NN/tests/*.unit.test.{ts,py}`
- Feature tests: `spx/.../feature-NN/tests/*.integration.test.{ts,py}`
- Capability tests: `spx/.../capability-NN/tests/*.e2e.test.{ts,py}`

Tests stay with their spec. No graduation.
```

### Test Naming Convention

**Before:** Test level determined by directory (`tests/unit/`, `tests/integration/`)

**After:** Test level determined by suffix (`*.unit.test.ts`, `*.integration.test.ts`, `*.e2e.test.ts`)

### Status Tracking

**Before:** "Tests must pass before graduation"

**After:** "Test results recorded in `status.yaml`. Empty `failed:` list = outcome achieved"

## Tests

- [Integration: No graduation references](tests/no-graduation.integration.test.ts)
- [Integration: Documents suffix naming](tests/suffix-naming.integration.test.ts)
- [Integration: Documents status.yaml](tests/status-yaml.integration.test.ts)

## Completion Criteria

- [ ] No test graduation references
- [ ] Documents co-located tests in `spx/.../tests/`
- [ ] Documents suffix naming convention
- [ ] Documents `status.yaml` for test results
- [ ] Language-specific testing skills updated to reference this
