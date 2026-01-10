# TRD: [Name of Technical Change]

> **Purpose**: Documents the explored solution after discovery and serves as an authoritative blueprint for implementation.
>
> - Written AFTER user and agent explore the solution space together
> - Captures the agreed-upon approach to solving a technical problem
> - Authoritative: Changes to solution approach require user approval
> - Guides implementation: Spawns work items (features/stories) with binding acceptance criteria
> - No size constraints, no state tracking (OPEN/IN PROGRESS/DONE)
> - Can exist at: capability level (`.../capability/[Name of Technical Change].trd.md`) or feature level (`.../feature/[Name of Technical Change].trd.md`)

## Required Sections

A TRD is ready for decomposition into work items when all sections below contain complete information. Empty or placeholder content indicates the TRD is not ready.

| Section             | Purpose                                                                 |
| ------------------- | ----------------------------------------------------------------------- |
| Problem Statement   | Root cause of the technical limitation, not just symptoms               |
| Validation Strategy | Guarantees mapped to test levels with BDD scenarios                     |
| Test Infrastructure | Harnesses (Level 2) and credentials (Level 3) documented                |
| Solution Design     | Technical approach without prescribing implementation details           |
| Dependencies        | Work items, runtime requirements, and test infrastructure prerequisites |
| Pre-Mortem          | Risks identified with likelihood, impact, and mitigation                |

## Testing Methodology

This TRD follows the three-tier testing methodology:

- **Level 1 (Unit)**: Pure logic with dependency injection. No external systems.
- **Level 2 (Integration)**: Real infrastructure via test harnesses. No mocking.
- **Level 3 (E2E)**: Real credentials and services. Full user workflows.

**Build confidence bottom-up**: Level 1 → Level 2 → Level 3.

Before implementation, agents MUST consult:

- `/testing` — Foundational principles, decision protocol, anti-patterns
- `/testing-python` or `/testing-typescript` — Language-specific patterns and fixtures

## Problem Statement

### Technical Problem

When [user] tries to [task], they encounter [limitation]
because [underlying technical cause], which blocks [desired capability].

### Current Pain

- **Symptom**: [What users work around]
- **Root Cause**: [Technical reason this pain exists]
- **Impact**: [How this affects workflow]
- **Validation Challenge**: [What makes this hard to test/verify?]

## Project-Specific Constraints

Document constraints beyond standard development practices. If no special constraints apply, write "None identified" and proceed.

Constraints worth documenting include:

- External API dependencies with rate limits, auth requirements, or availability concerns
- Cross-system ID matching or data synchronization requirements
- Security restrictions on credential storage or transmission
- Regulatory or compliance requirements affecting implementation
- Legacy system integration constraints

| Constraint                                    | Impact on Implementation        | Impact on Testing                            |
| --------------------------------------------- | ------------------------------- | -------------------------------------------- |
| [e.g., Trakt.tv API rate limit: 1000 req/day] | Must implement request batching | Level 3 tests limited to 10 requests per run |

## Solution Design

### Technical Solution

```
Implement [technical solution] that enables [user] to [new capability]
through [interaction pattern], resulting in [improved workflow].
```

### Technical Architecture

#### Components

List major components and their responsibilities. Do not prescribe implementation details.

| Component               | Responsibility                              |
| ----------------------- | ------------------------------------------- |
| [e.g., PriceCalculator] | Applies discounts and computes final prices |
| [e.g., OrderRepository] | Persists and retrieves order records        |

#### Data Flow

Describe how data moves through the system for the primary use case.

```
[Input] → [Component A] → [Component B] → [Output/Storage]
```

#### Key Interfaces

Define boundaries between components. These become integration points for Level 2 testing.

| Interface                 | Input        | Output             | Notes              |
| ------------------------- | ------------ | ------------------ | ------------------ |
| [e.g., Repository.save()] | Order object | Persisted order ID | Must be idempotent |

## Validation Strategy

### Guarantees Required

List each behavior the solution must guarantee. Assign to the lowest test level that can verify it.

| #   | Guarantee                                                  | Level | Rationale                                            |
| --- | ---------------------------------------------------------- | ----- | ---------------------------------------------------- |
| G1  | [e.g., Price calculation handles edge cases]               | 1     | Pure arithmetic logic, no external dependencies      |
| G2  | [e.g., Database persists order records]                    | 2     | Requires real PostgreSQL; use Docker harness         |
| G3  | [e.g., Payment processing completes with real credentials] | 3     | Requires Stripe test account and real API connection |

### BDD Scenarios

Each scenario must reference a guarantee from the table above by number. Every guarantee requires at least one scenario.

**Scenario: [Descriptive name] [G1]**

- **Given** [Initial state or precondition]
- **When** [Action performed by user or system]
- **Then** [Observable, verifiable outcome]

**Scenario: [Descriptive name] [G2]**

- **Given** [Initial state or precondition]
- **When** [Action performed by user or system]
- **Then** [Observable, verifiable outcome]

## Test Infrastructure

### Level 2: Test Harnesses

Document every external dependency that Level 2 tests require. If a harness does not exist, note that it must be built before implementation.

| Dependency  | Harness Type     | Setup Command                               | Reset Command                                        |
| ----------- | ---------------- | ------------------------------------------- | ---------------------------------------------------- |
| PostgreSQL  | Docker container | `docker-compose -f test.yml up -d postgres` | `docker-compose exec postgres psql -c "TRUNCATE..."` |
| Redis       | Docker container | `docker-compose -f test.yml up -d redis`    | `docker-compose exec redis redis-cli FLUSHALL`       |
| Hugo binary | Local install    | Verify: `hugo version`                      | N/A                                                  |

### Level 3: Credentials and Test Accounts

Document every credential or test account that Level 3 tests require. Include rotation schedule if applicable.

| Credential        | Environment Variable                    | Source                                  | Notes                                  |
| ----------------- | --------------------------------------- | --------------------------------------- | -------------------------------------- |
| Stripe test key   | `STRIPE_TEST_API_KEY`                   | 1Password: Engineering/Test Credentials | Rotates quarterly                      |
| Test user account | `TEST_USER_EMAIL`, `TEST_USER_PASSWORD` | 1Password: Engineering/Test Credentials | Account: <test-automation@example.com> |

### Infrastructure Gaps

If any required harness or credential source is unknown, list it here. **A TRD with unresolved infrastructure gaps is not ready for implementation.**

| Gap                                                      | Blocking       |
| -------------------------------------------------------- | -------------- |
| [e.g., Message queue harness—RabbitMQ or in-memory?]     | Implementation |
| [e.g., Trakt.tv OAuth test credentials location unknown] | Level 3 tests  |

## Reference Test Implementations

Provide example test code at each applicable level. These serve as implementation guidance, not exhaustive coverage.

Adapt examples to the project's primary language. The patterns below use Python; for TypeScript projects, consult `/testing-typescript` for equivalent patterns using Vitest fixtures and async/await syntax.

### Level 1 Example (Unit)

```python
# Pure logic test—no external dependencies, no mocking
def test_price_calculation_applies_discount():
    # Given
    base_price = 100.00
    discount_percent = 15

    # When
    final_price = calculate_discounted_price(base_price, discount_percent)

    # Then
    assert final_price == 85.00
```

### Level 2 Example (Integration)

```python
# Integration test—requires database harness
@pytest.fixture
def database():
    harness = PostgresHarness()
    harness.start()
    harness.apply_migrations()
    yield harness.connection_string
    harness.stop()


def test_order_repository_persists_order(database):
    # Given
    repo = OrderRepository(database)
    order = Order(customer_id="cust-123", total=85.00)

    # When
    repo.save(order)
    retrieved = repo.find_by_id(order.id)

    # Then
    assert retrieved.total == 85.00
```

### Level 3 Example (E2E)

```python
# E2E test—requires real credentials
CREDENTIALS_SOURCE = """
Requires:
- STRIPE_TEST_API_KEY from 1Password vault
- TEST_USER credentials from 1Password vault
"""


def test_complete_checkout_workflow(browser, stripe_client):
    # Given
    browser.goto("/products/test-item")

    # When
    browser.click("[data-testid=add-to-cart]")
    browser.click("[data-testid=checkout]")
    browser.click("[data-testid=pay-now]")

    # Then
    browser.wait_for_url("/confirmation")
    order_id = browser.locator("[data-testid=order-id]").text()

    # Verify in Stripe
    charge = stripe_client.charges.list(limit=1).data[0]
    assert charge.metadata["order_id"] == order_id
```

## Dependencies

### Work Item Dependencies

Work items that must be completed before this TRD can be implemented.

| Dependency                        | Location                   | Status                               |
| --------------------------------- | -------------------------- | ------------------------------------ |
| [e.g., Authentication capability] | `specs/capabilities/auth/` | Complete / In Progress / Not Started |

### Runtime Dependencies

Tools, libraries, and services the solution requires at runtime.

| Dependency | Version Constraint | Purpose             |
| ---------- | ------------------ | ------------------- |
| Node.js    | >=18.0.0           | Runtime environment |
| PostgreSQL | >=14               | Primary data store  |
| Stripe API | v2023-10           | Payment processing  |

### Test Infrastructure Dependencies

Dependencies specific to running the test suite. Cross-reference with Test Infrastructure section.

| Dependency    | Required For                 | Provided By                     |
| ------------- | ---------------------------- | ------------------------------- |
| Docker        | Level 2 harnesses            | Local install or CI environment |
| 1Password CLI | Level 3 credential injection | `op` binary + vault access      |

## Pre-Mortem Analysis

### Technical Risks

| Risk                                         | Likelihood | Impact | Mitigation                                         |
| -------------------------------------------- | ---------- | ------ | -------------------------------------------------- |
| [e.g., Third-party API rate limits exceeded] | Medium     | High   | Implement backoff; use sandbox with higher limits  |
| [e.g., Test credentials expire mid-sprint]   | Low        | High   | Document rotation schedule; set calendar reminders |

### Test Infrastructure Risks

| Risk                                   | Likelihood | Impact | Mitigation                                                 |
| -------------------------------------- | ---------- | ------ | ---------------------------------------------------------- |
| [e.g., Docker harness startup is slow] | Medium     | Medium | Use persistent test containers; parallelize where possible |
| [e.g., Seed data becomes stale]        | Medium     | Medium | Automate seed refresh; version seed data with schema       |

### Integration Risks

| Risk                                           | Likelihood | Impact | Mitigation                          |
| ---------------------------------------------- | ---------- | ------ | ----------------------------------- |
| [e.g., Upstream API changes break integration] | Low        | High   | Pin API version; add contract tests |

## Readiness Criteria

A reviewing agent should verify the following before approving this TRD for decomposition:

1. **Problem Statement**: Identifies root cause, not just symptoms. Explains why current state is inadequate.

2. **Validation Strategy**:
   - Every guarantee has a unique identifier (G1, G2, etc.)
   - Every guarantee is assigned to exactly one test level
   - Level assignment rationale is coherent (pure logic → L1, real infra → L2, real credentials → L3)
   - Every BDD scenario references a guarantee by number (e.g., [G1])
   - Every guarantee has at least one corresponding BDD scenario
   - Scenarios use strict Given/When/Then format

3. **Test Infrastructure**:
   - All Level 2 dependencies have documented harnesses with setup/reset commands
   - All Level 3 dependencies have documented credential sources
   - No unresolved infrastructure gaps remain

4. **Dependencies**:
   - Work item dependencies are linked to actual specs
   - Runtime dependencies include version constraints
   - Test infrastructure dependencies are cross-referenced

5. **Pre-Mortem**:
   - At least one risk identified per category (technical, test infrastructure, integration)
   - Each risk has mitigation strategy
