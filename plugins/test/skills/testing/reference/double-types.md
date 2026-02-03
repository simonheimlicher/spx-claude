# Test Double Types (Meszaros Taxonomy)

This reference explains the different types of test doubles. **Only use test doubles when an exception case applies** (see Stage 5 in SKILL.md).

---

## Overview

| Double Type | Purpose                           | When to Use                                        |
| ----------- | --------------------------------- | -------------------------------------------------- |
| **Stub**    | Returns predetermined responses   | Testing failure modes, specific response scenarios |
| **Spy**     | Records calls for verification    | Testing interaction protocols, observability       |
| **Fake**    | Simplified working implementation | High-speed combinatorial testing                   |
| **Mock**    | Strict expectation verification   | Protocol sequence assertions (use sparingly)       |
| **Dummy**   | Placeholder that's never called   | Satisfying type requirements                       |

---

## Stub

A stub returns predetermined responses. It doesn't record calls or verify behavior—it just provides controlled outputs.

**Use for:** Exception 1 (Failure Modes), Exception 4 (Safety), Exception 7 (Contract Testing)

### When to Use Stubs

- Testing how your code handles specific error responses
- Simulating timeouts, rate limits, or network failures
- Replacing destructive operations (payments, deletions)
- Contract testing with known response variants

### Example: Failure Mode Testing

```typescript
// Testing retry logic under timeout conditions
test("retries on timeout", async () => {
  let attempts = 0;

  // Stub that fails twice, then succeeds
  const client: HttpClient = {
    async fetch(url) {
      attempts++;
      if (attempts < 3) {
        throw new TimeoutError("Request timed out");
      }
      return { status: 200, body: "ok" };
    },
  };

  const result = await fetchWithRetry(url, client);

  expect(attempts).toBe(3);
  expect(result.status).toBe(200);
});
```

### Example: Safety (Payment Stub)

```typescript
// Stub payment provider to avoid real charges
test("processes refund for cancelled order", async () => {
  const refunds: Array<{ amount: number; reason: string }> = [];

  const paymentProvider: PaymentProvider = {
    async refund(chargeId, amount, reason) {
      // Record the call but don't actually refund
      refunds.push({ amount, reason });
      return { refundId: "refund_123", status: "succeeded" };
    },
  };

  await cancelOrder(order, paymentProvider);

  expect(refunds).toEqual([{ amount: 99.99, reason: "order_cancelled" }]);
});
```

### Example: Contract Testing

```typescript
// Stub that returns documented response variants
const responseVariants = [
  { status: 200, body: { data: [{ id: 1 }] } },
  { status: 200, body: { data: [], next_cursor: "abc" } },
  { status: 429, body: { retry_after: 60 } },
  { status: 503, body: { message: "Service unavailable" } },
];

for (const variant of responseVariants) {
  test(`handles ${variant.status} response`, async () => {
    const client: HttpClient = {
      async fetch() {
        return variant;
      },
    };

    // Verify our code can parse this response
    expect(() => parseApiResponse(await client.fetch("/"))).not.toThrow();
  });
}
```

---

## Spy

A spy records calls for later verification. Unlike a stub, the primary purpose is to observe WHAT was called, not to control the response.

**Use for:** Exception 2 (Interaction Protocols), Exception 6 (Observability)

### When to Use Spies

- Verifying request parameters (headers, body, query params)
- Ensuring operations are batched correctly
- Checking idempotency keys are sent
- Verifying call ordering or count

### Example: Observability

```typescript
// Verify idempotency key is included in requests
test("includes idempotency key in payment request", async () => {
  const requests: Array<{ url: string; headers: Record<string, string> }> = [];

  const client: HttpClient = {
    async fetch(url, options) {
      // Record the request for inspection
      requests.push({ url, headers: options.headers });
      return { status: 200, body: { id: "charge_123" } };
    },
  };

  await chargeCard(card, amount, client);

  expect(requests).toHaveLength(1);
  expect(requests[0].headers["Idempotency-Key"]).toBeDefined();
  expect(requests[0].headers["Idempotency-Key"]).toMatch(/^[a-f0-9-]+$/);
});
```

### Example: Batching Verification

```typescript
// Verify operations are batched, not sent individually
test("batches database inserts", async () => {
  const queries: string[] = [];

  const db: Database = {
    async query(sql) {
      queries.push(sql);
      return { rowCount: 1 };
    },
  };

  await insertUsers([user1, user2, user3], db);

  // Should be ONE batch insert, not three individual inserts
  expect(queries).toHaveLength(1);
  expect(queries[0]).toMatch(/INSERT INTO users.*VALUES.*,.*,/);
});
```

### Example: No Extra Calls

```typescript
// Verify no unnecessary API calls (cost/rate limit concerns)
test("caches result and doesn't refetch", async () => {
  let fetchCount = 0;

  const api: Api = {
    async getUser(id) {
      fetchCount++;
      return { id, name: "Test User" };
    },
  };

  const cache = createCachingWrapper(api);

  await cache.getUser("123");
  await cache.getUser("123");
  await cache.getUser("123");

  expect(fetchCount).toBe(1); // Only one actual fetch
});
```

---

## Fake

A fake is a simplified but working implementation. Unlike a stub (which returns canned responses), a fake has real behavior—just simpler than production.

**Use for:** Exception 5 (Combinatorial Cost), Exception 3 (Time and Concurrency)

### When to Use Fakes

- Running 100+ test scenarios where real dependency is too slow
- Testing time-dependent behavior with fake clock
- Testing concurrency with controllable scheduler
- Hermetic tests that need realistic behavior at speed

### Example: Fake Clock

```typescript
// Testing lease renewal with controllable time
test("renews lease before expiry", async () => {
  const fakeClock = createFakeClock();
  let renewCount = 0;

  const lease = createLease({
    ttl: 30_000,
    renewAt: 25_000,
    onRenew: () => renewCount++,
    clock: fakeClock,
  });

  // Advance time just before renewal threshold
  await fakeClock.advance(24_000);
  expect(renewCount).toBe(0);

  // Advance past renewal threshold
  await fakeClock.advance(2_000);
  expect(renewCount).toBe(1);
});
```

### Example: Configurable Fake for Combinatorics

```typescript
// Testing 27 combinations of retry behavior
type FakeHttpConfig = {
  responses: Array<{ status: number; delay?: number }>;
};

function createConfigurableFakeHttp(config: FakeHttpConfig): HttpClient {
  let callIndex = 0;

  return {
    async fetch(url) {
      const response = config.responses[callIndex++];
      if (response.delay) {
        await sleep(response.delay);
      }
      return { status: response.status, body: {} };
    },
  };
}

// Generate all combinations
const scenarios = [
  { name: "success on first try", responses: [{ status: 200 }] },
  { name: "retry once then success", responses: [{ status: 500 }, { status: 200 }] },
  { name: "retry twice then success", responses: [{ status: 500 }, { status: 500 }, { status: 200 }] },
  { name: "all retries fail", responses: [{ status: 500 }, { status: 500 }, { status: 500 }] },
  // ... 23 more combinations
];

for (const scenario of scenarios) {
  test(`retry behavior: ${scenario.name}`, async () => {
    const client = createConfigurableFakeHttp(scenario);
    const result = await fetchWithRetry("/api", client);
    // Assert expected behavior
  });
}
```

### Fake vs Real In-Memory Implementation

**Important distinction:**

```typescript
// This is a FAKE - simplified behavior for testing
class FakeUserRepository {
  private users: User[] = [];

  async save(user: User) {
    this.users.push(user);
  }

  async findById(id: string) {
    return this.users.find((u) => u.id === id);
  }
}

// This is a REAL in-memory implementation - actual database behavior
import Database from "better-sqlite3";

const db = new Database(":memory:");
db.exec(`
  CREATE TABLE users (
    id TEXT PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL
  )
`);

// Has real constraints, real SQL parsing, real transactions
```

The fake is for high-speed combinatorial testing. The real in-memory implementation is for Level 2 integration tests. **Don't confuse them.**

---

## Mock

A mock verifies that specific interactions happen in a specific order. It's the strictest form of test double.

**Use for:** Exception 2 (Interaction Protocols) - specifically strict sequence verification

### When to Use Mocks (Sparingly)

- Saga compensation must happen in exact reverse order
- Protocol requires "must call A before B"
- "Must not call B if A failed"

### Why Use Sparingly

Mocks couple your tests to implementation details. If you refactor to call methods in a different order (but still correct), the test breaks. Prefer spies for most interaction testing.

### Example: Saga Compensation Order

```typescript
// Strict sequence: compensation must happen in reverse order
test("saga compensates in reverse order on failure", async () => {
  const calls: string[] = [];

  const steps = [
    {
      name: "reserveInventory",
      execute: async () => calls.push("reserve"),
      compensate: async () => calls.push("unreserve"),
    },
    {
      name: "chargePayment",
      execute: async () => calls.push("charge"),
      compensate: async () => calls.push("refund"),
    },
    {
      name: "shipOrder",
      execute: async () => {
        calls.push("ship");
        throw new Error("Shipping failed");
      },
      compensate: async () => calls.push("cancel-ship"),
    },
  ];

  await runSaga(steps).catch(() => {});

  // Verify exact order: execute forward, compensate backward
  expect(calls).toEqual([
    "reserve",
    "charge",
    "ship",
    "cancel-ship", // Most recent first
    "refund",
    "unreserve", // First step compensated last
  ]);
});
```

---

## Dummy

A dummy is a placeholder that satisfies a type requirement but is never actually used.

**Use for:** When a parameter is required but irrelevant to the test

### Example

```typescript
// The logger is required but we don't care about it for this test
test("calculates total correctly", () => {
  const dummyLogger: Logger = {
    log: () => {},
    error: () => {},
    warn: () => {},
  };

  // We're testing calculation, not logging
  const result = calculateTotal(items, dummyLogger);

  expect(result).toBe(150);
});
```

In practice, dummies are less useful in well-designed code. If you need many dummies, consider whether the function has too many dependencies.

---

## Mapping Exceptions to Double Types

| Exception                | Primary Double Type | Why                                  |
| ------------------------ | ------------------- | ------------------------------------ |
| 1. Failure Modes         | **Stub**            | Control what errors are returned     |
| 2. Interaction Protocols | **Spy** or Mock     | Observe/verify call patterns         |
| 3. Time and Concurrency  | **Fake**            | Fake clock, controllable scheduler   |
| 4. Safety                | **Stub**            | Don't execute destructive operations |
| 5. Combinatorial Cost    | **Fake**            | Configurable for many scenarios      |
| 6. Observability         | **Spy**             | Record request details               |
| 7. Contract Testing      | **Stub**            | Return documented response variants  |

---

## Anti-Patterns

### Using Mocking Frameworks to Intercept Real Objects

```typescript
// ❌ Framework intercepts real object - THIS IS MOCKING
jest.spyOn(fs, "readFile").mockResolvedValue("fake content");

// ✅ Dependency injection with explicit interface
const fileSystem: FileSystem = {
  async readFile(path) {
    return "fake content";
  },
};
await processFile(path, fileSystem);
```

### Using Doubles When Real System Works

```typescript
// ❌ Fake database when real DB is available and fast
const fakeDb = createFakeDatabase();
const result = await repository.findUser("123", fakeDb);

// ✅ Use real database at Level 2
const db = await createTestDatabase(); // Real DB in Docker
const result = await repository.findUser("123", db);
```

### Over-Specifying with Mocks

```typescript
// ❌ Brittle: breaks if implementation changes call order
mock.expects("methodA").once().before(mock.expects("methodB"));
mock.expects("methodB").once().before(mock.expects("methodC"));
mock.expects("methodC").once();

// ✅ Verify what matters, not every detail
const calls: string[] = [];
const spy = { methodA: () => calls.push("A"), ... };
// ... run code ...
expect(calls).toContain("A");
expect(calls).toContain("C");
// Order only if it actually matters for correctness
```

### Creating "InMemoryRepository" Fakes for Everything

```typescript
// ❌ This is a fake dressed as an "in-memory implementation"
class InMemoryUserRepository {
  private users = [];
  save(user) {
    this.users.push(user);
  }
  findById(id) {
    return this.users.find(u => u.id === id);
  }
}

// Problems:
// - Doesn't enforce constraints
// - Doesn't have transactions
// - Doesn't test your actual SQL
// - Proves nothing about real database behavior

// ✅ Use real database for integration tests
// Only use fakes for Exception 5 (combinatorial cost) scenarios
```
