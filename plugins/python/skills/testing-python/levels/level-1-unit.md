# Level 1: Unit Tests

**Speed**: <100ms | **Infrastructure**: OS primitives only | **Framework**: pytest

Level 1 tests verify our code logic is correct without any external dependencies.

## What Counts as "No Dependencies"

These are **NOT** external dependencies—they're part of the runtime environment:

| Tool                   | Why It's OK                                  |
| ---------------------- | -------------------------------------------- |
| `echo`, `env`, `cat`   | Foundational OS primitives, always available |
| `subprocess.run()`     | Testing OUR command-building logic           |
| `tempfile`, `tmp_path` | Ephemeral, reentrant, no persistent state    |
| `pathlib.Path`         | Standard library file operations             |

## What IS an External Dependency

| Tool     | Why It's Level 2+                 |
| -------- | --------------------------------- |
| rclone   | External binary with own behavior |
| Docker   | Requires daemon running           |
| Postgres | Requires server running           |

---

## Dependency Injection Pattern

Design code to accept dependencies as parameters.

### Production Code

```python
from dataclasses import dataclass
from typing import Callable, Protocol


class CommandRunner(Protocol):
    def run(self, cmd: list[str]) -> tuple[int, str, str]:
        """Returns (returncode, stdout, stderr)."""
        ...


@dataclass
class SyncDependencies:
    run_command: CommandRunner
    get_env: Callable[[str], str | None] = os.environ.get


def build_rclone_command(source: str, dest: str, *, checksum: bool = True) -> list[str]:
    """Build rclone command. Pure function, no I/O."""
    cmd = ["rclone", "sync", source, dest]
    if checksum:
        cmd.append("--checksum")
    return cmd


def sync_to_remote(source: str, dest: str, deps: SyncDependencies) -> SyncResult:
    cmd = build_rclone_command(source, dest)
    returncode, stdout, stderr = deps.run_command.run(cmd)
    return SyncResult(success=returncode == 0)
```

### Level 1 Tests

```python
class TestBuildRcloneCommand:
    """Level 1: Pure function tests—no dependencies."""

    def test_basic_command_structure(self):
        cmd = build_rclone_command("/source", "remote:dest")
        assert cmd[0] == "rclone"
        assert cmd[1] == "sync"

    def test_checksum_flag_included_by_default(self):
        cmd = build_rclone_command("/source", "remote:dest")
        assert "--checksum" in cmd

    def test_unicode_paths_preserved(self):
        cmd = build_rclone_command("/tank/фото", "remote:резервная")
        assert "/tank/фото" in cmd


class TestSyncToRemote:
    """Level 1: Testing sync logic with injected dependencies."""

    def test_success_when_command_returns_zero(self):
        class FakeRunner:
            def run(self, cmd: list[str]) -> tuple[int, str, str]:
                return (0, "Transferred: 5 files", "")

        deps = SyncDependencies(run_command=FakeRunner())
        result = sync_to_remote("/source", "remote:dest", deps)
        assert result.success is True

    def test_failure_when_command_returns_nonzero(self):
        class FakeRunner:
            def run(self, cmd: list[str]) -> tuple[int, str, str]:
                return (1, "", "Error: failed to sync")

        deps = SyncDependencies(run_command=FakeRunner())
        result = sync_to_remote("/source", "remote:dest", deps)
        assert result.success is False
```

---

## Using Temporary Directories

Temp directories are ephemeral and reentrant—they're Level 1.

```python
class TestFileOperations:
    def test_creates_directory_structure(self, tmp_path):
        nested = tmp_path / "a" / "b" / "c"
        prepare_sync_directory(nested)
        assert nested.exists()

    def test_copies_files_to_staging(self, tmp_path):
        source = tmp_path / "source"
        source.mkdir()
        (source / "file.txt").write_text("content")

        staging = tmp_path / "staging"
        stage_files(source, staging)

        assert (staging / "file.txt").read_text() == "content"
```

---

## When to Escalate to Level 2

| Behavior                     | Level 1 Sufficient? | Why                                 |
| ---------------------------- | ------------------- | ----------------------------------- |
| Command is well-formed       | ✅ Yes              | Pure function, can verify structure |
| rclone accepts the command   | ❌ No               | Need real rclone to verify          |
| Files actually sync          | ❌ No               | Need real rclone execution          |
| Config file parsed correctly | ✅ Yes              | Pure parsing logic                  |

---

## Anti-Patterns

### Mocking Instead of DI

```python
# ❌ Mocking couples test to implementation
@patch("cloud_mirror.sync.subprocess.run")
def test_calls_rclone(mock_run):
    mock_run.return_value = Mock(returncode=0)
    sync_to_remote(src, dest)
    mock_run.assert_called_with(["rclone", "sync", ...])


# ✅ DI tests behavior, not implementation
def test_sync_succeeds_when_command_succeeds():
    deps = SyncDependencies(run_command=FakeSuccessRunner())
    result = sync_to_remote(src, dest, deps)
    assert result.success
```

### Testing External Tool Behavior

```python
# ❌ This tests rclone, not our code
def test_rclone_sync_copies_files(tmp_path):
    subprocess.run(["rclone", "sync", str(src), str(dest)])
    assert (dest / "file.txt").exists()  # Tests rclone, not our code


# ✅ This tests our code's command building
def test_sync_command_includes_source_and_dest():
    cmd = build_rclone_command("/source", "/dest")
    assert "/source" in cmd
```

---

*Level 1 is the foundation. Get this right, and higher levels become verification, not debugging.*
