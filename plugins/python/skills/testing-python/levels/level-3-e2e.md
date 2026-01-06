# Level 3: E2E Tests (Real Services)

**Speed**: <10s | **Infrastructure**: Network + Test Accounts | **Framework**: pytest

Level 3 tests verify that real remote services work correctly using dedicated test accounts.

## What Level 3 Provides

| Component   | What's Real          | What's Controlled             |
| ----------- | -------------------- | ----------------------------- |
| Dropbox API | Real Dropbox service | Dedicated test account        |
| OAuth       | Real authentication  | Test credentials from secrets |
| Rate limits | Real rate limiting   | May need handling in tests    |

## Why Level 3 Exists

Level 2 proves tools work locally. Level 3 proves:

- OAuth authentication actually works
- Service-specific behaviors are handled (rate limits, quirks)
- Network error handling works
- Real API responses are parsed correctly

---

## Credential Management

### Document Required Credentials

```python
CREDENTIALS_DOC = """
Level 3 tests require these environment variables:

Required:
  DROPBOX_TEST_TOKEN    - From 1Password: "Engineering/Test Credentials"
  GITHUB_TEST_TOKEN     - Personal access token for test account

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
        raise RuntimeError(f"Missing credentials.\n{CREDENTIALS_DOC}")
    return creds
```

### Fixture with Automatic Skip

```python
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

@pytest.fixture
def dropbox_test_folder(dropbox_config) -> str:
    """Provide isolated test folder in Dropbox."""
    folder_name = f"test_{uuid4().hex[:8]}"
    remote = f"dropbox-test:cloud-mirror-tests/{folder_name}"

    yield remote

    # Cleanup
    subprocess.run([
        "rclone", "purge", remote,
        "--config", str(dropbox_config),
    ])
```

---

## Good Patterns

### Basic Service Verification

```python
@pytest.mark.e2e
class TestDropboxSync:
    def test_files_sync_to_dropbox(
        self, source_dir, dropbox_test_folder, dropbox_config
    ):
        (source_dir / "test.txt").write_text("test content")

        result = sync_dataset(
            str(source_dir), dropbox_test_folder, config=dropbox_config
        )

        assert result.success
        # Verify via rclone ls
        ls_result = subprocess.run(
            ["rclone", "ls", dropbox_test_folder, "--config", str(dropbox_config)],
            capture_output=True, text=True,
        )
        assert "test.txt" in ls_result.stdout

    def test_sync_is_idempotent(
        self, source_dir, dropbox_test_folder, dropbox_config
    ):
        (source_dir / "stable.txt").write_text("stable")

        # First sync
        sync_dataset(str(source_dir), dropbox_test_folder, config=dropbox_config)

        # Second sync
        result = sync_dataset(
            str(source_dir), dropbox_test_folder, config=dropbox_config
        )

        assert result.files_transferred == 0
```

### Authentication Testing

```python
@pytest.mark.e2e
class TestDropboxAuth:
    def test_valid_token_authenticates(self, dropbox_config):
        result = subprocess.run(
            ["rclone", "lsd", "dropbox-test:", "--config", str(dropbox_config)],
            capture_output=True,
        )
        assert result.returncode == 0

    def test_expired_token_reports_auth_error(self, tmp_path):
        bad_config = tmp_path / "bad.conf"
        bad_config.write_text("""
[dropbox-test]
type = dropbox
token = {"access_token":"invalid","token_type":"bearer"}
""")

        result = sync_dataset("/tmp/source", "dropbox-test:dest", config=bad_config)

        assert not result.success
        assert result.error.category == "authentication"
```

### Rate Limit Handling

```python
@pytest.mark.e2e
class TestRateLimits:
    def test_tpslimit_prevents_rate_errors(
        self, source_dir, dropbox_test_folder, dropbox_config
    ):
        # Create many small files
        for i in range(50):
            (source_dir / f"file_{i:03d}.txt").write_text(f"content {i}")

        result = sync_dataset(
            str(source_dir), dropbox_test_folder,
            config=dropbox_config, tpslimit=8,
        )

        assert result.success
        assert result.rate_limit_errors == 0
```

---

## Handling Test Flakiness

Internet tests can be flaky. Handle with care:

### Retry Decorator

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))
def sync_with_retry(source, dest, **kwargs):
    result = sync_dataset(source, dest, **kwargs)
    if not result.success and result.error.is_transient:
        raise TransientError(result.error)
    return result
```

### Flaky Test Marker

```python
@pytest.mark.e2e
@pytest.mark.flaky(reruns=3, reruns_delay=5)
def test_high_volume_sync(source_dir, dropbox_test_folder, dropbox_config):
    """This test may hit rate limits; retry if needed."""
    ...
```

---

## CI Configuration

```yaml
# .github/workflows/test.yml
jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/ -m "not integration and not e2e"

  integration-tests:
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/ -m "integration"

  e2e-tests:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' # Only on main
    steps:
      - run: pytest tests/ -m "e2e"
        env:
          DROPBOX_TEST_TOKEN: ${{ secrets.DROPBOX_TEST_TOKEN }}
```

---

## Anti-Patterns

### Using Production Credentials

```python
# ❌ NEVER use production credentials
@pytest.fixture
def dropbox_config():
    config.write_text(f"""
[dropbox]
token = {os.environ["DROPBOX_PROD_TOKEN"]}
""")

# ✅ Use dedicated test account
@pytest.fixture
def dropbox_config():
    config.write_text(f"""
[dropbox-test]
token = {os.environ["DROPBOX_TEST_TOKEN"]}
""")
```

### Not Cleaning Up

```python
# ❌ Leaves test data in Dropbox
@pytest.mark.e2e
def test_sync(dropbox_config):
    sync_dataset(source, "dropbox-test:permanent-folder")
    # No cleanup!

# ✅ Use fixtures that clean up
@pytest.mark.e2e
def test_sync(dropbox_test_folder, dropbox_config):  # Fixture cleans up
    sync_dataset(source, dropbox_test_folder)
```

---

_Level 3 is where you prove the real world works. Test accounts give confidence without risking production._
