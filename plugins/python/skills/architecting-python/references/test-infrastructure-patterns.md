# Test Infrastructure Patterns

Test infrastructure is an architectural concern. Design it once in ADRs, not ad-hoc during implementation.

## Core Principle

> **Test utilities are production code. Package them properly. Verify the environment before running tests.**

---

## The Test Environment Problem

### Symptom: "Module not found" When Running Tests

```bash
$ uv run pytest specs/work/doing/.../tests/test_foo.py
ModuleNotFoundError: No module named 'click'  # But click IS installed!
```

### Root Cause: Wrong pytest

```bash
$ uv run which pytest
/opt/homebrew/bin/pytest  # ❌ WRONG - System pytest, not project venv
```

The system pytest uses a different Python that doesn't have your project's dependencies.

### Fix: Install Dev Dependencies in Project Venv

```bash
# This installs pytest AND all dev deps in the project venv
uv pip install -e ".[dev]"

# Verify
uv run which pytest
# /path/to/project/.venv/bin/pytest  # ✅ CORRECT
```

---

## Test Utility Packaging

### The Problem: Tests Can't Import Shared Code

```python
# specs/work/doing/.../tests/test_foo.py
from tests.fixtures import create_user  # ❌ ModuleNotFoundError
```

### ❌ Wrong: Path Hacks

```python
# conftest.py - DON'T DO THIS
import sys
from pathlib import Path

sys.path.insert(0, str(Path(__file__).parent.parent))  # Brittle, breaks easily
```

### ✅ Right: Make Test Utilities an Installable Package

**Step 1: Structure**

```
project/
├── pyproject.toml
├── mypackage/              # Main package
│   └── ...
├── mypackage_testing/      # Test utilities package (NOT tests/)
│   ├── __init__.py
│   ├── fixtures/
│   │   ├── __init__.py
│   │   └── users.py        # create_user(), etc.
│   └── harnesses/
│       ├── __init__.py
│       └── cli.py          # CLIHarness, etc.
└── specs/                  # Co-located tests (per CODE framework)
    └── .../tests/
        └── test_foo.py     # from mypackage_testing.fixtures import create_user
```

**Step 2: pyproject.toml**

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["mypackage", "mypackage_testing"] # Both are installable
```

**Step 3: Install**

```bash
uv pip install -e ".[dev]"  # Installs both packages in editable mode
```

**Step 4: Import**

```python
# specs/work/doing/.../tests/test_foo.py
from mypackage_testing.fixtures import create_user  # ✅ Works everywhere
```

---

## pytest Configuration for Complex Layouts

### The Problem: Multiple test directories collide

```
project/
├── specs/.../story-21/tests/test_generics.py  # Co-located tests
├── specs/.../story-54/tests/test_generics.py  # DIFFERENT co-located tests (same filename!)
└── legacy/tests/test_generics.py              # Legacy tests
```

pytest gets confused: "imported module 'test_generics' has this **file** attribute..."

### Fix: Use importlib Import Mode

```toml
# pyproject.toml
[tool.pytest.ini_options]
python_files = ["test_*.py"]
python_functions = ["test_*"]
addopts = "-v --tb=short --import-mode=importlib" # KEY: importlib mode
pythonpath = ["."]
```

### Why importlib Mode Works

| Mode                | Behavior                                               | Problem                              |
| ------------------- | ------------------------------------------------------ | ------------------------------------ |
| `prepend` (default) | Adds test dir to sys.path, imports as top-level module | Multiple `test_foo.py` files collide |
| `importlib`         | Uses importlib to import each file independently       | Each file is isolated, no collisions |

---

## Excluding Out-of-Scope Code

### The Problem: Broken legacy code fails `just check`

```bash
$ just check
ERROR legacy/tests/test_emitter.py - ImportError: cannot import name 'HDLEmitterBase'
```

### Fix: Explicit Exclusions in Check Commands

```makefile
# justfile

# Run all checks - excludes legacy/
check: lint typecheck
    uv run pytest mypackage_testing/ specs/work/doing/ --ignore=legacy/

# Coverage commands also need exclusion
test-cov:
    uv run pytest --cov=mypackage --cov-report=term --ignore=legacy/

# Legacy has its own quarantined commands
test-legacy:
    uv run pytest legacy/tests/ -v  # Expected to fail
```

---

## ADR Section: Test Infrastructure

Every Python project ADR should include:

````markdown
## Test Infrastructure

### Test Utility Package

Test utilities (fixtures, harnesses, helpers) are packaged as `{project}_testing/`:

- Location: `{project}_testing/` (sibling to main package)
- Installation: `uv pip install -e ".[dev]"`
- Import: `from {project}_testing.fixtures import ...`

### pytest Configuration

```toml
[tool.pytest.ini_options]
addopts = "-v --tb=short --import-mode=importlib"
pythonpath = ["."]
```
````

### Test Locations

| Type             | Location           | Runs in `just check` |
| ---------------- | ------------------ | -------------------- |
| Co-located tests | `specs/.../tests/` | Yes                  |
| Regression tests | `tests/`           | Yes                  |
| Legacy tests     | `legacy/tests/`    | No (quarantined)     |

### Environment Verification

Before running tests, verify:

1. `uv run which pytest` → must be `.venv/bin/pytest`
2. Dev deps installed → `uv pip install -e ".[dev]"`

````
---

## Verification Checklist

Before running any tests, verify:

```bash
# 1. pytest is from project venv
uv run which pytest
# Expected: /path/to/project/.venv/bin/pytest
# If wrong: uv pip install -e ".[dev]"

# 2. Test utilities are importable
uv run python -c "from mypackage_testing.fixtures import ...; print('OK')"
# If fails: Check pyproject.toml packages list, re-run uv pip install -e ".[dev]"

# 3. pytest config has importlib mode
grep "import-mode" pyproject.toml
# Expected: --import-mode=importlib in addopts
````

---

## Key Principles

1. **Test utilities are packages** — Install them, don't path-hack them

2. **Verify environment first** — `uv run which pytest` before running tests

3. **Use importlib mode** — Required for projects with multiple test directories

4. **Exclude out-of-scope code** — Legacy/broken code gets quarantined, not blocking

5. **Document in ADRs** — Test infrastructure is an architectural decision
