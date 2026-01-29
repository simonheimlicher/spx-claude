---
name: orchestrating-python
description: Orchestrate Python story implementation through specs, testing, coding, and review. Use when implementing features with multiple stories or when auto-implementing a feature.
---

<objective>
Autonomous Python implementation orchestrator. Implements stories in a feature one after another, coordinating specs, testing, coding, and review skills in a strict sequence until all stories are complete.
</objective>

<essential_principles>
**NO MOCKING. BEHAVIOR TESTING. CONSTANTS PATTERN. FULL BSP PATHS. FOLLOW THE STRICT SEQUENCE.**

- **Strict Sequence of Skills:** Follow the workflow sequence **exactly** - no skipping steps
- **Full BSP Paths ALWAYS:** BSP numbers are sibling-unique, not globally unique. ALWAYS use full paths like `21-core-cli.capability/54-commands.feature/54-something.story`, never bare names like `54-something.story`
- **Behavior-Driven Development:** Tests are written first to verify **behavior**, never implementation, then code is written to (i) validate that the tests are appropriate and (ii) pass them. Finally code and tests are refactored until they pass reviewing skills without reservation
- **Do Not Repeat Yourself (DRY):** Do not use any literal string multiple times. Define **constants in the implementation,** then check for constants in tests (not literal strings)
- **Mandatory Review Quality Gate:** Each story and its tests must pass review before proceeding to the next

</essential_principles>

<quick_start>
**Given a feature with stories to implement:**

1. Run `spx spec next` to identify the next incomplete story
2. Run the 5-step implementation cycle
3. Repeat until `spx spec next` returns no more work items

```text
Story → Specs → Test Design → Implement → Review → Remediate → Next Story
         ↓          ↓           ↓          ↓          ↓
   understanding  testing    coding   manual     reviewing
      -specs      -python    -python  review     -python
```

**CLI commands for status:**

- `spx spec next` - Get next work item (respects BSP ordering)
- `spx spec status --format table` - View project status

</quick_start>

<workflow>

## Part A: Story Implementation

**For each story in the feature:**

**Step 1: Load Story Context**
Invoke `/understanding-specs` on the story to load:

- Story specification
- Parent feature/capability requirements
- Relevant ADRs and constraints

If context loading fails, STOP.

Use the `AskUserQuestion` tool to discover with the user's help what the missing document should contain. Then use `/managing-specs` to create missing specifications, then restart this workflow.

**Step 2: Design Tests**
Invoke `/testing-python` to design the test strategy.

Invoke `/testing-python` for **MANDATORY TESTING METHODOLOGY** (no mocking, behavior testing, DRY using constants pattern).

**Step 3: Implement**
Invoke `/coding-python` with the story spec to:

- **RED phase:** Write tests following the test design
- **GREEN phase:** Write high quality production code
- **REFACTOR phase:** Run type checking and linting. Refactor the implementation code and tests. Always ensure all tests pass

**Step 4: Manual Review**
Before invoking the automated reviewer, manually verify:

- [ ] Tests verify the correct behaviors at the correct level using adequate test harnesses for integration and e2e tests
- [ ] Code matches story requirements
- [ ] No missing edge cases
- [ ] Code is clean and readable
- [ ] No obvious issues that might fail review

Fix any issues found before proceeding.

**Step 5: Automated Review**
Invoke `/reviewing-python` to review the implementation.

**If review identifies issues:**

1. Use `/coding-python` to fix issues
2. Re-invoke `/reviewing-python`
3. Repeat until the reviewer approves

**If review approves:**

- Story is complete
- Proceed to next story

**Step 6: Next Story**
Return to Step 1 with the next story in the feature.

Continue until all stories in the feature are implemented and approved.

---

## Part B: Feature Completion

**After ALL stories in a feature are approved**, complete the feature:

**Step 7: Feature-Level Tests**

1. **Read the feature spec** to find `## Outcomes` section with Test Files tables
2. **Check for integration tests** (Level 2 harness references)
3. **If Level 2 tests are specified:**
   - Implement each test following the spec's Gherkin
   - Tests go in the feature's `tests/` directory with `.integration.test.py` suffix
   - Use real infrastructure via test harnesses (no mocking)
   - Run tests: `uv run --extra dev pytest spx/{capability}/{feature}/tests/ -v`
4. **If no Level 2 tests specified:** Feature uses Level 1 only (documented in Test Strategy section)

**Step 8: Stamp Feature pass.csv**

Run `spx spec test --stamp` to update the feature's pass.csv:

- Validates all tests in `spx/{capability}/{feature}/tests/` pass
- Records `spec_blob` and `test_blob` SHAs
- Verifies child story pass.csv files are current

**Step 9: Next Feature**
Return to Part A, Step 1 with the first story of the next feature.

Continue until all features in the capability are implemented.

---

## Part C: Capability Completion

**After ALL features in a capability are approved**, complete the capability:

**Step 10: Capability-Level Tests**

1. **Read the capability spec** to find `## Outcomes` section with Test Files tables
2. **Check for E2E tests** (Level 3 harness references)
3. **If Level 3 tests are specified:**
   - Implement each test following the spec's Gherkin
   - Tests go in the capability's `tests/` directory with `.e2e.test.py` suffix
   - Use real credentials and services (full user workflows)
   - Run tests: `uv run --extra dev pytest spx/{capability}/tests/ -v`
4. **If no Level 3 tests specified:** Capability uses Level 1-2 only (documented in Test Strategy section)

**Step 11: Stamp Capability pass.csv**

Run `spx spec test --stamp` to update the capability's pass.csv:

- Validates all tests in `spx/{capability}/tests/` pass
- Records `spec_blob` and `test_blob` SHAs
- Verifies child feature pass.csv files are current
- Capability is complete when pass.csv is valid and all children are passing

</workflow>

<skill_invocations>
**Skills this orchestrator invokes:**

| Skill                  | Purpose                             | When                 |
| ---------------------- | ----------------------------------- | -------------------- |
| `/understanding-specs` | Load story context and requirements | Start of each story  |
| `/testing-python`      | Design test strategy                | After context loaded |
| `/coding-python`       | Implement code and tests            | After test design    |
| `/reviewing-python`    | Automated code review               | After implementation |

**Invocation syntax (ALWAYS use full paths):**

```text
# ✅ CORRECT: Full path
/understanding-specs 21-core-cli.capability/54-commands.feature/54-run.story

# ❌ WRONG: Bare story name (ambiguous)
/understanding-specs 54-run.story
```

</skill_invocations>

<progress_tracking>
**Track progress through the capability (ALWAYS use full paths):**

```text
21-core-cli.capability/
├── 54-commands.feature/
│   ├── 10-init.story/     [pass.csv ✓] All tests passing
│   ├── 20-build.story/    [pass.csv ✓] All tests passing
│   ├── 30-run.story/      [→] In Progress (Step 3)
│   └── 40-test.story/     [pending] Not started
│   └── pass.csv           [pending] Stamp after all stories pass
├── 65-config.feature/
│   └── (stories pending)
│   └── pass.csv           [pending]
└── pass.csv               [pending] Stamp after all features pass
```

**Legend:**

- `[pass.csv ✓]` = All tests passing, pass.csv valid
- `[→]` = In progress
- `[pending]` = Not started or tests not passing
- Tests live in each container's `tests/` directory
- Level indicated by test filename suffix (`.unit.test.py`, `.integration.test.py`, `.e2e.test.py`)

Update this tracking as you complete each work item.

</progress_tracking>

<success_criteria>

## Story Complete

- [ ] Story passed approval by `/reviewing-python`
- [ ] Tests co-located in `spx/{capability}/{feature}/{story}/tests/`
- [ ] `spx spec test --stamp` generates valid pass.csv

## Feature Complete

- [ ] All child story pass.csv files are valid
- [ ] Feature-level integration tests implemented (if specified in spec)
- [ ] `spx spec test --stamp` generates valid feature pass.csv
- [ ] All tests pass (`uv run --extra dev pytest spx/{capability}/{feature}/ -v`)
- [ ] Type checking passes (`uv run --extra dev mypy src/`)
- [ ] Linting passes (`uv run --extra dev ruff check src/`)

## Capability Complete

- [ ] All child feature pass.csv files are valid
- [ ] Capability-level E2E tests implemented (if specified in spec)
- [ ] `spx spec test --stamp` generates valid capability pass.csv
- [ ] All tests pass at all levels

Implementation quality (no mocking, constants pattern) is verified by `/reviewing-python`.

</success_criteria>
