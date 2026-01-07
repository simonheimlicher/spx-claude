---
name: testing
description: "FOUNDATIONAL skill. All other skills MUST consult this before writing code, tests, or architecture. Starts from ONE question: How do I GUARANTEE this works in the real world? No mocking. dependency injection, test harness and real infrastructure."
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

# Test Strategy (Foundational Skill)

You are the **foundational test strategy authority**. All other skills—architect, coder, reviewer—MUST consult you before making decisions.

---

## Fundamentals

In this projec, we follow the Behavior-Driven Development methodology.

> Tests are not everything—but without tests, everything is nothing.

### The One Question That Matters

> **When a user runs your CLI, visits your website, or opens your app—will it ACTUALLY WORK?**
>
> Not "did the tests pass?" Not "is coverage high?"
> **Will. It. Actually. Work.**

Everything in this skill flows from that question. Tests exist to give you **justified confidence** that the answer is YES.

---

### The Confidence Pyramid

```
┌─────────────────────┐
│      LEVEL 3        │  "Does it work in the real world?"
│   System / E2E      │  Real credentials, real services
│                     │  Full user workflows
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│      LEVEL 2        │  "Does our code work with real infrastructure?"
│    Integration      │  Real binaries, real databases
│                     │  Test harnesses required
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│      LEVEL 1        │  "Is the logic of our functions correct?"
│    Unit / Pure      │  No external dependencies
│                     │  Dependency injection, temp dirs, test runner only
└─────────────────────┘
```

**Build from the bottom up.** Each level answers a question the previous level cannot.

---

### 🚨 The Cardinal Rule: No Mocking

> **Mocking is a confession that your code is poorly designed—or that you're testing at the wrong level.**

If you need to mock something:

1. **Redesign** to use dependency injection, OR
2. **Don't test that interaction at this level**—push it to Level 2 or 3

Mocking gives you a test that passes while your production code fails. This is worse than no test at all.

---

### 🚨 Progress Tests vs Regression Tests

> **CRITICAL INVARIANT: The production test suite (`test/` or `tests/`) MUST ALWAYS PASS.**

| Location            | Name                 | May Fail? | Purpose                          |
| ------------------- | -------------------- | --------- | -------------------------------- |
| `specs/.../tests/`  | **Progress tests**   | YES       | TDD red-green during development |
| `test/` or `tests/` | **Regression tests** | NO        | Protect working functionality    |

**The Rule**: Never write failing tests directly in the regression suite. Write progress tests first, graduate them when complete.

---

## Level 1: Unit / Pure Logic

### The Question This Level Answers

> **"Is our logic correct, independent of any external system?"**

If a user's request fails, Level 1 tests help you instantly rule out (or identify) bugs in your core logic.

### What You Can Use

| Allowed              | Examples                                   | Why It's OK                   |
| -------------------- | ------------------------------------------ | ----------------------------- |
| Test runner          | pytest, vitest, jest, go test              | Part of dev environment       |
| Language primitives  | temp files, env vars, in-memory structures | Part of runtime               |
| Standard dev tools   | git, cat, grep, curl                       | Available in CI without setup |
| Dependency injection | Pass interfaces, not implementations       | Enables isolation             |
| Factories/builders   | Generate test data programmatically        | Reproducible tests            |

**Standard dev tools** are those available in CI environments without installation (git, cat, grep, curl, sed, awk, etc.). Project-specific tools (make, pip, npm, hugo) are Level 2.

### What You Cannot Use

| Forbidden                                | Why                                               |
| ---------------------------------------- | ------------------------------------------------- |
| Real databases                           | That's Level 2                                    |
| Real HTTP calls                          | That's Level 2 or 3                               |
| Project-specific binaries (ffmpeg, hugo) | That's Level 2                                    |
| Project-specific build tools (make, npm) | That's Level 2                                    |
| Mocks of external systems                | Never. Use DI and don't test the interaction here |

### The Key Insight

**If you can't test something without mocking an external system, you're at the wrong level.**

Don't mock the database—design your code so the business logic doesn't know about the database. Then test the business logic here, and test the database integration at Level 2.

### Pattern: Dependency Injection

```python
## ❌ BAD: Hardcoded dependency, requires mocking to test
class OrderProcessor:
    def process(self, order):
        db = PostgresDatabase()  # Hardcoded!
        db.save(order)
        EmailService().send(order.customer, "Order confirmed")


## ✅ GOOD: Injected dependencies, testable without mocks
class OrderProcessor:
    def __init__(self, repository, notifier):
        self.repository = repository
        self.notifier = notifier

    def process(self, order):
        self.repository.save(order)
        self.notifier.notify(order.customer, "Order confirmed")


## Level 1 test: Use simple in-memory implementations
def test_order_processing_saves_and_notifies():
    # These are NOT mocks—they're real implementations with test-friendly behavior
    saved_orders = []
    notifications = []

    class InMemoryRepo:
        def save(self, order):
            saved_orders.append(order)

    class InMemoryNotifier:
        def notify(self, to, msg):
            notifications.append((to, msg))

    processor = OrderProcessor(InMemoryRepo(), InMemoryNotifier())
    processor.process(Order(customer="alice@test.com", items=["book"]))

    assert len(saved_orders) == 1
    assert notifications == [("alice@test.com", "Order confirmed")]
```

### Pattern: Pure Function Testing

```python
## Pure functions are Level 1's sweet spot
def calculate_shipping(weight_kg: float, distance_km: float, express: bool) -> float:
    base = weight_kg * 0.5 + distance_km * 0.01
    return base * 1.5 if express else base


def test_shipping_calculation():
    assert calculate_shipping(10, 100, express=False) == 6.0
    assert calculate_shipping(10, 100, express=True) == 9.0
```

### Pattern: Temporary Directories

```python
import tempfile
from pathlib import Path


def test_config_file_generation():
    with tempfile.TemporaryDirectory() as tmpdir:
        config_path = Path(tmpdir) / "config.yaml"

        generate_config(config_path, {"debug": True})

        content = config_path.read_text()
        assert "debug: true" in content
```

### What Level 1 Tests Prove

✅ Your business logic handles edge cases correctly\
✅ Your parsing/validation works\
✅ Your algorithms produce correct results\
✅ Your error handling works as expected

### What Level 1 Tests Cannot Prove

❌ That PostgreSQL accepts your queries\
❌ That the HTTP API returns what you expect\
❌ That the CLI binary does what the docs say\
❌ That the real system works end-to-end

**This is why Level 2 exists.**

---

## Level 2: Integration

### The Question This Level Answers

> **"Does our code correctly interact with real external dependencies?"**

Level 1 proved your logic is correct. Level 2 proves your code actually works with the real databases, binaries, and services it depends on.

### The Critical Requirement: Test Harnesses

> **Before writing ANY Level 2 test, you must identify or build the test harness for each external dependency.**

#### What Is a Test Harness?

A test harness is the infrastructure that lets you run tests against a real dependency in a controlled, repeatable way.

| Dependency Type | Harness Examples                                                 |
| --------------- | ---------------------------------------------------------------- |
| Database        | Docker container with test schema, or test database with cleanup |
| HTTP API        | Local mock server, or sandbox/staging environment                |
| CLI binary      | Installed binary with known version, fixture files               |
| File system     | Temp directories with fixture data                               |
| Message queue   | Docker container or embedded instance                            |

#### 🚨 THE RULE: If You Don't Know the Harness, STOP

> **If you cannot describe the test harness for a dependency, you MUST ask the user before proceeding.**

Do not guess. Do not assume. Ask:

```
I need to write integration tests for [dependency].

To proceed, I need to know:
1. What test harness exists or should I build?
2. How do I start/stop/reset it?
3. Where are fixture files or seed data?
4. What environment variables configure it?

Please provide this information or point me to existing test infrastructure.
```

### Pattern: Document Your Harnesses

Every project should have a `test/harnesses/` directory or documentation:

```
test/
├── harnesses/
│   ├── README.md           # Overview of all harnesses
│   ├── postgres.py         # Start/stop/reset Postgres container
│   ├── redis.py            # Start/stop/reset Redis container
│   └── hugo.py             # Verify Hugo binary, create fixture sites
├── fixtures/
│   ├── sample-site/        # Fixture data for Hugo tests
│   └── seed-data.sql       # Seed data for Postgres tests
└── integration/
    ├── test_database.py
    └── test_hugo_build.py
```

### Pattern: Real Database Testing

```python
import pytest
from harnesses.postgres import PostgresHarness


@pytest.fixture(scope="module")
def database():
    """Harness: Docker Postgres with schema applied"""
    harness = PostgresHarness()
    harness.start()
    harness.apply_schema("schema.sql")
    yield harness.connection_string
    harness.stop()


def test_user_repository_saves_and_retrieves(database):
    repo = UserRepository(database)

    user = User(email="test@example.com", name="Test User")
    repo.save(user)

    retrieved = repo.find_by_email("test@example.com")

    assert retrieved is not None
    assert retrieved.name == "Test User"
```

### Pattern: Real Binary Testing

```python
import subprocess
import tempfile
from pathlib import Path


@pytest.fixture
def hugo_site():
    """Harness: Temp directory with minimal Hugo site structure"""
    with tempfile.TemporaryDirectory() as tmpdir:
        site_dir = Path(tmpdir)
        # Create minimal Hugo site
        (site_dir / "config.toml").write_text('title = "Test Site"')
        (site_dir / "content").mkdir()
        (site_dir / "content" / "_index.md").write_text("# Home")
        yield site_dir


def test_hugo_builds_site(hugo_site):
    """Level 2: Verify Hugo binary actually builds our site structure"""
    result = subprocess.run(
        ["hugo", "--source", str(hugo_site)], capture_output=True, text=True
    )

    assert result.returncode == 0
    assert (hugo_site / "public" / "index.html").exists()
```

### Pattern: Real HTTP API Testing

```python
import pytest
import httpx
from harnesses.api_server import LocalAPIServer


@pytest.fixture(scope="module")
def api_server():
    """Harness: Local instance of the API under test"""
    server = LocalAPIServer(port=8089)
    server.start()
    server.wait_until_ready()
    yield server.base_url
    server.stop()


def test_api_creates_resource(api_server):
    response = httpx.post(f"{api_server}/resources", json={"name": "test-resource"})

    assert response.status_code == 201
    assert response.json()["name"] == "test-resource"
```

### What Level 2 Tests Prove

✅ PostgreSQL accepts your queries and returns expected results\
✅ Hugo builds your site structure correctly\
✅ The HTTP client handles real responses correctly\
✅ File operations work on real file systems

### What Level 2 Tests Cannot Prove

❌ That production credentials work\
❌ That third-party APIs behave the same in prod\
❌ That the full user workflow succeeds\
❌ That performance is acceptable under load

**This is why Level 3 exists.**

---

## Level 3: System / End-to-End

### The Question This Level Answers

> **"Does the complete system work the way users will actually use it?"**

Level 2 proved your integrations work locally. Level 3 proves the **entire system** works with **real credentials** against **real (test) environments**.

### The Critical Requirement: Credentials & Test Accounts

> **Before writing ANY Level 3 test, you must know where the credentials are and what test accounts exist.**

#### 🚨 THE RULE: No Credentials, No Level 3 Tests

> **If you do not have explicit information about test credentials and accounts, you MUST ask the user before proceeding.**

Do not guess. Do not use production credentials. Ask:

```
I need to write end-to-end tests that use [external service].

To proceed, I need to know:
1. Where are the test credentials stored? (env vars, secrets manager, etc.)
2. What test accounts/environments exist?
3. Are there rate limits or quotas on the test account?
4. How do I reset test data between runs?
5. Is there a staging/sandbox environment, or do tests run against production?

Please provide this information before I proceed with Level 3 tests.
```

### Pattern: Credential Management

```python
import os
import pytest

## Document where credentials come from
CREDENTIALS_SOURCE = """
Level 3 tests require these environment variables:
- STRIPE_TEST_API_KEY: From 1Password vault "Engineering/Test Credentials"
- SENDGRID_TEST_API_KEY: From .env.test (not committed)
- TEST_USER_EMAIL: test-automation@example.com
- TEST_USER_PASSWORD: In 1Password vault "Engineering/Test Credentials"
"""


@pytest.fixture(scope="session")
def stripe_client():
    api_key = os.environ.get("STRIPE_TEST_API_KEY")
    if not api_key:
        pytest.skip(f"STRIPE_TEST_API_KEY not set.\n{CREDENTIALS_SOURCE}")
    return StripeClient(api_key=api_key)


@pytest.fixture(scope="session")
def authenticated_user(browser):
    email = os.environ.get("TEST_USER_EMAIL")
    password = os.environ.get("TEST_USER_PASSWORD")
    if not email or not password:
        pytest.skip(f"Test user credentials not set.\n{CREDENTIALS_SOURCE}")

    # Log in once per session
    browser.goto("/login")
    browser.fill("[name=email]", email)
    browser.fill("[name=password]", password)
    browser.click("[type=submit]")
    browser.wait_for_url("/dashboard")

    yield browser
```

### Pattern: Full User Workflow

```python
def test_complete_purchase_workflow(authenticated_user, stripe_client):
    """
    Level 3: Complete user workflow with real services

    Prerequisites:
    - TEST_USER has a saved payment method in Stripe test mode
    - Product "test-product" exists in test catalog
    """
    browser = authenticated_user

    # User browses to product
    browser.goto("/products/test-product")

    # User adds to cart
    browser.click("[data-testid=add-to-cart]")
    browser.wait_for_selector("[data-testid=cart-count]:has-text('1')")

    # User checks out
    browser.goto("/checkout")
    browser.click("[data-testid=pay-now]")

    # Verify order completed
    browser.wait_for_url("/order-confirmation")
    order_id = browser.locator("[data-testid=order-id]").text_content()

    # Verify in Stripe
    charges = stripe_client.charges.list(limit=1)
    assert charges.data[0].metadata["order_id"] == order_id
```

### Pattern: CLI End-to-End

```python
import subprocess
import os


def test_cli_full_workflow():
    """
    Level 3: CLI works with real credentials against real services

    Prerequisites:
    - LHCI_TOKEN set in environment
    - Test site deployed at https://staging.example.com
    """
    # Verify prerequisites
    if not os.environ.get("LHCI_TOKEN"):
        pytest.skip("LHCI_TOKEN not set")

    result = subprocess.run(
        [
            "hugolit",
            "run",
            "--url",
            "https://staging.example.com",
            "--upload",  # Uploads to LHCI server
        ],
        capture_output=True,
        text=True,
        env={**os.environ, "CI": "true"},
    )

    assert result.returncode == 0
    assert "Report uploaded" in result.stdout
```

### What Level 3 Tests Prove

✅ Real credentials work\
✅ Real third-party APIs behave as expected\
✅ The full user workflow succeeds\
✅ All integrations work together in a real environment

### When Level 3 Tests Fail

Level 3 failures are the most serious because they mean **users will experience failures**. When a Level 3 test fails:

1. **Check credentials**: Did they expire? Get rotated?
2. **Check third-party status**: Is the external service down?
3. **Check test data**: Did seed data get corrupted or deleted?
4. **Then** look at your code

---

## The Testing Decision Protocol

When you need to test a feature, execute these phases IN ORDER.

### Phase 1: List the Guarantees You Need

Before writing any test, list what you need to guarantee:

```markdown
### Guarantees Needed for "User Registration"

1. Email validation logic rejects invalid formats
2. Password hashing produces correct hashes
3. Database correctly stores and retrieves user records
4. Email service actually sends the welcome email
5. Complete signup flow works from the user's perspective
```

### Phase 2: Assign Each Guarantee to a Level

| Guarantee                       | Level   | Why This Level?                       |
| ------------------------------- | ------- | ------------------------------------- |
| Email validation logic          | Level 1 | Pure function, no dependencies        |
| Password hashing                | Level 1 | Pure function, deterministic          |
| Database stores/retrieves users | Level 2 | Needs real database                   |
| Email service sends email       | Level 2 | Needs email harness (Mailhog/sandbox) |
| Complete signup flow            | Level 3 | Needs real credentials, real services |

### Phase 3: Identify Harnesses (Level 2) and Credentials (Level 3)

Before writing Level 2 or Level 3 tests:

**For Level 2, document your harnesses:**

```markdown
### Test Harnesses Required

- **PostgreSQL**: Docker container via `docker-compose.test.yml`
  - Start: `docker-compose -f docker-compose.test.yml up -d postgres`
  - Reset: `docker-compose exec postgres psql -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"`
- **Email (Mailhog)**: Docker container captures all outgoing email
  - Start: `docker-compose -f docker-compose.test.yml up -d mailhog`
  - API: `http://localhost:8025/api/v2/messages`
```

**For Level 3, document your credentials:**

```markdown
### Test Credentials Required

- **TEST_USER_EMAIL**: `test-automation@example.com`
- **TEST_USER_PASSWORD**: In 1Password vault "Engineering/Test Credentials"
- **SENDGRID_API_KEY**: In `.env.test` (get from team lead)

### Test Environment

- Staging URL: `https://staging.example.com`
- Test Stripe account: Uses `sk_test_*` keys (auto-set in staging)
```

### Phase 4: Write Tests Bottom-Up

Start with Level 1. Only move to Level 2 when Level 1 is complete. Only move to Level 3 when Level 2 is complete.

```python
## test/unit/test_registration.py (Level 1)
def test_email_validation_rejects_invalid():
    assert validate_email("notanemail") is False
    assert validate_email("also@invalid") is False
    assert validate_email("valid@example.com") is True


def test_password_hashing_is_deterministic():
    hash1 = hash_password("secret123")
    hash2 = hash_password("secret123")
    assert verify_password("secret123", hash1)
    assert verify_password("secret123", hash2)


## test/integration/test_registration.py (Level 2)
def test_user_repository_creates_user(database):
    repo = UserRepository(database)
    user = repo.create(email="new@example.com", password_hash="hash123")

    retrieved = repo.find_by_email("new@example.com")
    assert retrieved.id == user.id


def test_email_service_sends_welcome_email(mailhog):
    service = EmailService(smtp_url=mailhog.smtp_url)
    service.send_welcome("recipient@example.com")

    messages = mailhog.get_messages()
    assert len(messages) == 1
    assert messages[0]["To"] == "recipient@example.com"


## test/e2e/test_registration.py (Level 3)
def test_complete_signup_workflow(browser, test_credentials):
    browser.goto("https://staging.example.com/signup")
    browser.fill("[name=email]", "e2e-test@example.com")
    browser.fill("[name=password]", "SecurePass123!")
    browser.click("[type=submit]")

    browser.wait_for_url("/welcome")
    assert "Welcome" in browser.title()
```

---

## Anti-Patterns

### Anti-Pattern: Mocking External Systems

```python
## ❌ NEVER DO THIS
@patch("myapp.database.PostgresClient")
def test_saves_user(mock_db):
    mock_db.save.return_value = {"id": 1}
    result = save_user({"email": "test@example.com"})
    mock_db.save.assert_called_once()  # What did we prove? NOTHING.
```

**Instead**: Use dependency injection at Level 1, real database at Level 2.

### Anti-Pattern: Skipping Levels

```python
## ❌ Going straight to E2E without unit/integration coverage
def test_full_checkout():
    # If this fails, you have no idea if it's:
    # - Your pricing logic (Level 1)
    # - Your database queries (Level 2)
    # - Your payment integration (Level 2)
    # - The third-party service (Level 3)
    # - Your test credentials (Level 3)
    ...
```

**Instead**: Build confidence from the bottom up.

### Anti-Pattern: Guessing at Harnesses or Credentials

```python
## ❌ Assuming a database exists
def test_integration():
    db = connect("postgresql://localhost:5432/test")  # Does this exist? Who knows!


## ❌ Hardcoding credentials
def test_e2e():
    stripe = Stripe(api_key="sk_test_abc123")  # Will this work? For how long?
```

**Instead**: Document harnesses and credential sources explicitly. Ask if you don't know.

### Anti-Pattern: Testing Implementation

```python
## ❌ Testing HOW, not WHAT
def test_uses_correct_query():
    repo = UserRepository(mock_db)
    repo.find_active_users()
    mock_db.query.assert_called_with("SELECT * FROM users WHERE active = true")


## ✅ Test the BEHAVIOR
def test_returns_only_active_users(database):
    seed_data(
        database,
        [
            {"email": "active@test.com", "active": True},
            {"email": "inactive@test.com", "active": False},
        ],
    )

    repo = UserRepository(database)
    users = repo.find_active_users()

    assert len(users) == 1
    assert users[0].email == "active@test.com"
```

---

## Quick Reference: When to Use Each Level

| If you need to verify...        | Use Level |
| ------------------------------- | --------- |
| Business logic correctness      | 1         |
| Parsing/validation              | 1         |
| Algorithm output                | 1         |
| Error handling                  | 1         |
| Database queries work           | 2         |
| HTTP calls work                 | 2         |
| CLI binary works                | 2         |
| File I/O works                  | 2         |
| Full user workflow              | 3         |
| Real credentials work           | 3         |
| Production-like environment     | 3         |
| Third-party service integration | 3         |

---

## Checklist Before Declaring Tests Complete

- [ ] All critical guarantees have tests at the appropriate level
- [ ] Level 1 tests use DI, not mocking
- [ ] Level 2 harnesses are documented and reproducible
- [ ] Level 3 credentials are documented (not hardcoded)
- [ ] Tests verify behavior, not implementation
- [ ] Fast-failing: environment checks run first
- [ ] Progress tests in `specs/`, regression tests in `test/`
- [ ] All regression tests pass

---

## When You're Stuck: The Questions to Ask

**For Level 1:**

> "Can I verify this behavior using only the test runner, language primitives, and dependency injection?"
>
> If no → move to Level 2

**For Level 2:**

> "What test harness do I need? How do I start/stop/reset it?"
>
> If you don't know → **STOP AND ASK THE USER**

**For Level 3:**

> "Where are the credentials? What test accounts exist?"
>
> If you don't know → **STOP AND ASK THE USER**

---

*Remember: The goal is not "passing tests." The goal is **justified confidence that your code works in the real world**. Every test should move you closer to that confidence. If a test doesn't, delete it.*

---

*Remember: The goal is not "passing tests." The goal is **justified confidence that your code works in the real world**. Every test should move you closer to that confidence. If a test doesn't, delete it.*
