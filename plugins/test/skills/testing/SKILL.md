---
name: testing
description: Learn how to test code without mocking. Use when learning testing approach or before writing tests.
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

<objective>
Route testing decisions through a five-stage methodology to determine test level, eliminate unnecessary test doubles, and ensure tests provide evidence of production correctness. This is the foundational testing authority—all architecting, coding, and reviewing skills must consult this skill for any testing decisions.
</objective>

<quick_start>
Before writing ANY test, answer Stage 1: "What evidence do I need to convince the user that my code correctly implements the specification?"

Then follow the router through Stages 2-5. Most paths terminate before test doubles are ever considered. If you reach for a fake or stub without completing the router, you are doing it wrong and will be caught by the reviewing skill.

DO NOT EVEN THINK ABOUT SKIPPING ANY STEPS OR USING MOCKS, FAKES, OR ANY OTHER TEST DOUBLES WITHOUT EXPLICIT APPROVAL FROM THE USER!
</quick_start>

<essential_principles>
**Tests are evidence, not bureaucracy.** Every test must be the answer to the question: "Will the code under test correctly implement the specified outcomes?" If a test doesn't provide evidence about production behavior, delete it.

**No mocking. Ever.** Mocking (using frameworks to intercept and stub arbitrary methods) gives you tests that pass while production code fails. This is worse than no test at all. If you feel you need to mock, either redesign with dependency injection or test at a different level.

**Reality is the oracle.** Test against real systems whenever possible. A test that passes against a fake, which is nothing bot your imagination of how a database works, proves nothing about the code that you could not have anticipated.

**Test doubles are exceptions, not defaults.** The seven exception cases in Stage 5 are the ONLY legitimate uses for test doubles. If your situation doesn't match an exception, don't use doubles.
</essential_principles>

<intake>
You are about to write a test. Before proceeding, route through this skill to determine:

1. What evidence you need (Stage 1)
2. At what level that evidence lives (Stage 2)
3. What kind of code this is (Stage 3)
4. Whether the real system can produce the behavior (Stage 4)
5. Which exception applies, if any (Stage 5)

**Start at Stage 1. Do not skip ahead.**
</intake>

<routing>

| Stage | Outcome                                               | Next Step                                               |
| ----- | ----------------------------------------------------- | ------------------------------------------------------- |
| 1     | Evidence identified                                   | Stage 2                                                 |
| 2     | Level 2 or 3 required                                 | Use real dependencies. DONE.                            |
| 2     | Level 1 appropriate                                   | Stage 3                                                 |
| 3A    | Pure computation                                      | Test directly, no doubles. DONE.                        |
| 3B    | Can extract pure part                                 | Extract, test pure at L1, integration at L2. DONE.      |
| 3C    | Glue/orchestration code                               | Stage 4                                                 |
| 4     | Real system works (reliable, safe, cheap, observable) | Use real at Level 2. DONE.                              |
| 4     | Real system doesn't work for testing                  | Stage 5                                                 |
| 5     | Exception case matches                                | Use appropriate double, document which exception. DONE. |
| 5     | No exception matches                                  | Don't write L1 for this code. Test at L2. DONE.         |

</routing>

<stage_1 name="What Evidence Do You Need?">
Before writing any test, answer these questions:

1. **What behavior could be wrong in production?** Not "what code am I testing" but what could actually fail for users?

2. **If this test passes, what does that prove about the real system?** A test that proves nothing about production is waste.

3. **What failure would this test catch that would otherwise reach users?** If you can't name a concrete failure, you don't need the test.

**The Evidence Trap**

Agents often skip this stage. They see code and think "I need to test this." That's backwards.

- **Wrong approach**: See `OrderProcessor` that calls `repository.save()` → think "I need to test OrderProcessor" → create `InMemoryRepository` fake → write test that passes
- **Right approach**: Ask "What evidence do I need?" → answer "Evidence that orders persist correctly" → realize a fake repository proves nothing about persistence → test at Level 2 with real database

</stage_1>

<stage_2 name="At What Level Does That Evidence Live?">
Evidence can only be proven at specific levels. Use the five factors to determine where.

<five_factors>

**Factor 1: What Does the Spec Promise?**

The acceptance criteria drive the minimum test level:

| Spec Promise                      | Minimum Level | Why                                |
| --------------------------------- | ------------- | ---------------------------------- |
| "Prices are calculated correctly" | Level 1       | Pure calculation                   |
| "User can export data as CSV"     | Level 1       | File I/O with temp dirs is Level 1 |
| "CLI processes Hugo site"         | Level 2       | Project-specific binary            |
| "Database query returns users"    | Level 2       | Real database required             |
| "User can complete checkout"      | Level 3       | Real payment provider              |
| "Works in Safari"                 | Level 3       | Real browser required              |

**Factor 2: What Dependencies Are Involved?**

| Dependency                             | Minimum Level |
| -------------------------------------- | ------------- |
| None (pure function)                   | Level 1       |
| File system (temp dirs)                | Level 1       |
| Standard dev tools (git, node, curl)   | Level 1       |
| Database                               | Level 2       |
| External HTTP API                      | Level 2       |
| Project-specific binary (Hugo, ffmpeg) | Level 2       |
| Browser API                            | Level 3       |
| Third-party service (live)             | Level 3       |
| Real credentials                       | Level 3       |

**Factor 3: How Complex Is YOUR Code?**

| Code Type                               | Level 1 Value                |
| --------------------------------------- | ---------------------------- |
| Your logic (algorithms, parsers, rules) | HIGH - test thoroughly       |
| Library wiring (argparse, Zod, YAML)    | LOW - trust the library      |
| Simple glue code                        | LOW - covered by integration |

**Trust the library**: Don't test that argparse parses flags. Test YOUR behavior that uses the parsed result.

**Factor 4: Debuggability Needs**

When this Level 2/3 test fails, will a Level 1 test help find the bug faster?

| Scenario                                        | Add Level 1? | Reason                                 |
| ----------------------------------------------- | ------------ | -------------------------------------- |
| Integration test fails on complex algorithm     | YES          | Level 1 isolates the algorithm         |
| Integration test fails on argparse flag parsing | NO           | Trust argparse; check your usage       |
| E2E test fails on payment flow                  | MAYBE        | If payment calculation is complex, yes |
| E2E test fails on clipboard                     | NO           | Browser API call, nothing to unit test |

**Factor 5: Where Does Achievable Confidence Live?**

Some guarantees can ONLY be proven at certain levels:

| You Want to Know...           | Achievable At |
| ----------------------------- | ------------- |
| Your math is correct          | Level 1       |
| Your SQL is valid             | Level 2       |
| The API accepts your requests | Level 2       |
| Users can complete the flow   | Level 3       |
| It works in Safari            | Level 3       |

</five_factors>

**Level Selection Decision**

Based on the five factors:

- Evidence lives at Level 3 → Use real environment. DONE. (Consider: add Level 1 for debuggability of complex parts?)
- Evidence lives at Level 2 → Use real dependencies. DONE. (Consider: add Level 1 for debuggability of complex parts?)
- Evidence lives at Level 1 → Go to Stage 3

**If evidence requires Level 2 or 3, stop here.** Use real dependencies. Do not fake what you can test for real.

</stage_2>

<stage_3 name="What Kind of Level 1 Code Is This?">
You've determined Level 1 is appropriate. Now classify the code.

**3A: Pure Computation**

Given inputs, compute outputs. No external state, no side effects.

Examples: validation logic, parsing functions, business calculations, data transformations, command/argument building.

**Decision**: Test directly. No doubles needed. DONE.

**3B: Code with Dependencies - Can You Extract?**

Can you extract the pure computation from the dependency interaction?

**If YES**: Extract it, test the pure part at Level 1, test integration at Level 2. DONE.

The pattern: factor tangled code into a pure function (validation, calculation, transformation) and a thin wrapper that calls the dependency. Test the pure function exhaustively at Level 1. Test the wrapper at Level 2 with real dependencies.

**The fake repository was never needed.**

**3C: Glue/Orchestration Code - Can't Extract**

The behavior IS the interaction with the dependency. The code exists to orchestrate, coordinate, or handle dependency behavior. You can't extract it without losing the thing you're testing.

Examples: retry logic, circuit breakers, saga orchestration, caching invalidation logic, rate limiting, multi-step workflows.

**If you're here**: Go to Stage 4.

</stage_3>

<stage_4 name="Can the Real System Produce the Behavior?">
You have glue/orchestration code that you can't extract. Before reaching for test doubles, ask:

| Question                                                   | If YES   | If NO         |
| ---------------------------------------------------------- | -------- | ------------- |
| **Reliably?** (deterministic, not flaky)                   | Continue | Go to Stage 5 |
| **Safely?** (won't charge money, send emails, delete data) | Continue | Go to Stage 5 |
| **Cheaply?** (won't cost $$ or take hours)                 | Continue | Go to Stage 5 |
| **Observably?** (can see what you need to assert)          | Continue | Go to Stage 5 |

**If YES to all**: Use real system at Level 2. DONE.

**If NO to any**: Go to Stage 5.

</stage_4>

<stage_5 name="Which Exception Applies?">
You've proven:

1. Evidence lives at Level 1
2. Code can't be factored into pure computation
3. Real system can't produce the behavior reliably/safely/cheaply/observably

**Now and only now** may you consider test doubles. But you must match a specific exception.

<exception_cases>

**Exception 1: Failure Modes**

**When**: You need to test behavior under specific failure conditions that the real system can't reliably produce.

Examples: timeouts at specific points, connection resets mid-stream, partial writes/reads, throttling/rate limits, retryable vs non-retryable error codes, DNS failures, TLS handshake failures.

**Double type**: Stub that returns predetermined errors.

**Exception 2: Interaction Protocols**

**When**: Correctness depends on the conversation pattern, not just input/output.

Examples: multi-step workflows (create → poll → finalize), pagination loops and cursor handling, transactional outbox patterns, sagas and compensating actions, "must call A before B", "must not call B if A failed", ensuring no extra calls (cost, rate limits).

**Double type**: Spy that records calls, or Mock for strict sequence assertions.

**Exception 3: Time and Concurrency**

**When**: You need deterministic control over timing and scheduling.

Examples: scheduled retries with jitter, token refresh races, lease renewal loops, debounce/throttle logic, "eventually consistent" wait loops, visibility timeouts.

**Double type**: Fake clock, controllable scheduler.

**Exception 4: Safety**

**When**: The real system is destructive or irreversible.

Examples: payment providers (charges/refunds), email/SMS sending, destructive admin APIs (delete, rotate keys), systems with strict quotas/billing.

**Double type**: Stub that records but doesn't execute.

**Exception 5: Combinatorial Cost**

**When**: You need 100+ scenarios and real system is too slow/expensive.

Examples: HTTP client behavior matrix (status codes × retry rules × idempotency × timeout budgets), queue consumer (redelivery × poison messages × DLQ routing × batch sizes), caching (TTL × stale-while-revalidate × negative caching × stampede protection).

**Double type**: Fake that can be configured for each scenario.

**Note**: Speed alone is not an exception. A test that takes 2 seconds with a real DB is fine. You need BOTH 100+ scenarios AND hours of runtime to justify this exception.

**Exception 6: Observability**

**When**: You need to verify details the real system can't expose.

Examples: "Did we include this header?", "Did we batch these operations?", "Did we send the idempotency key?", "Did we avoid N+1 calls?", "Did we handle pagination correctly?"

**Double type**: Spy that records request details.

**Exception 7: Contract Testing**

**When**: Third-party API you don't control, need to verify your serialization/parsing.

Examples: verify request serialization matches API spec, verify you can parse all documented response variants, pin behavior when provider changes subtly.

**Double type**: Contract stub that enforces expected format.

</exception_cases>

**If No Exception Applies**

STOP. You should not use test doubles.

Options:

1. Re-examine Stage 3B: Can you really not extract pure logic?
2. Test at Level 2: Use real dependencies
3. Accept no Level 1 test: Some glue code doesn't need unit tests

</stage_5>

<test_organization name="4-Part Test Progression">
Once you've determined the right level and approach, organize tests using this progression:

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

**Phase 1: Typical Cases**

Start with scenarios users encounter most often. What's the most common input? What does successful execution look like?

**Phase 2: Edge and Boundary Cases**

Test limits, special values, error conditions:

| Category           | Examples                                     |
| ------------------ | -------------------------------------------- |
| Boundaries         | 0, 1, max-1, max, max+1                      |
| Empty inputs       | Empty string, empty array, null, undefined   |
| Special values     | NaN, Infinity, negative numbers              |
| Error conditions   | Invalid input, missing fields                |
| Boundary crossings | Just below threshold, exactly at, just above |

**Phase 3: Systematic Coverage**

For code with loops, state machines, or combinatorial behavior:

- **Loop coverage**: zero iterations, one iteration, multiple iterations, max iterations
- **State transitions**: all valid transitions, verify invalid ones are rejected
- **Combinatorial testing**: key combinations of independent parameters

**Phase 4: Property-Based Tests**

Define invariants that should hold for ALL valid inputs, then let the test framework generate inputs.

**MANDATORY for these code types:**

- Parsers (parse(format(x)) === x)
- Mathematical operations
- Serialization/deserialization
- Complex algorithms where edge cases are hard to enumerate

**Common properties:**

| Property               | Description                     | Example                       |
| ---------------------- | ------------------------------- | ----------------------------- |
| Idempotency            | f(f(x)) === f(x)                | Formatting, normalization     |
| Round-trip             | decode(encode(x)) === x         | Serialization                 |
| Invariant preservation | Property holds before and after | Sorting doesn't lose elements |
| Commutativity          | f(a, b) === f(b, a)             | Set operations                |

Property-based testing is NOT optional for the categories listed above. If you're testing a parser or serializer without property-based tests, you're not done.

</test_organization>

<double_types name="Test Double Taxonomy">
When an exception case applies, use the appropriate double type:

| Double Type | Purpose                           | Use For                                          |
| ----------- | --------------------------------- | ------------------------------------------------ |
| **Stub**    | Returns predetermined responses   | Exceptions 1 (Failure), 4 (Safety), 7 (Contract) |
| **Spy**     | Records calls for verification    | Exceptions 2 (Interaction), 6 (Observability)    |
| **Fake**    | Simplified working implementation | Exceptions 3 (Time), 5 (Combinatorial)           |
| **Mock**    | Strict expectation verification   | Exception 2 (strict sequence only)               |
| **Dummy**   | Placeholder that's never called   | Satisfying type requirements                     |

**Stub**: Returns predetermined responses. Doesn't record calls or verify behavior—just provides controlled outputs. Use for testing how your code handles specific error responses, simulating failures, or replacing destructive operations.

**Spy**: Records calls for later verification. The primary purpose is to observe WHAT was called, not to control the response. Use for verifying request parameters, ensuring operations are batched, checking call ordering or count.

**Fake**: A simplified but working implementation. Unlike a stub (canned responses), a fake has real behavior—just simpler than production. Use for running many test scenarios at speed, or controlling time/scheduling.

**Mock**: Verifies that specific interactions happen in a specific order. The strictest form of test double. Use sparingly—only when exact sequence matters for correctness (saga compensation order, protocol requirements). Mocks couple tests to implementation details.

**Dummy**: A placeholder that satisfies a type requirement but is never actually used. If you need many dummies, the function may have too many dependencies.

**Critical distinction:**

- **Mock** (BAD default): Framework intercepts method calls on real objects
- **Test double** (OK when exception applies): You provide an alternative implementation via dependency injection

</double_types>

<anti_patterns>

**Anti-Pattern 1: Skipping the Evidence Question**

Wrong thinking: "I see `OrderService` calls `repository.save()`, so I need to test `OrderService` with a fake repository."

Right thinking: "What evidence do I need? Evidence that orders persist correctly. A fake repository proves nothing about persistence. Test at Level 2 with real database."

**Anti-Pattern 2: Calling Fakes "In-Memory Implementations"**

Wrong: Creating a class with an array that records calls and returns canned data, then calling it an "in-memory implementation."

Right: An in-memory implementation is a REAL implementation that happens to use memory (like SQLite `:memory:`). It has real SQL parsing, real constraints, real transactions. An array that records calls is a fake.

**Anti-Pattern 3: Faking for Speed**

Wrong thinking: "Real DB is slow (2 seconds), so I'll fake it."

Right thinking: Speed alone is not an exception. You need 100+ scenarios AND hours of runtime (Exception 5). A 2-second test is fine. Don't optimize developer experience at the cost of test validity.

**Anti-Pattern 4: Faking What Can Be Tested Real**

Wrong: Creating a fake HTTP client instead of testing against a real HTTP server (local test server or docker container).

Right: If you're testing "does my code handle HTTP responses correctly?"—use a real HTTP server. The fake proves your code works with your imagination of HTTP, not actual HTTP.

**Anti-Pattern 5: Testing Library Behavior**

Wrong: Testing that argparse correctly parses the `--verbose` flag.

Right: Testing YOUR behavior that uses the parsed result (e.g., verbose mode produces detailed output with DEBUG prefix).

Trust the library. Test your code.

</anti_patterns>

<failure_modes>
Common mistakes from actual usage:

**Failure 1: Skipped Stage 1 entirely**

What happened: Agent saw code with dependencies and immediately created fakes for everything.

Why it failed: Never asked "what evidence do I need?" Created tests that pass but prove nothing about production behavior.

How to avoid: ALWAYS start at Stage 1. Write down the answer before proceeding.

**Failure 2: Confused fake with real in-memory implementation**

What happened: Agent created `InMemoryUserRepository` with array storage, claimed it satisfied "DI with real in-memory implementations."

Why it failed: An array is not a database. It doesn't enforce constraints, handle transactions, or validate SQL.

How to avoid: Real in-memory = actual database/system in memory mode. Fake = canned responses you wrote.

**Failure 3: Used speed as sole justification**

What happened: Agent said "Real DB is slow, so I'll fake it for faster tests."

Why it failed: Speed alone doesn't qualify for Exception 5. Need 100+ scenarios AND hours of runtime.

How to avoid: Ask: "Do I have 100+ scenarios? Would real system take hours?" If no to either, use real system.

**Failure 4: Tested library instead of own code**

What happened: Agent wrote tests verifying that Zod validates schemas correctly.

Why it failed: Tested the library, not the application. Zod works. Test what YOUR code does with validation results.

How to avoid: Ask: "Am I testing MY behavior or the library's behavior?" Trust libraries; test your code.

**Failure 5: Used exception without documenting which one**

What happened: Agent created test doubles but didn't note which exception case justified them.

Why it failed: No way to verify the doubles are legitimate. Future maintainers can't assess whether the exception still applies.

How to avoid: When using doubles, add a comment citing the specific exception (e.g., "Exception 1: testing timeout retry behavior").

</failure_modes>

<cardinal_rule>
**Mocking is always wrong. There is no exception.**

Mocking (using mocking frameworks to intercept and stub arbitrary methods) gives you a test that passes while your production code fails. This is worse than no test at all.

If you feel you need to mock:

1. Redesign using dependency injection with real implementations or legitimate doubles (matching an exception case), OR
2. Test at a different level—push to Level 2 or 3 where real dependencies are available

The seven exception cases use **test doubles** (stubs, spies, fakes), not mocks. The distinction:

- **Mock**: Framework intercepts method calls on real objects
- **Test double**: You provide an alternative implementation via dependency injection

</cardinal_rule>

<success_criteria>
Testing is complete when:

- [ ] Stage 1 question answered: "What evidence do I need?" is written down for each test
- [ ] Evidence exists at the level where it can be proven (not faked)
- [ ] No test doubles without matching exception case
- [ ] Exception case documented in comments if doubles are used
- [ ] Property-based tests present for parsers, serializers, math operations, and complex algorithms
- [ ] Tests verify behavior (assertions on outputs), not implementation (assertions on call counts)
- [ ] All tests pass

</success_criteria>
