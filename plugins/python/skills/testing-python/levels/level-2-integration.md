# Level 2: Integration Tests

**Speed**: <1s | **Infrastructure**: Docker/Colima/local binaries | **Framework**: pytest

Level 2 tests verify that real tools work together in a controlled local environment.

## What Level 2 Provides

| Component  | What's Real             | What's Controlled       |
| ---------- | ----------------------- | ----------------------- |
| Database   | Real Postgres in Docker | Ephemeral test database |
| Binaries   | Real rclone/hugo binary | Local backend           |
| Filesystem | Real Linux filesystem   | Temp directories        |

## Why Level 2 Exists

Level 1 proves our logic is correct. Level 2 proves:

- The real binary accepts our commands
- Database queries actually work
- File permissions work as expected

---

## Test Harness Pattern

### Docker Harness

```python
from dataclasses import dataclass
import subprocess


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

    def _wait_for_ready(self, timeout: int = 30) -> None:
        import time

        deadline = time.time() + timeout
        while time.time() < deadline:
            result = subprocess.run(
                ["pg_isready", "-h", "localhost", "-p", str(self.port)],
                capture_output=True,
            )
            if result.returncode == 0:
                return
            time.sleep(0.5)
        raise TimeoutError("Postgres not ready")

    def stop(self) -> None:
        subprocess.run(["docker", "rm", "-f", self.container_name])

    def reset(self) -> None:
        # Truncate all tables or recreate schema
        pass

    @property
    def connection_string(self) -> str:
        return f"postgresql://postgres:test@localhost:{self.port}/postgres"
```

### Real Binary Harness

```python
@dataclass
class HugoHarness:
    site_dir: Path
    output_dir: Path

    def build(self, args: list[str] | None = None) -> subprocess.CompletedProcess:
        args = args or []
        return subprocess.run(
            [
                "hugo",
                "--source",
                str(self.site_dir),
                "--destination",
                str(self.output_dir),
            ]
            + args,
            capture_output=True,
            text=True,
        )

    def cleanup(self) -> None:
        import shutil

        shutil.rmtree(self.output_dir, ignore_errors=True)
```

---

## Using Harnesses with pytest

```python
import pytest


@pytest.fixture(scope="module")
def database():
    """Start database once per module."""
    harness = PostgresHarness()
    harness.start()
    yield harness
    harness.stop()


@pytest.fixture(autouse=True)
def reset_database(database):
    """Reset database state between tests."""
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

---

## Good Patterns

### Testing Real Binary Operations

```python
@pytest.mark.integration
class TestHugoBuild:
    def test_build_creates_output(self, hugo_harness):
        result = hugo_harness.build()

        assert result.returncode == 0
        assert (hugo_harness.output_dir / "index.html").exists()

    def test_build_with_minify(self, hugo_harness):
        result = hugo_harness.build(["--minify"])

        assert result.returncode == 0
```

### Testing Real Sync Operations

```python
@pytest.mark.integration
class TestRcloneSync:
    def test_sync_creates_files_at_destination(
        self, source_dir, test_remote, rclone_config
    ):
        (source_dir / "photo.jpg").write_bytes(b"fake image")

        result = sync_dataset(str(source_dir), test_remote, config=rclone_config)

        assert result.success
        remote_path = Path(test_remote.split(":")[1])
        assert (remote_path / "photo.jpg").exists()

    def test_sync_preserves_directory_structure(
        self, source_dir, test_remote, rclone_config
    ):
        nested = source_dir / "a" / "b" / "c"
        nested.mkdir(parents=True)
        (nested / "deep.txt").write_text("deep")

        sync_dataset(str(source_dir), test_remote, config=rclone_config)

        remote_path = Path(test_remote.split(":")[1])
        assert (remote_path / "a" / "b" / "c" / "deep.txt").exists()
```

### Testing Error Conditions

```python
@pytest.mark.integration
class TestDatabaseErrors:
    def test_connection_failure_handled(self):
        # Point to non-existent database
        repo = UserRepository("postgresql://localhost:59999/nonexistent")

        with pytest.raises(ConnectionError) as exc_info:
            repo.find_by_email("test@example.com")

        assert "connection" in str(exc_info.value).lower()

    def test_constraint_violation_raises_specific_error(self, database):
        repo = UserRepository(database.connection_string)
        user = User(email="test@example.com", name="Test")
        repo.save(user)

        with pytest.raises(DuplicateKeyError):
            repo.save(user)  # Same email
```

---

## When to Escalate to Level 3

| Behavior              | Level 2 Sufficient? | Why                      |
| --------------------- | ------------------- | ------------------------ |
| rclone syncs files    | ✅ Yes              | Local backend works same |
| Postgres queries work | ✅ Yes              | Real database            |
| Dropbox OAuth works   | ❌ No               | Need real Dropbox        |
| Rate limiting handled | ❌ No               | Local never rate limits  |

---

## Performance Expectations

Level 2 tests should complete in **<1 second** each:

```python
@pytest.mark.integration
def test_sync_completes_quickly(source_dir, test_remote, rclone_config):
    for i in range(10):
        (source_dir / f"file_{i}.txt").write_text(f"content {i}")

    start = time.perf_counter()
    sync_dataset(str(source_dir), test_remote, config=rclone_config)
    duration = time.perf_counter() - start

    assert duration < 1.0
```

---

## Anti-Patterns

### Skipping Integration Tests

```python
# ❌ Don't skip because they're "slow"
@pytest.mark.skip("Too slow")
@pytest.mark.integration
def test_database_query(): ...


# ✅ Integration tests should be fast (<1s)
@pytest.mark.integration
def test_database_query(database):
    # This runs in <100ms
    ...
```

### Using Level 2 When Level 1 Suffices

```python
# ❌ This doesn't need Docker
@pytest.mark.integration
def test_command_has_checksum_flag(database):
    cmd = build_command("/source", "dest")
    assert "--checksum" in cmd


# ✅ Level 1 is sufficient
def test_command_has_checksum_flag():
    cmd = build_command("/source", "dest")
    assert "--checksum" in cmd
```

---

*Level 2 is where theory meets reality. If it works here, you have real confidence.*
