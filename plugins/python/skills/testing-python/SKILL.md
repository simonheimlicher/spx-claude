---
name: testing-python
description: Python-specific testing patterns with dependency injection and real infrastructure. Use when testing Python code or writing Python tests.
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

# Python Testing Patterns

> **PREREQUISITE: Run through the `/testing` router first.**
>
> This skill provides Python-specific implementations for decisions made there. Do NOT skip the router—it determines WHAT to test and at WHAT level. This skill shows HOW to implement that decision in Python.

---

## Router Decision → Python Implementation

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

---

## Python Tooling by Level

| Level          | Infrastructure                                   | Speed  |
| -------------- | ------------------------------------------------ | ------ |
| 1: Unit        | Python stdlib + temp dirs + standard dev tools   | <100ms |
| 2: Integration | Docker containers + project-specific binaries    | <1s    |
| 3: E2E         | Network services + external APIs + test accounts | <10s   |

**Standard dev tools** (Level 1): git, cat, grep, curl—available in CI without setup.
**Project-specific tools** (Level 2): Docker, PostgreSQL, Hugo, ffmpeg—require installation.

---

## Level 1: Pure Computation (Stage 3A)

When the router determines your code is pure computation, test it directly.

### Pure Function Testing

```python
def test_command_includes_checksum_flag():
    cmd = build_rclone_command("/source", "remote:dest", checksum=True)
    assert "--checksum" in cmd


def test_unicode_paths_preserved():
    cmd = build_rclone_command("/tank/фото", "remote:резервная")
    assert "/tank/фото" in cmd


def test_validates_empty_order():
    result = validate_order(Order(items=[]))
    assert result.ok is False
    assert "empty" in result.error.lower()
```

### Data Factories

Generate test data programmatically. Never use arbitrary literals.

```python
from dataclasses import dataclass, field
from typing import Iterator
import itertools

_id_counter: Iterator[int] = itertools.count(1)


@dataclass
class AuditResultFactory:
    url: str = field(default_factory=lambda: f"https://example.com/{next(_id_counter)}")
    performance: int = 90
    accessibility: int = 100

    def build(self) -> dict:
        return {
            "url": self.url,
            "scores": {
                "performance": self.performance,
                "accessibility": self.accessibility,
            },
        }


def test_fails_on_low_performance():
    result = AuditResultFactory(performance=45).build()
    analysis = analyze_results([result], thresholds)
    assert analysis.passed is False
```

### Temporary Directories

Temp dirs are NOT external dependencies—use them freely at Level 1.

```python
import tempfile
from pathlib import Path


def test_loads_yaml_config():
    with tempfile.TemporaryDirectory() as tmpdir:
        config_path = Path(tmpdir) / "config.yaml"
        config_path.write_text("""
site_dir: ./site
base_url: http://localhost:1313
""")

        config = load_config(config_path)

        assert config.site_dir == "./site"
        assert config.base_url == "http://localhost:1313"
```

---

## Level 1: Extracted Logic (Stage 3B)

When the router says "extract the pure part," factor your code.

### Before: Tangled Code

```python
class OrderProcessor:
    def __init__(self, repository):
        self.repository = repository

    def process(self, order: Order) -> None:
        # Validation (pure) mixed with persistence (integration)
        if not order.items:
            raise ValidationError("Empty order")
        if order.total < 0:
            raise ValidationError("Negative total")
        self.repository.save(order)
```

### After: Extracted

```python
# Pure computation - test at Level 1, no doubles
def validate_order(order: Order) -> ValidationResult:
    if not order.items:
        return ValidationResult(ok=False, error="Empty order")
    if order.total < 0:
        return ValidationResult(ok=False, error="Negative total")
    return ValidationResult(ok=True)


# Thin wrapper - test at Level 2 with real database
class OrderProcessor:
    def __init__(self, repository):
        self.repository = repository

    def process(self, order: Order) -> None:
        result = validate_order(order)
        if not result.ok:
            raise ValidationError(result.error)
        self.repository.save(order)
```

Now test them separately:

```python
# Level 1: Test validation logic exhaustively
def test_validates_empty_order():
    result = validate_order(Order(items=[]))
    assert result.ok is False


def test_validates_negative_total():
    result = validate_order(Order(items=[item], total=-10))
    assert result.ok is False


def test_accepts_valid_order():
    result = validate_order(Order(items=[item], total=100))
    assert result.ok is True


# Level 2: Test persistence with real database (see Level 2 section)
```

---

## Level 1: Dependency Injection Pattern

When code has dependencies but you've determined Level 1 is appropriate (via router Stage 3), use DI with Protocols and dataclasses.

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
    return SyncResult(success=returncode == 0, output=stdout)


# Test with controlled implementation
def test_sync_returns_success_on_zero_exit():
    class FakeRunner:
        def run(self, cmd: list[str]) -> tuple[int, str, str]:
            return (0, "Transferred: 5 files", "")

    deps = SyncDependencies(run_command=FakeRunner())
    result = sync_to_remote("/src", "remote:dest", deps)
    assert result.success is True
```

---

## Exception Case Implementations (Python)

When the router reaches Stage 5 and an exception applies, here's how to implement each in Python.

### Exception 1: Failure Modes

Testing retry logic, error handling, circuit breakers.

```python
from typing import Protocol


class HttpClient(Protocol):
    def fetch(self, url: str) -> dict: ...


def test_retries_on_timeout():
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


def test_circuit_breaker_opens_after_failures():
    failure_count = 0

    class FailingClient:
        def fetch(self, url: str) -> dict:
            nonlocal failure_count
            failure_count += 1
            raise ConnectionError("Connection refused")

    breaker = CircuitBreaker(threshold=3)

    for _ in range(5):
        try:
            breaker.call(lambda: FailingClient().fetch("/"))
        except (ConnectionError, CircuitOpenError):
            pass

    # Circuit should open after 3 failures, preventing further calls
    assert failure_count == 3
    assert breaker.state == "open"
```

### Exception 2: Interaction Protocols

Testing call sequences, saga compensation, "no extra calls."

```python
def test_saga_compensates_in_reverse_order():
    calls: list[str] = []

    class Step1:
        def execute(self):
            calls.append("step1-execute")

        def compensate(self):
            calls.append("step1-compensate")

    class Step2:
        def execute(self):
            calls.append("step2-execute")
            raise RuntimeError("Step 2 failed")

        def compensate(self):
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


def test_caches_and_does_not_refetch():
    fetch_count = 0

    class CountingClient:
        def get_user(self, user_id: str) -> dict:
            nonlocal fetch_count
            fetch_count += 1
            return {"id": user_id, "name": "Test"}

    cache = CachingWrapper(CountingClient())

    cache.get_user("123")
    cache.get_user("123")
    cache.get_user("123")

    assert fetch_count == 1  # Only one actual fetch
```

### Exception 3: Time and Concurrency

Testing time-dependent behavior with controlled time.

```python
from unittest.mock import patch


def test_lease_renews_before_expiry():
    with patch("time.time") as mock_time:
        mock_time.return_value = 1000.0

        renewed = False

        def on_renew():
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


def test_generates_deterministic_ids():
    with patch("secrets.token_hex") as mock_token:
        mock_token.return_value = "abc123"

        result = create_resource("test")

        assert result.id == "abc123"
```

### Exception 4: Safety

Testing destructive operations without executing them.

```python
def test_processes_refund_for_cancelled_order():
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


def test_does_not_send_real_emails():
    sent_emails: list[dict] = []

    class FakeEmailService:
        def send(self, to: str, subject: str, body: str) -> None:
            sent_emails.append({"to": to, "subject": subject})

    notifier = OrderNotifier(email=FakeEmailService())
    notifier.notify_shipped(order)

    assert len(sent_emails) == 1
    assert sent_emails[0]["to"] == order.customer_email
```

### Exception 6: Observability

Testing request details the real system can't expose.

```python
def test_includes_idempotency_key():
    requests: list[dict] = []

    class SpyHttpClient:
        def post(self, url: str, headers: dict, body: dict) -> dict:
            requests.append({"url": url, "headers": headers, "body": body})
            return {"status": 200}

    client = PaymentClient(http=SpyHttpClient())
    client.charge(amount=100, card_token="tok_123")

    assert len(requests) == 1
    assert "Idempotency-Key" in requests[0]["headers"]
    assert requests[0]["headers"]["Idempotency-Key"] is not None


def test_batches_inserts():
    queries: list[str] = []

    class SpyDatabase:
        def execute(self, sql: str, params: list) -> None:
            queries.append(sql)

    repo = UserRepository(db=SpyDatabase())
    repo.bulk_insert([user1, user2, user3])

    # Should be ONE batch insert, not three
    assert len(queries) == 1
    assert "INSERT INTO users" in queries[0]
```

---

## Level 2: Integration Patterns

When the router determines Level 2 is appropriate, use real dependencies via harnesses.

### Docker Harness

```python
import subprocess
from dataclasses import dataclass


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

    def reset(self) -> None:
        # Truncate all tables
        pass

    def _wait_for_ready(self, timeout: int = 30) -> None:
        # Poll until postgres accepts connections
        pass

    @property
    def connection_string(self) -> str:
        return f"postgresql://postgres:test@localhost:{self.port}/postgres"
```

### pytest Fixtures for Harnesses

```python
import pytest


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

    assert retrieved is not None
    assert retrieved.name == "Test User"


@pytest.mark.integration
def test_enforces_unique_email_constraint(database):
    repo = UserRepository(database.connection_string)
    repo.save(User(email="dupe@example.com", name="First"))

    with pytest.raises(IntegrityError):
        repo.save(User(email="dupe@example.com", name="Second"))
```

### Binary Harness

```python
@dataclass
class HugoHarness:
    site_dir: Path
    output_dir: Path
    _temp_dir: tempfile.TemporaryDirectory | None = None

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
        if self._temp_dir:
            self._temp_dir.cleanup()


@pytest.fixture
def hugo():
    harness = create_hugo_harness()
    yield harness
    harness.cleanup()


@pytest.mark.integration
def test_builds_site_successfully(hugo):
    result = hugo.build()

    assert result.returncode == 0
    assert (hugo.output_dir / "index.html").exists()
```

---

## Level 3: E2E Patterns

When the router determines Level 3 is required (real credentials, external services).

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


def require_credentials() -> dict:
    creds = load_credentials()
    if not creds:
        raise RuntimeError("Missing required credentials.\n\n" + CREDENTIALS_DOC)
    return creds
```

### Skip If No Credentials

```python
credentials = load_credentials()
skip_no_creds = pytest.mark.skipif(
    credentials is None, reason="E2E credentials not configured"
)


@skip_no_creds
@pytest.mark.e2e
def test_full_sync_workflow(dropbox_test_folder):
    result = sync_to_dropbox(local_path, dropbox_test_folder)

    assert result.success
    assert result.files_transferred > 0


@skip_no_creds
@pytest.mark.e2e
def test_handles_api_rate_limit(dropbox_test_folder):
    # This tests real rate limiting behavior
    results = [sync_to_dropbox(local_path, dropbox_test_folder) for _ in range(10)]

    # Should handle rate limits gracefully
    assert all(r.success for r in results)
```

---

## pytest Configuration

```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["spx"]
python_files = ["*.test.py"]
markers = [
  "integration: Level 2 tests requiring local infrastructure",
  "e2e: Level 3 tests requiring credentials and external services",
]

# Run by level:
# pytest spx/ -k "unit"              # Level 1 only
# pytest spx/ -k "integration"       # Level 2
# pytest spx/ -k "e2e"               # Level 3
# pytest spx/                        # All tests
```

---

## Test Organization (CODE Framework)

Tests are co-located with specs in `spx/`. Level is indicated by suffix:

```
spx/
└── {capability}/
    └── {feature}/
        ├── {feature}.md                  # Feature spec
        └── tests/
            ├── {name}.unit.test.py       # Level 1
            ├── {name}.integration.test.py # Level 2
            ├── {name}.e2e.test.py        # Level 3, non-browser
            └── {name}.e2e.spec.py        # Level 3, browser (Playwright)
```

### Shared Test Infrastructure

```
myproject_testing/          # Installable via uv pip install -e ".[dev]"
├── __init__.py
├── harnesses/
│   ├── postgres.py         # PostgreSQL harness
│   ├── docker.py           # Generic Docker harness
│   └── factories.py        # Data factories
└── fixtures/
    └── values.py           # TYPICAL, EDGES collections
```

Import in tests:

```python
from myproject_testing.harnesses.factories import UserFactory
from myproject_testing.harnesses.postgres import PostgresHarness
```

---

## Quick Reference

| Aspect       | Level 1                    | Level 2            | Level 3              |
| ------------ | -------------------------- | ------------------ | -------------------- |
| Dependencies | DI with Protocol/dataclass | Real via harness   | Real via credentials |
| Data         | Factories + tmp_path       | Fixtures + harness | Test accounts        |
| Speed        | <100ms                     | <1s                | <10s                 |
| CI           | Every commit               | Every commit       | Nightly/pre-release  |

---

## Python-Specific Anti-Patterns

### Using unittest.mock.patch on External Services

```python
# ❌ WRONG: Mocking external service
@patch("httpx.Client.get")
def test_fetches_user(mock_get):
    mock_get.return_value = Mock(json=lambda: {"id": 1})
    user = api_client.get_user(1)
    assert user.id == 1  # Proves nothing about real API


# ✅ RIGHT: Use DI for Level 1, real service for Level 2/3
def test_parses_user_response():
    # Level 1: Test parsing logic with known data
    response = {"id": 1, "name": "Test", "email": "test@example.com"}
    user = parse_user_response(response)
    assert user.id == 1


@pytest.mark.e2e
def test_fetches_real_user(test_credentials):
    # Level 3: Test against real API
    client = ApiClient(credentials=test_credentials)
    user = client.get_user(known_test_user_id)
    assert user is not None
```

### Overusing pytest-mock

```python
# ❌ WRONG: mocker fixture everywhere
def test_sync(mocker):
    mocker.patch("os.path.exists", return_value=True)
    mocker.patch("subprocess.run", return_value=Mock(returncode=0))
    result = sync_files(src, dest)
    assert result.success  # What did we prove? Nothing.


# ✅ RIGHT: DI for controllable behavior
def test_sync_returns_success_on_zero_exit():
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
def test_parses_verbose_flag():
    parser = create_parser()
    args = parser.parse_args(["--verbose"])
    assert args.verbose is True  # Testing argparse, not your code


# ✅ RIGHT: Test YOUR behavior that uses parsed args
def test_verbose_mode_produces_detailed_output():
    output = run_command(Config(verbose=True))
    assert "DEBUG:" in output
```
