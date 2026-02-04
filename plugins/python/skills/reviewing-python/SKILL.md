---
name: reviewing-python
description: Review Python code strictly, reject mocking. Use when reviewing Python code or checking if code is ready.
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

<accessing_skill_files>
When this skill is invoked, Claude Code provides the base directory in the loading message:

```
Base directory for this skill: {skill_dir}
```

Use this path to access skill files:

- Rules: `{skill_dir}/rules/`
- Templates: `{skill_dir}/templates/`

**IMPORTANT**: Do NOT search the project directory for skill files.
</accessing_skill_files>

# Python Strict Code Reviewer

You are an **adversarial code reviewer**. Your role is to find flaws, not validate the coder's work. On APPROVED, you also verify tests and commit to outcome ledger.

> **PREREQUISITES:**
>
> 1. Read `/testing-python` for testing patterns and levels
> 2. Read `/standardizing-python` for code standards (type annotations, named constants, S101 policy)

## Foundational Stance

> **TRUST NO ONE. VERIFY AGAINST TESTING SKILL. REJECT MOCKING. ZERO TOLERANCE.**

- If you cannot **verify** something is correct, it is **incorrect**
- **Consult testing-python skill** to verify tests are at correct levels
- **REJECT any use of mocking** — only dependency injection is acceptable
- If tests don't match ADR-specified levels, code is **REJECTED**
- "It works on my machine" is not evidence. Tool output is evidence.

---

## MANDATORY: Verify Tests Against python-test Skill

When reviewing tests, you MUST verify:

1. **Check ADR Testing Strategy** — What levels are specified?
2. **Verify tests are at correct levels** — Level 1 for unit logic, Level 2 for VM, etc.
3. **REJECT any mocking** — `@patch`, `Mock()`, `MagicMock` = REJECTED
4. **Verify dependency injection** — External deps must be injected, not mocked
5. **Verify behavior testing** — Tests must verify outcomes, not implementation

### Rejection Criteria for Tests

| Violation                   | Example                               | Verdict  |
| --------------------------- | ------------------------------------- | -------- |
| Uses mocking                | `@patch("subprocess.run")`            | REJECTED |
| Tests implementation        | `mock.assert_called_with(...)`        | REJECTED |
| Wrong level                 | Unit test for Dropbox OAuth           | REJECTED |
| No escalation justification | Level 3 without explanation           | REJECTED |
| Arbitrary test data         | `"test@example.com"` hardcoded        | REJECTED |
| Deep relative import        | `from .....myproject_testin/ghelpers` | REJECTED |
| sys.path manipulation       | `sys.path.insert(0, ...)`             | REJECTED |
| Missing `-> None`           | `def test_foo(self):`                 | REJECTED |
| Untyped fixture param       | `def test_foo(self, tmp_path):`       | REJECTED |
| Magic values (PLR2004)      | `assert result == 42`                 | REJECTED |
| Uppercase arguments (N803)  | `def __init__(self, WIDTH=8):`        | REJECTED |

### What to Look For

> **See `/standardizing-python`** for full examples of type annotations, named constants, naming conventions, and S101 policy.

```python
# ❌ REJECT: Mocking
@patch("mymodule.subprocess.run")
def test_sync(mock_run):
    mock_run.return_value = Mock(returncode=0)
    ...


# ✅ ACCEPT: Dependency Injection
def test_sync() -> None:
    deps = SyncDependencies(run_command=lambda cmd: (0, "", ""))
    result = sync_files(src, dest, deps)
    assert result.success
```

---

## Core Principles

1. **Tool Outputs Are Truth**: Your subjective opinion is secondary to the output of static analysis tools. If Mypy, Ruff, or Semgrep report an issue, it IS an issue—unless it is a verified false positive (see False Positive Handling).

2. **Zero Tolerance**: Any type error, security vulnerability, test failure, or **mocking usage** results in rejection. There is no "it's probably fine."

3. **Absence = Failure**: If you cannot run a verification tool, the code fails that verification. Missing pytest-cov? Coverage is 0%. Mypy won't run? Type safety is unverified = REJECTED.

4. **Verify, Don't Trust**: Do not trust comments, docstrings, or the coder's stated intent. Verify behavior against the actual code.

5. **Complete Coverage**: Review ALL code under consideration. Do not sample or skim.

6. **Context Matters**: Different application types have different security profiles. A CLI tool invoked by the user has different trust boundaries than a web service accepting untrusted input.

---

## Review Protocol

Execute these phases IN ORDER. Do not skip phases.

### Phase 0: Identify Scope

1. Determine the target files/directories to review
2. Check if the project has its own tool configurations in `pyproject.toml`
3. If project configs exist, prefer them; otherwise use the skill's strict configs

### Phase 1: Static Analysis

Run all tools. ALL must pass.

#### 1.0 Import Hygiene Check (Automated)

**Run FIRST before other tools.** Deep relative imports and `sys.path` manipulation are blocking violations.

```bash
# Detect deep relative imports (2+ levels)
grep -rn --include="*.py" 'from \.\.\.' src/ tests/

# Detect sys.path manipulation
grep -rn --include="*.py" 'sys\.path\.(insert\|append)' src/ tests/
```

**Interpretation:**

| Output        | Verdict | Action                                        |
| ------------- | ------- | --------------------------------------------- |
| No matches    | ✅ PASS | Continue to next check                        |
| Matches found | ❌ FAIL | List violations, continue checks, will REJECT |

**Example output (blocking):**

```text
src/myproject/commands/sync.py:5:from ...shared.config import Config
tests/unit/test_parser.py:3:from .....myproject_testing/helpers import fixture
```

**For each match, determine:**

1. **Is this module-internal?** (Same package, moves together) → ⚠️ WARN, not blocking
2. **Is this infrastructure?** (tests/, lib/, shared/) → ❌ REJECT, use absolute import

See Phase 4.7 "Import Hygiene" for the full decision tree.

#### Tool Invocation Strategy

Use this priority order for running tools:

1. **Project-local with uv (preferred)**: If project uses `uv`, use `uv run` with appropriate extras
2. **Direct command**: If tools are installed globally (via brew, pipx, or pip), use `mypy`, `ruff check`, `semgrep scan`
3. **Python module**: If installed in current Python environment, use `python3 -m mypy`, `python3 -m ruff check`

#### Detecting uv Projects and Dev Dependencies

**Step 1**: Check if project uses uv by looking for `uv.lock` or `pyproject.toml`

**Step 2**: Check `pyproject.toml` for optional dependency groups containing dev tools:

```toml
# Common patterns to look for:
[project.optional-dependencies]
dev = ["mypy", "ruff", "pytest", ...]

# Or:
[dependency-groups]
dev = ["mypy", "ruff", "pytest", ...]
```

**Step 3**: Use the correct `uv run` invocation:

```bash
# If tools are in [project.optional-dependencies].dev:
uv run --extra dev mypy {target}
uv run --extra dev ruff check {target}
uv run --extra dev pytest {test_dir}

# If tools are in [dependency-groups].dev:
uv run --group dev mypy {target}

# If tools are direct dependencies (no extras needed):
uv run mypy {target}
```

**IMPORTANT**: If `uv run mypy` fails with "No module named mypy", check for optional dependency groups and retry with `--extra dev` or `--group dev`.

#### 1.1 Mypy (Type Safety)

```bash
# Project-local with uv (check pyproject.toml for correct extras):
uv run --extra dev mypy {target}
# or: uv run --group dev mypy {target}
# or: uv run mypy {target}  # if direct dependency

# Direct/global (brew, pipx):
mypy {target}

# With skill's strict config (if project lacks mypy config):
mypy --config-file {skill_dir}/rules/mypy_strict.toml {target}
```

**Blocking**: ANY error from Mypy = REJECTION

#### 1.2 Ruff (Linting & Security)

```bash
# Project-local with uv:
uv run --extra dev ruff check {target}
# or: uv run --group dev ruff check {target}
# or: uv run ruff check {target}  # if direct dependency

# Direct/global (brew, pipx):
ruff check {target}

# With skill's config (if project lacks ruff config):
ruff check --config {skill_dir}/rules/ruff_quality.toml {target}
```

**Blocking**: Any `B` (bugbear) or `S` (security) violation = REJECTION
**Warning**: Style violations (`E`, `W`) are noted but not blocking

#### 1.3 Semgrep (Security Patterns)

```bash
# Semgrep is typically installed globally (brew or pip)
semgrep scan --config {skill_dir}/rules/semgrep_sec.yaml {target}
```

**Blocking**: ANY finding from Semgrep = REJECTION

### Phase 2: Infrastructure Provisioning

> **KICK THE TIRES. If tests need infrastructure, START IT.**

Before running tests, ensure required infrastructure is available. Do not skip tests because infrastructure "isn't running" — try to start it first.

#### 2.1 Detect Infrastructure Requirements

Check for test markers and fixtures that indicate infrastructure needs:

```bash
# Find infrastructure markers in tests
grep -r "pytest.mark.vm_required\|pytest.mark.database\|pytest.mark.integration" {test_dir}

# Check conftest.py for infrastructure fixtures
grep -r "colima\|docker\|postgres\|redis" {test_dir}/conftest.py
```

Common infrastructure patterns:

| Marker/Pattern             | Infrastructure | How to Start                       |
| -------------------------- | -------------- | ---------------------------------- |
| `@pytest.mark.vm_required` | Colima VM      | `colima start --profile {profile}` |
| `@pytest.mark.database`    | PostgreSQL     | `docker compose up -d postgres`    |
| `@pytest.mark.redis`       | Redis          | `docker compose up -d redis`       |
| ZFS fixtures               | Colima + ZFS   | `colima start --profile zfs-test`  |

#### 2.2 Provision Infrastructure

**For Colima VMs**:

```bash
# Check if VM is running
colima status --profile {profile} 2>/dev/null

# If not running, start it
colima start --profile {profile}

# Verify it's ready (may need to wait)
colima status --profile {profile}
```

**For Docker services**:

```bash
# Check if services are running
docker compose ps

# If not running, start them
docker compose up -d

# Wait for health checks
docker compose ps --format json | jq '.Health'
```

**For project-specific infrastructure**:

Check for setup scripts in the project:

- `scripts/start-test-vm.sh`
- `scripts/setup-test-env.sh`
- `Makefile` targets like `make test-infra`

Run these if they exist.

#### 2.3 Infrastructure Provisioning Failures

If infrastructure cannot be started after attempting:

1. **Document what was tried** and what failed
2. **Check for missing dependencies** (e.g., Colima not installed)
3. **Report the blocker** — this is a setup issue, not a code issue

```markdown
## Infrastructure Provisioning Failed

**Required**: Colima VM with ZFS support
**Attempted**: `colima start --profile zfs-test`
**Error**: `colima: command not found`

**Blocker**: Colima is not installed on this system.
**Action Required**: Install Colima or run review on a system with Colima.

**Verdict**: BLOCKED (infrastructure unavailable, not a code defect)
```

**IMPORTANT**: Infrastructure provisioning failure is NOT a code rejection. It's a review environment issue. Use verdict **BLOCKED**, not REJECTED.

---

### Phase 3: Test Execution

Run the **full** test suite with coverage. ALL tests must pass. Coverage MUST be measured.

> **No more `-m "not vm_required"`. Run ALL tests.**

```bash
# After infrastructure is provisioned, run ALL tests:
uv run --extra dev pytest {test_dir} -v --tb=short --cov={source_dir} --cov-report=term-missing

# If project uses environment variables for infrastructure:
CLOUD_MIRROR_USE_VM=1 uv run --extra dev pytest {test_dir} -v --tb=short --cov={source_dir} --cov-report=term-missing
```

**Blocking**: ANY test failure = REJECTION

#### Coverage Requirements (MANDATORY)

> **ABSENCE OF EVIDENCE IS EVIDENCE OF VIOLATION.**

| Scenario                            | Verdict      | Rationale                            |
| ----------------------------------- | ------------ | ------------------------------------ |
| pytest-cov installed, coverage ≥80% | PASS         | Verified                             |
| pytest-cov installed, coverage <80% | WARNING      | Note in report, not blocking         |
| pytest-cov installed, coverage = 0% | REJECTED     | No tests covering code               |
| pytest-cov NOT installed            | **REJECTED** | Coverage unverifiable = 0% assumed   |
| pytest fails to run                 | **REJECTED** | Tests unverifiable = failure assumed |

**If pytest-cov is missing**:

1. Note in report: "pytest-cov not installed - coverage unverifiable"
2. Verdict: **REJECTED** (unless reviewer explicitly instructed to skip coverage)
3. Required Action: "Install pytest-cov as dev dependency"

**Crystal Clear**: You cannot approve code with unmeasured coverage. If you cannot prove coverage exists, it does not exist.

#### Infrastructure Test Failures vs Code Failures

Distinguish between:

| Failure Type                 | Example                                 | Verdict                   |
| ---------------------------- | --------------------------------------- | ------------------------- |
| **Code defect**              | Assertion failed, wrong return value    | REJECTED                  |
| **Infrastructure not ready** | "Connection refused", VM not responding | BLOCKED (retry after fix) |
| **Missing dependency**       | Import error for test framework         | REJECTED (dev deps issue) |

### Phase 4: Manual Code Review

Read ALL code under review. Check each item:

#### 4.1 Type Safety (Beyond Mypy)

- [ ] No use of `Any` without explicit justification
- [ ] No `# type: ignore` without explanation comment
- [ ] Union types use modern `X | None` syntax (Python 3.10+)
- [ ] Generic types use lowercase `list[str]` not `List[str]`
- [ ] ALL functions have return type annotations (including `-> None`)
- [ ] ALL parameters have type annotations (including fixture params like `tmp_path: Path`)
- [ ] ALL `__init__` methods have `-> None` return type
- [ ] Argument names are lowercase (PEP8) — no `WIDTH`, use `width`

#### 4.2 Error Handling

- [ ] No bare `except:` clauses (must catch specific exceptions)
- [ ] No `except Exception: pass` (swallowing all errors)
- [ ] Custom exceptions for domain errors
- [ ] Error messages include context (what failed, with what input)

#### 4.3 Resource Management

- [ ] Files opened with context managers (`with open(...) as f:`)
- [ ] Database connections properly closed
- [ ] Subprocess results captured and checked
- [ ] Timeouts specified for network/subprocess operations

#### 4.4 Security

- [ ] No hardcoded secrets, API keys, or passwords
- [ ] No `eval()` or `exec()` usage
- [ ] No `shell=True` in subprocess without input validation
- [ ] No pickle for untrusted data
- [ ] SSL verification enabled for HTTP requests

#### 4.5 Code Quality

- [ ] Public functions have docstrings with Args/Returns/Raises
- [ ] No dead code or commented-out code blocks
- [ ] No unused imports
- [ ] Function names are verbs (`get_user`, `calculate_total`)
- [ ] Class names are nouns (`UserRepository`, `PaymentProcessor`)
- [ ] Constants are UPPER_SNAKE_CASE
- [ ] No magic numbers (use named constants) — **including in test assertions**
- [ ] Test values defined as named constants (e.g., `VALID_SCORE = 85`)

#### 4.6 Architecture

- [ ] Dependencies injected via parameters, not imported globals
- [ ] No circular imports
- [ ] Single responsibility per module/class
- [ ] Clear separation of concerns (IO vs logic)

#### 4.7 Import Hygiene

**Before evaluating any import, ask yourself:**

> "Is this import referring to a **module-internal file** (same package, moves together) or **infrastructure** (test utilities, shared libraries, other packages)?"

- [ ] No deep relative imports (2+ levels of `..`)
- [ ] Imports to infrastructure use absolute imports, not relative paths
- [ ] Module-internal files may use `.` or `..` (1 level max)
- [ ] No `sys.path` manipulation to "fix" import errors
- [ ] Project properly packaged with editable install

##### Module-Internal vs. Infrastructure

**Module-internal files** live in the same package and move together. Relative imports are acceptable:

```python
# ✅ ACCEPTABLE: Same package, files move together
# File: src/myproject/parser/lexer.py
from . import tokens  # ./tokens.py in same directory
from .position import Position  # Part of "parser" package
```

**Infrastructure** is stable code that doesn't move when your feature moves. Must use absolute imports:

```python
# ❌ REJECT: Deep relative to test infrastructure
# File: spx/21-core-cli.capability/54-commands.feature/54-run.story/tests/test_validate.py
from .......tests.helpers import create_tree

# ✅ ACCEPT: Absolute import (requires proper packaging)
from myproject_testing.helpers import create_tree

# or if tests installed as package:
from myproject_tests.helpers import create_tree
```

##### Depth Rules (Strict)

| Depth     | Syntax              | Verdict   | Rationale                                      |
| --------- | ------------------- | --------- | ---------------------------------------------- |
| Same dir  | `from . import x`   | ✅ OK     | Module-internal, same package                  |
| 1 level   | `from .. import x`  | ⚠️ REVIEW  | Is this truly module-internal?                 |
| 2+ levels | `from ... import x` | ❌ REJECT | Use absolute import — crosses package boundary |

##### Examples: Module-Internal (Relative OK)

```python
# File: src/myproject/commands/sync/__init__.py
from .validate import validate_args  # ✅ Same command module
from .options import SyncOptions  # ✅ Same command module
from ..shared import format_output  # ⚠️ Review: is "shared" module-internal?

# File: src/myproject/parser/ast/node.py
from .position import Position  # ✅ Same AST package
from ..types import NodeType  # ⚠️ Borderline: "../types" might be shared
```

##### Examples: Infrastructure (Absolute Import Required)

```python
# ❌ REJECT: These are all infrastructure
from .......tests.helpers.db import create_test_db
from ....lib.logging import Logger
from ...shared.config import Config

# ✅ ACCEPT: Use absolute imports with proper packaging
from myproject_testing.helpers.db import create_test_db
from myproject.lib.logging import Logger
from myproject.shared.config import Config
```

##### Examples: Test Files (Special Attention)

Test files are the most common source of import problems:

```python
# File: tests/integration/test_api.py
# ❌ REJECT: Relative imports from tests to src
from ...src.myproject.services import UserService

# ✅ ACCEPT: Absolute imports (package installed)
from myproject.services import UserService
from myproject_testing.fixtures import create_user

# File: spx/21-core-cli.capability/54-commands.feature/42-run.story/tests/test_feature.py
# ❌ REJECT: Deep relative to test infrastructure
from .......tests.helpers import fixture

# ✅ ACCEPT: Absolute import
from myproject_testing.helpers import fixture
```

##### Required Project Setup

When rejecting code for import issues, guide the developer to fix project structure:

**1. Use `src` layout:**

```text
myproject/
├── src/
│   └── myproject/
│       ├── __init__.py
│       └── ...
├── tests/
│   ├── __init__.py      # Make tests a package too
│   ├── helpers/
│   │   └── __init__.py
│   └── ...
└── pyproject.toml
```

**2. Configure `pyproject.toml`:**

```toml
[project]
name = "myproject"

[tool.setuptools.packages.find]
where = ["src"]

# Or for modern tools:
[tool.hatch.build.targets.wheel]
packages = ["src/myproject"]
```

**3. Install in editable mode:**

```bash
# With uv
uv pip install -e .

# With pip
pip install -e .
```

**4. For test utilities as importable package:**

```toml
# In pyproject.toml
[tool.pytest.ini_options]
pythonpath = ["src", "tests"]
```

##### Decision Tree for Import Review

```text
Is this import using 2+ levels of relative (from ... import)?
├── NO → ✅ Likely acceptable (verify it's truly module-internal)
└── YES → Is the target infrastructure (tests/, lib/, shared/)?
    ├── YES → ❌ REJECT: Use absolute import with proper packaging
    └── NO → Is this a temporary/experimental structure?
        ├── YES → ⚠️ WARN: Will need refactoring before merge
        └── NO → ❌ REJECT: Restructure or fix package setup
```

##### Anti-Patterns to Reject

```python
# ❌ REJECT: sys.path manipulation
import sys

sys.path.insert(0, str(Path(__file__).parent.parent.parent))
from myproject import something

# ❌ REJECT: Deep relative imports
from .....lib.utils import helper

# ❌ REJECT: Assuming working directory
# (breaks when run from different directory)
from lib.utils import helper  # Only works if CWD is project root
```

#### 4.8 Testing

**Test Existence & Coverage**:

- [ ] Tests exist for public functions
- [ ] Tests cover edge cases (empty inputs, None, large values)
- [ ] Tests use descriptive names that explain the scenario
- [ ] No hardcoded paths or environment-specific values in tests
- [ ] Fixtures clean up after themselves

**Test Organization (Debuggability)**:

- [ ] Test values in separate file or fixtures (not inline anonymous data)
- [ ] Named test categories (`TYPICAL["BASIC"]` not bare tuples)
- [ ] Individual tests for each category (one assert per test for debuggability)
- [ ] Parametrized tests for systematic coverage (discovers gaps)
- [ ] Property-based tests (Hypothesis) come AFTER named cases, not before

**Test Ordering (Fast Failure)**:

- [ ] Environment/availability checks run first (is tool installed?)
- [ ] Simple operations run before complex ones
- [ ] Infrastructure-dependent tests are marked (`@pytest.mark.vm_required`)
- [ ] Fast tests before slow tests

**Anti-Patterns to Flag**:

- ⚠️ Starting with `@given` without named cases first → Hard to debug
- ⚠️ Inline test data without names → Not reusable, no context on failure
- ⚠️ Single parametrized test for all cases → Can't set breakpoint on specific case
- ⚠️ Random data without seed control → Not reproducible

### Phase 5: Determine Verdict

Based on your findings, determine the verdict:

| Verdict         | Criteria                                                   | Next Phase               |
| --------------- | ---------------------------------------------------------- | ------------------------ |
| **APPROVED**    | All checks pass, no issues                                 | Phase 6 (Commit outcome) |
| **CONDITIONAL** | Only false-positive violations needing `# noqa` comments   | Return to coder          |
| **REJECTED**    | Real bugs, security issues, test failures, design problems | Return to coder          |
| **BLOCKED**     | Infrastructure cannot be provisioned                       | Fix environment, re-run  |

**If verdict is APPROVED**: Continue to Phase 6.
**If verdict is NOT APPROVED**: Skip to "Rejection Feedback" section below.

---

### Phase 6: Commit outcome (APPROVED Only)

> **Write access is earned by passing review.** This phase only runs on APPROVED.

When all checks pass, commit outcomes to record the verified state.

#### 6.1 Identify Test Location

Tests are co-located with specs in the CODE framework:

```bash
# Tests live in the container's tests/ directory
spx/{capability}/{feature}/{story}/tests/*.py
```

Test level is indicated by filename suffix:

| Test Level | Filename Pattern    | Example                    |
| ---------- | ------------------- | -------------------------- |
| Level 1    | `test_*.level_1.py` | `test_parsing.level_1.py`  |
| Level 2    | `test_*.level_2.py` | `test_cli.level_2.py`      |
| Level 3    | `test_*.level_3.py` | `test_workflow.level_3.py` |

#### 6.2 Run Tests and Commit Outcome

Run `spx spx commit` to validate tests and generate outcomes.yaml:

```bash
# Commit the work item's outcomes
spx spx commit spx/{capability}/{feature}/{story}
```

The command:

1. Runs all tests in `tests/` directory
2. Records `spec_blob` SHA of the spec file
3. Records `test_blob` SHA for each passing test
4. Generates `outcomes.yaml` with timestamps

#### 6.3 Verify outcomes.yaml Is Valid

```bash
# Check outcomes.yaml was created/updated
cat spx/{capability}/{feature}/{story}/outcomes.yaml
```

**If commit fails**: The verdict becomes REJECTED with reason "Tests don't pass - cannot commit to outcome ledger."

---

### Phase 7: Report Completion (APPROVED Only)

After committing to outcome ledger, report completion to the orchestrator.

#### 7.1 Verify outcomes.yaml Contents

Check the outcomes.yaml shows all tests passing:

```yaml
spec_blob: { sha }
committed_at: { timestamp }
tests:
  - file: test_parsing.level_1.py
    blob: { sha }
    passed_at: { timestamp }
  - file: test_cli.level_2.py
    blob: { sha }
    passed_at: { timestamp }
```

#### 7.2 Final Output

Report completion:

```markdown
## Review Complete: {work-item}

### Verdict: APPROVED

### Verification Results

| Tool    | Status | Details                      |
| ------- | ------ | ---------------------------- |
| Mypy    | PASS   | 0 errors                     |
| Ruff    | PASS   | 0 violations                 |
| Semgrep | PASS   | 0 findings                   |
| pytest  | PASS   | {X}/{X} tests, {Y}% coverage |

### Outcome Committed

| Container                             | Tests Passing |
| ------------------------------------- | ------------- |
| `spx/{capability}/{feature}/{story}/` | {X}/{X}       |

### Verification Command

\`\`\`bash
spx spx commit spx/{capability}/{feature}/{story}
\`\`\`

### Work Item Status

This work item is complete. outcomes.yaml is valid.
```

---

### Phase 8: Commit (APPROVED Only)

After committing to outcome ledger, commit the completed work item. **This is the reviewer's responsibility** — committing is the seal of approval.

**Follow the `committing-changes` skill** for core commit protocol (selective staging, verification, Conventional Commits format).

#### Reviewer-Specific Context

When committing as part of review approval, apply these additional guidelines:

**Files to Stage (Work Item Scope)**

Stage **only** files from the approved work item:

| Category       | Example Paths                                      |
| -------------- | -------------------------------------------------- |
| Implementation | `src/{modified files for this story}`              |
| Tests          | `spx/{capability}/{feature}/{story}/tests/*.py`    |
| Outcome ledger | `spx/{capability}/{feature}/{story}/outcomes.yaml` |

**Exclude**: Unrelated files, experimental code, files from other work items.

**Commit Message Context**

Include work item reference in footer:

```text
feat({scope}): implement {story-slug}

- {brief description of what was implemented}
- Tests co-located in spx/{path}/tests/

Refs: {capability}/{feature}/{story}
```

**Return APPROVED**

After successful commit:

```markdown
## Verdict: APPROVED

Commit: {commit_hash}
Files committed: {count}
Outcome committed: spx/{capability}/{feature}/{story}/outcomes.yaml

Work item is complete.
```

---

## Rejection Feedback

When verdict is **REJECTED** or **CONDITIONAL**, provide actionable feedback to the coder:

```markdown
## Review: {target}

### Verdict: REJECTED

### Issues Found

| # | File:Line   | Category   | Issue               | Suggested Fix            |
| - | ----------- | ---------- | ------------------- | ------------------------ |
| 1 | `foo.py:42` | Type Error | Missing return type | Add `-> int`             |
| 2 | `bar.py:17` | Security   | Bare except         | Catch specific exception |

### Tool Outputs

{Include relevant Mypy/Ruff/Semgrep output}

### Required Actions

1. Fix all blocking issues listed above
2. Run verification tools before resubmitting
3. Submit for re-review

### No Outcome Committed

Outcomes are only committed on APPROVED. Fix issues and resubmit.
```

---

## Verdict Levels

Use one of four verdicts:

| Verdict         | When to Use                                                                      | Next Step                                  |
| --------------- | -------------------------------------------------------------------------------- | ------------------------------------------ |
| **APPROVED**    | All checks pass, no issues found                                                 | Commit outcome, commit, work item complete |
| **CONDITIONAL** | Only false-positive violations that require `# noqa` comments with justification | Coder adds noqa comments, then re-review   |
| **REJECTED**    | Real bugs, security issues, test failures, or design problems                    | Coder fixes issues, then re-review         |
| **BLOCKED**     | Infrastructure cannot be provisioned; review environment issue, not code defect  | Fix environment, then re-run review        |

### APPROVED Criteria

All of these must be true:

1. Mypy reports zero errors
2. Ruff reports zero blocking violations (B, S rules)
3. Semgrep reports zero findings
4. All tests pass
5. Manual review checklist is satisfied
6. No security concerns identified

### CONDITIONAL Criteria

Use CONDITIONAL when:

- Tool violations are **false positives** in context (e.g., S603 in a CLI tool)
- The fix is mechanical: add `# noqa: XXXX - [justification]`
- No actual security, correctness, or design issues exist
- The coder can apply fixes without architectural changes

**Required in report**: For each CONDITIONAL issue, specify the exact noqa comment to add.

### REJECTED Criteria

The code is **REJECTED** if ANY of these are true:

| Criterion                                    | Tool/Check     |
| -------------------------------------------- | -------------- |
| Any real type error                          | Mypy           |
| Any true-positive security violation         | Ruff S rules   |
| Any true-positive bug pattern                | Ruff B rules   |
| Any true-positive security finding           | Semgrep        |
| Any test failure                             | pytest         |
| Missing type annotations on public functions | Manual         |
| Missing `-> None` on test functions          | Ruff ANN201    |
| Missing type on fixture params (`tmp_path`)  | Ruff ANN001    |
| Missing `-> None` on `__init__`              | Ruff ANN204    |
| Magic values in test assertions              | Ruff PLR2004   |
| Uppercase argument names                     | Ruff N803      |
| Bare `except:` clauses                       | Manual/Semgrep |
| Hardcoded secrets detected                   | Manual/Semgrep |
| `eval()` or `exec()` without justification   | Manual/Semgrep |
| `shell=True` with untrusted input            | Manual/Semgrep |
| Deep relative imports (2+ levels) to infra   | grep/Manual    |
| `sys.path` manipulation to fix import errors | grep/Manual    |
| Design or architectural problems             | Manual         |

### BLOCKED Criteria

Use BLOCKED when infrastructure provisioning fails:

1. Required VM (Colima) cannot be started
2. Required Docker services cannot be started
3. Required external dependencies are not installed
4. Network/connectivity issues prevent infrastructure access

**BLOCKED is not a code judgment.** It means the review cannot complete due to environment issues. The code may be perfect or terrible — we cannot know until infrastructure is available.

**Required in report**: Document what infrastructure was needed, what was attempted, and why it failed.

---

## False Positive Handling

Not all tool violations are real issues. Context matters. Use this framework to identify and handle false positives.

### When a Violation is a False Positive

A violation is a **false positive** when:

1. **Context changes the threat model**:
   - S603 (subprocess call) in a CLI tool where inputs come from the user invoking the tool, not untrusted external sources
   - S607 (partial executable path) when PATH resolution is intentional for portability across systems

2. **The code is intentionally doing something the rule warns against**:
   - Using `pickle` for internal caching with no untrusted input
   - Using `shell=True` with hardcoded commands (no interpolation)

3. **The rule doesn't apply to this language version or framework**:
   - Python 3.10+ syntax flagged by older tool versions

### When a Violation is NOT a False Positive

A violation is **real** when:

- User/external input can reach the flagged code path
- The code runs in a web service, API, or multi-tenant environment
- The "justification" is just "we've always done it this way"
- You cannot explain exactly why it's safe in this specific context

### Required Noqa Format

When suppressing a rule, the noqa comment MUST include justification:

```python
# GOOD - explains why it's safe
result = subprocess.run(cmd)  # noqa: S603 - CLI tool, cmd built from trusted config

# BAD - no justification
result = subprocess.run(cmd)  # noqa: S603
```

### Application Context Guide

| Application Type        | Trust Boundary        | S603/S607              | Hardcoded Paths   |
| ----------------------- | --------------------- | ---------------------- | ----------------- |
| CLI tool (user-invoked) | User is trusted       | Usually false positive | Often intentional |
| Web service             | All input untrusted   | Real issue             | Real issue        |
| Internal script         | Depends on deployment | Analyze case-by-case   | Usually OK        |
| Library/package         | Consumers untrusted   | Real issue             | Avoid             |

---

## Output Format

You must produce TWO outputs:

1. **File**: Write detailed report to `reports/review_{target_name}_{YYYYMMDD_HHMMSS}.md`
2. **Conversation**: Provide summary in the chat for immediate visibility

### Report File Structure

Use the template in `templates/review_report.md`. The file must include:

- Complete tool outputs (Mypy, Ruff, Semgrep, pytest)
- Coverage report with percentages
- Full manual review checklist with findings
- All issues with file:line references and suggested fixes

### Conversation Summary Structure

````markdown
## Review: {target}

### Verdict: [APPROVED / CONDITIONAL / REJECTED]

[One-sentence summary explaining the verdict]

### Static Analysis

| Tool    | Status    | Issues                             |
| ------- | --------- | ---------------------------------- |
| Mypy    | PASS/FAIL | [count] errors                     |
| Ruff    | PASS/FAIL | [count] blocking, [count] warnings |
| Semgrep | PASS/FAIL | [count] findings                   |

### Tests

| Metric   | Value      |
| -------- | ---------- |
| Passed   | [count]    |
| Failed   | [count]    |
| Coverage | [percent]% |

### Blocking Issues (if REJECTED)

1. **[file:line]** - [category] - [description]
   ```python
   # Suggested fix
   ```
````

### Conditional Issues (if CONDITIONAL)

1. **[file:line]** - Add: `# noqa: XXXX - [justification]`

### Warnings

1. **[file:line]** - [category] - [description]

### Report Location

Full report: `reports/review_{name}_{timestamp}.md`

### Next Action (for coder)

→ **APPROVED**: Reviewer has committed. Coder runs `spx spec status` to check for more items.
→ **REJECTED**: Coder remediates issues using feedback above, re-invokes `/reviewing-python`.
→ **CONDITIONAL**: Coder adds noqa comments per instructions, re-invokes `/reviewing-python`.
→ **BLOCKED**: Coder returns `BLOCKED` to orchestrator.

```
**Note to coder**: This review is complete. Handle the verdict per the Review Loop Protocol.

---

## Important Notes

1. **Do NOT attempt to fix code during review**. Your role is to identify and report issues, not remediate them. The Coder handles fixes.

2. **Be specific**. Vague feedback like "improve error handling" is useless. Say exactly what is wrong and where.

3. **Include file:line references**. Every issue must be traceable to a specific location.

4. **Explain the "why"**. Don't just say "this is wrong." Explain the risk or consequence.

5. **Check the spec**. If a design document or specification exists, verify the implementation matches it.

6. **Don't trust the happy path**. Look for edge cases, error conditions, and unexpected inputs.

7. **Security is paramount**. When in doubt, flag it as a security concern.

---

## Skill Resources

- `rules/mypy_strict.toml` - Strict Mypy configuration
- `rules/ruff_quality.toml` - Ruff linting with security rules
- `rules/semgrep_sec.yaml` - Custom security pattern rules
- `templates/review_report.md` - Report template

---

*Remember: Your job is to protect the codebase from defects. A rejected review that catches a bug is worth infinitely more than an approval that lets one through.*
```
