---
name: testing-python
description: Write Python tests for a story spec. Use when writing tests for Python code, before implementation.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

<objective>
Write test files for a story specification. Given a story spec with Gherkin outcomes, produce actual test files that verify those outcomes at the appropriate test levels.

**This skill WRITES tests. It does not just design or plan.**

</objective>

<quick_start>
**Input:** Story spec path (e.g., `spx/01-capability/02-feature/21-story.story/`)

**Output:** Test files written to `{story}/tests/` directory

**Workflow:**

1. Read the story spec to understand outcomes
2. Consult `/testing` methodology (5 stages) to determine test levels
3. Reference `/standardizing-python-testing` for Python patterns
4. Write test files that verify each outcome
5. Run tests to confirm they fail (RED phase)

</quick_start>

<workflow>

## Step 1: Load Context

Read the story spec and related files:

```bash
# Read story spec
cat {story_path}/{story_name}.story.md

# Read parent feature for context
cat {feature_path}/{feature_name}.feature.md

# Check for ADRs that constrain testing approach
ls {capability_path}/*.adr.md {feature_path}/*.adr.md 2>/dev/null
```

Extract from the spec:

- **Outcomes** - Gherkin scenarios to verify
- **Test Strategy** - Which levels are specified
- **Harnesses** - Any referenced test harnesses

## Step 2: Determine Test Levels

For each outcome, apply the `/testing` methodology:

**Stage 1:** What evidence do I need?
**Stage 2:** At what level does that evidence live? (5 factors)
**Stage 3:** What kind of code is this? (Pure/Extract/Glue)
**Stage 4:** Can the real system produce the behavior?
**Stage 5:** Which exception applies, if any?

| Evidence Type                   | Minimum Level |
| ------------------------------- | ------------- |
| Pure computation/algorithm      | 1             |
| File I/O with temp dirs         | 1             |
| Standard dev tools (git, curl)  | 1             |
| Project-specific binary         | 2             |
| Database, Docker                | 2             |
| Real credentials, external APIs | 3             |

## Step 3: Design Test Cases

For each outcome, design test cases that:

1. **Verify behavior, not implementation** - What the code does, not how
2. **Use the Gherkin as guide** - GIVEN/WHEN/THEN maps to arrange/act/assert
3. **Cover edge cases** - Boundary conditions, error conditions
4. **Use named constants** - No magic values (per `/standardizing-python-testing`)

**Test case structure:**

```python
# Named constants at module level
VALID_INPUT = "expected_input"
EXPECTED_OUTPUT = "expected_output"
ERROR_INPUT = "invalid_input"


@pytest.mark.level_1
def test_outcome_name_happy_path() -> None:
    """GIVEN valid input WHEN processed THEN returns expected output."""
    # Arrange
    input_data = VALID_INPUT

    # Act
    result = function_under_test(input_data)

    # Assert
    assert result == EXPECTED_OUTPUT


@pytest.mark.level_1
def test_outcome_name_error_case() -> None:
    """GIVEN invalid input WHEN processed THEN raises appropriate error."""
    with pytest.raises(ValidationError):
        function_under_test(ERROR_INPUT)
```

## Step 4: Write Test Files

Create test files in the story's tests directory:

```text
{story_path}/
├── {story_name}.story.md
└── tests/
    ├── test_{outcome_name}.level_1.py
    ├── test_{outcome_name}.level_2.py  # if Level 2 needed
    └── conftest.py                      # fixtures if needed
```

**File naming convention:**

- `test_{name}.level_1.py` - Unit tests
- `test_{name}.level_2.py` - Integration tests
- `test_{name}.level_3.py` - E2E tests

**Mandatory elements per `/standardizing-python-testing`:**

- `@pytest.mark.level_N` on every test
- `-> None` return type on every test function
- Type annotations on all parameters
- Named constants for all test values
- Property-based tests for parsers/serializers/math (`@given`)
- No mocking - use dependency injection

## Step 5: Verify Tests Fail (RED)

Run the tests to confirm they fail for the right reasons:

```bash
# Run tests - they should fail because implementation doesn't exist
uv run --extra dev pytest {story_path}/tests/ -v

# Expected: Tests fail with ImportError or AssertionError
# NOT: Tests pass (would mean tests are trivial)
# NOT: Tests fail with unexpected errors
```

**If tests pass:** Tests are not testing anything useful. Revise.
**If tests fail with wrong errors:** Fix test setup, imports, etc.
**If tests fail with expected errors:** Proceed to implementation.

</workflow>

<test_writing_checklist>

Before declaring tests complete:

- [ ] Each Gherkin outcome has at least one test
- [ ] Test level matches the evidence type (per `/testing` Stage 2)
- [ ] File names include level suffix (`.level_1.py`, etc.)
- [ ] All tests marked with `@pytest.mark.level_N`
- [ ] All test functions have `-> None` return type
- [ ] All parameters have type annotations
- [ ] Named constants used (no magic values)
- [ ] Parsers/serializers have property-based tests (`@given`)
- [ ] No mocking - dependency injection where doubles needed
- [ ] Tests run and fail for expected reasons (RED phase)

</test_writing_checklist>

<patterns_reference>

See `/standardizing-python-testing` for:

- **Level patterns** - How to write Level 1, 2, 3 tests
- **Exception implementations** - The 7 exception cases in Python
- **Property-based testing** - Hypothesis patterns
- **Data factories** - Factory and builder patterns
- **DI patterns** - Protocol and dataclass dependencies
- **Harness patterns** - Docker, subprocess harnesses
- **Anti-patterns** - What to avoid

</patterns_reference>

<output_format>

When tests are written, report:

````markdown
## Tests Written

### Story: {story_path}

### Test Files Created

| File                        | Level | Outcomes Covered |
| --------------------------- | ----- | ---------------- |
| `tests/test_foo.level_1.py` | 1     | Outcome 1, 2     |
| `tests/test_foo.level_2.py` | 2     | Outcome 3        |

### Test Run (RED Phase)

```bash
$ uv run --extra dev pytest {story_path}/tests/ -v
# Output showing expected failures
```
````

### Ready for Implementation

Tests are ready. Implementation should make these tests pass.

```
</output_format>

<success_criteria>

Task is complete when:

- [ ] Test files exist in `{story}/tests/` directory
- [ ] Each outcome from spec has corresponding test(s)
- [ ] Tests follow `/standardizing-python-testing` standards
- [ ] Tests run and fail for expected reasons
- [ ] Output report shows files created and test run results

</success_criteria>
```
