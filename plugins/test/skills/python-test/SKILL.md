---
name: python-test
description: "Python-specific testing patterns. REQUIRES reading /test first for foundational principles."
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

# Python Testing Patterns

> **PREREQUISITE:** Read the foundational `/test` skill first. This skill only provides Python-specific patterns.

This skill provides Python-specific implementations of the three-tier testing methodology defined in `/test`.

---

## Python Tooling

| Level | Tools | Speed |
| --- | --- | --- |
| 1: Unit | pytest, temp directories, dependency injection | <100ms |
| 2: Integration | pytest + Docker/Colima, real binaries, test harnesses | <1s |
| 3: E2E | pytest + real services, test accounts, credentials | <30s |

---

## Level 1: Unit Patterns (Python)

### Dependency Injection

```python
# ❌ BAD: Hardcoded dependency requiring mocks
def sync_to_remote(source: Path, dest: str) -> None:
    subprocess.run(["rclone", "sync", str(source), dest])

# ✅ GOOD: Dependencies as parameters
@dataclass
class SyncDeps:
    run_command: Callable[[list[str]], subprocess.CompletedProcess]

def sync_to_remote(source: Path, dest: str, deps: SyncDeps) -> SyncResult:
    result = deps.run_command(["rclone", "sync", str(source), dest])
    return SyncResult(success=result.returncode == 0)

# Test with controlled implementation
def test_sync_returns_success_on_zero_exit():
    deps = SyncDeps(
        run_command=lambda cmd: subprocess.CompletedProcess(cmd, returncode=0)
    )
    result = sync_to_remote(Path("/src"), "remote:dest", deps)
    assert result.success is True
```

### Pure Function Testing

```python
def test_build_rclone_command():
    cmd = build_rclone_command(
        source="/tank/photos",
        dest="remote:backup",
        checksum=True
    )
    assert cmd == ["rclone", "sync", "/tank/photos", "remote:backup", "--checksum"]

def test_parse_config_rejects_missing_required():
    result = parse_config({"base_url": "http://localhost"})
    assert result.ok is False
    assert "site_dir" in result.errors[0]
```

### Temporary Directories

```python
def test_config_file_generation(tmp_path: Path):
    config_path = tmp_path / "config.yaml"

    generate_config(config_path, {"debug": True})

    content = config_path.read_text()
    assert "debug: true" in content
```

### Data Factories

```python
from dataclasses import dataclass, field
from typing import Iterator
import itertools

_id_counter: Iterator[int] = itertools.count(1)

@dataclass
class AuditResultFactory:
    url: str = field(default_factory=lambda: f"https://example.com/page-{next(_id_counter)}")
    performance: int = 90
    accessibility: int = 100

    def build(self) -> dict:
        return {
            "url": self.url,
            "scores": {
                "performance": self.performance,
                "accessibility": self.accessibility,
            }
        }

def create_audit_result(**kwargs) -> dict:
    return AuditResultFactory(**kwargs).build()

# Usage
def test_fails_on_low_performance():
    result = create_audit_result(performance=45)
    analysis = analyze_results([result], deps)
    assert analysis.passed is False
```

---

## Level 2: Integration Patterns (Python)

### Test Harness: Docker/Colima

```python
# test/harnesses/postgres.py
from dataclasses import dataclass
import subprocess

@dataclass
class PostgresHarness:
    container_name: str = "test-postgres"
    port: int = 5432

    def start(self) -> None:
        subprocess.run([
            "docker", "run", "-d",
            "--name", self.container_name,
            "-p", f"{self.port}:5432",
            "-e", "POSTGRES_PASSWORD=test",
            "postgres:15"
        ], check=True)
        self._wait_for_ready()

    def stop(self) -> None:
        subprocess.run(["docker", "rm", "-f", self.container_name])

    def reset(self) -> None:
        # Truncate all tables or recreate schema
        pass

    @property
    def connection_string(self) -> str:
        return f"postgresql://postgres:test@localhost:{self.port}/postgres"
```

### Test Harness: Real Binary

```python
# test/harnesses/hugo.py
import tempfile
import subprocess
from pathlib import Path
from dataclasses import dataclass

@dataclass
class HugoHarness:
    site_dir: Path
    output_dir: Path
    _temp_dir: tempfile.TemporaryDirectory | None = None

    def build(self, args: list[str] | None = None) -> subprocess.CompletedProcess:
        args = args or []
        return subprocess.run(
            ["hugo", "--source", str(self.site_dir),
             "--destination", str(self.output_dir)] + args,
            capture_output=True,
            text=True
        )

    def cleanup(self) -> None:
        if self._temp_dir:
            self._temp_dir.cleanup()

def create_hugo_harness() -> HugoHarness:
    temp_dir = tempfile.TemporaryDirectory(prefix="hugo-test-")
    site_dir = Path(temp_dir.name)
    _create_minimal_site(site_dir)
    return HugoHarness(
        site_dir=site_dir,
        output_dir=site_dir / "public",
        _temp_dir=temp_dir
    )
```

### Using Harnesses with pytest

```python
import pytest
from harnesses.postgres import PostgresHarness

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

def test_user_repository_saves_and_retrieves(database):
    repo = UserRepository(database.connection_string)

    user = User(email="test@example.com", name="Test User")
    repo.save(user)

    retrieved = repo.find_by_email("test@example.com")
    assert retrieved.name == "Test User"
```

---

## Level 3: E2E Patterns (Python)

### Credential Management

```python
# test/e2e/credentials.py
import os

CREDENTIALS_DOC = """
Level 3 tests require these environment variables:

Required:
  DROPBOX_TEST_TOKEN    - From 1Password: "Engineering/Test Credentials"
  TEST_USER_EMAIL       - test-automation@example.com

Setup:
  cp .env.test.example .env.test
  # Fill in values from 1Password
"""

def load_credentials() -> dict | None:
    token = os.environ.get("DROPBOX_TEST_TOKEN")
    email = os.environ.get("TEST_USER_EMAIL")

    if not token or not email:
        return None

    return {"token": token, "email": email}

def require_credentials() -> dict:
    creds = load_credentials()
    if not creds:
        raise RuntimeError(f"Missing credentials.\n{CREDENTIALS_DOC}")
    return creds
```

### Skip If No Credentials

```python
import pytest
from credentials import load_credentials

credentials = load_credentials()
skip_no_creds = pytest.mark.skipif(
    credentials is None,
    reason="E2E credentials not configured"
)

@skip_no_creds
def test_full_sync_workflow():
    result = sync_to_dropbox(local_path, "dropbox:backup")
    assert result.success
    assert result.files_synced > 0
```

### CLI E2E Testing

```python
import subprocess
import os

@skip_no_creds
def test_cli_full_workflow():
    result = subprocess.run(
        ["python", "-m", "myapp", "sync",
         "--source", "/test/data",
         "--dest", "dropbox:backup"],
        capture_output=True,
        text=True,
        env={
            **os.environ,
            "DROPBOX_TOKEN": credentials["token"],
        }
    )

    assert result.returncode == 0
    assert "Sync complete" in result.stdout
```

---

## pytest Configuration

```python
# conftest.py
import pytest

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

## Quick Reference

| Pattern | Level 1 | Level 2 | Level 3 |
| --- | --- | --- | --- |
| Dependencies | Injected callables/dataclasses | Real via harness | Real via credentials |
| Data | Factories + tmp_path | Fixtures + harness | Test accounts |
| Speed | <100ms | <1s | <30s |
| CI | Every commit | Every commit | Nightly/pre-release |

---

_For foundational principles (no mocking, progress vs regression tests, escalation justification), see `/test`._
