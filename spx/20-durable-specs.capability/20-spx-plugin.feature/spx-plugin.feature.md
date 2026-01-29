# Feature: SPX Plugin Migration

## Observable Outcome

The spx plugin skills work with the CODE structure: `spx/` root, `pass.csv` for state, no TRDs, co-located tests.

## Skills to Update

| Skill               | Changes Required                                                                                               |
| ------------------- | -------------------------------------------------------------------------------------------------------------- |
| managing-specs      | Update paths from `specs/work/` to `spx/`, remove TRD templates, add `pass.csv` creation, update BSP discovery |
| understanding-specs | Update hierarchy traversal to `spx/`, remove TRD verification, add `pass.csv` reading                          |
| writing-prd         | Update output paths to `spx/`, no other changes                                                                |
| writing-trd         | **DELETE** - TRDs no longer exist in CODE                                                                      |

## Key Changes

### managing-specs

1. **Path changes**: All references to `specs/work/backlog|doing|done/` become flat `spx/` paths
2. **Status tracking**: Replace `DONE.md` with `pass.csv` creation/updates
3. **Templates**: Remove TRD template, update capability/feature/story templates
4. **BSP discovery**: Update to find work items in flat `spx/` structure

### understanding-specs

1. **Hierarchy loading**: Update to traverse `spx/capability-NN/feature-NN/story-NN/`
2. **Document verification**: Remove TRD from required documents list
3. **Status reading**: Parse `pass.csv` to determine work item state

### writing-prd

1. **Output path**: Write to `spx/` not `specs/work/doing/`
2. **Template**: No TRD reference in "next steps"

### writing-trd

1. **Delete entirely** - TRDs dropped from CODE model

## Tests

- [Integration: managing-specs creates spx/ structure](tests/managing-specs-spx.integration.test.ts)
- [Integration: understanding-specs traverses spx/](tests/understanding-specs-spx.integration.test.ts)
- [Integration: No TRD references in any skill](tests/no-trd-refs.integration.test.ts)

## Completion Criteria

- [ ] managing-specs works with `spx/` structure
- [ ] understanding-specs traverses `spx/` hierarchy
- [ ] writing-prd outputs to `spx/`
- [ ] writing-trd skill deleted
- [ ] No TRD references in any spx plugin skill
- [ ] `pass.csv` documented and used for state
