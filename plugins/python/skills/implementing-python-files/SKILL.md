---
name: implementing-python-files
description: Implement Python code for specific files using test-driven workflow. Use when given file paths to implement, coding specific modules, or applying TDD to explicit files.
args: file_paths
---

<objective>
File-focused Python implementation skill. Implements code for one or more specific files using test-driven development, processing each file sequentially through test design, implementation, and review.
</objective>

<essential_principles>
**NO MOCKING. BEHAVIOR TESTING. CONSTANTS PATTERN. SEQUENTIAL FILE PROCESSING.**

- **File-Centric Workflow:** Each file goes through the complete cycle before moving to the next
- **Behavior-Driven Development:** Tests are written first to verify **behavior**, then code is written to pass them
- **Do Not Repeat Yourself (DRY):** Define **constants in the implementation**, then check for constants in tests (not literal strings)
- **Mandatory Review Quality Gate:** Each file must pass review before proceeding to the next

</essential_principles>

<quick_start>
**Given file path(s) to implement:**

1. Parse the file path(s) from arguments
2. For each file, run the 4-step implementation cycle
3. Continue until all files are complete

```text
File → Test Design → Implement → Review → Next File
          ↓             ↓          ↓
       testing       coding    reviewing
       -python       -python    -python
```

</quick_start>

<arguments>
**Input:** One or more file paths to implement

**Accepted formats:**

```bash
# Single file
/implementing-python-files src/mymodule/handler.py

# Multiple files (space-separated)
/implementing-python-files src/mymodule/handler.py src/mymodule/processor.py

# Glob pattern (expanded by shell)
/implementing-python-files src/mymodule/*.py
```

**File validation:**

- Files may or may not exist yet (new files are created)
- Paths should be relative to project root or absolute
- For existing files, tests may already exist (will be augmented)

</arguments>

<workflow>

## Step 0: Parse and Validate Input

Extract file paths from arguments. For each path:

1. **Normalize path** - Convert to absolute or project-relative
2. **Determine test location** - Co-located in `tests/` subdirectory or sibling `test_*.py`
3. **Check for existing tests** - Will augment, not replace

```text
src/mymodule/handler.py → tests/unit/mymodule/test_handler.py
                        → or: src/mymodule/tests/test_handler.unit.py (if spx/ style)
```

---

## For Each File (Sequential)

### Step 1: Analyze Context

Before writing tests, understand:

1. **Read the file** (if exists) - Understand current implementation
2. **Read related files** - Imports, dependencies, interfaces
3. **Identify behaviors** - What should this code do?

If file doesn't exist yet, determine expected behaviors from:

- User's request/description
- Interface contracts (if implementing an interface)
- Calling code (if being called by existing code)

### Step 2: Design Tests

Invoke `/testing-python` for **MANDATORY TESTING METHODOLOGY**.

Design tests that verify:

- Expected behaviors (not implementation details)
- Edge cases and error conditions
- Integration with dependencies (via injection, not mocking)

**Test file naming:**

```text
# For src/mymodule/handler.py
tests/unit/mymodule/test_handler.py          # Unit tests (Level 1)
tests/integration/mymodule/test_handler.py   # Integration tests (Level 2)
```

### Step 3: Implement

Invoke `/coding-python` to implement:

**RED phase:**

- Write failing tests following the test design
- Tests should fail for the right reasons (missing behavior)

**GREEN phase:**

- Write minimal production code to pass tests
- Focus on correctness, not optimization

**REFACTOR phase:**

- Clean up code while keeping tests green
- Apply constants pattern (extract repeated literals)
- Run type checking: `uv run --extra dev mypy src/`
- Run linting: `uv run --extra dev ruff check src/`

### Step 4: Review

Invoke `/reviewing-python` to review the implementation.

**If review identifies issues:**

1. Use `/coding-python` to fix issues
2. Re-invoke `/reviewing-python`
3. Repeat until the reviewer approves

**If review approves:**

- File is complete
- Proceed to next file

---

## After All Files Complete

Run full verification:

```bash
# All tests pass
uv run --extra dev pytest -v

# Type checking
uv run --extra dev mypy src/

# Linting
uv run --extra dev ruff check src/
```

</workflow>

<skill_invocations>
**Skills this workflow invokes:**

| Skill               | Purpose                        | When                 |
| ------------------- | ------------------------------ | -------------------- |
| `/testing-python`   | Test design methodology        | Before writing tests |
| `/coding-python`    | Implementation (RED/GREEN/REF) | After test design    |
| `/reviewing-python` | Code review                    | After implementation |

</skill_invocations>

<progress_tracking>
**Track progress through file list:**

```text
Files to implement:
├── src/mymodule/handler.py      [✓] Tests passing, reviewed
├── src/mymodule/processor.py    [→] In Progress (Step 3)
├── src/mymodule/validator.py    [pending] Not started
└── src/mymodule/formatter.py    [pending] Not started
```

**Legend:**

- `[✓]` = Tests passing, review approved
- `[→]` = In progress
- `[pending]` = Not started

Update tracking as you complete each file.

</progress_tracking>

<success_criteria>

## File Complete

- [ ] Tests exist and verify correct behaviors
- [ ] Tests use dependency injection (no mocking)
- [ ] Implementation passes all tests
- [ ] Code uses constants pattern (no repeated literals)
- [ ] File passed `/reviewing-python` approval

## All Files Complete

- [ ] All files individually approved
- [ ] All tests pass: `uv run --extra dev pytest -v`
- [ ] Type checking passes: `uv run --extra dev mypy src/`
- [ ] Linting passes: `uv run --extra dev ruff check src/`

</success_criteria>

<error_handling>

**File doesn't exist and no context provided:**
→ Ask user for expected behaviors before proceeding

**Tests already exist with mocking:**
→ Flag to user, offer to refactor tests to use DI

**Review fails repeatedly (3+ attempts):**
→ Stop and report issues to user for guidance

**Circular dependency detected:**
→ Flag architectural issue, suggest DI pattern

**Type errors that can't be resolved:**
→ Stop and report, may need interface changes

</error_handling>
