---
name: testing
description: Learn how to test code without mocking. Use when learning testing approach or before writing tests.
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

# Test Strategy Router

You are the **foundational test strategy authority**. All other skills—architect, coder, reviewer—MUST consult you before making decisions about testing.

This skill is a **decision router**. Follow the stages in order. Most paths terminate before you ever consider test doubles.

---

## The Overarching Question

> **What evidence do I need to convince the user that my code correctly implements the specification?**

Every test exists to answer this question. Tests are not bureaucracy—they are **evidence** that your code works. If a test doesn't provide evidence, delete it.

---

## Stage 1: What Evidence Do You Need?

**Before writing any test, answer these questions:**

1. **What behavior could be wrong in production?**
   Not "what code am I testing" but what could actually fail for users?

2. **If this test passes, what does that prove about the real system?**
   A test that proves nothing about production is waste.

3. **What failure would this test catch that would otherwise reach users?**
   If you can't name a concrete failure, you don't need the test.

### The Evidence Trap

Agents often skip this stage. They see code and think "I need to test this." But that's backwards.

**Wrong approach:**

1. See `OrderProcessor` that calls `repository.save()`
2. Think "I need to test OrderProcessor"
3. Create `InMemoryRepository` fake
4. Write test that passes

**Right approach:**

1. Ask "What evidence do I need?"
2. Evidence needed: "orders persist correctly"
3. Realize: a fake repository proves nothing about persistence
4. Test at Level 2 with real database

---

## Stage 2: At What Level Does That Evidence Live?

Evidence can only be proven at specific levels. Use the five factors to determine where.

### The Five Factors

#### 1. What Does the Spec Promise?

The acceptance criteria drive the minimum test level:

| Spec Promise                      | Minimum Level | Why                                |
| --------------------------------- | ------------- | ---------------------------------- |
| "User can export data as CSV"     | Level 1       | File I/O with temp dirs is Level 1 |
| "Prices are calculated correctly" | Level 1       | Pure calculation                   |
| "CLI processes Hugo site"         | Level 2       | Project-specific binary            |
| "Database query returns users"    | Level 2       | Real database required             |
| "User can complete checkout"      | Level 3       | Real payment provider              |
| "Works in Safari"                 | Level 3       | Real browser required              |

#### 2. What Dependencies Are Involved?

| Dependency                             | Minimum Level |
| -------------------------------------- | ------------- |
| None (pure function)                   | Level 1       |
| File system (temp dirs)                | Level 1       |
| Standard dev tools (git, node)         | Level 1       |
| Database                               | Level 2       |
| External HTTP API                      | Level 2       |
| Project-specific binary (Hugo, ffmpeg) | Level 2       |
| Browser API                            | Level 3       |
| Third-party service (live)             | Level 3       |
| Real credentials                       | Level 3       |

#### 3. How Complex Is YOUR Code?

| Code Type                                   | Level 1 Value                |
| ------------------------------------------- | ---------------------------- |
| **Your logic** (algorithms, parsers, rules) | HIGH - test thoroughly       |
| **Library wiring** (argparse, Zod, YAML)    | LOW - trust the library      |
| **Simple glue code**                        | LOW - covered by integration |

**Trust the library**: Don't test that argparse parses flags. Test YOUR behavior that uses the parsed result.

#### 4. Debuggability Needs

> When this Level 2/3 test fails, will a Level 1 test help me find the bug faster?

| Scenario                                        | Add Level 1? | Reason                                        |
| ----------------------------------------------- | ------------ | --------------------------------------------- |
| Integration test fails on complex algorithm     | YES          | Level 1 isolates the algorithm                |
| Integration test fails on argparse flag parsing | NO           | Trust argparse; check your usage              |
| E2E test fails on payment flow                  | MAYBE        | If payment calculation is complex, yes        |
| E2E test fails on clipboard                     | NO           | It's a browser API call, nothing to unit test |

#### 5. Where Does Achievable Confidence Live?

Some guarantees can ONLY be proven at certain levels:

| You Want to Know...           | Achievable At |
| ----------------------------- | ------------- |
| Your math is correct          | Level 1       |
| Your SQL is valid             | Level 2       |
| The API accepts your requests | Level 2       |
| Users can complete the flow   | Level 3       |
| It works in Safari            | Level 3       |

### Level Selection Decision

```
Based on the five factors:
├─→ Evidence lives at Level 3 → Use real environment. DONE.
│   (Consider: add Level 1 for debuggability of complex parts?)
│
├─→ Evidence lives at Level 2 → Use real dependencies. DONE.
│   (Consider: add Level 1 for debuggability of complex parts?)
│
└─→ Evidence lives at Level 1 → Go to Stage 3
```

**If evidence requires Level 2 or 3, stop here.** Use real dependencies. Do not fake what you can test for real.

---

## Stage 3: What Kind of Level 1 Code Is This?

You've determined Level 1 is appropriate. Now classify the code.

### 3A: Pure Computation

**Definition:** Given inputs, compute outputs. No external state, no side effects.

Examples:

- Validation logic
- Parsing functions
- Business calculations
- Data transformations
- Command/argument building

**Decision:** Test directly. No doubles needed. DONE.

```typescript
// Pure computation - test at Level 1, no doubles
function calculateDiscount(order: Order, coupon: Coupon): number {
  if (coupon.expired) return 0;
  if (order.total < coupon.minPurchase) return 0;
  return Math.min(order.total * coupon.percentage, coupon.maxDiscount);
}

// Test directly
test("applies percentage discount", () => {
  const order = { total: 100 };
  const coupon = { percentage: 0.1, maxDiscount: 50, minPurchase: 0, expired: false };
  expect(calculateDiscount(order, coupon)).toBe(10);
});
```

### 3B: Code with Dependencies - Can You Extract?

**Question:** Can you extract the pure computation from the dependency interaction?

**If YES:** Extract it, test the pure part at Level 1, test integration at Level 2. DONE.

```typescript
// BEFORE: Tangled code
class OrderProcessor {
  async process(order: Order): Promise<void> {
    // Validation (pure)
    if (!order.items.length) throw new ValidationError("Empty order");
    if (order.total < 0) throw new ValidationError("Negative total");

    // Persistence (integration)
    await this.repository.save(order);
  }
}

// AFTER: Extracted
function validateOrder(order: Order): ValidationResult {
  // Pure computation - test at Level 1, no doubles
  if (!order.items.length) return { ok: false, error: "Empty order" };
  if (order.total < 0) return { ok: false, error: "Negative total" };
  return { ok: true };
}

class OrderProcessor {
  async process(order: Order): Promise<void> {
    // Glue code - test at Level 2 with real DB
    const validation = validateOrder(order);
    if (!validation.ok) throw new ValidationError(validation.error);
    await this.repository.save(order);
  }
}
```

Now:

- `validateOrder` → Level 1, no doubles, full 4-part progression
- `OrderProcessor.process` → Level 2, real database

**The fake repository was never needed.**

### 3C: Glue/Orchestration Code - Can't Extract

**Definition:** The behavior IS the interaction with the dependency. The code exists to orchestrate, coordinate, or handle dependency behavior. You can't extract it without losing the thing you're testing.

Examples:

- Retry logic
- Circuit breakers
- Saga orchestration
- Caching invalidation logic
- Rate limiting
- Multi-step workflows

**If you're here:** Go to Stage 4.

---

## Stage 4: Can the Real System Produce the Behavior?

You have glue/orchestration code that you can't extract. Before reaching for test doubles, ask:

| Question                                                   | If YES   | If NO         |
| ---------------------------------------------------------- | -------- | ------------- |
| **Reliably?** (deterministic, not flaky)                   | Continue | Go to Stage 5 |
| **Safely?** (won't charge money, send emails, delete data) | Continue | Go to Stage 5 |
| **Cheaply?** (won't cost $$ or take hours)                 | Continue | Go to Stage 5 |
| **Observably?** (can see what you need to assert)          | Continue | Go to Stage 5 |

**If YES to all:** Use real system at Level 2. DONE.

**If NO to any:** Go to Stage 5.

---

## Stage 5: Which Exception Applies?

You've proven:

1. Evidence lives at Level 1
2. Code can't be factored into pure computation
3. Real system can't produce the behavior reliably/safely/cheaply/observably

**Now and only now** may you consider test doubles. But you must match a specific exception.

### The Seven Exception Cases

Test doubles are ONLY legitimate when you need **deterministic control over behavior the real system can't reliably produce**.

#### Exception 1: Failure Modes

**When:** You need to test behavior under specific failure conditions that the real system can't reliably produce.

Examples:

- Timeouts at specific points in a call
- Connection resets mid-stream
- Partial writes / partial reads
- Throttling / rate limits
- Retryable vs non-retryable error codes
- DNS failures, TLS handshake failures

**Double type:** Stub that returns predetermined errors

```typescript
// Testing retry logic under timeout
const timeoutingClient: HttpClient = {
  async fetch(url) {
    throw new TimeoutError("Request timed out");
  },
};

test("retries on timeout", async () => {
  let attempts = 0;
  const client: HttpClient = {
    async fetch(url) {
      attempts++;
      if (attempts < 3) throw new TimeoutError("timeout");
      return { status: 200, body: "ok" };
    },
  };

  const result = await fetchWithRetry(url, client);
  expect(attempts).toBe(3);
  expect(result.status).toBe(200);
});
```

#### Exception 2: Interaction Protocols

**When:** Correctness depends on the conversation pattern, not just input/output.

Examples:

- Multi-step workflows (create → poll → finalize)
- Pagination loops and cursor handling
- Transactional outbox patterns
- Sagas and compensating actions
- "Must call A before B"
- "Must not call B if A failed"
- Ensuring no extra calls (cost, rate limits)

**Double type:** Spy that records calls, or Mock for strict sequence assertions

```typescript
// Testing that compensating action is called on failure
test("saga calls compensation on step 2 failure", async () => {
  const calls: string[] = [];

  const step1 = {
    execute: async () => {
      calls.push("step1");
    },
  };
  const step2 = {
    execute: async () => {
      calls.push("step2");
      throw new Error("fail");
    },
    compensate: async () => {
      calls.push("step2-compensate");
    },
  };
  const step1Compensate = {
    compensate: async () => {
      calls.push("step1-compensate");
    },
  };

  await runSaga([step1, step2]).catch(() => {});

  expect(calls).toEqual(["step1", "step2", "step2-compensate", "step1-compensate"]);
});
```

#### Exception 3: Time and Concurrency

**When:** You need deterministic control over timing and scheduling.

Examples:

- Scheduled retries with jitter
- Token refresh races
- Lease renewal loops
- Debounce/throttle logic
- "Eventually consistent" wait loops
- Visibility timeouts

**Double type:** Fake clock, controllable scheduler

```typescript
// Testing lease renewal with fake clock
test("renews lease before expiry", async () => {
  const fakeClock = createFakeClock();
  let renewCount = 0;

  const lease = createLease({
    ttl: 30_000,
    renewAt: 25_000,
    onRenew: () => {
      renewCount++;
    },
    clock: fakeClock,
  });

  await fakeClock.advance(24_000);
  expect(renewCount).toBe(0);

  await fakeClock.advance(2_000); // Now at 26s
  expect(renewCount).toBe(1);
});
```

#### Exception 4: Safety

**When:** The real system is destructive or irreversible.

Examples:

- Payment providers (charges/refunds)
- Email/SMS sending
- Destructive admin APIs (delete, rotate keys)
- Systems with strict quotas/billing

**Double type:** Stub that records but doesn't execute

```typescript
// Testing payment flow without charging
test("processes refund for cancelled order", async () => {
  const refunds: Array<{ amount: number; reason: string }> = [];

  const paymentProvider: PaymentProvider = {
    async refund(chargeId, amount, reason) {
      refunds.push({ amount, reason });
      return { refundId: "refund_123", status: "succeeded" };
    },
  };

  await cancelOrder(order, paymentProvider);

  expect(refunds).toEqual([{ amount: 99.99, reason: "order_cancelled" }]);
});
```

#### Exception 5: Combinatorial Cost

**When:** You need 100+ scenarios and real system is too slow/expensive.

Examples:

- HTTP client behavior matrix: status codes × retry rules × idempotency × timeout budgets
- Queue consumer: redelivery × poison messages × DLQ routing × batch sizes
- Caching: TTL × stale-while-revalidate × negative caching × stampede protection

**Double type:** Fake that can be configured for each scenario

```typescript
// Testing 27 combinations of retry behavior
const scenarios = generateRetryScenarios(); // 27 combinations

for (const scenario of scenarios) {
  test(`retry behavior: ${scenario.name}`, async () => {
    const client = createConfigurableClient(scenario.responses);
    const result = await fetchWithRetry(url, client, scenario.config);
    expect(result).toMatchSnapshot();
  });
}
```

#### Exception 6: Observability

**When:** You need to verify details the real system can't expose.

Examples:

- "Did we include this header?"
- "Did we batch these operations?"
- "Did we send the idempotency key?"
- "Did we avoid N+1 calls?"
- "Did we handle pagination correctly?"

**Double type:** Spy that records request details

```typescript
// Testing that idempotency key is sent
test("includes idempotency key in payment request", async () => {
  const requests: Array<{ headers: Record<string, string> }> = [];

  const client: HttpClient = {
    async fetch(url, options) {
      requests.push({ headers: options.headers });
      return { status: 200 };
    },
  };

  await chargeCard(card, amount, client);

  expect(requests[0].headers["Idempotency-Key"]).toBeDefined();
});
```

#### Exception 7: Contract Testing

**When:** Third-party API you don't control, need to verify your serialization/parsing.

Examples:

- Verify request serialization matches API spec
- Verify you can parse all documented response variants
- Pin behavior when provider changes subtly

**Double type:** Contract stub that enforces expected format

```typescript
// Testing response parsing for all documented variants
const responseVariants = [
  { status: 200, body: { data: [...] } },           // Success
  { status: 200, body: { data: [], next: "..." } }, // Paginated
  { status: 429, body: { retry_after: 60 } },       // Rate limited
  { status: 503, body: { message: "..." } },        // Service unavailable
];

for (const variant of responseVariants) {
  test(`handles ${variant.status} response`, async () => {
    const client = createContractClient(variant);
    const result = await apiClient.fetch(client);
    expect(() => parseResponse(result)).not.toThrow();
  });
}
```

### If No Exception Applies

**STOP.** You should not use test doubles.

Options:

1. **Re-examine Stage 3B**: Can you really not extract pure logic?
2. **Test at Level 2**: Use real dependencies
3. **Accept no Level 1 test**: Some glue code doesn't need unit tests

---

## Anti-Patterns: What Agents Get Wrong

### Anti-Pattern 1: Skipping the Evidence Question

**Wrong:**

```
Agent sees: OrderService calls repository.save()
Agent thinks: "I need to test OrderService"
Agent does: Creates InMemoryRepository
```

**Right:**

```
Agent asks: "What evidence do I need?"
Agent answers: "Evidence that orders persist correctly"
Agent realizes: Fake repository proves nothing about persistence
Agent does: Tests at Level 2 with real database
```

### Anti-Pattern 2: Calling Fakes "In-Memory Implementations"

The skill says "DI with real in-memory implementations" is allowed at Level 1. Agents read this as permission to create fakes.

**Fake (not allowed by default):**

```typescript
// This is a FAKE dressed up as an "in-memory implementation"
class InMemoryUserRepository {
  private users: User[] = [];

  async save(user: User) {
    this.users.push(user); // Records call, doesn't implement real behavior
  }

  async findById(id: string) {
    return this.users.find(u => u.id === id); // Canned behavior
  }
}
```

**Real in-memory implementation (allowed):**

```typescript
// SQLite in-memory - actually implements database behavior
const db = new Database(":memory:");
db.exec(schema);

// This is a REAL implementation that happens to be in-memory
// It has real SQL parsing, real constraints, real transactions
```

### Anti-Pattern 3: Faking for Speed

**Wrong thinking:** "Real DB is slow, so I'll fake it"

Speed alone is not an exception. You need:

- **100+ scenarios** (Exception 5: Combinatorial Cost), AND
- Real system would take **hours**, not just seconds

A test that takes 2 seconds with a real DB is fine. Don't optimize developer experience at the cost of test validity.

### Anti-Pattern 4: Faking What Can Be Tested Real

**Wrong:**

```typescript
// Creating FakeHttpClient instead of testing against real endpoint
const fakeClient: HttpClient = {
  async fetch(url) {
    return { status: 200, body: { users: [] } };
  },
};
```

**Right:** Use Level 2 test with real HTTP server (local test server or docker container).

If you're testing "does my code handle HTTP responses correctly?" - use a real HTTP server. The fake proves your code works with your imagination of HTTP, not actual HTTP.

### Anti-Pattern 5: Testing Library Behavior

**Wrong:**

```typescript
// Tests that argparse works, not your code
test("parses --verbose flag", () => {
  const args = parser.parse(["--verbose"]);
  expect(args.verbose).toBe(true); // Testing argparse!
});
```

**Right:**

```typescript
// Tests YOUR behavior that uses the parsed result
test("verbose mode produces detailed output", () => {
  const output = runCommand({ verbose: true });
  expect(output).toContain("DEBUG:"); // Testing your code!
});
```

---

## The Cardinal Rule: No Mocking

> **Mocking is always wrong. There is no exception.**

Mocking (using mocking frameworks to intercept and stub arbitrary methods) gives you a test that passes while your production code fails. This is worse than no test at all.

**If you feel you need to mock:**

1. **Redesign using dependency injection** with real implementations or legitimate doubles (matching an exception case), OR
2. **Test at a different level**—push to Level 2 or 3 where real dependencies are available

The seven exception cases use **test doubles** (stubs, spies, fakes), not mocks. The distinction:

- **Mock**: Framework intercepts method calls on real objects
- **Test double**: You provide an alternative implementation via dependency injection

---

## Quick Reference: The Router Flow

```
STAGE 1: What evidence do I need?
         ↓
STAGE 2: At what level does that evidence live?
         ├─→ Level 2/3: Use real dependencies. DONE.
         └─→ Level 1: Continue
                ↓
STAGE 3: What kind of Level 1 code?
         ├─→ 3A Pure computation: No doubles. DONE.
         ├─→ 3B Can extract pure part: Extract, test pure at L1, integration at L2. DONE.
         └─→ 3C Glue code (can't extract): Continue
                ↓
STAGE 4: Can real system produce the behavior?
         ├─→ YES (reliable, safe, cheap, observable): Use real at Level 2. DONE.
         └─→ NO: Continue
                ↓
STAGE 5: Which exception applies?
         ├─→ Exception matches: Use appropriate double. Document which exception.
         └─→ No exception: Don't write L1 for this code. Test at L2.
```

---

## Reference Material

For detailed patterns at each level, see:

- [Level 1: Unit/Pure Logic](levels/level-1-unit.md) - DI patterns, pure function testing
- [Level 2: Integration](levels/level-2-integration.md) - Harness patterns, fixtures
- [Level 3: E2E](levels/level-3-e2e.md) - Credential management, workflow testing

For test organization:

- [4-Part Progression](reference/4-part-progression.md) - Typical/Edge/Systematic/Property-based
- [Double Types](reference/double-types.md) - Meszaros taxonomy when doubles ARE needed

---

## Checklist Before Declaring Tests Complete

- [ ] Evidence exists at the level where it can be proven
- [ ] Stage 1 question answered: "What evidence do I need?"
- [ ] No test doubles without matching exception case
- [ ] Exception case documented if doubles are used
- [ ] Level 2 harnesses documented
- [ ] Level 3 credentials documented (not hardcoded)
- [ ] Tests verify behavior, not implementation
- [ ] Regression tests all pass
