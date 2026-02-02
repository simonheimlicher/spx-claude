---
name: migrating-spec-to-spx
description: Domain knowledge for migrating capabilities from legacy specs/ to CODE spx/ structure. Use when migrating or reviewing migrations.
---

<objective>

Provide the domain knowledge needed to migrate capabilities from the legacy `specs/work/` structure to the CODE framework `spx/` structure, including naming conventions, test reverse-graduation, and coverage verification.

This skill is referenced by:

- `spec-to-spx-migrator` agent - executes migrations
- `spec-to-spx-reviewer` agent - verifies migrations

</objective>

<two_systems>

## Legacy System: specs/

```text
specs/
  work/
    backlog/           # Not started
    doing/             # In progress
    done/              # Completed
      capability-NN_slug/
        slug.capability.md
        slug.prd.md           # Optional PRD
        feature-NN_slug/
          slug.feature.md
          slug.trd.md         # Optional TRD
          story-NN_slug/
            slug.story.md
            tests/
              DONE.md         # KEY: Documents graduated tests
decisions/
  adr-NN_slug.md              # Separate ADR directory
tests/
  unit/                       # Graduated tests live HERE
  integration/
  e2e/
```

**Key characteristics:**

- **Work items move** between backlog/doing/done directories
- **Tests graduate** from `specs/.../tests/` to `tests/{level}/`
- **DONE.md** documents which tests graduated and where
- **Status** = which directory the work item is in
- **Naming**: `{type}-{BSP}_{slug}/` (e.g., `capability-27_spec-domain/`)

## Target System: spx/ (CODE Framework)

```text
spx/
  {product}.prd.md
  NN-{slug}.adr.md            # ADRs interleaved with containers
  NN-{slug}.capability/
    {slug}.capability.md
    outcomes.yaml             # Test verification ledger
    tests/                    # Tests STAY here
      *.unit.test.{ts,py}
      *.integration.test.{ts,py}
    NN-{slug}.adr.md
    NN-{slug}.feature/
      {slug}.feature.md
      outcomes.yaml
      tests/
      NN-{slug}.story/
        {slug}.story.md
        outcomes.yaml
        tests/
```

**Key characteristics:**

- **Specs stay in place** - Nothing moves because work is "done"
- **Tests co-located** - Tests live with specs permanently, no graduation
- **outcomes.yaml** - Machine-generated verification ledger (replaces DONE.md + directory location)
- **Status** = derived from outcomes.yaml, not directory location
- **Naming**: `{BSP}-{slug}.{type}/` (e.g., `27-spec-domain.capability/`)
- **No TRDs** - Technical details belong in feature.md itself

</two_systems>

<naming_transformation>

## Naming Convention Transformation

| Element            | Legacy (specs/)                     | CODE (spx/)                    |
| ------------------ | ----------------------------------- | ------------------------------ |
| Directory pattern  | `{type}-{BSP}_{slug}/`              | `{BSP}-{slug}.{type}/`         |
| Capability example | `capability-27_spec-domain/`        | `27-spec-domain.capability/`   |
| Feature example    | `feature-32_parsing/`               | `32-parsing.feature/`          |
| Story example      | `story-54_validate-args/`           | `54-validate-args.story/`      |
| Spec file          | `{slug}.{type}.md`                  | `{slug}.{type}.md` (unchanged) |
| ADR location       | `decisions/adr-NN_slug.md`          | `NN-{slug}.adr.md` (in tree)   |
| Status tracking    | Directory location (backlog/doing/) | outcomes.yaml verification     |
| Test location      | `tests/{level}/` (graduated)        | `spx/.../tests/` (co-located)  |

**Key transformations:**

1. **BSP comes first** in directory names (for sort order)
2. **Hyphen** (`-`) separates BSP from slug (not underscore)
3. **ADRs interleaved** in the tree, not in separate `decisions/` directory
4. **No TRDs** - technical details go in feature.md

### Parsing Legacy Names

```text
Input:  capability-27_spec-domain
        ├─────────┘ ├┘ └──────┘
        type        BSP  slug

Output: 27-spec-domain.capability
        ├┘ └──────┘    └────────┘
        BSP  slug        type
```

</naming_transformation>

<bsp_uniqueness>

## CRITICAL: BSP Numbers Are SIBLING-UNIQUE, Not Global

**BSP numbers are ONLY unique among siblings at the same level.**

```text
capability-21/feature-32/story-54  ← One story-54
capability-28/feature-32/story-54  ← DIFFERENT story-54
capability-21/feature-87/story-54  ← DIFFERENT story-54
```

**ALWAYS use FULL PATHS when referencing work items:**

| Wrong (Ambiguous) | Correct (Unambiguous)                          |
| ----------------- | ---------------------------------------------- |
| "story-54"        | "capability-21/feature-54/story-54"            |
| "feature-32"      | "capability-27_spec-domain/feature-32_parsing" |

</bsp_uniqueness>

<done_md_format>

## Understanding DONE.md Format

DONE.md documents test graduation for completed stories. Example:

```markdown
# Story Complete: validate-args

## Graduated Tests

| Requirement                  | Test File                            | Level       |
| ---------------------------- | ------------------------------------ | ----------- |
| Parses --config flag         | tests/unit/cli/parsing.test.ts       | Unit        |
| Validates config file exists | tests/integration/cli/config.test.ts | Integration |
| Full CLI workflow            | tests/e2e/cli/workflow.test.ts       | E2E         |

## Coverage

- Lines: 94%
- Branches: 87%

## Completion Date

2025-01-15
```

**Key information to extract:**

1. **Graduated Tests table** - Maps requirements to test file locations
2. **Test Level** - Determines target suffix (unit → `.unit.test.ts`)
3. **Original location** - Where to find the test in the legacy `tests/` directory

</done_md_format>

<test_file_naming>

## Test File Naming in CODE Framework

Tests use suffix naming to indicate level:

| Level       | Pattern                        | Example                        |
| ----------- | ------------------------------ | ------------------------------ |
| Unit        | `*.unit.test.{ts,py}`          | `parsing.unit.test.ts`         |
| Integration | `*.integration.test.{ts,py}`   | `cli.integration.test.ts`      |
| E2E         | `*.e2e.test.{ts,py}`           | `workflow.e2e.test.ts`         |
| E2E (PW)    | `*.e2e.spec.{ts,py}` (browser) | `login.e2e.spec.ts` (optional) |

**Transformation:**

```text
tests/unit/parsing.test.ts           → spx/.../tests/parsing.unit.test.ts
tests/integration/cli.test.ts        → spx/.../tests/cli.integration.test.ts
tests/e2e/workflow.test.ts           → spx/.../tests/workflow.e2e.test.ts
```

</test_file_naming>

<shared_test_files>

## CRITICAL: Shared Legacy Test Files

**Multiple stories often graduate tests to the SAME legacy file.** For example:

- story-32 → `tests/integration/status/state.integration.test.ts`
- story-43 → `tests/integration/status/state.integration.test.ts`
- story-54 → `tests/integration/status/state.integration.test.ts`

This means:

- **Coverage verification** must be at the **legacy file level**, not story level
- **A legacy file can only be removed after ALL stories that contributed to it are migrated**
- Individual story coverage is meaningless - only the combined coverage matters

### Building the Sharing Map

Before migrating, scan ALL DONE.md files in the feature to build:

```text
legacy_file -> [story1, story2, ...]
```

Example output:

```text
tests/unit/status/state.test.ts:
  - story-21 (5 tests)

tests/integration/status/state.integration.test.ts:
  - story-32 (7 tests)
  - story-43 (4 tests)
  - story-54 (8 tests)
```

</shared_test_files>

<worktree_requirement>

## Reference Worktree Requirement

The worktree contains the original DONE.md files - the source of truth for which tests graduated from which work item.

```bash
PROJECT_ROOT=$(git rev-parse --show-toplevel)
PROJECT_NAME=$(basename "$PROJECT_ROOT")
WORKTREE_PATH="$(dirname "$PROJECT_ROOT")/${PROJECT_NAME}_pre-spx"

# Find last commit before spx/ existed
REF_COMMIT=$(git log --oneline --diff-filter=A --all -- 'spx/' | tail -1 | cut -d' ' -f1)^

# Create if doesn't exist (idempotent)
if [ ! -d "$WORKTREE_PATH" ]; then
  git worktree add "$WORKTREE_PATH" "$REF_COMMIT"
fi

# Verify: spx/ should NOT exist in worktree
[ ! -d "$WORKTREE_PATH/spx" ] && echo "✓ Worktree valid"
```

**CRITICAL:** Always read DONE.md from `$WORKTREE_PATH`, never from main repo.

</worktree_requirement>

<coverage_verification>

## Coverage Verification Requirements

### Granularity: Legacy File Level

- **Coverage is verified at the LEGACY FILE level, not story level**
- Multiple stories may contribute to the same legacy file
- A legacy file can only be removed after ALL contributing stories are migrated
- Individual story coverage percentages are meaningless - only combined coverage matters

### Verification Process

```bash
# Run ALL legacy tests that will be removed
pnpm vitest run tests/unit/status/state.test.ts tests/integration/status/state.integration.test.ts --coverage

# Run ALL SPX tests for the feature
pnpm vitest run spx/.../NN-slug.feature --coverage

# Compare coverage on target source files - MUST MATCH
```

**STOP if coverage doesn't match.** Identify which story's tests are incomplete.

</coverage_verification>

<valid_migration_checklist>

## What Constitutes a Valid Migration

A migration is complete and valid when:

### Structure

- [ ] SPX directory exists with correct naming (`{BSP}-{slug}.{type}/`)
- [ ] All spec files copied and renamed correctly
- [ ] ADRs moved from `decisions/` to in-tree location
- [ ] Test directories created at appropriate levels

### Tests

- [ ] All DONE.md entries have corresponding SPX tests
- [ ] Test files renamed with level suffix (`.unit.test.ts`, etc.)
- [ ] Imports updated if needed (`FIXTURES_ROOT` instead of `__dirname`)
- [ ] No orphaned tests (tests without DONE.md reference)

### Coverage

- [ ] Coverage verified at legacy file level (not story level)
- [ ] SPX coverage matches legacy coverage for shared source files
- [ ] All contributing stories migrated before legacy file removal

### Cleanup

- [ ] Legacy tests removed with `git rm` (never `rm`)
- [ ] Legacy specs removed with `git rm -r`
- [ ] SPX-MIGRATION.md documents the migration

### Documentation

- [ ] SPX-MIGRATION.md at feature level shows:
  - Legacy test file → contributing stories mapping
  - Coverage comparison results
  - Files removed

</valid_migration_checklist>

<constraints>

## Invariants

### Source of Truth

- **DONE.md in worktree** is the ONLY source of truth for which tests belong to which work item
- Never guess test mappings based on filename patterns
- Never trust SPX-MIGRATION.md written by previous runs - always verify against worktree DONE.md

### Deletion Safety

- **NEVER** use `rm` or `rm -r` to delete files
- **ALWAYS** use `git rm` or `git rm -r`
- This ensures deletions are tracked and reversible

### Processing Order

- Process stories in BSP order (lowest number first)
- Within a feature: story-21 before story-32 before story-43
- Within a capability: feature-21 before feature-32

### Reentrancy

Every operation MUST be reentrant (can be interrupted and resumed):

| Step                    | If interrupted  | On restart                      |
| ----------------------- | --------------- | ------------------------------- |
| Read DONE.md            | No state change | Reads again from worktree       |
| Create SPX-MIGRATION.md | Partial file    | Overwrites with correct content |
| Copy tests              | Some copied     | Skips existing, copies rest     |
| Verify coverage         | No state change | Runs again                      |
| git rm legacy tests     | Some removed    | Skips already-removed           |
| git rm old specs        | Some removed    | Skips already-removed           |

</constraints>
