---
name: auto-typescript
description: Implement TypeScript stories sequentially with specs, testing, coding, and review. Use when implementing features with multiple stories or when auto-implementing a feature.
---

<objective>
Autonomous TypeScript implementation orchestrator. Implements stories in a feature one after another, coordinating specs, testing, coding, and review skills in a strict sequence until all stories are complete.
</objective>

<essential_principles>
**NO MOCKING. BEHAVIOR TESTING. CONSTANTS PATTERN. EVERY CREATING SKILL IS FOLLOWED BY A REVIEWING SKILL. FOLLOW THE STRICT SEQUENCE.**

- **Strict Sequence of Skills:** Follow the workflow sequence **exactly** - no skipping steps
- **Behavior-Driven Development:** Tests are written first to verify **behavior**, never implementation, then code is written to (i) validate that the tests are appropriate and (ii) pass them. Finally code and tests are refactored until they pass reviewing skills without reservation
- **Do Not Repeat Yourself (DRY):** Do not use any literal string multiple times. Define **constants in the implementation,** then check for constants in tests (not literal strings)
- **Mandatory Review Quality Gate:** Each story and its tests must pass review before proceeding to the next

</essential_principles>

<quick_start>
**Given a feature with stories to implement:**

1. Identify the first/next incomplete story
2. Run the 5-step implementation cycle
3. Repeat until all stories complete

```text
Story → Specs → Test Design → Implement → Review → Remediate → Next Story
         ↓          ↓           ↓          ↓          ↓
   understanding  testing    coding   manual     reviewing
      -specs    -typescript -typescript review   -typescript
```

</quick_start>

<workflow>
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

Continue until all stories in the feature are implemented and approved. Then continue in the same manner with the subsequent feature until all features of the capability are implemented.

</workflow>

<skill_invocations>
**Skills this orchestrator invokes:**

| Skill                   | Purpose                             | When                 |
| ----------------------- | ----------------------------------- | -------------------- |
| `/understanding-specs`  | Load story context and requirements | Start of each story  |
| `/testing-typescript`   | Design test strategy                | After context loaded |
| `/coding-typescript`    | Implement code and tests            | After test design    |
| `/reviewing-typescript` | Automated code review               | After implementation |

**Invocation syntax:**

```text
/understanding-specs {story-path}
/testing-typescript
/coding-typescript
/reviewing-typescript
```

</skill_invocations>

<progress_tracking>
**Track progress through the feature:**

```text
Feature: {feature-name}
├── [✓] story-10_first - Approved
├── [→] story-20_second - In Progress (Step 3: Implementing)
├── [ ] story-30_third - Pending
└── [ ] story-40_fourth - Pending
```

Update this tracking as you complete each story.

</progress_tracking>

<success_criteria>
Feature implementation is complete when:

- [ ] All stories in the feature have been processed
- [ ] Each story passed approval by `/reviewing-typescript` and its tests have been graduated
- [ ] All tests pass (`vitest run`)
- [ ] Type checking passes (`tsc --noEmit`)
- [ ] Linting passes (`eslint`)

Implementation quality (no mocking, constants pattern) is verified by `/reviewing-typescript`.

</success_criteria>
