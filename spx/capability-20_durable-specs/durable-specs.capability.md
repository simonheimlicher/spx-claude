# Capability: Durable Specs Migration

## Purpose

Migrate all plugins from the current `specs/work/backlog|doing|done/` structure to the CODE (Customer Outcome Driven Engineering) model where specs are a durable map that never moves.

## Success Metric

- **Baseline**: Plugins reference `specs/work/` paths, use test graduation, lack `status.yaml` awareness
- **Target**: All plugins reference `spx/` paths, use co-located tests, understand `status.yaml`
- **Measurement**: All plugin skills/commands pass validation with new structure

## Key Changes from Current to CODE

| Aspect         | Current                           | CODE                               |
| -------------- | --------------------------------- | ---------------------------------- |
| Root directory | `specs/`                          | `spx/`                             |
| Work state     | `backlog/doing/done/` directories | `status.yaml` per container        |
| Test location  | Graduate to `tests/unit           | integration                        |
| Test naming    | Directory-based (`tests/unit/`)   | Suffix-based (`*.unit.test.ts`)    |
| TRDs           | Separate `*.trd.md` files         | Dropped (content in feature.md)    |
| Status         | `DONE.md` marker files            | `status.yaml` with pass/fail lists |

## Plugins to Update

| Plugin     | Skills/Commands Affected                                                              |
| ---------- | ------------------------------------------------------------------------------------- |
| spx        | managing-specs, understanding-specs, writing-prd (drop writing-trd)                   |
| python     | coding-python, reviewing-python, testing-python, orchestrating-python                 |
| typescript | coding-typescript, reviewing-typescript, testing-typescript, orchestrating-typescript |
| test       | testing (foundational skill)                                                          |
| core       | handoff, pickup                                                                       |

## Tests

- [E2E: All plugins work with spx/ structure](tests/spx-structure.e2e.test.ts)
- [E2E: Status.yaml correctly determines work state](tests/status-yaml.e2e.test.ts)

## Completion Criteria

- [ ] All plugin skills reference `spx/` not `specs/work/`
- [ ] Test instructions use co-located `tests/` with suffix naming
- [ ] No references to test graduation
- [ ] No references to TRDs
- [ ] `status.yaml` documented as status mechanism
- [ ] All features complete
