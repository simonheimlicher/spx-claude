---
name: testing
description: Learn how to test code without mocking. Use when learning testing approach or before writing tests.
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

# Test Strategy (Foundational Skill)

You are the **foundational test strategy authority**. All other skills—architect, coder, reviewer—MUST consult you before making decisions about testing.

---

## The Overarching Question

> **What evidence do I need to convince the user that my code correctly implements the specification?**

Every test exists to answer this question. Tests are not bureaucracy—they are **evidence** that your code works. If a test doesn't provide evidence, delete it.

---

## Tests Serve Three Purposes

| Purpose          | What It Means                     | Example                                                    |
| ---------------- | --------------------------------- | ---------------------------------------------------------- |
| **Code quickly** | Fast feedback loop while building | Run Level 1 tests in <1 second to verify logic as you code |
| **Evidence**     | Prove the spec is implemented     | Show user that acceptance criteria are met                 |
| **Debug**        | Find bugs when regressions occur  | Named test cases point directly to the broken behavior     |

---

## The Cardinal Rule: No Mocking

> **Mocking is always wrong. There is no exception.**

Mocking gives you a test that passes while your production code fails. This is worse than no test at all.

If you feel you need to mock:

1. **Redesign** using dependency injection with real in-memory implementations, OR
2. **Test at a different level**—push to Level 2 or 3 where real dependencies are available

---

## The Three Levels

```
┌─────────────────────┐
│      LEVEL 3        │  "Does it work in the real world?"
│   System / E2E      │  Real credentials, real services
│                     │  Full user workflows
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│      LEVEL 2        │  "Does it work with real infrastructure?"
│    Integration      │  Real binaries, real databases
│                     │  Test harnesses required
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│      LEVEL 1        │  "Is our logic correct?"
│    Unit / Pure      │  Standard dev environment only
│                     │  DI with real in-memory implementations
└─────────────────────┘
```

### Level 1: Standard Dev Environment

**The question**: Is our logic correct, independent of any external system?

**Allowed:**

| Resource           | Examples                               | Why                     |
| ------------------ | -------------------------------------- | ----------------------- |
| Test runner        | pytest, vitest, jest, go test          | Dev environment         |
| Temp directories   | `tempfile.mkdtemp()`, `os.tmpdir()`    | OS-provided, isolated   |
| Environment vars   | Set/read env vars within test          | Language runtime        |
| Standard dev tools | git, node, npm (tool), python, curl    | CI without setup        |
| DI implementations | In-memory repositories, stub notifiers | Real code, test-focused |
| Factories/builders | Generate test data programmatically    | Reproducible            |

**Forbidden:**

| Resource                  | Why                                      |
| ------------------------- | ---------------------------------------- |
| Real databases            | Level 2                                  |
| Real HTTP APIs            | Level 2 or 3                             |
| Project-specific binaries | Level 2 (ffmpeg, hugo, custom tools)     |
| Installing dependencies   | `npm install`, `pip install` are Level 2 |
| Mocking external systems  | Never. Redesign with DI instead.         |

**Critical filesystem rule**: All Level 1 tests MUST use OS-provided temporary directories exclusively. Never write outside temp directories.

### Level 2: Project-Specific Dependencies

**The question**: Does our code correctly interact with real external dependencies?

**Covers:**

- Project-specific tools: Hugo, Caddy, FFmpeg, custom binaries
- Project-specific build tools: Make, Gradle, Maven
- Containerized services: Docker databases, message queues

**Required before writing**: Document the test harness for each dependency.

**If you don't know the harness, STOP and ask:**

> I need to write integration tests for [dependency].
>
> To proceed, I need to know:
>
> 1. What test harness exists or should I build?
> 2. How do I start/stop/reset it?
> 3. Where are fixture files or seed data?
> 4. What environment variables configure it?

### Level 3: Real Environment

**The question**: Does the complete system work the way users will actually use it?

**Covers:**

- Real credentials against real (test) environments
- Third-party services in production or staging
- Browser-based testing for web applications
- Complete user workflows end-to-end

**Required before writing**: Document credentials and test accounts.

**If you don't know the credentials, STOP and ask:**

> I need to write end-to-end tests that use [external service].
>
> To proceed, I need to know:
>
> 1. Where are the test credentials stored?
> 2. What test accounts/environments exist?
> 3. Are there rate limits or quotas?
> 4. How do I reset test data between runs?

---

## The Three Dimensions

Every test level offers different tradeoffs across three orthogonal dimensions:

### 1. Detection: What Bugs Can This Level Find?

Each level has a **different lens**—not progressively broader, but genuinely different:

| Level  | Detects                                             | Cannot Detect                                 |
| ------ | --------------------------------------------------- | --------------------------------------------- |
| **L1** | Algorithmic bugs, edge cases, invariant violations  | Race conditions, integration mismatches       |
| **L2** | Race conditions, integration mismatches, contracts  | Pure logic bugs hidden in stack, graphical UX |
| **L3** | Workflow breaks, UX failures, real-world edge cases | Algorithmic bugs, intermittent races          |

**Key insight**: L1 property-based tests catch algorithmic bugs that L3 almost never detects. L2 randomized harnesses catch race conditions that L1 can't see.

### 2. Validity: What Does a Passing Test Prove?

| Level  | A Passing Test Proves                           | False Confidence Risk                               |
| ------ | ----------------------------------------------- | --------------------------------------------------- |
| **L1** | The isolated logic is correct for tested inputs | Low—tests exactly what it claims                    |
| **L2** | Components integrate correctly with the harness | Medium—depends on harness fidelity to production    |
| **L3** | The workflow succeeds in the test environment   | High—can pass while feature is broken in production |

**Key insight**: "E2E tests pass" does NOT mean "it works for users." L3 has the highest false confidence risk.

### 3. Cost: What Investment Does This Level Require?

| Aspect          | L1                          | L2                                             | L3                                             |
| --------------- | --------------------------- | ---------------------------------------------- | ---------------------------------------------- |
| **Upfront**     | Low (no setup, just code)   | Medium (harnesses, fixtures, containers)       | High (real env, credentials, test accounts)    |
| **Per-run**     | Milliseconds                | Seconds to minutes                             | Minutes to hours                               |
| **Maintenance** | Low (stable, deterministic) | Medium (harness evolution, dependency updates) | High (brittle to UI changes, flaky, env drift) |

**Key insight**: A test cheap to write can be expensive to maintain. L3 tests often become "tests you're afraid to touch."

---

## Where Evidence Lives

Some outcomes can only be proven at specific levels:

| Outcome                             | Minimum Level | Best Combination                                  |
| ----------------------------------- | ------------- | ------------------------------------------------- |
| Algorithm correctness               | 1             | L1 with property-based testing                    |
| Parser handles grammar              | 1             | L1 typical + edges + properties                   |
| User can export data as CSV         | 1             | L1 with temp directory (file I/O is Level 1)      |
| Database query returns correct data | 2             | L2 integration; L1 only if query logic is complex |
| CLI binary behaves correctly        | 2             | L2 with fixture files                             |
| API contract is honored             | 2 or 3        | L2 against test server; L3 against staging        |
| Clipboard works in browsers         | 3 only        | L3 in real browser; L1/L2 prove nothing           |
| Payment flow completes              | 3 only        | L3 with test credentials                          |

### The Clipboard Example

A "copy to clipboard" React component:

- **L1 test** (component renders): Proves nothing about clipboard functionality
- **L3 test** (actually copies in browser): Proves the feature works

**Best combination**: Skip L1/L2 entirely, write L3 tests in target browsers.

### The Pricing Engine Example

A complex pricing calculation with discounts, taxes, promotions:

- **L2 only** (integration test checks total): Knows IF wrong, not WHERE the bug is
- **L1 + L2**: L1 isolates which rule is broken; L2 confirms end-to-end

**Best combination**: L1 with full 4-part progression + L2 for integration.

---

## Add Lower Levels for Debuggability

When Level 2 or 3 is required for evidence, add Level 1 tests ONLY if:

1. **The code is complex**—your logic (algorithms, parsers, rules), not library wiring
2. **Debugging will be hard**—when the higher-level test fails, will you know where to look?
3. **Property-based testing adds value**—would generated inputs find edge cases?

| Scenario                                        | Add Level 1? | Reason                                        |
| ----------------------------------------------- | ------------ | --------------------------------------------- |
| Integration test fails on a complex algorithm   | YES          | Level 1 isolates the algorithm                |
| Integration test fails on argparse flag parsing | NO           | Trust argparse; check your usage              |
| E2E test fails on payment flow                  | MAYBE        | If payment calculation logic is complex, yes  |
| E2E test fails on clipboard                     | NO           | It's a browser API call, nothing to unit test |

---

## Trust the Library

Libraries like argparse, Zod, pydantic, js-yaml are battle-tested. Don't test that they work—test YOUR logic.

**Don't test library behavior:**

```python
# BAD: Tests argparse, not your code
def test_verbose_flag_is_parsed():
    args = parser.parse_args(["--verbose"])
    assert args.verbose is True
```

**Do test your behavior:**

```python
# GOOD: Tests your logic that uses the parsed result
def test_verbose_mode_produces_detailed_output():
    output = run_command(verbose=True)
    assert "DEBUG:" in output
```

---

## Randomized Test Harnesses

Always ask: **What is the data structure that describes the fixture?**

Example: When testing directory tree operations:

1. The underlying structure is a DAG (directed acyclic graph)
2. Generate a DAG data structure first
3. Test your logic against the DAG at Level 1
4. Convert to actual directories in temp directory for Level 2

### Seeding

- Seeds should derive from system time (different every run)
- Show seed on failure for reproduction
- This maximizes variety while enabling reproduction

---

## The 4-Part Progression

Organize tests at any level to serve all three purposes (code/evidence/debug):

### Part 0: Shared Test Values

Create a test values file with named, typed data:

```typescript
export const TYPICAL = {
  BASIC: { input: "simple", expected: 42 },
  COMPLEX: { input: "with-flags", expected: 100 },
} as const;

export const EDGES = {
  EMPTY: { input: "", expected: 0 },
  MAX: { input: "x".repeat(1000), expected: "ERROR" },
} as const;
```

### Part 1: Named Typical Cases

One `it()` per category. When test fails, you know EXACTLY which case.

### Part 2: Named Edge Cases

One `it()` per boundary condition. Each boundary is independently debuggable.

### Part 3: Systematic Coverage Loop

Loop over all known cases. Should ONLY fail if Parts 1-2 missed a category.

### Part 4: Generated/Property-Based

Reproducible via seed. Escalate from debuggable loops to comprehensive properties.

### Level Breadth

| Level   | Typical Parts      | Why                                   |
| ------- | ------------------ | ------------------------------------- |
| Level 1 | All 4 parts        | Cheapest—can afford full breadth      |
| Level 2 | Parts 1-2, maybe 3 | More expensive—focus on key scenarios |
| Level 3 | Part 1 only        | Most expensive—critical flows only    |

---

## Progress Tests vs Regression Tests

| Location                                | Name             | May Fail? | Purpose                          |
| --------------------------------------- | ---------------- | --------- | -------------------------------- |
| `specs/.../tests/`                      | Progress tests   | YES       | TDD red-green during development |
| `tests/{level}/{capability}/{feature}/` | Regression tests | NO        | Protect working functionality    |

**The invariant**: The regression test suite MUST ALWAYS PASS.

**Graduation**: When a story is complete, tests graduate from `specs/work/doing/{capability}/{feature}/{story}/tests/` to `tests/{level}/{capability}/{feature}/`.

Stories are ephemeral—they disappear as a directory level, becoming test files within the feature.

---

## Test Infrastructure

Keep test infrastructure separate from tests. Categories:

### 1. Test Environment Context Managers

Shared utilities (like `withTestEnv`) that handle:

- Seeding and reproducibility
- Temp directory lifecycle
- Environment variable isolation
- Shared setup/teardown

### 2. Containerized Services

Local databases, dev servers, message queues. Managed via docker-compose.

### 3. Fixtures

Named test values (TYPICAL, EDGES)—static data collections.

### 4. Generators

Randomized data generation with seeding for reproducibility.

---

## Dependency Injection Pattern

```python
# BAD: Hardcoded dependency, requires mocking
class OrderProcessor:
    def process(self, order):
        db = PostgresDatabase()  # Hardcoded!
        db.save(order)


# GOOD: Injected dependencies, testable without mocks
class OrderProcessor:
    def __init__(self, repository):
        self.repository = repository

    def process(self, order):
        self.repository.save(order)


# Level 1 test: Real in-memory implementation
def test_order_processing_saves():
    saved = []

    class InMemoryRepo:
        def save(self, order):
            saved.append(order)

    processor = OrderProcessor(InMemoryRepo())
    processor.process(Order(customer="alice"))

    assert len(saved) == 1
```

---

## Quick Reference: Level Selection

| Evidence needed for...  | Level |
| ----------------------- | ----- |
| Business logic          | 1     |
| Parsing/validation      | 1     |
| Algorithm output        | 1     |
| File I/O with temp dirs | 1     |
| Database queries        | 2     |
| HTTP calls              | 2     |
| CLI binary behavior     | 2     |
| Full user workflow      | 3     |
| Real credentials        | 3     |
| Browser behavior        | 3     |
| Third-party services    | 3     |

---

## Checklist Before Declaring Tests Complete

- [ ] Evidence exists at the level where it can be proven
- [ ] No mocking anywhere—DI with real implementations
- [ ] Level 2 harnesses documented
- [ ] Level 3 credentials documented (not hardcoded)
- [ ] Tests verify behavior, not implementation
- [ ] Regression tests all pass

---

## When You're Stuck

**For Level 1:**

> Can I verify this behavior using only the test runner, language primitives, temp dirs, and DI?
>
> If no → move to Level 2

**For Level 2:**

> What test harness do I need?
>
> If you don't know → **STOP AND ASK THE USER**

**For Level 3:**

> Where are the credentials?
>
> If you don't know → **STOP AND ASK THE USER**

---

*The goal is not "passing tests" or "high coverage"—it's **justified confidence that your code works in the real world**.*
