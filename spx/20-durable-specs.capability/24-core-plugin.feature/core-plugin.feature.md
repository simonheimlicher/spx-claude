# Feature: Core Plugin Migration

## Observable Outcome

Core plugin commands (handoff, pickup) work with CODE structure if they reference specs.

## Commands to Evaluate

| Command | Changes Required                          |
| ------- | ----------------------------------------- |
| handoff | Evaluate if spec references need updating |
| pickup  | Evaluate if spec references need updating |
| commit  | No spec references - no changes           |

## Key Changes

### Session Directory

Session handoffs currently use `.spx/sessions/` which is independent of the spec structure. This remains unchanged.

### Spec References in Handoffs

If handoff documents reference work items:

**Before:**

```markdown
Working on: specs/work/doing/capability-10/feature-10/story-10/
```

**After:**

```markdown
Working on: spx/capability-10/feature-10/story-10/
Status: See outcomes.yaml (failed: 2 tests remaining)
```

### Work Item Discovery

If pickup needs to find next work item:

**Before:** Scan `specs/work/backlog/` for OPEN items

**After:** Scan `spx/` for containers where `outcomes.yaml` is missing or has non-empty `failed:` list

## Tests

- [Integration: Handoff references spx/ paths](tests/handoff-spx-paths.integration.test.ts)
- [Integration: Pickup discovers work from outcomes.yaml](tests/pickup-status-discovery.integration.test.ts)

## Completion Criteria

- [ ] Handoff documents use `spx/` paths
- [ ] Pickup understands `outcomes.yaml` for work state
- [ ] No references to `specs/work/backlog|doing|done/`
- [ ] Session directory (`.spx/sessions/`) unchanged
