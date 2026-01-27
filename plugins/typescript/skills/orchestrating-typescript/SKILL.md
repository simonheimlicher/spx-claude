---
name: orchestrating-typescript
description: Orchestrate TypeScript story implementation through specs, testing, coding, and review. Use when implementing features with multiple stories or when auto-implementing a feature.
---

<objective>
Autonomous TypeScript implementation orchestrator. Implements stories in a feature one after another, coordinating specs, testing, coding, and review skills in a strict sequence until all stories are complete.
</objective>

<essential_principles>
**NO MOCKING. BEHAVIOR TESTING. CONSTANTS PATTERN. FULL BSP PATHS. FOLLOW THE STRICT SEQUENCE.**

- **Strict Sequence of Skills:** Follow the workflow sequence **exactly** - no skipping steps
- **Full BSP Paths ALWAYS:** BSP numbers are sibling-unique, not globally unique. ALWAYS use full paths like `capability-21/feature-54/story-54`, never bare numbers like `story-54`
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
      -specs    -typescript -typescript review   -typescript
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
Invoke `/testing-typescript` to design the test strategy.

Invoke `/testing-typescript` for **MANDATORY TESTING METHODOLOGY** (no mocking, behavior testing, DRY using constants pattern).

**Step 3: Implement**
Invoke `/coding-typescript` with the story spec to:

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
Invoke `/reviewing-typescript` to review the implementation.

**If review identifies issues:**

1. Use `/coding-typescript` to fix issues
2. Re-invoke `/reviewing-typescript`
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

1. **Read the feature spec** to find `## Feature Integration Tests (Level 2)` section
2. **Check for specified tests** (FI1, FI2, etc.)
3. **If Level 2 tests are specified:**
   - Implement each test following the spec's pseudocode
   - Tests go in `tests/integration/{capability}/{feature}/`
   - Use real infrastructure via test harnesses (no mocking)
   - Run tests to verify they pass: `vitest run tests/integration/`
4. **If no Level 2 tests specified:** Document why in feature DONE.md (e.g., "Level 1 sufficient—no external integrations")

**Step 8: Feature DONE.md**

Create `DONE.md` in the feature's spec directory:

- Document all graduated Level 1 tests (from stories)
- Document all graduated Level 2 tests (from this step)
- Verify all story DONE.md files exist
- Include test run output showing all feature tests pass

**Step 9: Next Feature**
Return to Part A, Step 1 with the first story of the next feature.

Continue until all features in the capability are implemented.

---

## Part C: Capability Completion

**After ALL features in a capability are approved**, complete the capability:

**Step 10: Capability-Level Tests**

1. **Read the capability spec** to find `## Capability E2E Tests (Level 3)` section
2. **Check for specified tests** (E2E1, E2E2, etc.)
3. **If Level 3 tests are specified:**
   - Implement each test following the spec's pseudocode
   - Tests go in `tests/e2e/{capability}/`
   - Use real credentials and services (full user workflows)
   - Run tests to verify they pass: `vitest run tests/e2e/`
4. **If no Level 3 tests specified:** Document why in capability DONE.md (e.g., "Level 2 sufficient—no external services")

**Step 11: Capability DONE.md**

Create `DONE.md` in the capability's spec directory:

- Document all graduated tests across all levels
- Verify all feature DONE.md files exist
- Include test run output showing all capability tests pass
- Summary of delivered value

</workflow>

<skill_invocations>
**Skills this orchestrator invokes:**

| Skill                   | Purpose                             | When                 |
| ----------------------- | ----------------------------------- | -------------------- |
| `/understanding-specs`  | Load story context and requirements | Start of each story  |
| `/testing-typescript`   | Design test strategy                | After context loaded |
| `/coding-typescript`    | Implement code and tests            | After test design    |
| `/reviewing-typescript` | Automated code review               | After implementation |

**Invocation syntax (ALWAYS use full paths):**

```text
# ✅ CORRECT: Full path
/understanding-specs capability-21/feature-54/story-54

# ❌ WRONG: Bare story number (ambiguous)
/understanding-specs story-54
```

</skill_invocations>

<progress_tracking>
**Track progress through the capability (ALWAYS use full paths):**

```text
Capability: capability-21_core-cli
├── Feature: capability-21/feature-54_commands
│   ├── [✓] capability-21/feature-54/story-10_init - DONE.md ✓
│   ├── [✓] capability-21/feature-54/story-20_build - DONE.md ✓
│   ├── [→] capability-21/feature-54/story-30_run - In Progress (Step 3)
│   └── [ ] capability-21/feature-54/story-40_test - Pending
│   └── [L2] Feature Integration Tests - Pending (after all stories)
│   └── [ ] Feature DONE.md - Pending
├── Feature: capability-21/feature-65_config
│   └── [ ] (stories pending)
│   └── [L2] Feature Integration Tests - Pending
│   └── [ ] Feature DONE.md - Pending
└── [L3] Capability E2E Tests - Pending (after all features)
└── [ ] Capability DONE.md - Pending
```

**Legend:**

- `[✓]` = Complete with DONE.md
- `[→]` = In progress
- `[ ]` = Pending
- `[L2]` = Level 2 integration tests (feature-level)
- `[L3]` = Level 3 E2E tests (capability-level)

Update this tracking as you complete each work item.

</progress_tracking>

<success_criteria>

## Story Complete

- [ ] Story passed approval by `/reviewing-typescript`
- [ ] Story Level 1 tests graduated to `tests/unit/`
- [ ] Story DONE.md created

## Feature Complete

- [ ] All stories in the feature have DONE.md
- [ ] Feature Level 2 tests implemented (if specified in feature spec)
- [ ] Feature Level 2 tests graduated to `tests/integration/`
- [ ] Feature DONE.md created
- [ ] All tests pass (`vitest run`)
- [ ] Type checking passes (`tsc --noEmit`)
- [ ] Linting passes (`eslint`)

## Capability Complete

- [ ] All features in the capability have DONE.md
- [ ] Capability Level 3 tests implemented (if specified in capability spec)
- [ ] Capability Level 3 tests graduated to `tests/e2e/`
- [ ] Capability DONE.md created
- [ ] All tests pass at all levels

Implementation quality (no mocking, constants pattern) is verified by `/reviewing-typescript`.

</success_criteria>
