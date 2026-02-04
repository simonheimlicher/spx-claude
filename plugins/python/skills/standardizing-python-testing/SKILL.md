---
name: standardizing-python-testing
description: Python testing standards enforced across all skills. Reference skill for property-based testing, factories, markers, and harness patterns.
allowed-tools: Read
---

<objective>
Define Python testing standards that other skills reference. Not invoked directly—invoke `/testing-python`, `/reviewing-python-tests`, or `/coding-python` instead. These standards apply to ALL Python test code.
</objective>

<quick_start>
Reference this skill for standards on:

- Property-based testing with Hypothesis (MANDATORY for parsers, serializers, math, complex algorithms)
- Data factories with dataclasses (NO magic values)
- Test markers (`@pytest.mark.level_N`) - see `/testing` for level definitions
- File naming (`.level_1.py`, `.level_2.py`, `.level_3.py`)
- DI patterns with Protocols
- Harness patterns for Level 2 tests
- Test function signatures (`-> None`, type annotations)
- Credential management (fail loudly, not skip silently)

</quick_start>

<property_based_testing>
Property-based testing is **MANDATORY** for these code types:

| Code Type               | Property to Test         | Example                      |
| ----------------------- | ------------------------ | ---------------------------- |
| Parsers                 | `parse(format(x)) == x`  | JSON, YAML, CLI args         |
| Mathematical operations | Algebraic properties     | Commutativity, associativity |
| Serialization           | `decode(encode(x)) == x` | Protocol buffers, msgpack    |
| Complex algorithms      | Invariant preservation   | Sorting, tree operations     |

<hypothesis_configuration>

```python
from hypothesis import given, settings, strategies as st


# ✅ REQUIRED: All property tests use @given decorator
@given(st.text())
def test_round_trips_through_encoding(text: str) -> None:
    encoded = encode(text)
    decoded = decode(encoded)
    assert decoded == text


# ✅ REQUIRED: Settings for slow generators
@settings(max_examples=500, deadline=None)
@given(st.binary(min_size=1, max_size=10_000))
def test_handles_arbitrary_binary(data: bytes) -> None:
    result = process(data)
    assert result.valid or result.error is not None


# ✅ REQUIRED: Composite strategies for complex data
@st.composite
def valid_orders(draw: st.DrawFn) -> Order:
    items = draw(st.lists(st.builds(OrderItem), min_size=1, max_size=10))
    return Order(items=items, total=sum(item.price for item in items))


@given(valid_orders())
def test_order_validation_accepts_valid(order: Order) -> None:
    result = validate_order(order)
    assert result.ok is True
```

</hypothesis_configuration>

<common_properties>

```python
# ✅ Idempotency: f(f(x)) == f(x)
@given(st.text())
def test_normalization_is_idempotent(text: str) -> None:
    once = normalize(text)
    twice = normalize(once)
    assert once == twice


# ✅ Commutativity: f(a, b) == f(b, a)
@given(st.integers(), st.integers())
def test_merge_is_commutative(a: int, b: int) -> None:
    assert merge(a, b) == merge(b, a)


# ✅ Invariant preservation: property holds before and after
@given(st.lists(st.integers()))
def test_sort_preserves_elements(items: list[int]) -> None:
    sorted_items = sort(items)
    assert sorted(items) == sorted(sorted_items)
    assert len(items) == len(sorted_items)
```

</common_properties>

**If testing a parser or serializer without property-based tests, the tests are INCOMPLETE.**

</property_based_testing>

<data_factories>
Test data MUST use factories with named constants. Never use arbitrary literals.

<dataclass_factory_pattern>

```python
from dataclasses import dataclass, field
from typing import Iterator
import itertools

# Module-level counter for unique IDs
_id_counter: Iterator[int] = itertools.count(1)

# Named constants at module level
DEFAULT_PERFORMANCE_SCORE = 90
DEFAULT_ACCESSIBILITY_SCORE = 100
FAILING_PERFORMANCE_THRESHOLD = 45


@dataclass
class AuditResultFactory:
    """Factory for creating test audit results."""

    url: str = field(default_factory=lambda: f"https://example.com/{next(_id_counter)}")
    performance: int = DEFAULT_PERFORMANCE_SCORE
    accessibility: int = DEFAULT_ACCESSIBILITY_SCORE

    def build(self) -> dict:
        return {
            "url": self.url,
            "scores": {
                "performance": self.performance,
                "accessibility": self.accessibility,
            },
        }


# ✅ CORRECT: Factory with named constant
def test_fails_on_low_performance() -> None:
    result = AuditResultFactory(performance=FAILING_PERFORMANCE_THRESHOLD).build()
    analysis = analyze_results([result])
    assert analysis.passed is False


# ❌ REJECTED: Magic value (PLR2004)
def test_fails_on_low_performance_bad() -> None:
    result = {"scores": {"performance": 45}}  # What is 45?
    analysis = analyze_results([result])
    assert analysis.passed is False
```

</dataclass_factory_pattern>

<builder_pattern>

```python
@dataclass
class UserBuilder:
    """Builder for test users with sensible defaults."""

    email: str = field(default_factory=lambda: f"user{next(_id_counter)}@test.com")
    name: str = "Test User"
    role: str = "member"
    active: bool = True

    def with_admin_role(self) -> "UserBuilder":
        self.role = "admin"
        return self

    def inactive(self) -> "UserBuilder":
        self.active = False
        return self

    def build(self) -> User:
        return User(
            email=self.email, name=self.name, role=self.role, active=self.active
        )


# ✅ CORRECT: Builder with fluent interface
def test_admin_can_delete_users() -> None:
    admin = UserBuilder().with_admin_role().build()
    target = UserBuilder().build()
    result = delete_user(admin, target)
    assert result.success is True
```

</builder_pattern>

</data_factories>

<test_markers>
All tests MUST be marked with their level using pytest markers. See `/testing` for level definitions.

<marker_usage>

```python
import pytest


# ✅ REQUIRED: Mark every test with its level
@pytest.mark.level_1
def test_validates_empty_order() -> None: ...


@pytest.mark.level_2
def test_saves_to_database(database: PostgresHarness) -> None: ...


@pytest.mark.level_3
def test_full_checkout_flow(credentials: dict) -> None: ...
```

</marker_usage>

<pytest_configuration>

```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["spx"]
python_files = ["test_*.py"]
markers = [
  "level_1: See /testing for level definitions",
  "level_2: See /testing for level definitions",
  "level_3: See /testing for level definitions",
]

# Run by level:
# pytest -m level_1    # Level 1 only
# pytest -m level_2    # Level 2 only
# pytest -m level_3    # Level 3 only
# pytest               # All tests
```

</pytest_configuration>

</test_markers>

<file_naming>
Test level is indicated by filename suffix:

| Level | Suffix             | Example                                 |
| ----- | ------------------ | --------------------------------------- |
| 1     | `.level_1.py`      | `test_validation.level_1.py`            |
| 2     | `.level_2.py`      | `test_database.level_2.py`              |
| 3     | `.level_3.py`      | `test_checkout.level_3.py`              |
| 3     | `.level_3.spec.py` | `checkout.level_3.spec.py` (Playwright) |

<directory_structure>

```
spx/
└── {capability}/
    └── {feature}/
        ├── {feature}.md
        └── tests/
            ├── test_{name}.level_1.py
            ├── test_{name}.level_2.py
            └── test_{name}.level_3.py
```

</directory_structure>

</file_naming>

<dependency_injection>
When Stage 3 of `/testing` determines DI is appropriate, use Protocols.

<protocol_definition>

```python
from typing import Protocol


class CommandRunner(Protocol):
    """Protocol for running shell commands."""

    def run(self, cmd: list[str]) -> tuple[int, str, str]:
        """Run command, return (exit_code, stdout, stderr)."""
        ...


class HttpClient(Protocol):
    """Protocol for HTTP operations."""

    def fetch(self, url: str) -> dict:
        """Fetch URL, return response as dict."""
        ...
```

</protocol_definition>

<dataclass_dependencies>

```python
from dataclasses import dataclass
from typing import Callable
import os


@dataclass
class SyncDependencies:
    """Dependencies for sync operation, injectable for testing."""

    run_command: CommandRunner
    get_env: Callable[[str], str | None] = os.environ.get


def sync_to_remote(source: str, dest: str, deps: SyncDependencies) -> SyncResult:
    """Sync files to remote, using injected dependencies."""
    cmd = build_command(source, dest)
    returncode, stdout, stderr = deps.run_command.run(cmd)
    return SyncResult(success=returncode == 0, output=stdout)
```

</dataclass_dependencies>

<test_double_implementation>

```python
# ✅ CORRECT: Test double via DI (Exception 1: Failure modes)
def test_handles_command_failure() -> None:
    class FailingRunner:
        def run(self, cmd: list[str]) -> tuple[int, str, str]:
            return (1, "", "Connection refused")

    deps = SyncDependencies(run_command=FailingRunner())
    result = sync_to_remote("/src", "remote:dest", deps)
    assert result.success is False


# ❌ REJECTED: Mocking via patch
@patch("subprocess.run")
def test_handles_command_failure_bad(mock_run: Mock) -> None:
    mock_run.return_value = Mock(returncode=1)
    result = sync_to_remote("/src", "remote:dest")  # No DI!
    assert result.success is False
```

</test_double_implementation>

</dependency_injection>

<test_harnesses>
Level 2 tests require harnesses for real dependencies.

<docker_harness>

```python
from dataclasses import dataclass
import subprocess


@dataclass
class DockerHarness:
    """Base class for Docker-based test harnesses."""

    container_name: str
    image: str
    port: int

    def start(self) -> None:
        subprocess.run(
            [
                "docker",
                "run",
                "-d",
                "--name",
                self.container_name,
                "-p",
                f"{self.port}:{self.port}",
                self.image,
            ],
            check=True,
        )
        self._wait_for_ready()

    def stop(self) -> None:
        subprocess.run(["docker", "rm", "-f", self.container_name])

    def _wait_for_ready(self, timeout: int = 30) -> None:
        raise NotImplementedError


@dataclass
class PostgresHarness(DockerHarness):
    """PostgreSQL test harness."""

    container_name: str = "test-postgres"
    image: str = "postgres:15"
    port: int = 5432
    password: str = "test"

    def start(self) -> None:
        subprocess.run(
            [
                "docker",
                "run",
                "-d",
                "--name",
                self.container_name,
                "-p",
                f"{self.port}:5432",
                "-e",
                f"POSTGRES_PASSWORD={self.password}",
                self.image,
            ],
            check=True,
        )
        self._wait_for_ready()

    @property
    def connection_string(self) -> str:
        return f"postgresql://postgres:{self.password}@localhost:{self.port}/postgres"
```

</docker_harness>

<harness_fixtures>

```python
import pytest


@pytest.fixture(scope="module")
def database() -> PostgresHarness:
    harness = PostgresHarness()
    harness.start()
    yield harness
    harness.stop()


@pytest.fixture(autouse=True)
def reset_database(database: PostgresHarness) -> None:
    yield
    database.reset()
```

</harness_fixtures>

</test_harnesses>

<test_signatures>
All test functions MUST have complete type annotations.

```python
import pytest
from pathlib import Path


# ✅ REQUIRED: -> None on all test functions
def test_validates_input() -> None:
    result = validate("test")
    assert result.valid


# ✅ REQUIRED: Type annotations on fixture parameters
def test_creates_file(tmp_path: Path) -> None:
    file = tmp_path / "test.txt"
    file.write_text("content")
    assert file.exists()


# ✅ REQUIRED: Return type on fixtures
@pytest.fixture
def config(tmp_path: Path) -> Config:
    return Config(path=tmp_path)


# ❌ REJECTED: Missing -> None (ANN201)
def test_something(self):
    pass


# ❌ REJECTED: Missing parameter type (ANN001)
def test_with_fixture(self, tmp_path) -> None:
    pass
```

</test_signatures>

<credential_management>
E2E tests requiring credentials MUST fail loudly, not skip silently.

```python
CREDENTIALS_DOC = """
Level 3 tests require these environment variables:

Required:
  STRIPE_TEST_KEY    - From 1Password: "Engineering/Test Credentials"

Setup:
  cp .env.test.example .env.test
  # Fill in values from 1Password
"""


def load_credentials() -> dict | None:
    key = os.environ.get("STRIPE_TEST_KEY")
    if not key:
        return None
    return {"key": key}


def require_credentials() -> dict:
    creds = load_credentials()
    if not creds:
        raise RuntimeError(f"Missing required credentials.\n\n{CREDENTIALS_DOC}")
    return creds


# ✅ CORRECT: Fail loudly if credentials required but missing
@pytest.mark.level_3
def test_charges_card() -> None:
    creds = require_credentials()
    client = StripeClient(creds["key"])
    result = client.charge(amount=100)
    assert result.success


# ❌ REJECTED: Silent skip on required credentials
@pytest.mark.skipif(not load_credentials(), reason="No credentials")
@pytest.mark.level_3
def test_charges_card_bad() -> (
    None
): ...  # CI goes green with zero payment verification!
```

</credential_management>

<rejection_criteria>

| Issue                              | Example                                   | Rule/Reason           |
| ---------------------------------- | ----------------------------------------- | --------------------- |
| Missing property tests for parser  | `test_parse_json` without `@given`        | Mandatory for parsers |
| Magic values in assertions         | `assert score == 45`                      | PLR2004               |
| Missing `-> None` on test          | `def test_foo(self):`                     | ANN201                |
| Missing marker                     | No `@pytest.mark.level_N`                 | Level not indicated   |
| Mocking                            | `@patch("module.func")`                   | Use DI instead        |
| Silent skip on required dependency | `@pytest.mark.skipif(not has_postgres())` | Must fail, not skip   |
| Wrong filename suffix              | `test_db.py` for Level 2 test             | Use `.level_2.py`     |
| Untyped fixture parameter          | `def test_foo(tmp_path) -> None:`         | ANN001                |

</rejection_criteria>

<success_criteria>
Code follows these standards when:

- [ ] Parsers, serializers, math operations have property-based tests with `@given`
- [ ] Test data uses factories with named constants (no magic values)
- [ ] All tests marked with `@pytest.mark.level_N`
- [ ] File names indicate level (`.level_1.py`, `.level_2.py`, `.level_3.py`)
- [ ] DI uses Protocols, not mocking
- [ ] Level 2 tests use harnesses with real dependencies
- [ ] All test functions have `-> None` return type
- [ ] All fixture parameters have type annotations
- [ ] Required credentials fail loudly, not skip silently

</success_criteria>
