---
name: testing-python
description: Python-specific testing patterns with dependency injection and real infrastructure. Use when testing Python code or writing Python tests.
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

<objective>
Provide Python-specific implementations for testing decisions made by the `/testing` router. This skill shows HOW to implement tests in Python—the router determines WHAT to test and at WHAT level.
</objective>

<quick_start>
**PREREQUISITE**: Run through `/testing` first to determine test level and approach.

1. Read `/testing` for methodology (5 stages, 5 factors, 7 exceptions)
2. Mentally apply to your context
3. Consult `/standardizing-python-testing` for standards (property-based, factories, markers)
4. Use this skill for Python-specific implementation patterns

When providing insights to users, cite sources:

- "Per /testing Stage 2 Factor 2, database dependency requires Level 2"
- "Per /standardizing-python-testing, parsers MUST have property-based tests"

</quick_start>

<router_to_python_mapping>
After running through the `/testing` router, use this mapping:

| Router Decision                                 | Python Implementation                          |
| ----------------------------------------------- | ---------------------------------------------- |
| **Stage 2 → Level 1**                           | pytest + temp dirs + dataclasses for DI        |
| **Stage 2 → Level 2**                           | pytest fixtures + Docker/subprocess harnesses  |
| **Stage 2 → Level 3**                           | pytest + skip decorators + credential loading  |
| **Stage 3A** (Pure computation)                 | Pure functions, test directly                  |
| **Stage 3B** (Extract pure part)                | Factor into pure functions + thin wrappers     |
| **Stage 5 Exception 1** (Failure modes)         | Protocol + stub returning errors               |
| **Stage 5 Exception 2** (Interaction protocols) | Spy class recording calls                      |
| **Stage 5 Exception 3** (Time/concurrency)      | `patch("time.time")`, `patch("random.random")` |
| **Stage 5 Exception 4** (Safety)                | Stub that records but doesn't execute          |
| **Stage 5 Exception 6** (Observability)         | Spy class capturing request details            |

</router_to_python_mapping>

<level_tooling>

| Level          | Infrastructure                                   | Speed  |
| -------------- | ------------------------------------------------ | ------ |
| 1: Unit        | Python stdlib + temp dirs + standard dev tools   | <100ms |
| 2: Integration | Docker containers + project-specific binaries    | <1s    |
| 3: E2E         | Network services + external APIs + test accounts | <10s   |

**Standard dev tools** (Level 1): git, cat, grep, curl—available in CI without setup.
**Project-specific tools** (Level 2): Docker, PostgreSQL, Hugo, ffmpeg—require installation.

</level_tooling>

<level_1_patterns>

## Pure Computation (Stage 3A)

When the router determines your code is pure computation, test it directly.

```python
# ✅ Test pure functions directly, no doubles needed
def test_command_includes_checksum_flag() -> None:
    cmd = build_rclone_command("/source", "remote:dest", checksum=True)
    assert "--checksum" in cmd


def test_unicode_paths_preserved() -> None:
    cmd = build_rclone_command("/tank/фото", "remote:резервная")
    assert "/tank/фото" in cmd


def test_validates_empty_order() -> None:
    result = validate_order(Order(items=[]))
    assert result.ok is False
    assert "empty" in result.error.lower()
```

## Temporary Directories

Temp dirs are NOT external dependencies—use freely at Level 1.

```python
import tempfile
from pathlib import Path


def test_loads_yaml_config() -> None:
    with tempfile.TemporaryDirectory() as tmpdir:
        config_path = Path(tmpdir) / "config.yaml"
        config_path.write_text("""
site_dir: ./site
base_url: http://localhost:1313
""")
        config = load_config(config_path)
        assert config.site_dir == "./site"
```

## Extracted Logic (Stage 3B)

When the router says "extract the pure part," factor your code.

**Before** (tangled):

```python
class OrderProcessor:
    def __init__(self, repository) -> None:
        self.repository = repository

    def process(self, order: Order) -> None:
        # Validation (pure) mixed with persistence (integration)
        if not order.items:
            raise ValidationError("Empty order")
        self.repository.save(order)
```

**After** (factored):

```python
# Pure computation - test at Level 1, no doubles
def validate_order(order: Order) -> ValidationResult:
    if not order.items:
        return ValidationResult(ok=False, error="Empty order")
    return ValidationResult(ok=True)


# Thin wrapper - test at Level 2 with real database
class OrderProcessor:
    def __init__(self, repository) -> None:
        self.repository = repository

    def process(self, order: Order) -> None:
        result = validate_order(order)
        if not result.ok:
            raise ValidationError(result.error)
        self.repository.save(order)
```

Now test separately:

```python
# Level 1: Test validation logic exhaustively
def test_validates_empty_order() -> None:
    result = validate_order(Order(items=[]))
    assert result.ok is False


# Level 2: Test persistence with real database (see Level 2 section)
```

## Dependency Injection Pattern

When code has dependencies but Level 1 is appropriate (per router Stage 3), use DI with Protocols.

See `/standardizing-python-testing` for Protocol and dataclass patterns.

```python
from dataclasses import dataclass
from typing import Protocol


class CommandRunner(Protocol):
    def run(self, cmd: list[str]) -> tuple[int, str, str]: ...


@dataclass
class SyncDependencies:
    run_command: CommandRunner


def sync_to_remote(source: str, dest: str, deps: SyncDependencies) -> SyncResult:
    cmd = build_command(source, dest)
    returncode, stdout, stderr = deps.run_command.run(cmd)
    return SyncResult(success=returncode == 0, output=stdout)


# Test with controlled implementation
def test_sync_returns_success_on_zero_exit() -> None:
    class FakeRunner:
        def run(self, cmd: list[str]) -> tuple[int, str, str]:
            return (0, "Transferred: 5 files", "")

    deps = SyncDependencies(run_command=FakeRunner())
    result = sync_to_remote("/src", "remote:dest", deps)
    assert result.success is True
```

</level_1_patterns>

<exception_implementations>

## Exception Case Implementations

When the router reaches Stage 5 and an exception applies, here's how to implement each in Python.

### Exception 1: Failure Modes

Testing retry logic, error handling, circuit breakers.

```python
from typing import Protocol


class HttpClient(Protocol):
    def fetch(self, url: str) -> dict: ...


def test_retries_on_timeout() -> None:
    attempts = 0

    class TimeoutingClient:
        def fetch(self, url: str) -> dict:
            nonlocal attempts
            attempts += 1
            if attempts < 3:
                raise TimeoutError("Request timed out")
            return {"status": 200, "body": "ok"}

    result = fetch_with_retry("https://api.example.com", TimeoutingClient())
    assert attempts == 3
    assert result["status"] == 200
```

### Exception 2: Interaction Protocols

Testing call sequences, saga compensation, "no extra calls."

```python
def test_saga_compensates_in_reverse_order() -> None:
    calls: list[str] = []

    class Step1:
        def execute(self) -> None:
            calls.append("step1-execute")

        def compensate(self) -> None:
            calls.append("step1-compensate")

    class Step2:
        def execute(self) -> None:
            calls.append("step2-execute")
            raise RuntimeError("Step 2 failed")

        def compensate(self) -> None:
            calls.append("step2-compensate")

    saga = Saga([Step1(), Step2()])

    with pytest.raises(RuntimeError):
        saga.run()

    assert calls == [
        "step1-execute",
        "step2-execute",
        "step2-compensate",
        "step1-compensate",
    ]
```

### Exception 3: Time and Concurrency

Testing time-dependent behavior with controlled time.

```python
from unittest.mock import patch


def test_lease_renews_before_expiry() -> None:
    with patch("time.time") as mock_time:
        mock_time.return_value = 1000.0

        renewed = False

        def on_renew() -> None:
            nonlocal renewed
            renewed = True

        lease = Lease(ttl=30, renew_at=25, on_renew=on_renew)

        # Before renewal threshold
        mock_time.return_value = 1024.0
        lease.tick()
        assert renewed is False

        # After renewal threshold
        mock_time.return_value = 1026.0
        lease.tick()
        assert renewed is True
```

### Exception 4: Safety

Testing destructive operations without executing them.

```python
def test_processes_refund_for_cancelled_order() -> None:
    refunds: list[dict] = []

    class FakePaymentProvider:
        def refund(self, charge_id: str, amount: float, reason: str) -> dict:
            refunds.append({"charge_id": charge_id, "amount": amount, "reason": reason})
            return {"refund_id": "refund_123", "status": "succeeded"}

    processor = OrderProcessor(payment=FakePaymentProvider())
    processor.cancel_order(order_with_charge)

    assert refunds == [
        {"charge_id": "ch_123", "amount": 99.99, "reason": "order_cancelled"}
    ]
```

### Exception 6: Observability

Testing request details the real system can't expose.

```python
def test_includes_idempotency_key() -> None:
    requests: list[dict] = []

    class SpyHttpClient:
        def post(self, url: str, headers: dict, body: dict) -> dict:
            requests.append({"url": url, "headers": headers, "body": body})
            return {"status": 200}

    client = PaymentClient(http=SpyHttpClient())
    client.charge(amount=100, card_token="tok_123")

    assert len(requests) == 1
    assert "Idempotency-Key" in requests[0]["headers"]
```

</exception_implementations>

<level_2_patterns>

## Integration Patterns

When the router determines Level 2 is appropriate, use real dependencies via harnesses.

See `/standardizing-python-testing` for harness base patterns.

### pytest Fixtures for Harnesses

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


@pytest.mark.level_2
def test_user_repository_saves_and_retrieves(database: PostgresHarness) -> None:
    repo = UserRepository(database.connection_string)
    user = User(email="test@example.com", name="Test User")

    repo.save(user)
    retrieved = repo.find_by_email("test@example.com")

    assert retrieved is not None
    assert retrieved.name == "Test User"
```

### Binary Harness

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


@pytest.mark.level_2
def test_builds_site_successfully(hugo: HugoHarness) -> None:
    result = hugo.build()
    assert result.returncode == 0
    assert (hugo.output_dir / "index.html").exists()
```

</level_2_patterns>

<level_3_patterns>

## E2E Patterns

When the router determines Level 3 is required (real credentials, external services).

See `/standardizing-python-testing` for credential management patterns.

### Skip If No Credentials

```python
credentials = load_credentials()
skip_no_creds = pytest.mark.skipif(
    credentials is None, reason="E2E credentials not configured"
)


@skip_no_creds
@pytest.mark.level_3
def test_full_sync_workflow(dropbox_test_folder: str) -> None:
    result = sync_to_dropbox(local_path, dropbox_test_folder)
    assert result.success
    assert result.files_transferred > 0
```

**Note**: Use `skipif` only for optional E2E tests. Required tests must fail loudly—see `/standardizing-python-testing`.

</level_3_patterns>

<anti_patterns>

## Python-Specific Anti-Patterns

### Using unittest.mock.patch on External Services

```python
# ❌ WRONG: Mocking external service
@patch("httpx.Client.get")
def test_fetches_user(mock_get: Mock) -> None:
    mock_get.return_value = Mock(json=lambda: {"id": 1})
    user = api_client.get_user(1)
    assert user.id == 1  # Proves nothing about real API


# ✅ RIGHT: Use DI for Level 1, real service for Level 2/3
def test_parses_user_response() -> None:
    # Level 1: Test parsing logic with known data
    response = {"id": 1, "name": "Test", "email": "test@example.com"}
    user = parse_user_response(response)
    assert user.id == 1


@pytest.mark.level_3
def test_fetches_real_user(test_credentials: dict) -> None:
    # Level 3: Test against real API
    client = ApiClient(credentials=test_credentials)
    user = client.get_user(known_test_user_id)
    assert user is not None
```

### Overusing pytest-mock

```python
# ❌ WRONG: mocker fixture everywhere
def test_sync(mocker: MockerFixture) -> None:
    mocker.patch("os.path.exists", return_value=True)
    mocker.patch("subprocess.run", return_value=Mock(returncode=0))
    result = sync_files(src, dest)
    assert result.success  # What did we prove? Nothing.


# ✅ RIGHT: DI for controllable behavior
def test_sync_returns_success_on_zero_exit() -> None:
    class FakeRunner:
        def run(self, cmd: list[str]) -> tuple[int, str, str]:
            return (0, "Done", "")

    deps = SyncDeps(runner=FakeRunner())
    result = sync_files(src, dest, deps)
    assert result.success
```

### Testing Library Behavior

```python
# ❌ WRONG: Testing that argparse works
def test_parses_verbose_flag() -> None:
    parser = create_parser()
    args = parser.parse_args(["--verbose"])
    assert args.verbose is True  # Testing argparse, not your code


# ✅ RIGHT: Test YOUR behavior that uses parsed args
def test_verbose_mode_produces_detailed_output() -> None:
    output = run_command(Config(verbose=True))
    assert "DEBUG:" in output
```

</anti_patterns>

<success_criteria>
Testing is complete when:

- [ ] `/testing` router consulted for level and approach
- [ ] `/standardizing-python-testing` consulted for standards
- [ ] Property-based tests present for parsers, serializers, math, complex algorithms
- [ ] All tests have `-> None` return type
- [ ] All tests marked with appropriate level (`@pytest.mark.level_1/level_2/level_3`)
- [ ] File naming follows convention (`.level_1.py`, `.level_2.py`, `.level_3.py`)
- [ ] No mocking—DI with Protocols where doubles are needed
- [ ] Exception case documented when test doubles are used
- [ ] All tests pass

</success_criteria>
