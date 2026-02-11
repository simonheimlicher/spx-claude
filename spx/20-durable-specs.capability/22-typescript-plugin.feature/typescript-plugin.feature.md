# Feature: TypeScript Plugin Migration

## Observable Outcome

TypeScript plugin skills work with Outcome Engineering structure: reference `spx/` paths, use co-located tests with suffix naming, understand `outcomes.yaml`.

## Skills to Update

| Skill                    | Changes Required                                                       |
| ------------------------ | ---------------------------------------------------------------------- |
| coding-typescript        | Update spec references to `spx/`, test output to co-located `tests/`   |
| reviewing-typescript     | Update spec references to `spx/`, review co-located tests              |
| testing-typescript       | Update test paths from `tests/unit/` to `spx/.../tests/*.unit.test.ts` |
| architecting-typescript  | Update ADR output to `spx/` hierarchy                                  |
| orchestrating-typescript | Update workflow to use `spx/` paths, `outcomes.yaml` for progress      |

## Key Changes

### Test Location Changes

**Before (graduation model):**

```
specs/work/doing/capability-10/feature-10/story-10/tests/  → tests/unit/
```

**After (co-location model):**

```
spx/capability-10/feature-10/story-10/tests/*.unit.test.ts  (stays here)
```

### Test Naming Changes

**Before:**

```
tests/unit/story-10/parsing.test.ts
tests/integration/feature-10/validation.test.ts
```

**After:**

```
spx/.../story-10/tests/parsing.unit.test.ts
spx/.../feature-10/tests/validation.integration.test.ts
```

### Vitest Configuration

**Before:**

```typescript
export default {
  include: ["tests/**/*.test.ts"],
};
```

**After:**

```typescript
export default {
  include: ["spx/**/tests/**/*.test.ts"],
};
```

## Tests

- [Integration: coding-typescript writes to spx/.../tests/](tests/coding-typescript-tests.integration.test.ts)
- [Integration: testing-typescript uses suffix naming](tests/testing-typescript-naming.integration.test.ts)
- [Integration: No test graduation references](tests/no-graduation.integration.test.ts)

## Completion Criteria

- [ ] All skills reference `spx/` not `specs/work/`
- [ ] Test output uses co-located `tests/` with `*.unit.test.ts` naming
- [ ] No references to test graduation
- [ ] `outcomes.yaml` used for work state determination
- [ ] No TRD references
- [ ] vitest.config.ts example updated
