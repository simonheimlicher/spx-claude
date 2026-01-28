---
name: testing-python
description: Write tests for Python code with three levels (Unit/Integration/E2E). Use when testing Python or writing Python tests.
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

# Python Testing Patterns

> **PREREQUISITE:** Read the foundational `/testing` skill first. This skill provides Python-specific implementations.

## Foundational Stance

> **MAXIMUM CONFIDENCE. MINIMUM DEPENDENCIES. NO MOCKING. REALITY IS THE ORACLE.**

- Every dependency you add must **justify itself** with confidence gained
- Mocking external services is a **confession** that your code is poorly designed
- Reality is the only oracle that matters

---

## Python Tooling

| Level          | Infrastructure                                       | Speed  |
| -------------- | ---------------------------------------------------- | ------ |
| 1: Unit        | Python stdlib + Git + standard tools + temp fixtures | <100ms |
| 2: Integration | Project-specific binaries/tools (Docker, ZFS, etc.)  | <1s    |
| 3: E2E         | Network services + external APIs + test accounts     | <10s   |

**Standard dev tools** are available in CI without installation (git, cat, grep, curl, sed, awk, etc.).
**Project-specific tools** require installation/setup (make, pip, Docker, ZFS, Hugo, etc.).

---

## The Mocking Prohibition

**THESE VIOLATIONS WILL CAUSE YOUR CODE TO BE REJECTED:**

❌ `unittest.mock.patch` for external services (GitHub, Stripe, Dropbox, etc.)
❌ `@patch("httpx.Client")` for HTTP boundaries
❌ `respx.mock` for internet APIs

### Allowed "Mocking" (Level 1 ONLY)

These control YOUR code's environment, not external services:

✅ `patch("time.time")` for deterministic timestamps
✅ `patch("secrets.token_hex")` for predictable IDs
✅ `patch("os.getenv")` for config injection
✅ Git operations via subprocess (standard dev tool, always available)

---

## Level Decision Tree

```
What am I testing?
│
├─ Pure logic / data transformation
│  └─ Level 1: DI with fake implementations
│
├─ External service interaction
│  │
│  ├─ Can service run locally? (Docker/VM)
│  │  ├─ YES → Level 2: Real service in Docker
│  │  └─ NO  → Level 3: Real service on internet
│  │
│  └─ Thinking of mocking?
│     └─ STOP. Redesign with DI (Level 1) or use real service (Level 2/3)
```

**SaaS APIs (GitHub, Stripe, Trakt, OpenAI):** Level 2 does NOT exist. Use Level 1 (pure DI) + Level 3 (real service).

**Note:** Git is a standard dev tool (Level 1), while GitHub API is a network service (Level 3). Don't confuse local Git operations with GitHub API calls.

---

## Level 1: Unit Patterns

### Dependency Injection

```python
from dataclasses import dataclass
from typing import Callable, Protocol


class CommandRunner(Protocol):
    def run(self, cmd: list[str]) -> tuple[int, str, str]: ...


@dataclass
class SyncDependencies:
    run_command: CommandRunner
    get_env: Callable[[str], str | None] = os.environ.get


def sync_to_remote(source: str, dest: str, deps: SyncDependencies) -> SyncResult:
    cmd = build_command(source, dest)
    returncode, stdout, stderr = deps.run_command.run(cmd)
    return SyncResult(success=returncode == 0)


# Test with controlled implementation
def test_sync_returns_success_on_zero_exit():
    class FakeRunner:
        def run(self, cmd: list[str]) -> tuple[int, str, str]:
            return (0, "Transferred: 5 files", "")

    deps = SyncDependencies(run_command=FakeRunner())
    result = sync_to_remote("/src", "remote:dest", deps)
    assert result.success is True
```

### Pure Function Testing

```python
def test_command_includes_checksum_flag():
    cmd = build_rclone_command("/source", "remote:dest", checksum=True)
    assert "--checksum" in cmd


def test_unicode_paths_preserved():
    cmd = build_rclone_command("/tank/фото", "remote:резервная")
    assert "/tank/фото" in cmd
```

### Data Factories

```python
from dataclasses import dataclass, field
from typing import Iterator
import itertools

_id_counter: Iterator[int] = itertools.count(1)


@dataclass
class AuditResultFactory:
    url: str = field(default_factory=lambda: f"https://example.com/{next(_id_counter)}")
    performance: int = 90

    def build(self) -> dict:
        return {"url": self.url, "scores": {"performance": self.performance}}


def test_fails_on_low_performance():
    result = AuditResultFactory(performance=45).build()
    analysis = analyze_results([result], deps)
    assert analysis.passed is False
```

**See**: `levels/level-1-unit.md`

---

## Level 2: Integration Patterns

### Test Harness: Docker

```python
@dataclass
class PostgresHarness:
    container_name: str = "test-postgres"
    port: int = 5432

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
                "POSTGRES_PASSWORD=test",
                "postgres:15",
            ],
            check=True,
        )
        self._wait_for_ready()

    def stop(self) -> None:
        subprocess.run(["docker", "rm", "-f", self.container_name])

    @property
    def connection_string(self) -> str:
        return f"postgresql://postgres:test@localhost:{self.port}/postgres"
```

### Using Harnesses with pytest

```python
@pytest.fixture(scope="module")
def database():
    harness = PostgresHarness()
    harness.start()
    yield harness
    harness.stop()


@pytest.fixture(autouse=True)
def reset_database(database):
    yield
    database.reset()


@pytest.mark.integration
def test_user_repository_saves_and_retrieves(database):
    repo = UserRepository(database.connection_string)
    user = User(email="test@example.com", name="Test User")
    repo.save(user)

    retrieved = repo.find_by_email("test@example.com")
    assert retrieved.name == "Test User"
```

**See**: `levels/level-2-integration.md`

---

## Level 3: E2E Patterns

### Credential Management

```python
CREDENTIALS_DOC = """
Level 3 tests require these environment variables:

Required:
  DROPBOX_TEST_TOKEN    - From 1Password: "Engineering/Test Credentials"

Setup:
  cp .env.test.example .env.test
  # Fill in values from 1Password
"""


def load_credentials() -> dict | None:
    token = os.environ.get("DROPBOX_TEST_TOKEN")
    if not token:
        return None
    return {"token": token}


@pytest.fixture
def dropbox_config(tmp_path) -> Path | None:
    creds = load_credentials()
    if not creds:
        pytest.skip("DROPBOX_TEST_TOKEN not set")

    config = tmp_path / "rclone.conf"
    config.write_text(f"""
[dropbox-test]
type = dropbox
token = {creds["token"]}
""")
    return config
```

### Skip If No Credentials

```python
credentials = load_credentials()
skip_no_creds = pytest.mark.skipif(
    credentials is None, reason="E2E credentials not configured"
)


@skip_no_creds
@pytest.mark.e2e
def test_full_sync_workflow(dropbox_test_folder, dropbox_config):
    result = sync_to_dropbox(local_path, dropbox_test_folder, config=dropbox_config)
    assert result.success
```

**See**: `levels/level-3-e2e.md`

---

## pytest Configuration

```python
# conftest.py
def pytest_configure(config):
    config.addinivalue_line("markers", "integration: requires test harness")
    config.addinivalue_line("markers", "e2e: requires credentials")


# pyproject.toml
[tool.pytest.ini_options]
markers = [
    "integration: requires test harness (Docker, real binaries)",
    "e2e: requires credentials and external services",
]
testpaths = ["tests"]

# Run by level:
# pytest -m "not integration and not e2e"  # Level 1 only
# pytest -m "integration"                   # Level 2
# pytest -m "e2e"                           # Level 3
```

---

## Anti-Patterns

### Mock Everything

```python
# ❌ Mocking destroys confidence
@patch("subprocess.run")
@patch("os.path.exists")
def test_sync(mock_exists, mock_run):
    mock_exists.return_value = True
    mock_run.return_value = Mock(returncode=0)
    result = sync_files(src, dest)
    assert result.success  # What did we prove? NOTHING.
```

### Skip Levels

```python
# ❌ Jumping to Level 3 without Level 1/2 coverage
@pytest.mark.e2e
def test_sync_to_dropbox():
    sync_to_dropbox(local_path, "dropbox:backup")
    # If this fails, we don't know if it's our code or Dropbox
```

### Test Implementation Details

```python
# ❌ Testing HOW, not WHAT
def test_uses_rclone_sync_command():
    with capture_subprocess() as captured:
        sync_dataset(src, dest)
    assert "rclone sync" in captured.command  # Implementation detail!


# ✅ Test the observable behavior instead
def test_files_synced():
    sync_dataset(src, dest)
    assert (dest / "file.txt").exists()
```

---

## Quick Reference

| Pattern      | Level 1                        | Level 2            | Level 3              |
| ------------ | ------------------------------ | ------------------ | -------------------- |
| Dependencies | Injected callables/dataclasses | Real via harness   | Real via credentials |
| Data         | Factories + tmp_path           | Fixtures + harness | Test accounts        |
| Speed        | <100ms                         | <1s                | <10s                 |
| CI           | Every commit                   | Every commit       | Nightly/pre-release  |

---

*For foundational principles (progress vs regression tests, escalation justification), see `/testing`.*

---

## Test Infrastructure Paths

Test infrastructure lives alongside tests:

```
tests/
├── harness/                 # Active code for tests
│   ├── __init__.py
│   ├── context.py           # Test environment context manager (withTestEnv)
│   ├── postgres.py          # PostgreSQL harness
│   ├── docker.py            # Generic Docker harness
│   └── factories.py         # Seeded data factories
├── fixtures/                # Static test data
│   ├── sample-config.json
│   └── values.py            # TYPICAL, EDGES collections
├── unit/
│   └── {capability}/
│       └── {feature}/
├── integration/
│   └── {capability}/
│       └── {feature}/
└── e2e/
    └── {capability}/
        └── {feature}/
```

**harness/** = Code that runs (context managers, harnesses, factories)
**fixtures/** = Data that's read (JSON files, sample configs, test values)
