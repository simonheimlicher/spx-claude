---
name: migrating-spec-to-spx
description: Domain knowledge for migrating capabilities from legacy specs/ to CODE spx/ structure. Use when migrating or reviewing migrations.
---

<context>

This is a **reference skill** providing domain knowledge for migration operations. It does NOT execute migrations - it provides the knowledge for agents that do.

**Users should invoke**:

- `spx:spec-to-spx-migrator` agent - to execute a migration
- `spx:spec-to-spx-reviewer` agent - to verify a migration

**Agents using this skill should**:

- Read it fully before starting work
- Consult it during execution when uncertain
- Use it as the authoritative source for naming, coverage, and verification rules

</context>

<objective>

Provide the domain knowledge needed to migrate capabilities from the legacy `specs/work/` structure to the CODE framework `spx/` structure, including naming conventions, test reverse-graduation, and coverage verification.

This skill is referenced by:

- `spec-to-spx-migrator` agent - executes migrations
- `spec-to-spx-reviewer` agent - verifies migrations

</objective>

<quick_start>

## For Agents Using This Skill

**If you're migrating**, read these sections in order:

1. `<success_criteria>` - Know what success looks like FIRST
2. `<verification_gates>` - Understand where to STOP and check
3. `<failure_modes>` - Learn from past mistakes
4. `<two_systems>` - Understand legacy vs target structure
5. Consult other sections as needed during migration

**If you're reviewing a migration**, read `<success_criteria>` and `<verification_gates>`, then verify each criterion.

</quick_start>

<success_criteria>

## How to Recognize a Successful Migration

**Read this FIRST.** A migration is successful when you can answer YES to all of these:

### Per-Story Success

| Criterion                       | How to Verify                                                   |
| ------------------------------- | --------------------------------------------------------------- |
| DONE.md exists in worktree      | `cat $WORKTREE_PATH/specs/work/done/.../story-NN/tests/DONE.md` |
| SPX tests exist                 | `ls spx/.../NN-slug.story/tests/*.test.ts`                      |
| SPX tests match DONE.md entries | Count tests in DONE.md table vs SPX test file                   |

### Per-Feature Success

| Criterion                            | How to Verify                                   |
| ------------------------------------ | ----------------------------------------------- |
| ALL stories in feature migrated      | Every story with DONE.md has SPX tests          |
| Coverage parity at legacy file level | See concrete example below                      |
| SPX-MIGRATION.md exists              | `cat spx/.../NN-slug.feature/SPX-MIGRATION.md`  |
| Legacy tests removed                 | `git status` shows deletions, not modifications |

### Per-Capability Success

| Criterion             | How to Verify                                            |
| --------------------- | -------------------------------------------------------- |
| ALL features migrated | Every feature passes per-feature criteria                |
| Legacy specs removed  | `ls specs/work/done/capability-NN_slug/` returns nothing |
| Tests pass            | `pnpm test` shows 0 failures                             |
| Validation passes     | `pnpm run validate` succeeds                             |

### Concrete Coverage Verification Example

For 43-status-determination.feature, success looked like:

```text
Legacy tests:
  tests/unit/status/state.test.ts (5 tests)
  tests/integration/status/state.integration.test.ts (19 tests)
  Total: 24 tests
  Coverage on src/status/state.ts: 86.3%

SPX tests:
  spx/.../21-initial-state.story/tests/state.unit.test.ts (5 tests)
  spx/.../32-state-transitions.story/tests/state.integration.test.ts (7 tests)
  spx/.../43-concurrent-access.story/tests/state.integration.test.ts (4 tests)
  spx/.../54-status-edge-cases.story/tests/state.integration.test.ts (8 tests)
  Total: 24 tests
  Coverage on src/status/state.ts: 86.3%

Verdict: ✓ Test count matches, coverage matches, migration successful
```

**If coverage differs by more than 0.5%, STOP.** Find which tests are missing.

</success_criteria>

<verification_gates>

## Verification Gates (MUST STOP and Check)

**Do NOT proceed past a gate until it passes.**

### Gate 1: Before Writing Any SPX Tests

- [ ] Worktree exists at `$WORKTREE_PATH`
- [ ] `spx/` does NOT exist in worktree (confirms correct commit)
- [ ] Can read at least one DONE.md from worktree

```bash
# Verify gate 1
[ -d "$WORKTREE_PATH" ] && [ ! -d "$WORKTREE_PATH/spx" ] && echo "GATE 1: PASS"
```

### Gate 2: Before Removing Any Legacy Tests

- [ ] ALL stories in the feature have SPX tests
- [ ] Legacy test file sharing map is complete
- [ ] Coverage comparison run shows parity (±0.5%)

```bash
# Verify gate 2 - coverage comparison
pnpm vitest run tests/unit/status/state.test.ts tests/integration/status/state.integration.test.ts --coverage 2>&1 | grep "state.ts"
pnpm vitest run spx/.../43-status-determination.feature --coverage 2>&1 | grep "state.ts"
# Numbers must match
```

### Gate 3: Before Committing

- [ ] `pnpm test` passes (0 failures)
- [ ] `pnpm run validate` passes
- [ ] `git status` shows only expected changes (SPX additions, legacy deletions)
- [ ] No files deleted with `rm` (all deletions via `git rm`)

```bash
# Verify gate 3
pnpm test && pnpm run validate && git status
```

### Gate 4: Before Creating Handoff

- [ ] Current feature is FULLY migrated (not partial)
- [ ] SPX-MIGRATION.md documents what was done
- [ ] Commit created for completed work

**Never hand off in the middle of a feature.** Either complete the feature or abandon and reset.

</verification_gates>

<failure_modes>

## Failure Modes (Learn from Past Mistakes)

These failures occurred during actual migrations. Avoid them.

### Failure 1: Comparing Coverage at Wrong Granularity

**What happened:** Agent compared coverage per-story, saw "39.72%" for one story and panicked.

**Why it failed:** Multiple stories contribute to the same legacy test file. Story-level coverage is meaningless.

**How to avoid:** ALWAYS compare at the legacy file level. If `tests/integration/status/state.integration.test.ts` has tests from stories 32, 43, and 54, compare the COMBINED coverage of all three SPX story tests against that ONE legacy file.

### Failure 2: Not Reading DONE.md Files

**What happened:** Agent guessed which tests belonged to which story based on file names.

**Why it failed:** File names don't reliably indicate origin. Only DONE.md documents the actual mapping.

**How to avoid:** ALWAYS read DONE.md from the worktree. The "Graduated Tests" table is the ONLY source of truth.

### Failure 3: Trusting Previous Handoff Claims

**What happened:** Agent accepted handoff claim that "coverage would drop" without verifying.

**Why it failed:** The claim was based on wrong-granularity comparison (see Failure 1).

**How to avoid:** Verify ALL claims from previous handoffs. Re-run coverage comparisons yourself. Trust, but verify.

### Failure 4: Removing Legacy Files Too Early

**What happened:** Agent removed `tests/integration/cli.test.ts` after migrating story-32, but stories 43 and 54 also used that file.

**Why it failed:** Didn't build the sharing map first.

**How to avoid:** Build the legacy file → stories map BEFORE starting migration. Only remove a legacy file after ALL contributing stories are migrated.

### Failure 5: Using `rm` Instead of `git rm`

**What happened:** Files disappeared from working directory but Git still tracked them. Caused confusion on next commit.

**Why it failed:** `rm` doesn't update Git index.

**How to avoid:** ALWAYS use `git rm`. This removes the file AND stages the deletion.

### Failure 6: Partial Feature Handoff

**What happened:** Agent migrated 2 of 4 stories, then handed off. Next agent didn't know which stories were done.

**Why it failed:** Partial state is hard to communicate. SPX-MIGRATION.md wasn't updated mid-feature.

**How to avoid:** Complete entire features before handoff. If you must stop mid-feature, update SPX-MIGRATION.md with explicit "Stories migrated: X, Y. Stories remaining: Z" section.

</failure_modes>

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
