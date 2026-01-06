# Debuggability-First Test Organization

**The Problem**: When property-based tests fail with random values, you have no context and cannot debug.

**The Solution**: Progress from debuggable named cases to comprehensive property tests.

## Part 0: Shared Test Values

`test/fixtures/values.ts`:

```typescript
export interface TestCase<I, E> {
  readonly input: I;
  readonly expected: E;
}

export const TYPICAL = {
  BASIC: { input: "simple", expected: 42 },
  COMPLEX: { input: "with-flags", expected: 100 },
} as const satisfies Record<string, TestCase<string, number>>;

export const EDGES = {
  EMPTY: { input: "", expected: 0 },
  MAX_LENGTH: { input: "x".repeat(1000), expected: -1 }, // Error case
} as const satisfies Record<string, TestCase<string, number>>;
```

## Part 1: Named Typical Cases

One test per category:

```typescript
import { TYPICAL } from "../fixtures/values";

describe("GIVEN typical inputs", () => {
  it("WHEN processing BASIC input THEN returns expected", () => {
    const testCase = TYPICAL.BASIC;

    const result = process(testCase.input);

    expect(result).toBe(testCase.expected); // ← Set breakpoint, inspect testCase
  });

  it("WHEN processing COMPLEX input THEN returns expected", () => {
    const testCase = TYPICAL.COMPLEX;

    const result = process(testCase.input);

    expect(result).toBe(testCase.expected);
  });
});
```

**Why**: When test fails, you know WHICH category. Set breakpoint, inspect the named case.

## Part 2: Named Edge Cases

One test per boundary:

```typescript
import { EDGES } from "../fixtures/values";

describe("GIVEN boundary conditions", () => {
  it("WHEN processing EMPTY input THEN handles correctly", () => {
    const testCase = EDGES.EMPTY;

    const result = process(testCase.input);

    expect(result).toBe(testCase.expected);
  });

  it("WHEN processing MAX_LENGTH input THEN returns error", () => {
    const testCase = EDGES.MAX_LENGTH;

    const result = process(testCase.input);

    expect(result).toBe(testCase.expected);
  });
});
```

**Why**: Each boundary is independently debuggable.

## Part 3: Systematic Coverage

Parametrized test finds gaps:

```typescript
import { EDGES, TYPICAL } from "../fixtures/values";

describe("GIVEN all known cases", () => {
  const allCases = { ...TYPICAL, ...EDGES };

  it.each(Object.entries(allCases))(
    "WHEN testing %s THEN passes",
    (name, testCase) => {
      const result = process(testCase.input);

      expect(result).toBe(testCase.expected); // ← Breakpoint: inspect name, testCase
    },
  );
});
```

**Why**: Should ONLY fail if Parts 1-2 missed a category.

## Part 4: Property-Based Testing

fast-check for comprehensive coverage:

```typescript
import * as fc from "fast-check";

describe("GIVEN generated inputs", () => {
  it("WHEN processing any string THEN never throws unexpected exception", () => {
    fc.assert(
      fc.property(fc.string({ minLength: 0, maxLength: 100 }), (input) => {
        try {
          const result = process(input);
          return typeof result === "number";
        } catch (e) {
          return e instanceof ValidationError; // Expected for invalid inputs
        }
      }),
    );
  });
});
```

**Why**: Comprehensive coverage after debuggable cases are established.

## Test Ordering Strategy

Order tests from trivial to complex for fast failure:

```typescript
describe("TestModuleAvailability", () => {
  // Part 0: Environment checks (run first)
  it("module imports successfully", async () => {
    const { process } = await import("@/module");
    expect(process).toBeDefined();
  });
});

describe("TestBasicFunctionality", () => {
  // Part 1: Basic operations (run second)
  it("simple case works", () => {
    expect(process("hello")).toBe(5);
  });
});

describe("TestComplexBehavior", () => {
  // Part 2+: Complex operations (run last)
  // ...
});
```

## Critical Rules

1. **Separate file for test values** - DRY, reusable, type-safe with satisfies
2. **Named categories** - `TYPICAL.BASIC` not anonymous objects
3. **One test per category in Parts 1-2** - Immediately debuggable
4. **Part 3 discovers gaps** - Parametrized test failure reveals missing category
5. **Part 4 uses fast-check** - For comprehensive property testing

## Anti-Patterns

- ❌ Starting with property tests - No context when it fails
- ❌ Inline test data - Not reusable, no names
- ❌ Testing subset of cases - Part 3 must test ALL
- ❌ Random without seed - Use fast-check's `seed` option for reproducibility
