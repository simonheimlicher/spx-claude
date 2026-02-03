# The 4-Part Test Progression

Once you've determined the right level and approach (via the router in SKILL.md), organize your tests using this progression.

---

## Overview

| Phase                   | What You're Testing                      | Confidence Level |
| ----------------------- | ---------------------------------------- | ---------------- |
| 1. Typical cases        | Happy paths, common scenarios            | Baseline         |
| 2. Edge/boundary cases  | Limits, special values, error conditions | Robustness       |
| 3. Systematic coverage  | Loops, state transitions, combinations   | Completeness     |
| 4. Property-based tests | Invariants that hold for all inputs      | Deep correctness |

**Not every function needs all four phases.** Use judgment:

- Simple utilities: Phase 1 + 2 may be sufficient
- Complex algorithms: All four phases
- Glue code: Phase 1 only (integration tests handle the rest)

---

## Phase 1: Typical Cases

Start with the scenarios users will encounter most often. These are your "golden path" tests.

**Questions to ask:**

- What's the most common input?
- What does successful execution look like?
- What's the expected output for normal usage?

```typescript
describe("calculateDiscount", () => {
  test("applies 10% discount to order over minimum", () => {
    const order = { total: 100 };
    const coupon = { percentage: 0.1, minPurchase: 50 };

    const discount = calculateDiscount(order, coupon);

    expect(discount).toBe(10);
  });

  test("returns 0 for expired coupon", () => {
    const order = { total: 100 };
    const coupon = { percentage: 0.1, minPurchase: 50, expired: true };

    const discount = calculateDiscount(order, coupon);

    expect(discount).toBe(0);
  });
});
```

---

## Phase 2: Edge and Boundary Cases

Test the limits, special values, and error conditions.

**Categories to consider:**

| Category               | Examples                                     |
| ---------------------- | -------------------------------------------- |
| **Boundaries**         | 0, 1, max-1, max, max+1                      |
| **Empty inputs**       | Empty string, empty array, null, undefined   |
| **Special values**     | NaN, Infinity, negative numbers              |
| **Error conditions**   | Invalid input, missing fields                |
| **Boundary crossings** | Just below threshold, exactly at, just above |

```typescript
describe("calculateDiscount - edge cases", () => {
  test("returns 0 when order is exactly at minimum", () => {
    const order = { total: 50 };
    const coupon = { percentage: 0.1, minPurchase: 50 };

    // At boundary - should still apply
    expect(calculateDiscount(order, coupon)).toBe(5);
  });

  test("returns 0 when order is below minimum", () => {
    const order = { total: 49.99 };
    const coupon = { percentage: 0.1, minPurchase: 50 };

    // Just below boundary
    expect(calculateDiscount(order, coupon)).toBe(0);
  });

  test("caps discount at maximum", () => {
    const order = { total: 1000 };
    const coupon = { percentage: 0.5, minPurchase: 0, maxDiscount: 100 };

    // 50% of 1000 = 500, but max is 100
    expect(calculateDiscount(order, coupon)).toBe(100);
  });

  test("handles zero total", () => {
    const order = { total: 0 };
    const coupon = { percentage: 0.1, minPurchase: 0 };

    expect(calculateDiscount(order, coupon)).toBe(0);
  });
});
```

---

## Phase 3: Systematic Coverage

For code with loops, state machines, or combinatorial behavior, ensure you cover all paths systematically.

**Techniques:**

### Loop Coverage

Test: zero iterations, one iteration, multiple iterations, max iterations.

```typescript
describe("processItems - loop coverage", () => {
  test("handles empty array", () => {
    expect(processItems([])).toEqual({ processed: 0, errors: 0 });
  });

  test("handles single item", () => {
    expect(processItems([validItem])).toEqual({ processed: 1, errors: 0 });
  });

  test("handles multiple items", () => {
    expect(processItems([item1, item2, item3])).toEqual({ processed: 3, errors: 0 });
  });

  test("handles max batch size", () => {
    const items = Array(100).fill(validItem);
    expect(processItems(items)).toEqual({ processed: 100, errors: 0 });
  });
});
```

### State Transitions

For state machines, test all valid transitions and verify invalid ones are rejected.

```typescript
describe("OrderStateMachine", () => {
  // Valid transitions
  test("pending -> confirmed", () => {
    const order = createOrder("pending");
    order.confirm();
    expect(order.state).toBe("confirmed");
  });

  test("confirmed -> shipped", () => {
    const order = createOrder("confirmed");
    order.ship();
    expect(order.state).toBe("shipped");
  });

  // Invalid transitions
  test("pending -> shipped throws", () => {
    const order = createOrder("pending");
    expect(() => order.ship()).toThrow("Cannot ship pending order");
  });

  test("shipped -> pending throws", () => {
    const order = createOrder("shipped");
    expect(() => order.cancel()).toThrow("Cannot cancel shipped order");
  });
});
```

### Combinatorial Testing

For functions with multiple independent parameters, test key combinations.

```typescript
describe("formatOutput - combinations", () => {
  const cases = [
    // [format, verbose, colorized, expected]
    ["json", false, false, "{\"data\":\"test\"}"],
    ["json", true, false, "{\n  \"data\": \"test\"\n}"],
    ["table", false, false, "| data |\n| test |"],
    ["table", true, true, "\x1b[1m| data |\x1b[0m\n| test |"],
  ];

  test.each(cases)(
    "format=%s verbose=%s color=%s",
    (format, verbose, colorized, expected) => {
      const result = formatOutput({ data: "test" }, { format, verbose, colorized });
      expect(result).toBe(expected);
    },
  );
});
```

---

## Phase 4: Property-Based Tests

Define invariants that should hold for ALL valid inputs, then let the test framework generate inputs.

**When to use:**

- Complex algorithms where edge cases are hard to enumerate
- Parsers (parse(format(x)) === x)
- Mathematical operations
- Serialization/deserialization

**Common properties:**

| Property                   | Description                     | Example                       |
| -------------------------- | ------------------------------- | ----------------------------- |
| **Idempotency**            | f(f(x)) === f(x)                | Formatting, normalization     |
| **Round-trip**             | decode(encode(x)) === x         | Serialization                 |
| **Invariant preservation** | Property holds before and after | Sorting doesn't lose elements |
| **Commutativity**          | f(a, b) === f(b, a)             | Set operations                |

```typescript
import fc from "fast-check";

describe("URL parser - properties", () => {
  test("round-trip: parse then format equals original", () => {
    fc.assert(
      fc.property(fc.webUrl(), (url) => {
        const parsed = parseUrl(url);
        const formatted = formatUrl(parsed);
        // Normalized comparison
        expect(new URL(formatted).href).toBe(new URL(url).href);
      }),
    );
  });

  test("protocol is always lowercase", () => {
    fc.assert(
      fc.property(fc.webUrl(), (url) => {
        const parsed = parseUrl(url);
        expect(parsed.protocol).toBe(parsed.protocol.toLowerCase());
      }),
    );
  });
});

describe("sort - properties", () => {
  test("result has same length as input", () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        const sorted = sort(arr);
        expect(sorted.length).toBe(arr.length);
      }),
    );
  });

  test("result contains same elements as input", () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        const sorted = sort(arr);
        expect(sorted.sort()).toEqual([...arr].sort());
      }),
    );
  });

  test("result is ordered", () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        const sorted = sort(arr);
        for (let i = 1; i < sorted.length; i++) {
          expect(sorted[i]).toBeGreaterThanOrEqual(sorted[i - 1]);
        }
      }),
    );
  });
});
```

### Python Property-Based Testing

```python
from hypothesis import given, strategies as st


@given(st.text())
def test_roundtrip_encoding(s):
    """Encoding then decoding returns original string."""
    encoded = encode(s)
    decoded = decode(encoded)
    assert decoded == s


@given(st.lists(st.integers()))
def test_sort_preserves_length(lst):
    """Sorting doesn't add or remove elements."""
    sorted_lst = sort(lst)
    assert len(sorted_lst) == len(lst)


@given(st.lists(st.integers()))
def test_sort_is_ordered(lst):
    """Result is in ascending order."""
    sorted_lst = sort(lst)
    for i in range(1, len(sorted_lst)):
        assert sorted_lst[i] >= sorted_lst[i - 1]
```

---

## Applying the Progression to Different Code Types

### Pure Computation

Use all four phases. This is where the progression shines.

```
Phase 1: 2-3 typical cases
Phase 2: All boundary conditions
Phase 3: Loop/state coverage if applicable
Phase 4: Property-based tests for invariants
```

### Glue/Orchestration Code (with legitimate doubles)

Focus on Phase 1 and 2. The behavior IS the interaction pattern.

```
Phase 1: Happy path interaction
Phase 2: Error handling, edge cases
Phase 3: Usually not needed (integration tests cover this)
Phase 4: Rarely applicable
```

### Integration Tests (Level 2)

Focus on Phase 1 and key Phase 2 cases. Full coverage is expensive.

```
Phase 1: Primary workflows work with real dependencies
Phase 2: Key error conditions (connection failures, invalid data)
Phase 3: Rarely worth the cost
Phase 4: Not applicable
```

### E2E Tests (Level 3)

Phase 1 only. These are expensive; test the critical paths.

```
Phase 1: User can complete the workflow
Phase 2-4: Not appropriate at this level
```

---

## Anti-Patterns

### Testing Implementation, Not Behavior

```typescript
// ❌ Tests implementation details
test("calls helper function", () => {
  const spy = jest.spyOn(module, "helperFunction");
  doThing();
  expect(spy).toHaveBeenCalled(); // Who cares?
});

// ✅ Tests observable behavior
test("produces correct output", () => {
  const result = doThing();
  expect(result).toEqual(expectedOutput);
});
```

### Redundant Tests

```typescript
// ❌ Three tests that prove the same thing
test("returns true for positive", () => {
  expect(isPositive(1)).toBe(true);
});
test("returns true for 5", () => {
  expect(isPositive(5)).toBe(true);
});
test("returns true for 100", () => {
  expect(isPositive(100)).toBe(true);
});

// ✅ One typical case, focus on boundaries
test("returns true for positive numbers", () => {
  expect(isPositive(1)).toBe(true);
});
test("returns false for zero", () => {
  expect(isPositive(0)).toBe(false);
});
test("returns false for negative", () => {
  expect(isPositive(-1)).toBe(false);
});
```

### Skipping Boundaries

```typescript
// ❌ Only happy path
test("validates email", () => {
  expect(isValidEmail("user@example.com")).toBe(true);
});

// ✅ Include boundaries
test("accepts valid email", () => {
  expect(isValidEmail("user@example.com")).toBe(true);
});
test("rejects empty string", () => {
  expect(isValidEmail("")).toBe(false);
});
test("rejects missing @", () => {
  expect(isValidEmail("userexample.com")).toBe(false);
});
test("rejects multiple @", () => {
  expect(isValidEmail("user@@example.com")).toBe(false);
});
```
