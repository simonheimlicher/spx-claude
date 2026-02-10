I want to position my ouctome engineering framework on Hacker News.

Critically review my draft of the Product Tree and Outcome Engineering Framework

What would you change in terms of naming, positioning, wording and the underlying rigorous concepts before making it public?

What would be the strongest counter arguments and how would you preemptively counter them?

What do you think of dropping the CODE acronym and instead focusing on outcome engineering? I am aware that outcome is a loaded term but I also think that at the level of capabilities and features it is appropriate, whereas stories should focus on behavior.

I have registered the domains `spx.sh` and `outcome.engineering` as a preventative measure to keep my options open.

---

# The Product Tree: From Backlog to Momentum

## The Problem with Backlogs

Traditional backlogs are graveyards of good intentions. They grow infinitely. Anyone can add to them. Items sink to the bottom and rot. The "backlog grooming" ritual is really backlog mourning—acknowledging that most items will never ship.

Backlogs frame development as **debt reduction**. You start each sprint already behind. Progress means shrinking a pile that others keep growing. This is demoralizing by design.

## The Momentum Alternative

CODE replaces the backlog with a **Product Tree**—a living structure where ideas must earn their place through concrete definition.

**The fundamental shift:**

| Backlog Mindset    | Momentum Mindset                 |
| ------------------ | -------------------------------- |
| Reducing debt      | Creating and realizing potential |
| Checking off tasks | Growing observable capabilities  |
| Sprint velocity    | Continuous coherent growth       |
| Infinite wishlist  | Concrete ceiling                 |

### Creating Potential

When you write a spec, you aren't adding to a pile. You are **creating potential energy** in the system. The spec defines a state of the world that doesn't yet exist but should. This potential wants to become real.

The work of engineering is converting potential into reality. Tests are the proof that conversion happened. The outcome ledger records when each piece of potential became real.

### The Concrete Ceiling

> "It is not possible to add an infinite number of ideas that will never ship."

This is CODE's most radical constraint. You cannot dump vague ideas into the system. Every idea must be concrete enough to fit the physics of the product:

```
NN-slug.capability/ → NN-slug.feature/ → NN-slug.story/ → scenarios (Gherkin)
```

If you can't express it as a Gherkin scenario with Given/When/Then, it doesn't belong in the engineering system. This naturally filters wishlist items, scope creep, and "wouldn't it be nice if" features.

**The Concrete Ceiling is not gatekeeping—it's quality control.** Ideas that can't be expressed concretely aren't ready for engineering. They belong in discovery, not delivery.

## BSP: Binary Space Partitioning for Dependencies

The `NN` in `NN-slug.capability/`, `NN-slug.feature/`, `NN-slug.story/` isn't arbitrary. It's **BSP numbering**—a scheme that encodes dependency order while allowing insertion at any point.

### The Rules

| Rule                   | Value                                          |
| ---------------------- | ---------------------------------------------- |
| Range                  | [10, 99] (two digits)                          |
| Meaning                | Lower BSP = dependency, same BSP = independent |
| First item             | Start at 21 (room for ~10 before)              |
| Insert between X and Y | `floor((X + Y) / 2)`                           |
| Append after X         | `floor((X + 99) / 2)`                          |

### Why BSP?

Traditional sequential numbering (01, 02, 03) breaks when you discover dependencies late:

```
02-auth.feature/
03-dashboard.feature/
```

You realize auth needs a config system first. With sequential numbering, you must renumber everything. With BSP:

```
21-config.feature/      ← Insert: floor((10 + 37) / 2) = 23... or just use 21 if first
37-auth.feature/        ← Was 02, now has room before it
54-dashboard.feature/   ← Was 03, now has room before and after
```

### Parallel Work: Same Number, Different Names

Items with the **same BSP number** can be worked on in parallel—they all depend on the previous number, but not on each other:

```
21-setup.story/
37-auth.story/        ← All depend on 21
37-profile.story/     ← but NOT on each other
37-settings.story/    ← can work in parallel
54-integration.story/ ← depends on ALL 37s completing
```

This extends to capabilities:

```
21-test-harness.capability/      ← Infrastructure (dependency for others)
37-users.capability/             ← Functional (independent of each other)
37-billing.capability/           ← Functional (independent of each other)
37-reports.capability/           ← Functional (independent of each other)
54-linter.capability/            ← Improvement (depends on functional work)
```

### Strategic Insertion

BSP enables two critical patterns:

| Pattern                   | Strategy             | When to use                                             |
| ------------------------- | -------------------- | ------------------------------------------------------- |
| **Discovered dependency** | Insert LOWER number  | You realize functional work needs a test harness first  |
| **Improvement/polish**    | Insert HIGHER number | You want to add a linter after core functionality works |

This means the same concept can appear at different BSP numbers:

```
21-auth.capability/       ← Core auth (dependency for others)
54-auth.capability/       ← Auth improvements (depends on other features)
```

### Unified Number Space: BSP First, Type Last

Within a container, ALL items share the same BSP number space—ADRs, features, stories. The BSP number comes first (for sorting), the type comes last (for identification).

**Example: Capability with interleaved ADRs and features**

```
37-users.capability/
├── 10-bootstrap.feature/             ← No ADR dependency, can start immediately
├── 15-config.feature/                ← No ADR dependency
├── 21-auth-strategy.adr.md           ← Architectural decision
├── 22-login.feature/                 ← Depends on ADR 21
├── 22-registration.feature/          ← Depends on ADR 21 (parallel with login)
├── 37-oauth.feature/                 ← Depends on features 22
├── 54-session-management.adr.md      ← Later decision (after OAuth works)
├── 55-logout.feature/                ← Depends on ADR 54
└── 55-token-refresh.feature/         ← Depends on ADR 54 (parallel with logout)
```

**Example: Feature with interleaved ADRs and stories**

```
22-login.feature/
├── login.feature.md                  ← Spec file (slug.type.md)
├── 10-parse-credentials.story/       ← No ADR needed
├── 21-password-hashing.adr.md        ← Security decision
├── 22-hash-password.story/           ← Implements ADR 21
├── 22-verify-password.story/         ← Implements ADR 21 (parallel)
├── 37-rate-limiting.adr.md           ← Performance decision
├── 38-throttle-attempts.story/       ← Implements ADR 37
└── 54-login-flow.story/              ← Integration (depends on all above)
```

**What this enables:**

| Pattern             | Example                                            | Meaning                                        |
| ------------------- | -------------------------------------------------- | ---------------------------------------------- |
| Feature before ADR  | `10-bootstrap.feature/` before `21-auth.adr.md`    | Some work doesn't need architectural decisions |
| ADR blocks features | `21-auth.adr.md` blocks `22-login.feature/`        | Decision must be made before dependent work    |
| Parallel after ADR  | `22-login.feature/` and `22-registration.feature/` | Both depend on ADR, not on each other          |
| Later ADR           | `54-session.adr.md` after features 22-37           | Some decisions emerge from implementation      |

**The name tells you both WHEN (BSP prefix) and WHAT (type suffix).**

### Sibling-Unique, Not Global

BSP numbers are only unique among siblings (combined with slug):

```
21-foo.capability/21-bar.feature/54-baz.story/  ← One story-54
21-foo.capability/37-qux.feature/54-baz.story/  ← DIFFERENT story-54
37-other.capability/21-bar.feature/54-baz.story/  ← DIFFERENT story-54
```

**Always use full paths** when referencing work items. "story-54" is ambiguous; the full path is not.

## The Product Tree Structure

```
spx/
├── product.prd.md                        # The trunk: what is this product?
├── 21-core-decision.adr.md               # Product-level ADR
│
├── 21-test-harness.capability/           # Infrastructure (dependency for others)
│   ├── test-harness.capability.md
│   ├── outcomes.yaml
│   └── tests/
│
├── 37-users.capability/                  # Functional (parallel with billing)
│   ├── users.capability.md
│   ├── outcomes.yaml
│   │
│   ├── 10-bootstrap.feature/             # Feature before any ADR
│   │   └── bootstrap.feature.md
│   │
│   ├── 21-auth-strategy.adr.md           # ADR at position 21
│   │
│   ├── 22-login.feature/                 # Feature depends on ADR 21
│   │   ├── login.feature.md
│   │   ├── outcomes.yaml
│   │   │
│   │   ├── 21-password-hashing.adr.md        # Feature-level ADR
│   │   ├── 22-hash-password.story/           # Story depends on ADR 21
│   │   │   ├── hash-password.story.md
│   │   │   ├── outcomes.yaml
│   │   │   └── tests/
│   │   │       └── hash-password.unit.test.ts
│   │   │
│   │   └── 22-verify-password.story/         # Parallel with hash-password
│   │       ├── verify-password.story.md
│   │       └── tests/
│   │
│   └── 37-profile.feature/               # Feature depends on login (22)
│       └── profile.feature.md
│
├── 37-billing.capability/                # Parallel with users (same BSP)
│   └── billing.capability.md
│
└── 54-linter.capability/                 # Improvement (after functional)
    └── linter.capability.md
```

**Key observations:**

- Directory format: `{BSP}-{slug}.{type}/` (BSP first, then slug, type suffix)
- Spec file format: `{slug}.{type}.md` (matches directory naming)
- ADR format: `{BSP}-{slug}.adr.md` (flat files, interleaved with containers)
- Status derived from the outcome ledger, not a separate status file
- Everything sorts by BSP number first—humans see dependency order at a glance

### Why a Tree?

1. **Trees grow coherently.** You can't have a leaf without a branch, a branch without a trunk. Ideas must connect to existing structure.

2. **Trees are prunable.** Removing a capability removes its features and stories. No orphaned tickets.

3. **Trees show relationships.** The path from root to leaf IS the context. No need for "blocked by" or "related to" metadata.

4. **Trees are navigable.** You can zoom to any level—product vision, capability area, specific behavior—and understand where you are.

## Story States: Precision Over Poetry

States must communicate **what needs to happen**, not just describe the situation. Poetry in philosophy, precision in indicators.

| State         | Meaning                            | Required Action                 |
| ------------- | ---------------------------------- | ------------------------------- |
| **Unknown**   | Test Files links don't resolve     | Write tests                     |
| **Pending**   | Tests exist, not all passing       | Fix code or fix tests           |
| **Stale**     | Spec or test blob changed          | Re-commit with `spx spx commit` |
| **Passing**   | All tests pass, blobs unchanged    | None—potential realized         |
| **Regressed** | Was passing, now fails, blobs same | Investigate and fix             |

### Why Not "Aspiration" and "Realized"?

The original proposal suggested renaming these states to feel more positive:

- "Pending" → "Aspiration"
- "Passing" → "Realized"
- "Stale" → "Evolving"

The problem: **"Evolving" sounds like progress when it's actually a problem.** "Stale" is ugly but communicates urgency. When scanning a status board, you need to instantly know what needs attention.

Reserve inspirational language for the philosophy. Keep indicators clinical.

## Momentum Metrics

Traditional metrics (velocity, burndown) measure how fast you're emptying a bucket that keeps refilling. Momentum metrics measure the health of the tree.

### Tree Health Indicators

| Metric               | What it measures                                 |
| -------------------- | ------------------------------------------------ |
| **Realization Rate** | Stories moving from Pending → Passing per week   |
| **Drift**            | % of Passing stories that Regressed this week    |
| **Potential Energy** | Count of Pending + Stale stories (work waiting)  |
| **Coverage Depth**   | % of capabilities with passing integration tests |

### Understanding Drift

Drift measures how often *unrelated* changes break *previously working* stories. A story regresses when its tests fail without anyone touching its spec or tests—something elsewhere in the codebase caused the failure.

| Drift Rate | Interpretation                            |
| ---------- | ----------------------------------------- |
| < 1%       | Well-isolated architecture                |
| 1-5%       | Normal coupling, monitor trends           |
| 5-10%      | High coupling, consider refactoring       |
| > 10%      | Brittle architecture, intervention needed |

**Drift is unavoidable but manageable.** All codebases drift. The goal isn't zero drift—it's understanding your drift rate and keeping it sustainable.

### The Momentum Dashboard

```
Product: spx-cli
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Realized      ████████████████████░░░░  42/52 stories (81%)
In Motion     ████░░░░░░░░░░░░░░░░░░░░   6/52 stories
Potential     ██░░░░░░░░░░░░░░░░░░░░░░   4/52 stories

Drift: 2.4% (1/42 regressed this week)
This week: +3 realized, +2 potential created
```

This dashboard tells a story of growth, not debt.

## The Philosophy in Practice

### Writing a New Spec = Creating Potential

When you create `54-export-csv.story/export-csv.story.md`, you're not adding to a backlog. You're defining a piece of reality that should exist. The system now has potential energy—a gap between what is defined and what is proven.

### Passing Tests = Realizing Potential

When the outcome ledger records that all scenarios pass, potential has become reality. The story isn't "done" (done implies it goes away). The story is **realized**—it describes something true about the product.

### Editing a Spec = Raising the Bar

When you modify a realized story, you're not "breaking" anything. You're creating new potential. The spec now describes a better reality than what exists. Tests become stale because reality needs to catch up to the new vision.

### Deleting a Spec = Pruning the Tree

When you remove a story, you're not "closing a ticket." You're pruning—deciding this branch no longer serves the tree's growth. The product becomes simpler, more focused.

## Migration from Backlog Thinking

If your team uses traditional backlogs, the shift is cultural before it's technical:

1. **Stop saying "tickets."** Say "stories" or "outcomes."
2. **Stop saying "close."** Say "realize" or "prove."
3. **Stop measuring velocity.** Measure realization rate.
4. **Stop grooming.** Start pruning.
5. **Stop estimating.** Start decomposing until stories are obvious.

The tools follow the language. Change how you talk, and the process changes with it.

---

## Summary

The Product Tree replaces the infinite backlog with a coherent, growable structure. The Concrete Ceiling ensures only well-defined work enters the system. Momentum metrics measure growth rather than debt reduction.

**The key insight:** Development isn't about emptying a pile. It's about growing a tree—creating potential and converting it to reality, one proven behavior at a time.

---

# Customer Outcome Driven Engineering (CODE)

## Rationale

CODE treats specs as durable, version-controlled product structure—not "work to implement." The `spx/` tree is the always-current map of the product. Each container holds its spec, tests, and outcome ledger together, so agents can discover, validate, and evolve the system with minimal guesswork. Outcomes are proven by the outcome ledger, validated at precommit and insured by CI.

**Important**: Use the `spx` CLI for all structural operations (like `gh` for GitHub).

---

## Principles

1. **Durable map**
   The spec tree is the always-on system map. Nothing moves because work is "done."

2. **Co-location**
   Each container holds its spec file, tests directory, and outcome ledger together. No parallel trees to synchronize.

3. **Type in names, order in directories**
   Container type is in both directory and file names. Ordering (BSP) is in directory names only, so rebalancing never requires file renames.

4. **Tests are the executable proof**
   The spec describes intent, constraints, and strategy. Tests prove the implementation works. The outcome ledger is the machine-verifiable contract.

5. **Incomplete is valid**
   A container with 2 of 5 tests passing is in progress, not broken. The outcome ledger lists only passing tests.

6. **Blob-based staleness**
   Staleness is detected by comparing git blob SHAs, not timestamps. This is deterministic across rebases, checkouts, and touched files.

7. **Precommit is primary, CI is insurance**
   Agents get feedback at precommit. CI validates that precommit wasn't bypassed.

8. **Decisions are durable and updated in-place**
   No "superseded" decisions. When a decision changes, update it in place.

9. **Tool-mediated refactors**
   Moves/renames are performed via `spx` CLI to keep structure consistent.

10. **Test infrastructure is first-class**
    Harnesses are production code requiring their own specs and test coverage.

11. **Higher levels unaware of lower level breakdown**
    Features don't list story outcomes. Capabilities don't list feature outcomes. Completion bubbles up through the outcome ledger, not spec references. This prevents drift between levels.

12. **Analysis sections are not contracts**
    Story specs include analysis of files, constants, and configuration to prove the agent examined the codebase. Implementation may diverge as understanding deepens—this is expected, not a failure.

13. **No stale references in specs**
    Specs contain only durable information. Dependencies are encoded in the tree (BSP ordering), not listed in spec files. Harness references appear in test tables, not separate sections. Nothing that can go stale belongs in a spec.

---

## Structure

### Container hierarchy

```
spx/
  {product-slug}.prd.md              # Product requirements (one per product)
  NN-{slug}.adr.md                   # Product-wide decisions
  13-test-infrastructure.capability/ # Harnesses are first-class
    {slug}.capability.md
    NN-{slug}.feature/
      {slug}.feature.md
      outcomes.yaml
      tests/
  NN-{slug}.capability/
    {slug}.capability.md
    outcomes.yaml                         # Outcome ledger (may be incomplete)
    tests/
    NN-{slug}.adr.md
    NN-{slug}.feature/
      {slug}.feature.md
      outcomes.yaml
      tests/
      NN-{slug}.adr.md
      NN-{slug}.story/
        {slug}.story.md
        outcomes.yaml
        tests/
```

### Directory naming

```
{BSP}-{slug}.{type}/
```

Examples:

- `13-test-infrastructure.capability/`
- `15-validation.capability/`
- `21-testable-validation.feature/`
- `47-validation-commands.story/`

See [Appendix: Recursive Decimal BSP](#appendix-recursive-decimal-bsp) for insertion algorithms.

### Spec file naming

Inside each container, the canonical spec file:

```
{slug}.{type}.md
```

Examples:

- `15-validation.capability/validation.capability.md`
- `21-testable-validation.feature/testable-validation.feature.md`
- `47-validation-commands.story/validation-commands.story.md`

BSP is in directory only—renumbering never touches spec files.

### Decisions (ADR)

- Location: in the container where the decision applies
- Naming: `NN-{slug}.adr.md`
- Updated in-place; no "superseded" workflow

---

## Capability and Feature Spec Format

Capabilities and features describe **intent, constraints, and strategy**. Tests are the **executable proof**.

### Structure

```markdown
# {Capability|Feature}: {Name}

## Purpose

What this container delivers and why it matters.

## Requirements

Prose description of functional and quality requirements.
Constraints and invariants that tests must verify.

## Test Strategy

| Component         | Level | Harness     | Rationale                       |
| ----------------- | ----- | ----------- | ------------------------------- |
| Argument parsing  | 1     | -           | Pure function, standard dev env |
| Config validation | 1     | -           | Pure function with temp dirs    |
| CLI integration   | 2     | cli-harness | Needs real spx binary           |
| Full workflow     | 3     | e2e-harness | Needs credentials               |

### Escalation Rationale

- **1 → 2**: [What confidence does Level 2 add that Level 1 cannot provide?]
- **2 → 3**: [What confidence does Level 3 add that Level 2 cannot provide?]

## Outcomes

### 1. [Scenario name]

\`\`\`gherkin
GIVEN [precondition]
WHEN [action]
THEN [expected result]
\`\`\`

| File                                   | Level | Harness                                                                     |
| -------------------------------------- | ----- | --------------------------------------------------------------------------- |
| [{slug}.e2e](tests/{slug}.e2e.test.ts) | 3     | [e2e-harness](spx/NN-test-infrastructure.capability/NN-e2e-harness.feature) |

## Architectural Constraints

| ADR                              | Constraint                         |
| -------------------------------- | ---------------------------------- |
| [adr-NN_name](../adr-NN_name.md) | [What constraint this ADR imposes] |
```

### Rationale

- **No drift** between spec prose and outcome ledger—it is the sole contract
- **Test strategy** documents approach without duplicating test logic
- **Harness references** point to the harness spec (the durable contract), not the implementation code
- **Prose requirements** capture intent that tests alone cannot express
- **Outcomes** use the same numbered Gherkin pattern as stories for consistency
- **Architectural Constraints** reference ADRs that impose requirements on this container

---

## Story Spec Format

Stories describe **specific outcomes with implementation details**. Unlike capabilities and features, stories document the exact implementation path as proof of analysis.

### Structure

```markdown
# Story: {Name}

## Purpose

What this story delivers and why it matters.

## Outcomes

### 1. {Outcome name}

\`\`\`gherkin
GIVEN {precondition}
WHEN {action}
THEN {expected result}
AND {additional assertion}
\`\`\`

#### Test Files

| File                                     | Level | Harness |
| ---------------------------------------- | ----- | ------- |
| [{slug}.unit](tests/{slug}.unit.test.ts) | 1     | -       |

#### Analysis

*Implementation may diverge as understanding deepens.*

| File               | Intent         |
| ------------------ | -------------- |
| `src/path/file.ts` | {What changes} |
| `src/path/new.ts`  | {What it does} |

| Constant                       | Intent                    |
| ------------------------------ | ------------------------- |
| `src/constants.ts::CONST_NAME` | {Why using existing}      |
| `src/constants.ts::NEW_CONST`  | {Why new constant needed} |

| Config Parameter | Test Values      | Expected Behavior   |
| ---------------- | ---------------- | ------------------- |
| `ENV_VAR`        | `unset`, `value` | {Behavior for each} |

---

### 2. {Second outcome if needed}

{Same structure: gherkin + 4 tables}

---

## Architectural Constraints

| ADR                              | Constraint                         |
| -------------------------------- | ---------------------------------- |
| [NN-name.adr](../NN-name.adr.md) | {What constraint this ADR imposes} |
```

### Rationale

- **Gherkin is source of truth**—tests implement it, spec doesn't contain code
- **Test Files table** is the contract—harness references here, not in separate sections
- **Analysis section proves examination**—agent looked before coding, but implementation may diverge
- **No completion criteria**—stories are atomic; the outcome ledger is the contract
- **No inline code**—code in specs drifts from actual implementation

---

## Tests

Each container has a `tests/` subdirectory containing all tests for that outcome.

### Test naming

Test level is in the filename suffix:

- `*.unit.test.ts`
- `*.integration.test.ts`
- `*.e2e.test.ts`

### Path derivation

For each `test_file` in `outcomes.yaml`, SPX derives the runnable path as:

```
<container>/tests/<test_file>
```

The `tests/` prefix is never stored in `outcomes.yaml`.

### Test runner configuration

```ts
// vitest.config.ts
export default {
  include: ["spx/**/tests/*.test.ts"],
};
```

---

## Outcome Ledger

Each container MAY have an `outcomes.yaml` file listing tests that currently pass. This is the **machine-verifiable proof** for the container—the record of which potential has been realized.

### Purpose

The outcome ledger answers a question Git cannot: "Did this content pass tests, and when?"

Git provides cryptographic integrity of content (a Merkle tree of blobs). The outcome ledger provides verification state—a separate Merkle tree tracking which tests pass and how containers relate to each other.

### Key Properties

- **Incomplete ledgers are valid** - A container with 2 of 5 tests passing is in progress, not broken
- **Never hand-edited** - Generated by `spx spx claim`, verified by `spx spx verify`
- **Tree coupling** - Parent ledgers reference child ledgers, creating a hierarchy of verification state
- **States are derived** - Unknown, Pending, Stale, Passing, Regressed—computed from ledger contents

### Container States

| State         | Condition                         | Required Action     |
| ------------- | --------------------------------- | ------------------- |
| **Unknown**   | No tests exist                    | Write tests         |
| **Pending**   | Tests exist, not all claimed      | Fix code or claim   |
| **Stale**     | Descendant outcomes_blob mismatch | Re-claim            |
| **Passing**   | All tests pass, blobs match       | None                |
| **Regressed** | Claimed test fails                | Investigate and fix |

### Commands

```bash
spx spx test <container>     # Run tests, show results
spx spx claim <container>    # Assert tests pass, update outcomes.yaml
spx spx verify <container>   # Check that claims hold
spx spx status <container>   # Show states without running tests
```

For detailed format specification and design rationale, see [outcome-ledger.md](outcome-ledger.md)

---

## Precommit Validation

Precommit is the primary feedback loop. CI is insurance.

### What Precommit Catches

| Scenario                              | Result                     |
| ------------------------------------- | -------------------------- |
| Claimed test fails                    | Regression - blocks commit |
| Descendant ledger blob changed        | Stale - re-claim required  |
| Test file deleted but still in ledger | Phantom - blocks commit    |
| New test file not yet in ledger       | In progress - OK           |

### Development Flow

1. **Write spec**: define intent, constraints, and test strategy
2. **Implement**: write code and tests
3. **Claim outcome**: run `spx spx claim <container>` to update the outcome ledger
4. **Commit**: spec + implementation + tests + outcomes.yaml together
5. **Precommit**: validates no regressions, no phantoms, no staleness
6. **CI**: re-runs validation as insurance

For detailed validation rules, see [outcome-ledger.md](outcome-ledger.md)

---

## Test Infrastructure

Test harnesses are production code requiring their own specifications and test coverage.

### Location

Harness **specs** live in `spx/`; harness **code** lives in `tests/harness/` (same level as `src/`):

```
project/
├── src/                                  # Implementation code
├── tests/
│   └── harness/                          # Harness implementation code
│       ├── cli/                          # CLI harness code
│       └── e2e/                          # E2E harness code
└── spx/
    └── 13-test-infrastructure.capability/  # Harness specs
        ├── test-infrastructure.capability.md
        ├── 10-cli-harness.feature/
        │   ├── cli-harness.feature.md
        │   ├── outcomes.yaml
        │   └── tests/                    # Tests for the harness itself
        │       ├── setup.unit.test.ts
        │       └── isolation.integration.test.ts
        └── 20-e2e-harness.feature/
            ├── e2e-harness.feature.md
            ├── outcomes.yaml
            └── tests/
```

**Key distinction**: Harness specs in `spx/` define *what the harness must do*. Harness code in `tests/harness/` is *the implementation*. Tests in `spx/.../tests/` verify the harness works correctly.

**Harness references in Test Files tables point to the spec**, not the implementation. The spec is the durable contract; code organization can change.

### Why harnesses need specs

1. **Harness bugs break all dependent tests** - high impact requires tracking
2. **Harness refactoring needs regression protection** - the outcome ledger detects breakage
3. **Harness capabilities need documentation** - what can tests rely on?
4. **Harness dependencies need BSP ordering** - tests depend on harness (lower BSP)

### Harness spec format

````markdown
# Feature: CLI Harness

## Purpose

Provides isolated CLI execution environment for Level 2 tests.

## Capabilities

- Spawns spx binary in temp directory
- Captures stdout/stderr
- Manages fixture files
- Cleans up after test

## Test Strategy

| Component           | Level | Rationale                        |
| ------------------- | ----- | -------------------------------- |
| Temp dir management | 1     | Pure filesystem with os.tmpdir() |
| Process spawning    | 2     | Needs real spx binary            |

## Usage

```typescript
import { withCliHarness } from "tests/harness/cli";

it("runs spx spec next", async () => {
  await withCliHarness(async (harness) => {
    const result = await harness.run("spx", ["spec", "next"]);
    expect(result.exitCode).toBe(0);
  });
});
```

## Dependencies

- spx binary must be built
- Node.js >= 18
````

---

## Reference Layout

```text
spx/
  myproduct.prd.md
  21-testing-strategy.adr.md
  13-test-infrastructure.capability/
    test-infrastructure.capability.md
    10-cli-harness.feature/
      cli-harness.feature.md
      outcomes.yaml
      tests/
        setup.unit.test.ts
  15-validation.capability/
    validation.capability.md
    outcomes.yaml
    tests/
      validation.e2e.test.ts
    21-isolated-test-fixtures.adr.md
    21-testable-validation.feature/
      testable-validation.feature.md
      outcomes.yaml
      tests/
        validation.integration.test.ts
      04-level-2-integration-only.adr.md
      47-validation-commands.story/
        validation-commands.story.md
        outcomes.yaml
        tests/
          commands.unit.test.ts
```

---

## Appendix: Recursive Decimal BSP

**Binary Space Partitioning (BSP)** encodes dependency order: lower BSP items are dependencies that higher-BSP items may rely on; same BSP means independent. The "binary" refers to insertion by halving available space.

### Syntax

```text
{BSP}-{slug}.{type}
```

- **BSP**: Two digits (10-99), with `@` for recursive insertion
- **`-`**: Slug separator (hyphen, ASCII 45)
- **slug**: Human-readable name
- **type**: `.capability`, `.feature`, `.story`, or `.adr`

### Why Hyphen

Filesystems sort by ASCII code. Hyphen (45) sorts before `@` (64), ensuring parents appear before children:

| Character | ASCII | Sort Position |
| --------- | ----- | ------------- |
| `-`       | 45    | Parent        |
| `@`       | 64    | Children      |

```text
20-auth.capability/      ← Parent (- sorts first)
20@50-mfa.feature/       ← Child (@ sorts after -)
21-billing.capability/   ← Sibling (2 > @)
```

### Insertion Algorithms

**Append** (new sibling after 20):

```text
floor((20 + 99) / 2) = 59 → 59-billing
```

**Insert** (between 20 and 30):

```text
floor((20 + 30) / 2) = 25 → 25-subscriptions
```

**Recursive insert** (between 20 and 21, no integer space):

```text
20@floor((10 + 99) / 2) = 20@54 → 20@54-audit
```

**Deep recursion** (between 20@54 and 20@55):

```text
20@54@50-detailed-trace
```

### Rules

1. **Hyphen only**: Always `-` between BSP and slug, never `_`
2. **No renaming**: `20-auth` stays `20-auth` forever; use `@` to insert
3. **Rebalance at depth 3**: If you reach `20@50@50@...`, consider rebalancing
4. **Same BSP = parallel**: `20-auth.capability/` and `20-billing.capability/` can be built concurrently

===

# Outcome Ledger

The outcome ledger records verification state for the Product Tree. Each container MAY have an `outcomes.yaml` file that claims its tests pass.

## Format

```yaml
tests:
  - file: login.unit.test.ts
    passed_at: 2026-01-28T14:15:00Z
  - file: login.integration.test.ts
    passed_at: 2026-01-28T14:15:00Z
descendants:
  - path: 10-parse-credentials.story/
    outcomes_blob: a3f2b7c
  - path: 22-validate-token.story/
    outcomes_blob: 9bc4e1d
```

| Field                         | Description                                    |
| ----------------------------- | ---------------------------------------------- |
| `tests[].file`                | Test filename relative to container's `tests/` |
| `tests[].passed_at`           | ISO 8601 timestamp when test passed            |
| `descendants[].path`          | Child container directory name                 |
| `descendants[].outcomes_blob` | Git blob SHA of child's outcomes.yaml          |

## Container States

| State         | Condition                         | Required Action     |
| ------------- | --------------------------------- | ------------------- |
| **Unknown**   | No tests exist                    | Write tests         |
| **Pending**   | Tests exist, not all claimed      | Fix code or claim   |
| **Stale**     | Descendant outcomes_blob mismatch | Re-claim            |
| **Passing**   | All tests pass, blobs match       | None                |
| **Regressed** | Claimed test fails                | Investigate and fix |

States are mutually exclusive. Every container is in exactly one state.

## Tree Coupling

Parent outcomes.yaml stores `outcomes_blob` for each child. When a child's outcomes.yaml changes:

1. Child's Git blob changes
2. Parent's stored `outcomes_blob` no longer matches
3. Parent becomes **Stale**
4. Parent must re-claim to update references

This creates a Merkle tree of verification state, separate from Git's content Merkle tree.

## Commands

### `spx spx test [container [--tree] | --all]`

Run tests without changing anything.

```bash
spx spx test spx/21-auth.capability/22-login.feature/
# Run login's tests only, show results

spx spx test spx/21-auth.capability/ --tree
# Run capability + all descendants

spx spx test --all
# Run all tests in spx/
```

### `spx spx claim [container [--tree] | --all]`

Assert tests pass and update outcomes.yaml.

```bash
spx spx claim spx/21-auth.capability/22-login.feature/10-parse.story/
# 1. Run story's tests
# 2. All pass? Update outcomes.yaml
# 3. Any fail? Error, cannot claim

spx spx claim spx/21-auth.capability/ --tree
# Claim all descendants bottom-up, then claim capability
```

**Claim requires bottom-up order.** If a child changed, parent cannot claim until child is re-claimed.

### `spx spx verify [container | --all]`

Check that claims hold by running claimed tests.

```bash
spx spx verify --all
# Run all tests listed in outcomes.yaml files
# Report: Passing or Regressed
```

### `spx spx status [container | --all]`

Show current states without running tests.

```bash
spx spx status --all
# Show tree with states: Unknown, Pending, Stale, Claimed
```

Note: `status` shows "Claimed" for containers with outcomes.yaml. Use `verify` to confirm Passing vs Regressed.

## State Computation

### Leaf Containers (Stories)

Story state IS its aggregate state (no descendants).

### Non-Leaf Containers (Features, Capabilities)

Aggregate state = worst of (local state, descendant states).

**State ordering (worst to best):**

```
Regressed > Stale > Pending > Unknown > Passing
```

### Examples

```
Capability: Stale (child changed)
  Feature-1: Passing
    Story-1: Passing
  Feature-2: Stale (child changed)
    Story-2: Passing
    Story-3: Passing ← just re-claimed, blob changed
```

```
Capability: Regressed (worst of all)
  Feature-1: Passing
  Feature-2: Regressed
    Story-2: Regressed ← claimed test now fails
```

## Workflow

```bash
# 1. Development: run tests, iterate
spx spx test spx/path/to/container/

# 2. Ready: claim tests pass
spx spx claim spx/path/to/container/ --tree

# 3. Commit: record the claim
git add spx/ && git commit -m "feat: implement X"

# 4. CI: verify claims hold
spx spx verify --all
```

## Verification Reduction

Only tests in outcomes.yaml are run during `verify`. Containers without outcomes.yaml (Unknown/Pending) are not verified—there's nothing claimed to check.

This provides test reduction: verify runs claimed tests, not all possible tests.

---

# Outcome Ledger: Design Rationale

This document explains the design decisions behind the outcome ledger. For the specification, see [outcome-ledger.md](outcome-ledger.md).

## The Duality: Git and the Outcome Ledger

Two parallel Merkle trees serve different purposes:

| Aspect            | Git                             | Outcome Ledger                  |
| ----------------- | ------------------------------- | ------------------------------- |
| Question answered | "What is the content?"          | "Did this content pass tests?"  |
| Tracks            | Specs, tests, implementation    | Verification state              |
| Merkle structure  | Content blobs → trees → commits | Descendant outcomes_blobs       |
| Proof             | Cryptographic (hash integrity)  | Empirical (run tests to verify) |

Git provides content integrity. The outcome ledger provides verification state. Neither duplicates the other's job.

## Why We Don't Store Spec/Test/Implementation Blobs

Early designs stored `spec_blob` and `test_blob` in outcomes.yaml to detect staleness:

```yaml
# REJECTED DESIGN
spec_blob: abc123
tests:
  - file: test.ts
    blob: def456
```

**Problems:**

1. **Incomplete coverage**: We cannot store implementation blobs without enumerating all source files each test depends on. The staleness detection would miss implementation changes.

2. **Duplicates Git's job**: Git already tracks content. Storing blobs in outcomes.yaml partially duplicates Git's Merkle tree, but incompletely.

3. **False confidence**: If spec_blob and test_blob match, we might skip running tests—but implementation could have changed, causing failures.

**Conclusion:** Partial blob storage provides unreliable staleness detection. Better to not store content blobs at all and rely on running tests.

## Why We Store Descendant outcomes_blobs

While we don't store content blobs, we DO store `outcomes_blob` for each child:

```yaml
descendants:
  - path: 10-parse.story/
    outcomes_blob: a3f2b7c
```

**This provides tree coupling:**

1. Child claims tests pass → child's outcomes.yaml updated
2. Child's Git blob changes
3. Parent's stored `outcomes_blob` no longer matches
4. Parent is Stale → must re-claim

**Why this works but content blobs don't:**

- `outcomes_blob` captures the child's ENTIRE verification state (its tests + ITS descendants)
- One blob reference cascades all the way down
- We're building a Merkle tree of OUTCOMES, not of CONTENT

## Alternative Designs Considered

### Option 1: Store Content Blobs

```yaml
spec_blob: abc123
test_blobs:
  - file: test.ts
    blob: def456
```

**Rejected because:** Ignores implementation. A test might fail due to src/ changes, but we'd skip it because spec/test blobs match.

### Option 2: Separate Outcome Commits

Structure:

```
A: code + spec + tests (code commit)
B: outcomes.yaml only (outcome commit)
```

Detect staleness via `git diff A..HEAD`.

**Rejected because:** Any code change invalidates all outcomes. This helps for auditing but not for test reduction during development.

### Option 3: Pure Claims (No Blobs at All)

```yaml
tests:
  - file: test.ts
    passed_at: 2026-01-28T14:15:00Z
```

No tree coupling. Parent doesn't know if children changed.

**Rejected because:** Loses the Merkle property. Parent can be "Passing" while children changed underneath.

### Option 4: Minimal Merkle (Chosen)

```yaml
tests:
  - file: test.ts
    passed_at: 2026-01-28T14:15:00Z
descendants:
  - path: child/
    outcomes_blob: abc123
```

Store only descendant outcomes_blobs. Get tree coupling without duplicating Git.

**Chosen because:**

- Tree coupling via outcomes_blob
- No content blob duplication
- Clear separation: Git tracks content, ledger tracks verification

## Command Naming: `test` and `claim`

### Why `test` (not `run`, `check`)

- `test` is unambiguous: run the tests
- `run` is generic (run what?)
- `check` might imply status check without running

### Why `claim` (not `commit`, `record`, `pass`)

- `claim` emphasizes epistemological status: it's an assertion, not proof
- `commit` conflicts with `git commit`
- `record` is neutral but loses the claim semantics
- `pass` is confusing (verb vs test result)

**The word "claim" is intentionally precise:**

outcomes.yaml is a claim that can be verified independently. Timestamps are metadata, not evidence. Anyone can checkout the commit and run the claimed tests to verify the claim.

## Epistemology: Claims vs Proofs

The outcome ledger provides **claims**, not **proofs**.

| Aspect                | Claim                       | Proof                               |
| --------------------- | --------------------------- | ----------------------------------- |
| What outcomes.yaml is | Assertion that tests passed | Would require cryptographic witness |
| Timestamps            | Metadata (when claim made)  | Not evidence                        |
| Verification          | Run tests independently     | Would be redundant if proven        |
| Trust model           | Verify by re-running        | Trust the signature                 |

This is appropriate because:

1. **Tests are repeatable**: Anyone can run them
2. **Proofs are expensive**: Cryptographic witnesses add complexity
3. **Claims are sufficient**: CI verifies claims; if they hold, that's enough

## Test Reduction

The ledger reduces tests via a simple rule:

**Only run tests that are claimed to pass.**

- `verify` runs tests in outcomes.yaml
- Containers without outcomes.yaml are not verified (nothing claimed)
- Unknown/Pending containers don't block verification

This is NOT about skipping tests for "unchanged" code (unreliable). It's about knowing WHICH tests to run: the claimed ones.

## MECE Coverage

The design ensures every container is in exactly one state:

| State     | Detection                             |
| --------- | ------------------------------------- |
| Unknown   | No tests/ directory or empty          |
| Pending   | Tests exist, not all in outcomes.yaml |
| Stale     | Descendant outcomes_blob mismatch     |
| Passing   | verify succeeds                       |
| Regressed | verify fails                          |

These states are mutually exclusive and collectively exhaustive.

---

# Recursive Decimal BSP Strategy

This approach solves the "Infinite Insertion" problem while leveraging standard filesystem sorting rules (ASCII) to keep parents and children ordered correctly without renaming files.

## 1. The Syntax

```text
{BSP}[@{SubBSP}...]-{slug}.{type}
```

- **{BSP}**: Two digits (10-99)
- **@**: The recursion delimiter (read as "at")
- **-**: The slug separator. **Critical:** Must be a hyphen, not an underscore
- **{slug}**: Human-readable name
- **{type}**: `.capability`, `.feature`, `.story`, or `.adr`

## 2. The ASCII Physics

Filesystems sort characters by ASCII code. To ensure the **Parent** (20) always appears before the **Child** (20@50), we rely on the fact that the **Hyphen (-)** has a lower ASCII value than the **At Symbol (@)**.

| Character      | ASCII | Sort Position |
| -------------- | ----- | ------------- |
| **- (Hyphen)** | 45    | 1 (Parent)    |
| **0-9**        | 48-57 | 2 (Siblings)  |
| **@ (At)**     | 64    | 3 (Children)  |
| _ (Underscore) | 95    | Avoid         |

**Resulting sort order:**

```text
20-auth.capability/      ← Parent (- sorts first)
20@50-recovery.feature/  ← Child (@ sorts after -)
21-billing.capability/   ← Sibling (2 > 0)
```

## 3. The Algorithms

### A. Append (new sibling)

**Goal:** Add item after `20-auth`.

```text
floor((20 + 99) / 2) = 59 → 59-billing.capability/
```

### B. Insert (between siblings)

**Goal:** Insert between `20-auth` and `30-billing`.

```text
floor((20 + 30) / 2) = 25 → 25-subscriptions.capability/
```

### C. Recursive Insert (no integer space)

**Goal:** Insert between `20-auth` and `21-logging` (no room).

**Strategy:** "Zoom in" on 20. Treat it as a new 10-99 space.

```text
20 + @ + floor((10 + 99) / 2) = 20@54 → 20@54-audit.capability/
```

### D. Deep Recursion

**Goal:** Insert between `20@54` and `20@55`.

```text
20@54@50-detailed-trace.story/
```

## 4. File Tree Example

A complex, heavily edited project. Note how the hierarchy remains logical despite insertions.

```text
spx/
├── product.prd.md
│
├── 10-bootstrap.adr.md                # Start here
│
├── 20-auth.capability/                # Parent capability
│   ├── auth.capability.md             # Spec file (slug.type.md)
│   │
│   ├── 10-login.feature/              # Feature 1
│   ├── 20-signup.feature/             # Feature 2
│   │
│   ├── 20@50-recovery.feature/        # ← INSERTED (between 20 & 21)
│   │   ├── recovery.feature.md
│   │   ├── 10-forgot-pass.story/
│   │   └── 20-reset-link.story/
│   │
│   └── 21-logout.feature/             # Feature 3
│
├── 20@50-security.adr.md              # ← ADR inserted after Auth capability
├── 20@90-legacy-migration.adr.md      # ← ADR inserted near end of Auth block
│
├── 21-billing.capability/             # Next major sibling
│   ├── billing.capability.md
│   └── ...
```

## 5. Rules

1. **Hyphens only**: Always `-` between BSP and slug, never `_`
2. **No renaming**: `20-auth` stays `20-auth` forever; use `@` to insert
3. **Rebalance at depth 3**: If you reach `20@50@50@...`, consider rebalancing
4. **Works for files and directories**: `20-auth.capability/` and `20-auth.adr.md` follow the same pattern

This is the most robust, collision-free, and tool-friendly strategy.
