---
name: migrating-spec-to-spx
description: Use when migrating specs to spx, reviewing migrations, understanding spec-to-spx process, or debugging migration issues.
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
4. `<legacy_system>` and `<target_system>` - Understand legacy vs target structure
5. Consult other sections as needed during migration

**If you're reviewing a migration**, read `<success_criteria>` and `<verification_gates>`, then verify each criterion.

</quick_start>

<success_criteria>

## How to Recognize a Successful Migration

**Read this FIRST.** A migration is successful when you can answer YES to all of these:

### Per-Story Success

| Criterion                           | How to Verify                                                |
| ----------------------------------- | ------------------------------------------------------------ |
| SPX tests exist                     | `ls spx/.../NN-slug.story/tests/*.{unit,integration,e2e}.py` |
| SPX-MIGRATION.md exists             | `cat spx/.../NN-slug.story/SPX-MIGRATION.md`                 |
| DONE.md does NOT exist in spx/      | `! test -f spx/.../NN-slug.story/tests/DONE.md`              |
| If DONE.md in worktree: tests match | Count tests in DONE.md table vs SPX test files               |

**Note:** Stories WITHOUT DONE.md in the worktree still get migrated - they're incomplete stories. Migrate all tests found in `specs/.../tests/`. The SPX-MIGRATION.md documents what was found and migrated.

### Per-Feature Success

| Criterion                            | How to Verify                                            |
| ------------------------------------ | -------------------------------------------------------- |
| ALL stories in feature migrated      | Every story in worktree has corresponding SPX story      |
| ALL stories pass Per-Story Success   | Each story has SPX-MIGRATION.md, no DONE.md in spx/      |
| Coverage parity at legacy file level | See concrete example below                               |
| SPX-MIGRATION.md exists at feature   | `cat spx/.../NN-slug.feature/SPX-MIGRATION.md`           |
| No DONE.md anywhere in spx/ feature  | `! find spx/.../NN-slug.feature -name DONE.md \| grep .` |
| Legacy tests removed                 | `git status` shows deletions, not modifications          |

### Per-Capability Success

| Criterion                            | How to Verify                                           |
| ------------------------------------ | ------------------------------------------------------- |
| ALL features migrated                | Every feature passes Per-Feature Success criteria       |
| SPX-MIGRATION.md exists at cap level | `cat spx/NN-slug.capability/SPX-MIGRATION.md`           |
| No DONE.md anywhere in spx/ cap      | `! find spx/NN-slug.capability -name DONE.md \| grep .` |
| Legacy specs removed                 | `ls specs/work/*/capability-NN_slug/` returns nothing   |
| Tests pass                           | `just test` or `pnpm test` shows 0 failures             |
| Validation passes                    | `just check` or `pnpm run validate` succeeds            |

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

### Failure 7: Copying DONE.md to spx/ Without Renaming

**What happened:** Agent moved DONE.md to spx/ but kept the filename as DONE.md.

**Why it failed:** DONE.md is the legacy name. In spx/, the corrected record is called SPX-MIGRATION.md.

**How to avoid:**

- Use `git mv` to move AND rename in one operation:
  ```bash
  git mv specs/.../tests/DONE.md spx/.../NN-slug.story/SPX-MIGRATION.md
  ```
- Then edit SPX-MIGRATION.md to add the corrected record sections (see `<spx_migration_md>`)
- Verify with: `! find spx/ -name DONE.md | grep .`

### Failure 8: Skipping Stories Without DONE.md

**What happened:** Agent only migrated stories that had DONE.md, skipping "incomplete" stories.

**Why it failed:** DONE.md absence means the story wasn't marked complete - NOT that it should be skipped. The story's tests still exist and need to be migrated.

**How to avoid:** Migrate ALL stories in the capability, regardless of DONE.md presence. For stories without DONE.md, the SPX-MIGRATION.md documents "No completion record found - migrated all tests from specs/.../tests/".

</failure_modes>

<legacy_system>

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
              DONE.md         # MAY exist - documents completion
              *.py            # Tests MAY still be here
decisions/
  adr-NN_slug.md              # Separate ADR directory
tests/
  unit/                       # Tests MAY have graduated here (or not)
  integration/
  e2e/
```

**Key characteristics:**

- **Work items move** between backlog/doing/done directories
- **Tests MAY OR MAY NOT have graduated** - check actual file locations
- **DONE.md** MAY exist - documents completion but test locations may be inconsistent
- **Status** = which directory the work item is in (irrelevant for migration)
- **Naming**: `{type}-{BSP}_{slug}/` (e.g., `capability-27_spec-domain/`)

**⚠️ MESSY REALITY**: The legacy system evolved. Tests might be:

- Still in `specs/.../tests/` (never graduated)
- Copied to `tests/` but also still in `specs/.../tests/` (duplicated - specs/ copies may be stale)
- Moved to `tests/` with DONE.md referencing them
- Any combination of the above

**Phantom graduation**: DONE.md may claim tests graduated to `tests/unit/...` but:

- The `tests/` directory doesn't exist
- Tests are actually still in `specs/.../tests/`

```bash
# DONE.md says tests are here:
ls tests/unit/test_foo.py  # File not found!

# Tests are actually here:
ls specs/.../tests/test_foo.py  # Found!
```

**Migration approach**:

1. Read DONE.md to understand intended test locations
2. **Check if files actually exist** at the claimed location
3. If DONE.md says graduated AND tests exist in `tests/` → use those
4. If tests only exist in `specs/.../tests/` → use those (ignore DONE.md claims)
5. If tests exist in BOTH locations → `specs/.../tests/` may be stale duplicates
6. Remove from ALL original locations to avoid duplicates in final state

</legacy_system>

<target_system>

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

</target_system>

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

### ADRs Already in spx/

ADRs may have been migrated earlier with RENUMBERED BSPs. Before migrating:

1. Check if ADR content already exists in spx/ under a different number
2. If yes: **UPDATE REFERENCES** in spec files to point to existing spx/ ADR
3. If no: Migrate the ADR with appropriate BSP

```bash
# Check for existing ADRs
ls spx/*.adr.md

# Example: adr-07 was renumbered to 21
# decisions/adr-07_python-tooling.md  →  spx/21-python-tooling.adr.md (EXISTS)
# Action: Update references in spec files, do NOT re-migrate
```

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

DONE.md documents test completion for work items. **Read this file to find where tests are.**

Test locations in DONE.md may be:

- `specs/.../tests/` - Tests stayed in place (common now)
- `tests/unit/...` etc. - Tests were graduated (older pattern)

Example:

```markdown
# Story Complete: validate-args

## Tests

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

1. **Tests table** - Maps requirements to test file locations
2. **Test Level** - Determines target suffix (unit → `.unit.test.py`)
3. **Actual location** - Where to find the test (go get it from there)

</done_md_format>

<test_file_naming>

## Test File Naming in CODE Framework

### Determining Test Level

Read the test. Check what it uses. Apply the `/testing` skill's Quick Reference table.

| Evidence needed for...  | Level |
| ----------------------- | ----- |
| Business logic          | 1     |
| Parsing/validation      | 1     |
| File I/O with temp dirs | 1     |
| Database queries        | 2     |
| HTTP calls              | 2     |
| CLI binary behavior     | 2     |
| Full user workflow      | 3     |
| Real credentials        | 3     |
| Browser behavior        | 3     |

**Example:**

- Test reads `pyproject.toml` with `tomllib` → Level 1 (parsing)
- Test runs `subprocess.run(["pre-commit", ...])` → Level 2 (CLI binary)
- Test creates git repo + runs pre-commit hooks → Level 2 (project-specific tools)

### CRITICAL: Test Level ≠ Container Level

A story can have Level 2 tests. A capability can have Level 1 tests. Determine level by what the test USES, not where it lives.

### TypeScript Naming

TypeScript uses `.test.` SUFFIX:

| Level        | Pattern                   | Example                   |
| ------------ | ------------------------- | ------------------------- |
| Level 1      | `*.unit.test.ts`          | `parsing.unit.test.ts`    |
| Level 2      | `*.integration.test.ts`   | `cli.integration.test.ts` |
| Level 3      | `*.e2e.test.ts`           | `workflow.e2e.test.ts`    |
| Level 3 (PW) | `*.e2e.spec.ts` (browser) | `login.e2e.spec.ts`       |

### Python Naming

Python uses `test_` PREFIX (for pytest discovery):

| Level   | Pattern                 | Example                   |
| ------- | ----------------------- | ------------------------- |
| Level 1 | `test_*.unit.py`        | `test_parsing.unit.py`    |
| Level 2 | `test_*.integration.py` | `test_cli.integration.py` |
| Level 3 | `test_*.e2e.py`         | `test_workflow.e2e.py`    |

**Transformation examples:**

```text
# TypeScript
tests/unit/parsing.test.ts           → spx/.../tests/parsing.unit.test.ts
tests/integration/cli.test.ts        → spx/.../tests/cli.integration.test.ts

# Python
specs/.../tests/test_foo.py          → spx/.../tests/test_foo.unit.py
tests/integration/test_bar.py        → spx/.../tests/test_bar.integration.py
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

<migration_workflow>

## Migration Workflow

**Before starting any migration:**

1. **Create worktree** (once per migration session) - See worktree_requirement section

**For each work item being migrated:**

1. **Read DONE.md** - It documents what tests exist and where they are located
2. **Check for duplicates** - If DONE.md says tests graduated to `tests/`, verify:
   - Do graduated tests exist in `tests/`?
   - Do tests ALSO exist in `specs/.../tests/`?
3. **Use the authoritative source**:
   - If DONE.md says graduated AND tests exist in `tests/` → use those (they're authoritative)
   - If tests only exist in `specs/.../tests/` → use those
   - **WARNING**: If tests exist in BOTH locations, the `specs/.../tests/` copies may be stale duplicates
4. **Capture baseline coverage** - Run tests from worktree to get baseline (worktree is untouched)
5. **Move tests with `git mv`** to `spx/.../tests/` (preserves history):
   ```bash
   git mv specs/.../tests/test_foo.py spx/.../tests/test_foo.unit.py
   ```
   - Rename with level suffix during the move
   - **NEVER copy/rewrite tests** - use `git mv` to preserve history
6. **Move DONE.md to SPX-MIGRATION.md** (if DONE.md exists):
   ```bash
   git mv specs/.../tests/DONE.md spx/.../NN-slug.story/SPX-MIGRATION.md
   ```
   - Then edit SPX-MIGRATION.md to add corrected record sections (see `<spx_migration_md>`)
   - If no DONE.md exists, create SPX-MIGRATION.md documenting "No completion record found"
7. **Verify coverage matches** - Run moved tests, compare to baseline
8. **Remove stale duplicates** with `git rm` if tests existed in multiple locations:
   - Remove from `tests/` if graduated copies exist there
   - **Failure to remove from `tests/` leaves duplicates!**

**The directory location (backlog/doing/done) is irrelevant for migration. Migrate content regardless of which directory it's in.**

### Post-Migration Checklist

After migration, verify:

- [ ] NO tests remain in `tests/unit/` for this work item
- [ ] NO tests remain in `tests/integration/` for this work item
- [ ] NO tests remain in `tests/e2e/` for this work item
- [ ] NO files remain in `specs/.../tests/` (tests moved, DONE.md became SPX-MIGRATION.md)
- [ ] NO DONE.md exists anywhere in `spx/` (`! find spx/ -name DONE.md | grep .`)
- [ ] SPX-MIGRATION.md exists for every story that had tests

</migration_workflow>

<worktree_requirement>

## Reference Worktree (MANDATORY)

**Create the worktree first. One command. Just do it.**

```bash
git worktree add "../$(basename $(pwd))_pre-spx" \
  $(git log --oneline --diff-filter=A --all -- 'spx/' | tail -1 | cut -d' ' -f1)^
```

Then verify:

```bash
[ ! -d "../$(basename $(pwd))_pre-spx/spx" ] && echo "✓ Worktree valid"
```

**Why mandatory:**

- Baseline for test comparison
- Recovery if migration fails
- Debugging when tests behave differently

**Do NOT ask "should I create a worktree?" Just create it.**

</worktree_requirement>

<coverage_verification>

## Coverage Verification Requirements

### Granularity: Legacy File Level

- **Coverage is verified at the LEGACY FILE level, not story level**
- Multiple stories may contribute to the same legacy file
- A legacy file can only be removed after ALL contributing stories are migrated
- Individual story coverage percentages are meaningless - only combined coverage matters

### Verification Process

**Baseline from worktree** (always available):

```bash
# Run tests from worktree to get baseline coverage
cd "$WORKTREE_PATH" && just test "path/to/original/tests --cov --cov-report=json"
```

**After moving tests with `git mv`**:

```bash
# Run moved tests from spx/ location
just test "spx/.../tests/ --cov --cov-report=json"
# Compare coverage to baseline - MUST MATCH
```

**If coverage doesn't match** - debug using worktree:

```bash
# Compare test files
diff -u "$WORKTREE_PATH/path/to/test.py" spx/.../tests/test.py

# Run specific tests from worktree to identify what's missing
cd "$WORKTREE_PATH" && just test "path/to/test.py -v"
```

The worktree always has the original state - debugging is trivial.

</coverage_verification>

<valid_migration_checklist>

## What Constitutes a Valid Migration

A migration is complete and valid when:

### Structure

- [ ] SPX directory exists with correct naming (`{BSP}-{slug}.{type}/`)
- [ ] All spec files moved with `git mv` and renamed correctly
- [ ] ADRs moved with `git mv` from `decisions/` to in-tree location
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

- [ ] SPX-MIGRATION.md created at EVERY level where DONE.md existed
- [ ] SPX-MIGRATION.md contains ALL required sections (see `<spx_migration_md>`)

</valid_migration_checklist>

<spx_migration_md>

## SPX-MIGRATION.md: The Corrected Record

**DONE.md documents what was claimed. SPX-MIGRATION.md documents what actually happened.**

### Rule: Create at EVERY Level Where DONE.md Exists

If DONE.md exists at a level, SPX-MIGRATION.md MUST exist at the same level:

| If DONE.md exists here...               | Create SPX-MIGRATION.md here...               |
| --------------------------------------- | --------------------------------------------- |
| `specs/.../story-NN/tests/DONE.md`      | `spx/.../NN-slug.story/SPX-MIGRATION.md`      |
| `specs/.../feature-NN/tests/DONE.md`    | `spx/.../NN-slug.feature/SPX-MIGRATION.md`    |
| `specs/.../capability-NN/tests/DONE.md` | `spx/.../NN-slug.capability/SPX-MIGRATION.md` |

If NO DONE.md exists at a level but tests were migrated there, create SPX-MIGRATION.md anyway.

### Required Sections (ALL MANDATORY)

SPX-MIGRATION.md is NOT just a changelog. It is the **corrected verification record**.

```markdown
# SPX-MIGRATION: {spx-name}

## Original Completion Record

**From**: `{path to original DONE.md}`
**Verdict**: {APPROVED/etc from DONE.md}
**Original Date**: {from DONE.md}
**Migration Date**: {today}

## Test Location Corrections

DONE.md claimed tests were at locations that may not exist. This table shows:

- What DONE.md claimed
- Where tests actually were
- Where they are now in spx/

| DONE.md Claimed Location   | Actual Location Found         | SPX Location                            | Why This Level   |
| -------------------------- | ----------------------------- | --------------------------------------- | ---------------- |
| `tests/unit/foo.py`        | `specs/.../tests/test_foo.py` | `spx/.../tests/test_foo.unit.py`        | Pure computation |
| `tests/integration/bar.py` | `specs/.../tests/test_bar.py` | `spx/.../tests/test_bar.integration.py` | Uses subprocess  |

**"Why This Level"** must reference the testing skill's evidence table:

- Level 1: Business logic, parsing, file I/O with temp dirs
- Level 2: Database, HTTP, CLI binary behavior
- Level 3: Full workflow, real credentials, browser

## Complete Test Inventory

ALL test files at this level, with test counts:

| Test File                 | Test Count | Requirements Covered |
| ------------------------- | ---------- | -------------------- |
| `test_foo.unit.py`        | 11         | FR1, FR2, QR1        |
| `test_bar.integration.py` | 4          | FR3, FR4             |

**Total**: {N} tests

## ADR Reference Updates

If ANY spec file had ADR references updated:

| Spec File      | Old ADR Reference             | New ADR Reference          |
| -------------- | ----------------------------- | -------------------------- |
| `foo.story.md` | `decisions/adr-07_tooling.md` | `21-python-tooling.adr.md` |

If no ADR references were updated, state: "No ADR references required updates."

## Verification

\`\`\`bash

# Command to verify all tests pass

just test "spx/.../tests/"

# Result

{N} passed in {X}s
\`\`\`

## Files Moved/Removed

Legacy files and their disposition:

| Legacy File                   | Action   | Destination                            |
| ----------------------------- | -------- | -------------------------------------- |
| `specs/.../tests/test_foo.py` | `git mv` | `spx/.../tests/test_foo.unit.py`       |
| `specs/.../tests/DONE.md`     | `git mv` | `spx/.../SPX-MIGRATION.md` (this file) |
| `tests/unit/test_bar.py`      | `git rm` | (duplicate removed)                    |
```

### Failure Mode: Incomplete SPX-MIGRATION.md

**What happens:** Agent creates SPX-MIGRATION.md with just "migrated tests" and no details.

**Why it fails:** Next agent cannot verify:

- Which tests came from where
- Why tests are at their level
- What ADR references changed
- Whether all DONE.md entries are accounted for

**How to avoid:** Use the template above. Every section is mandatory. If a section doesn't apply, explicitly state "N/A" or "None".

### The Correction Requirement

DONE.md often contains **phantom graduations** - claims that tests are in `tests/unit/...` when they're actually in `specs/.../tests/`.

SPX-MIGRATION.md MUST:

1. Document what DONE.md **claimed**
2. Document what **actually existed**
3. Document where tests are **now**

This creates an audit trail that explains discrepancies.

</spx_migration_md>

<constraints>

## Invariants

### Source of Truth

- **DONE.md in the work item directory** documents which tests exist and where they are
- Read DONE.md first, then get tests from the location it specifies
- Never guess test mappings based on filename patterns
- Never trust SPX-MIGRATION.md written by previous runs - always verify against DONE.md
- **Worktree is mandatory** - use it for baseline coverage, debugging, and recovery

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

| Step                      | If interrupted  | On restart                         |
| ------------------------- | --------------- | ---------------------------------- |
| Create worktree           | No state change | Idempotent - skips if exists       |
| Read DONE.md              | No state change | Reads again (from worktree)        |
| Capture baseline          | No state change | Run from worktree again            |
| git mv tests              | Some moved      | Skips existing, moves rest         |
| git mv DONE→SPX-MIGRATION | Moved or not    | Skips if SPX-MIGRATION.md exists   |
| Edit SPX-MIGRATION.md     | Partial edit    | Re-edit (worktree has original)    |
| Verify coverage           | No state change | Runs again (worktree has baseline) |
| git rm duplicates         | Some removed    | Skips already-removed              |
| git rm old specs          | Some removed    | Skips already-removed              |

</constraints>
