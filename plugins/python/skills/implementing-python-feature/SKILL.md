---
name: implementing-python-feature
description: Implement Python stories in a feature through specs, testing, coding, and review. Use when implementing features with multiple stories or when auto-implementing a feature.
---

<objective>
Orchestrate Python feature implementation by invoking skills in sequence for each story. This is a 4-step loop: write tests → review tests → implement → review implementation.
</objective>

<quick_start>
**Input:** Feature or story path

**Workflow per story:**

```text
Story → Step 1 → Step 2 → Step 3 → Step 4 → Next Story
          │         │         │         │
      /testing   /reviewing  /coding   /reviewing
       -python    -python     -python   -python
                  -tests
```

1. **Load spec** - Invoke `/understanding-specs` on the story
2. **Write tests** - Invoke `/testing-python` → tests written
3. **Review tests** - Invoke `/reviewing-python-tests` → APPROVE/REJECT
4. **Implement** - Invoke `/coding-python` → code written
5. **Review code** - Invoke `/reviewing-python` → APPROVE/REJECT
6. **Next story** - Repeat until feature complete

</quick_start>

<workflow>

## Step 0: Identify Work Items

Determine which stories to implement:

```bash
# Option A: List stories in a feature
ls {feature_path}/*-*.story/

# Option B: Specific story provided by user
# Use the provided story path directly
```

## Step 1: Load Story Context

For each story, invoke `/understanding-specs`:

```text
/understanding-specs {story_path}
```

This loads:

- Story specification (outcomes, acceptance criteria)
- Parent feature/capability requirements
- Relevant ADRs/PDRs and constraints

**If context fails to load:** STOP. Ask user for guidance.

## Step 2: Write Tests

Invoke `/testing-python` with the story path:

```text
/testing-python
```

The skill will:

- Read story outcomes (Gherkin)
- Determine test levels per `/testing` methodology
- Write test files to `{story}/tests/`
- Run tests to confirm they fail (RED)

**Output:** Test files created, failing as expected.

## Step 3: Review Tests

Invoke `/reviewing-python-tests`:

```text
/reviewing-python-tests
```

The reviewer will check:

- Evidentiary value (can tests pass while assertion fails?)
- Spec compliance (all outcomes covered?)
- Standards compliance (markers, types, no mocking)

**If REJECT:**

1. Fix issues identified by reviewer
2. Re-invoke `/reviewing-python-tests`
3. Repeat until APPROVE

**If APPROVE:** Proceed to Step 4.

## Step 4: Implement

Invoke `/coding-python`:

```text
/coding-python
```

The skill will:

- Read failing tests
- Write implementation code
- Run tests until GREEN
- Self-verify (types, lint)

**Output:** Implementation complete, tests passing.

## Step 5: Review Implementation

Invoke `/reviewing-python`:

```text
/reviewing-python
```

The reviewer will check:

- Code quality (constants, DI, types)
- Spec compliance
- Security, performance

**If REJECT:**

1. Fix issues identified by reviewer
2. Re-invoke `/reviewing-python`
3. Repeat until APPROVE

**If APPROVE:** Story is complete.

## Step 6: Next Story

Return to Step 1 with the next story in the feature.

Continue until all stories are implemented and approved.

</workflow>

<skill_sequence>

| Step | Skill                     | Purpose      | Output                  |
| ---- | ------------------------- | ------------ | ----------------------- |
| 1    | `/understanding-specs`    | Load context | Spec loaded             |
| 2    | `/testing-python`         | Write tests  | Test files (failing)    |
| 3    | `/reviewing-python-tests` | Review tests | APPROVE/REJECT          |
| 4    | `/coding-python`          | Implement    | Code files (tests pass) |
| 5    | `/reviewing-python`       | Review code  | APPROVE/REJECT          |

**Rejection loops:** Steps 3 and 5 may loop until APPROVE.

</skill_sequence>

<progress_tracking>

Track progress through the feature:

```text
{feature_path}/
├── 10-first.story/      [✓] Tests approved, code approved
├── 20-second.story/     [→] In Progress (Step 4: implementing)
├── 30-third.story/      [pending] Not started
└── 40-fourth.story/     [pending] Not started
```

**Legend:**

- `[✓]` = Story complete (tests + code approved)
- `[→]` = In progress (show current step)
- `[pending]` = Not started

Update tracking after each story.

</progress_tracking>

<output_format>

When feature is complete, report:

````markdown
## Feature Implementation Complete

### Feature: {feature_path}

### Stories Implemented

| Story           | Tests      | Code       | Status   |
| --------------- | ---------- | ---------- | -------- |
| 10-first.story  | ✓ Approved | ✓ Approved | Complete |
| 20-second.story | ✓ Approved | ✓ Approved | Complete |
| 30-third.story  | ✓ Approved | ✓ Approved | Complete |

### Verification

```bash
$ uv run --extra dev pytest {feature_path}/ -v
# All tests passing

$ uv run --extra dev mypy src/
# No errors
```
````

### Next Steps

Feature complete. Ready for feature-level integration tests (if specified in feature spec).

```
</output_format>

<error_handling>

**Test review fails repeatedly (3+ attempts):**
→ Stop and report issues to user for guidance

**Code review fails repeatedly (3+ attempts):**
→ Stop and report issues to user for guidance

**Missing ADRs:**
→ Invoke `/architecting-python` before continuing

**Spec not found:**
→ Stop and ask user for correct path

</error_handling>

<success_criteria>

Feature is complete when:

- [ ] All stories have tests approved by `/reviewing-python-tests`
- [ ] All stories have code approved by `/reviewing-python`
- [ ] All tests pass: `uv run --extra dev pytest {feature}/`
- [ ] Type checking passes: `uv run --extra dev mypy src/`
- [ ] Linting passes: `uv run --extra dev ruff check src/`

</success_criteria>
```
