# Feature: Python Plugin Migration

## Observable Outcome

Python plugin skills work with CODE structure: reference `spx/` paths, use co-located tests with suffix naming, understand `status.yaml`.

## Skills to Update

| Skill                | Changes Required                                                       |
| -------------------- | ---------------------------------------------------------------------- |
| coding-python        | Update spec references to `spx/`, test output to co-located `tests/`   |
| reviewing-python     | Update spec references to `spx/`, review co-located tests              |
| testing-python       | Update test paths from `tests/unit/` to `spx/.../tests/*.unit.test.py` |
| architecting-python  | Update ADR output to `spx/` hierarchy                                  |
| orchestrating-python | Update workflow to use `spx/` paths, `status.yaml` for progress        |

## Key Changes

### Test Location Changes

**Before (graduation model):**

```
specs/work/doing/capability-10/feature-10/story-10/tests/  → tests/unit/
```

**After (co-location model):**

```
spx/capability-10/feature-10/story-10/tests/*.unit.test.py  (stays here)
```

### Test Naming Changes

**Before:**

```
tests/unit/story-10/test_parsing.py
tests/integration/feature-10/test_validation.py
```

**After:**

```
spx/.../story-10/tests/parsing.unit.test.py
spx/.../feature-10/tests/validation.integration.test.py
```

### Status Changes

**Before:** Check for `DONE.md` or directory location (`doing/` vs `done/`)

**After:** Read `status.yaml` - `failed: []` means outcome achieved

## Tests

- [Integration: coding-python writes to spx/.../tests/](tests/coding-python-tests.integration.test.ts)
- [Integration: testing-python uses suffix naming](tests/testing-python-naming.integration.test.ts)
- [Integration: No test graduation references](tests/no-graduation.integration.test.ts)

## Completion Criteria

- [ ] All skills reference `spx/` not `specs/work/`
- [ ] Test output uses co-located `tests/` with `*.unit.test.py` naming
- [ ] No references to test graduation
- [ ] `status.yaml` used for work state determination
- [ ] No TRD references
