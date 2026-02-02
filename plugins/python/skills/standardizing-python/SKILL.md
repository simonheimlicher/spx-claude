---
name: standardizing-python
description: Python code standards enforced across all skills. Reference skill for type annotations, naming conventions, and linting rules.
allowed-tools: Read
---

# Python Code Standards

> **This is a reference skill.** Other Python skills reference these standards. You typically don't invoke this directly—invoke `/coding-python`, `/testing-python`, or `/reviewing-python` instead.

These standards apply to ALL Python code: production and test code alike.

---

## Type Annotations (MANDATORY)

ALL functions require complete type annotations. No exceptions.

```python
# ✅ REQUIRED: Return types on ALL functions
def process_items(
    items: list[str],
    config: Config,
    logger: logging.Logger,
) -> ProcessResult:
    """Process items according to config."""


# ✅ REQUIRED: -> None on functions that return nothing
def test_validates_input(self) -> None:
    result = validate("test")
    assert result.valid


# ✅ REQUIRED: -> None on __init__
def __init__(self, config: Config) -> None:
    self.config = config


# ✅ REQUIRED: Type annotations on ALL parameters
def test_creates_file(self, tmp_path: Path) -> None:
    file = tmp_path / "test.txt"
    assert not file.exists()


# ✅ REQUIRED: Return types on fixtures
@pytest.fixture
def config(tmp_path: Path) -> Config:
    return Config(path=tmp_path)


# ❌ REJECTED: Missing return type (ANN201)
def test_something(self):
    pass


# ❌ REJECTED: Missing parameter type (ANN001)
def test_with_fixture(self, tmp_path) -> None:
    pass


# ❌ REJECTED: Missing __init__ return type (ANN204)
def __init__(self, config: Config):
    self.config = config
```

**Ruff rules enforced:**

| Rule   | What it catches                        |
| ------ | -------------------------------------- |
| ANN001 | Missing type annotation on parameter   |
| ANN201 | Missing return type on public function |
| ANN204 | Missing return type on `__init__`      |

---

## Named Constants (NO magic values)

Test values and configuration must use named constants, not inline literals.

```python
# ✅ REQUIRED: Named constants at module level
VALID_SCORE = 85
MIN_SCORE = 0
MAX_SCORE = 100
VALID_INPUT = "simple"
EXPECTED_RESULT = 42


class TestScoreValidation:
    def test_accepts_valid_score(self) -> None:
        assert validate_score(VALID_SCORE) is True

    def test_rejects_above_maximum(self) -> None:
        assert validate_score(MAX_SCORE + 1) is False


# ❌ REJECTED: Magic values (PLR2004)
class TestScoreValidationBad:
    def test_accepts_valid_score(self) -> None:
        assert validate_score(85) is True  # What is 85?

    def test_rejects_above_maximum(self) -> None:
        assert validate_score(101) is False  # Magic number
```

**Why named constants matter:**

- Sharing between tests and production code
- Clear documentation of what values mean
- Easy updates when requirements change
- Self-documenting test intent

**PLR2004 exemptions:** Ruff's magic value rule already exempts common idiomatic values: `0`, `1`, `""`, and `"__main__"`. You don't need constants for these.

```python
# ✅ OK: Idiomatic values are exempt
assert len(results) == 0
assert count == 1
if __name__ == "__main__":
    main()
```

---

## Naming Conventions (PEP8)

### Lowercase Argument Names (N803)

```python
# ❌ REJECTED: Uppercase argument names
def __init__(self, domain: ClockDomain, WIDTH: int = 8) -> None:
    pass


# ✅ REQUIRED: Lowercase argument names
def __init__(self, domain: ClockDomain, width: int = 8) -> None:
    pass
```

### Avoid Shadowing Builtins

```python
# ❌ BAD: Shadows builtin `input`
@pytest.mark.parametrize("input,expected", TEST_CASES)
def test_processing(self, input: str, expected: int) -> None:
    pass


# ✅ GOOD: Descriptive name, no shadowing
@pytest.mark.parametrize(("input_val", "expected"), TEST_CASES)
def test_processing(self, input_val: str, expected: int) -> None:
    pass
```

**Why avoid `input` as a parameter name:**

- Shadows Python's builtin `input()` function
- Causes A002 (argument shadows builtin) lint errors
- Makes code confusing if you need the actual `input()` function
- The tuple form `("input_val", "expected")` is also preferred by pytest for clarity

---

## S101 Policy (assert in tests)

Ruff's S101 rule flags `assert` statements because they can be disabled with Python's `-O` flag.

**Policy:** `assert` is ACCEPTED in test files because:

1. pytest rewrites assertions for better error messages
2. Tests are never run with `-O` optimization
3. The alternative (`if not x: raise AssertionError`) adds noise

**Required project configuration** in `pyproject.toml`:

```toml
[tool.ruff.lint.per-file-ignores]
"**/test_*.py" = ["S101"]
"**/tests/**/*.py" = ["S101"]
```

If the project hasn't configured this, tests will fail linting. Fix by adding the config, not by avoiding `assert`.

---

## Summary: Rejection Criteria

| Issue                      | Example                         | Ruff Rule |
| -------------------------- | ------------------------------- | --------- |
| Missing `-> None` on test  | `def test_foo(self):`           | ANN201    |
| Untyped fixture parameter  | `def test_foo(self, tmp_path):` | ANN001    |
| Missing `-> None` on init  | `def __init__(self, x: int):`   | ANN204    |
| Magic values in assertions | `assert result == 42`           | PLR2004   |
| Uppercase argument names   | `def __init__(self, WIDTH=8):`  | N803      |
| Shadowing builtins         | `def foo(input: str):`          | A002      |
