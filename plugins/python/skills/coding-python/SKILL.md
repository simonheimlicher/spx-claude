---
name: coding-python
description: Write Python implementation code to pass existing tests. Use when implementing code after tests are written.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

<objective>
Write implementation code that makes existing tests pass. Given a story spec and failing tests, produce production-grade Python code.

**This skill WRITES implementation. Tests should already exist and be failing.**

</objective>

<quick_start>
**Input:** Story spec path with existing failing tests

**Output:** Implementation code that makes tests pass

**Workflow:**

1. Read story spec and failing tests
2. Write implementation code (GREEN phase)
3. Refactor while keeping tests green (REFACTOR phase)
4. Self-verify (types, lint)

</quick_start>

<prerequisites>

Before invoking this skill:

1. **Tests must exist** - Written by `/testing-python`
2. **Tests must fail** - RED phase complete
3. **Spec must be loaded** - Context from `/understanding-specs`

If tests don't exist, invoke `/testing-python` first.

</prerequisites>

<workflow>

## Step 1: Understand the Spec and Tests

Read the story spec and existing tests:

```bash
# Read story spec
cat {story_path}/{story_name}.story.md

# Read failing tests
cat {story_path}/tests/*.py

# Run tests to see current failures
uv run --extra dev pytest {story_path}/tests/ -v
```

Understand:

- What behaviors the tests verify
- What interfaces are expected (function signatures, classes)
- What the tests import (where implementation should live)

## Step 2: Write Implementation (GREEN)

Write the minimal implementation that makes tests pass.

**Code standards (per `/standardizing-python`):**

```python
# ✅ Type annotations on ALL functions
def process_order(order: Order, config: Config) -> OrderResult: ...


# ✅ Named constants for all literals
MIN_ORDER_VALUE = 10
MAX_ITEMS = 100


def validate_order(order: Order) -> ValidationResult:
    if order.total < MIN_ORDER_VALUE:
        return ValidationResult(ok=False, error="Order below minimum")
    ...


# ✅ Dependency injection for external dependencies
@dataclass
class OrderProcessorDeps:
    run_command: CommandRunner
    get_env: Callable[[str], str | None] = os.environ.get


def process_order(order: Order, deps: OrderProcessorDeps) -> OrderResult: ...
```

**Import hygiene:**

```python
# ✅ Absolute imports for infrastructure
from myproject.shared.config import Config
from myproject_testing.harnesses import PostgresHarness

# ✅ Relative imports only for same-package modules
from .models import Order
from . import validators

# ❌ Deep relative imports (use absolute)
from .....tests.harnesses import ...  # WRONG
```

## Step 3: Run Tests (Verify GREEN)

```bash
# Run tests - they should now pass
uv run --extra dev pytest {story_path}/tests/ -v

# All tests should pass
# If any fail, fix implementation and re-run
```

## Step 4: Refactor (Keep GREEN)

Clean up the implementation while keeping tests green:

1. **Extract constants** - Any repeated literals become module-level constants
2. **Simplify** - Remove unnecessary complexity
3. **Clarify** - Rename for clarity
4. **DRY** - Extract shared logic

After each change, run tests to ensure they still pass.

## Step 5: Self-Verify

Run all verification tools:

```bash
# Type checking
uv run --extra dev mypy src/

# Linting
uv run --extra dev ruff check src/

# Tests one more time
uv run --extra dev pytest {story_path}/tests/ -v
```

**All must pass before declaring complete.**

</workflow>

<code_patterns>

## Mandatory Patterns

### Named Constants

```python
# ❌ REJECTED: Magic values
def validate_score(score: int) -> bool:
    return 0 <= score <= 100


# ✅ REQUIRED: Named constants
MIN_SCORE = 0
MAX_SCORE = 100


def validate_score(score: int) -> bool:
    return MIN_SCORE <= score <= MAX_SCORE
```

### Dependency Injection

```python
# ❌ REJECTED: Direct import of external dependency
import subprocess


def sync_files(src: str, dest: str) -> bool:
    result = subprocess.run(["rsync", src, dest])
    return result.returncode == 0


# ✅ REQUIRED: Dependency injection
from dataclasses import dataclass
from typing import Protocol


class CommandRunner(Protocol):
    def run(self, cmd: list[str]) -> tuple[int, str, str]: ...


@dataclass
class SyncDeps:
    run_command: CommandRunner


def sync_files(src: str, dest: str, deps: SyncDeps) -> bool:
    returncode, _, _ = deps.run_command.run(["rsync", src, dest])
    return returncode == 0
```

### Type Annotations

```python
# ✅ All functions have full type annotations
def get_user(user_id: int) -> User | None:
    users: list[User] = fetch_users()
    return next((u for u in users if u.id == user_id), None)


# ✅ -> None on functions that don't return
def log_event(event: Event) -> None:
    logger.info(f"Event: {event}")
```

### Modern Python (3.10+)

```python
# ✅ Union types with |
def find_user(id: int) -> User | None: ...


# ✅ list, dict lowercase (not List, Dict)
users: list[User] = []
config: dict[str, str] = {}

# ✅ Match statements
match command:
    case "start":
        start_server()
    case "stop":
        stop_server()
    case _:
        raise ValueError(f"Unknown command: {command}")
```

</code_patterns>

<output_format>

When implementation is complete, report:

````markdown
## Implementation Complete

### Story: {story_path}

### Files Created/Modified

| File                      | Action   | Description                  |
| ------------------------- | -------- | ---------------------------- |
| `src/mymodule/handler.py` | Created  | Order handler implementation |
| `src/mymodule/models.py`  | Modified | Added Order dataclass        |

### Verification

```bash
$ uv run --extra dev pytest {story_path}/tests/ -v
# All tests passing

$ uv run --extra dev mypy src/
# No errors

$ uv run --extra dev ruff check src/
# No issues
```
````

### Ready for Review

Implementation complete. Ready for `/reviewing-python`.

```
</output_format>

<success_criteria>

Task is complete when:

- [ ] All tests in `{story}/tests/` pass
- [ ] Type checking passes (`mypy`)
- [ ] Linting passes (`ruff check`)
- [ ] Code uses named constants (no magic values)
- [ ] Code uses dependency injection (no direct external imports)
- [ ] All functions have type annotations

</success_criteria>
```
