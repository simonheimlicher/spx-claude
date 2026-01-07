---
name: testing-typescript
description: |
  TypeScript-specific testing patterns with three-tier methodology (Unit/Integration/E2E).
  Use when writing tests, designing test architecture, or reviewing test code for TypeScript projects.
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

<skill_relationship>
**This skill provides TypeScript-specific testing patterns.**

**Required reading order:**

1. **FIRST**: Invoke `/testing` skill for foundational testing principles
2. **THEN**: Read this skill for TypeScript-specific implementations

This skill assumes you understand from `/testing`:

- The three testing levels (Unit/Integration/E2E)
- No mocking principle - use dependency injection
- Behavior vs implementation testing
- Reality is the oracle
- Test graduation workflow

**What this skill adds:** TypeScript-specific patterns, Vitest configuration, type-safe test factories, and concrete examples.
</skill_relationship>

<essential_principles>
**MAXIMUM CONFIDENCE. MINIMUM DEPENDENCIES. NO MOCKING. REALITY IS THE ORACLE.**

- Every dependency you add must **justify itself** with confidence gained
- If you can verify with pure functions, don't require binaries
- If you can verify with local binaries, don't require Chrome
- Mocking is a **confession** that your code is poorly designed
- Reality is the only oracle that matters
  </essential_principles>

<progress_vs_regression>
**CRITICAL INVARIANT: The production test suite (`test/`) MUST ALWAYS PASS.**

This is the deployment gate. A failing test in `test/` means undeployable code.

**The Two Test Locations:**

| Location           | Name                 | May Fail? | Purpose                                |
| ------------------ | -------------------- | --------- | -------------------------------------- |
| `specs/.../tests/` | **Progress tests**   | YES       | TDD red-green cycle during development |
| `test/`            | **Regression tests** | NO        | Protect working functionality          |

**Progress tests** allow TDD:

1. Write failing test in `specs/.../tests/` (RED)
2. Implement code until test passes (GREEN)
3. Graduate test to `test/` when work item complete

**Regression tests** protect the codebase:

- Run in CI on every commit
- Must always pass
- Failure = broken build = blocked deployment

**The Rule:**

> **NEVER write tests directly in `test/`**
>
> Writing a failing test in `test/` breaks CI until implementation is complete.
> Always write progress tests in `specs/.../tests/` first, then graduate them.

**Quick Decision:**

```
Am I implementing new functionality?
├── YES → Write test in specs/.../tests/ (progress test)
│         Graduate to test/ when DONE
└── NO  → Modify existing test in test/ (regression test)
          Must stay GREEN
```

</progress_vs_regression>

<core_principles>
**1. Behavior Only**

Tests verify **WHAT** the system does, not **HOW** it does it.

```typescript
// ❌ BAD: Testing implementation
it("uses execa to run hugo", async () => {
  const execaSpy = vi.spyOn(deps, "execa");
  await buildHugo(siteDir, deps);
  expect(execaSpy).toHaveBeenCalledWith("hugo"); // Tests HOW, not WHAT
});

// ✅ GOOD: Testing behavior
it("GIVEN site directory WHEN building THEN creates files in output", async () => {
  const result = await buildHugo(siteDir, deps);
  expect(result.buildDir).toBeDefined();
  expect(deps.existsSync(result.buildDir)).toBe(true); // Tests WHAT happened
});
```

**2. Dependency Injection Over Mocking**

Design code to accept dependencies as parameters. Then tests pass controlled implementations—no mocking framework needed.

```typescript
// ❌ BAD: Mocking external calls
vi.mock("execa", () => ({ execa: vi.fn() }));

it("runs hugo", async () => {
  await buildHugo(siteDir);
  expect(execa).toHaveBeenCalled();
});

// ✅ GOOD: Dependency injection
it("GIVEN valid site WHEN building THEN returns build directory", async () => {
  const deps: BuildDependencies = {
    execa: vi.fn().mockResolvedValue({ exitCode: 0 }),
    mkdtemp: vi.fn().mockResolvedValue("/tmp/hugolit-test"),
  };

  const result = await buildHugo(siteDir, deps);

  expect(result.buildDir).toBe("/tmp/hugolit-test");
});
```

**3. Escalation Requires Justification**

Each level adds dependencies. You must **justify** the confidence gained:

| Level | Dependencies Added   | Confidence Gained                                 |
| ----- | -------------------- | ------------------------------------------------- |
| 1 → 2 | Real binaries (Hugo) | "Unit tests pass, but does Hugo accept our args?" |
| 2 → 3 | Chrome + network     | "Integration works, but do real audits complete?" |

If you cannot articulate what confidence the next level adds, **don't escalate**.

**4. No Arbitrary Test Data**

All test data must be:

- **Generated**: Use factories, not literals
- **Ephemeral**: Created and destroyed within test scope
- **Randomized**: Expose ordering assumptions

```typescript
// ❌ BAD: Arbitrary strings
it("syncs files", async () => {
  await runLhci({ url: "http://example.com" });
});

// ✅ GOOD: Generated data
it("GIVEN config with URL set WHEN running THEN audits each URL", async () => {
  const config = createTestConfig({
    url_sets: { critical: ["/", "/about/"] },
  });

  await runLhci({ set: "critical" }, config, mockDeps);
});
```

**5. Fast Failure**

Tests run in order of speed and likelihood to fail:

1. **Environment checks first**: Is the tool installed? Is the server running?
2. **Simple operations before complex**: Can we parse config before running audits?
3. **Known failure modes early**: Check the most fragile assumptions first
   </core_principles>

<testing_levels>
**Level 1: Unit (No External Dependencies)**

- **Speed**: <50ms
- **Infrastructure**: Standard developer CLI tools + temp fixtures
- **Framework**: Vitest

Unit testing is fast and depends only on CLI tools installed by default on modern macOS and Linux developer machines:

**Standard tools (always available):**

- `git`, `node`, `npm`, `npx`, `curl`, `python`
- Node.js built-ins: `fs`, `path`, `os`, `crypto`

**Testing approach:**

- Dependency injection with controlled implementations
- Pure function testing
- **CRITICAL: MUST use `os.tmpdir()` exclusively**
- Never write outside OS-provided temporary directories
- Fast execution thanks to SSDs

The developer has bigger problems if git, node, npm, or curl are not available.
These are NOT external dependencies - they're part of the standard developer environment.

**See**: `levels/level-1-unit.md`

---

**Level 2: Integration (Local Binaries)**

- **Speed**: <1s
- **Infrastructure**: Project-specific tools OR virtualized environments

Integration testing covers two scenarios:

**1. Project-specific CLI tools:**

- Tools NOT installed by default: Claude Code, Hugo, Caddy, TypeScript compiler
- Integration with tools specific to your project
- Local execution only (no network)

**2. Virtualized environments:**

- Docker containers and containerized test environments
- Creates dependencies beyond standard developer setup

**See**: `levels/level-2-integration.md`

---

**Level 3: E2E (Full Workflow)**

- **Speed**: <30s
- **Infrastructure**: External dependencies for maximum confidence

Maximum confidence testing with external dependencies:

- Network services (GitHub API, external repos)
- Chrome and browser-based tools
- Complete real-world workflows

**Still respects filesystem boundaries:**

- MUST use `os.tmpdir()` exclusively OR containerized environments
- Never write to user directories or system locations

**See**: `levels/level-3-e2e.md`
</testing_levels>

<test_design_protocol>
Execute these phases IN ORDER when designing tests for a feature.

**Phase 1: Identify Behaviors**

List the behaviors to verify. Not implementation details—observable outcomes.

```markdown
## Behaviors to Verify

1. Given a Hugo site, build creates files in temp directory
2. Given URL set in config, audits each URL
3. Given LHCI failure, returns non-zero exit code
4. Given invalid config, throws descriptive error
```

**Phase 2: Assign Minimum Levels**

For each behavior, determine the **minimum** level that can verify it:

| Behavior                   | Minimum Level | Justification                 |
| -------------------------- | ------------- | ----------------------------- |
| Config parsing             | Level 1       | Pure function, can verify DI  |
| Command building           | Level 1       | Pure function, no external    |
| Hugo actually builds       | Level 2       | Need real Hugo binary         |
| Caddy serves files         | Level 2       | Need real Caddy binary        |
| Lighthouse scores returned | Level 3       | Need real Chrome + Lighthouse |

**Phase 3: Design Tests Bottom-Up**

Start at Level 1. Only escalate when Level 1 **cannot** verify the behavior.

```typescript
// Level 1: Verify command building logic
it("GIVEN checksum enabled WHEN building command THEN includes --checksum", () => {
  const cmd = buildLhciCommand({ checksum: true });

  expect(cmd).toContain("--checksum");
});

// Level 2: Verify Hugo accepts the command
it("GIVEN valid site WHEN building THEN Hugo exits with 0", async () => {
  const result = await buildHugo("test/fixtures/sample-site");

  expect(result.exitCode).toBe(0);
});

// Level 3: Verify full workflow
it("GIVEN sample site WHEN running hugolit THEN produces reports", async () => {
  const { exitCode, stdout } = await execa("node", ["bin/hugolit.js", "run"]);

  expect(exitCode).toBe(0);
  expect(stdout).toContain("reports written");
});
```

**Phase 4: Document Escalation Rationale**

In the test file, document why each level is necessary:

```typescript
/**
 * Test Levels for LHCI Runner:
 *
 * Level 1 (Unit):
 * - Command building logic
 * - Config parsing and validation
 * - Error message formatting
 * - Median/average calculations
 *
 * Level 2 (Integration):
 * - Hugo build with real binary
 * - Caddy server management
 * - Port selection
 *
 * Level 3 (E2E):
 * - Full LHCI audit with Chrome
 * - Lighthouse scores returned
 * - Report generation
 */
```

</test_design_protocol>

<anti_patterns>
**Anti-Pattern: Mock Everything**

```typescript
// ❌ Mocking destroys confidence
vi.mock("execa");
vi.mock("fs");
vi.mock("get-port");

it("runs lhci", async () => {
  await runLhci(config);
  expect(execa).toHaveBeenCalled(); // What did we prove? NOTHING.
});
```

**Anti-Pattern: Skip Levels**

```typescript
// ❌ Jumping to Level 3 without Level 1/2 coverage
it("runs full lighthouse audit", async () => {
  // This test is slow and requires Chrome
  const result = await runLighthouse({ url: "http://localhost:1313/" });
  // If this fails, we don't know if it's our code or Lighthouse
});
```

**Anti-Pattern: Test Implementation Details**

```typescript
// ❌ Testing HOW, not WHAT
it("uses execa with correct args", async () => {
  const spy = vi.spyOn(deps, "execa");
  await buildHugo(siteDir, deps);

  expect(spy).toHaveBeenCalledWith("hugo", ["--minify"]); // Implementation detail!
});

// ✅ Test the observable behavior instead
it("GIVEN site WHEN building THEN output is minified", async () => {
  const result = await buildHugo(siteDir, deps);

  // Verify the BEHAVIOR, not the implementation
  expect(result.minified).toBe(true);
});
```

**Anti-Pattern: Arbitrary Test Data**

```typescript
// ❌ Magic strings that hide assumptions
it("syncs files", async () => {
  await runLhci({ url: "http://localhost:1313/about/" });
});

// ✅ Generated, ephemeral data
it("GIVEN url set WHEN running THEN audits each URL", async () => {
  const config = createTestConfig({
    url_sets: { critical: ["/", "/about/"] },
  });

  await runLhci({ set: "critical" }, config, mockDeps);
});
```

</anti_patterns>

<level_references>
Detailed patterns and examples for each level:

| File                            | Purpose                                      |
| ------------------------------- | -------------------------------------------- |
| `levels/level-1-unit.md`        | Pure functions, DI, Node.js primitives       |
| `levels/level-2-integration.md` | Real binaries (Hugo, Caddy), local execution |
| `levels/level-3-e2e.md`         | Full workflow, Chrome, Lighthouse            |

</level_references>

<success_criteria>
Before declaring tests complete:

- [ ] All behaviors have tests at the minimum necessary level
- [ ] No mocking of external systems (DI instead)
- [ ] Escalation to each level is justified in comments
- [ ] Test data is generated, not hardcoded
- [ ] Fast failure: environment checks run first
- [ ] Each test verifies behavior, not implementation

*Remember: A test that passes because of mocks is worse than no test at all. It gives false confidence. Reality is the only oracle.*
</success_criteria>
