# Feature: Python Plugin Migration

## Observable Outcome

Python plugin skills work with Outcome Engineering structure: reference `spx/` paths, use co-located tests with suffix naming, understand `outcomes.yaml`.

## Skills to Update

| Skill                | Changes Required                                                       |
| -------------------- | ---------------------------------------------------------------------- |
| coding-python        | Update spec references to `spx/`, test output to co-located `tests/`   |
| reviewing-python     | Update spec references to `spx/`, review co-located tests              |
| testing-python       | Update test paths from `tests/unit/` to `spx/.../tests/test_*.unit.py` |
| architecting-python  | Update ADR output to `spx/` hierarchy                                  |
| orchestrating-python | Update workflow to use `spx/` paths, `outcomes.yaml` for progress      |

## Key Changes

### Test Location Changes

**Before (graduation model):**

```
specs/work/doing/capability-10/feature-10/story-10/tests/  → tests/unit/
```

**After (co-location model):**

```
spx/capability-10/feature-10/story-10/tests/test_*.unit.py  (stays here)
```

### Test Naming Changes

**Before:**

```
tests/unit/story-10/test_parsing.py
tests/integration/feature-10/test_validation.py
```

**After:**

```
spx/.../story-10/tests/test_parsing.unit.py
spx/.../feature-10/tests/test_validation.integration.py
```

### Status Changes

**Before:** Check for `DONE.md` or directory location (`doing/` vs `done/`)

**After:** Read `outcomes.yaml` - `failed: []` means outcome achieved

## Tests

- [Integration: coding-python writes to spx/.../tests/](tests/coding-python-tests.integration.test.ts)
- [Integration: testing-python uses suffix naming](tests/testing-python-naming.integration.test.ts)
- [Integration: No test graduation references](tests/no-graduation.integration.test.ts)

## Completion Criteria

- [ ] All skills reference `spx/` not `specs/work/`
- [ ] Test output uses co-located `tests/` with `test_*.unit.py` naming
- [ ] No references to test graduation
- [ ] `outcomes.yaml` used for work state determination
- [ ] No TRD references
