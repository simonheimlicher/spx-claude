# Outcome Engineering: Framework

Technical specification for the Spec Tree and its validation model. For the motivation and principles behind this design, see [The Spec Tree](../posts/01-spec-tree/spec-tree.md). For spec formats, naming conventions, test infrastructure, and fractional indexing details, see [Outcome Engineering Reference](outcome-engineering-reference.md).

---

## Spec Tree

The **Spec Tree** is a git-native product structure — the `spx/` directory — where each node co-locates a spec, its tests, and a lock file that binds them. Remaining work is not a list of tasks to complete. It is the set of assertions not yet satisfied, not yet stored in a lock file, or no longer current — progress measured by assertions validated, not tickets closed.

### Directory Template

```text
spx/
  {product-slug}.prd.md
  NN-{slug}.pdr.md
  NN-{slug}.adr.md
  NN-{slug}/
    {slug}.md
    spx-lock.yaml
    tests/
    NN-{slug}.adr.md
    NN-{slug}/
      {slug}.md
      spx-lock.yaml
      tests/
```

Each node contains its spec (`{slug}.md`), lock file (`spx-lock.yaml`), and tests (`tests/`). Decision records (PDRs and ADRs) are interleaved as sibling flat files at any level — a constraint sitting next to the code it governs.

### Numeric Prefixes

Every directory and decision record carries a numeric prefix (`NN-{slug}`). These prefixes encode **dependency order**: lower indices are dependencies, higher indices may rely on lower ones, same index means independent work. Siblings with the same prefix can be completed — i.e., their test assertions can be met — independently of each other.

When no integer space remains between adjacent siblings, fractional insertion preserves order without renaming: `20.54-audit` sorts between `20-auth` and `21-billing` because dot (ASCII 46) sorts after hyphen (ASCII 45). See [Fractional Indexing](outcome-engineering-reference.md#fractional-indexing) for insertion algorithms and rules.

### Example

```text
spx/
  billing.prd.md
  10-tax-compliance.pdr.md
  20-invoicing/
    invoicing.md
    15-rounding-rules.adr.md
    spx-lock.yaml
    tests/
    20-line-items/
      line-items.md
      spx-lock.yaml
      tests/
    20-aggregate-statistics/
      aggregate-statistics.md
      spx-lock.yaml
      tests/
    37-usage-breakdown/
  37-cancellation/
    cancellation.md
  54-annual-billing/
```

`10-tax-compliance.pdr.md` is a product-level constraint (PDR); `15-rounding-rules.adr.md` is a node-local architecture constraint (ADR). Both `20-line-items` and `20-aggregate-statistics` depend on `15-rounding-rules.adr.md`, but they can be completed independently of each other.

A spec like `invoicing.md` contains:

> **Purpose:** We believe showing itemized charges will reduce billing support tickets.
>
> **Assertions:** GIVEN a multi-service invoice WHEN the user opens it THEN each charge shows service name, quantity, and unit price.

The Purpose states why this node exists — an outcome hypothesis awaiting real-world evidence. The Assertions are testable output claims that tests can validate.

---

## Node Architecture

Each node carries a **Purpose** (an outcome hypothesis — why this node exists) and **Assertions** (testable output claims — what the software does). The Spec Tree does not prove outcomes; it proves which output claims are currently satisfied and whether that validation is still current.

| Dimension | Role                                                             | Evolves with                 |
| --------- | ---------------------------------------------------------------- | ---------------------------- |
| **WHY**   | Outcome hypothesis — what we believe this achieves               | Which outcomes matter        |
| **WHAT**  | Output claims — testable assertions about what the software does | Iteration toward the outcome |
| **HOW**   | Decisions (ADR/PDR) — constraints on implementation              | Iteration toward the outcome |

**The tree structure changes when understanding changes** — business understanding at higher levels, implementation understanding at lower levels. Content within nodes iterates freely.

- Changing outputs or decisions = iterating toward the outcome (frequent, normal)
- Restructuring the tree = changing which outcomes matter (rare, deliberate)
- Pruning a node = admitting this wasn't worth building (healthy, explicit)

The hardest thing in software isn't adding features; it's removing them. Pruning acts as garbage collection: delete the node, the co-located tests vanish, implementation code loses coverage, and CI exposes the dead code. Deleting the spec forces you to clean up the implementation.

Each structural element provides a specific guarantee: `spx-lock.yaml` ensures semantic integrity; interleaved decision records (PDRs/ADRs) ensure constraint compliance; co-located tests ensure garbage collection.

### What Tests Validate

| Spec section              | Locally testable?          |
| ------------------------- | -------------------------- |
| Purpose                   | No — requires real users   |
| Assertions                | Yes — tests validate these |
| Architectural Constraints | Checked through review     |

Assertions are structured output claims — Scenarios, Mappings, Conformance checks, Properties — testable statements about what the software does. Before tests validate them, assertions represent remaining work. After tests validate them, assertions become results tracked in the lock file.

---

## Lock File

Each node MAY have a `spx-lock.yaml` — a generated file tracking which assertions have been validated by tests.

### Purpose

Git tracks **content integrity** — a Merkle tree of blobs that knows when files change. But it cannot track **semantic integrity**: which outputs have been validated, and are those results still current? A spec can change while its tests still pass against the old behavior. The lock file closes this gap — binding a specific version of a spec to the specific version of the tests that validated it.

| Aspect            | Git                             | Lock File                      |
| ----------------- | ------------------------------- | ------------------------------ |
| Question answered | "What is the content?"          | "Did this content pass tests?" |
| Tracks            | Specs, tests, implementation    | Validation state               |
| Merkle structure  | Content blobs → trees → commits | Descendant digests             |
| Basis             | Cryptographic (hash integrity)  | Empirical (run tests to check) |

### Format

```yaml
tests:
  - file: login.unit.test.ts
    passed_at: 2026-01-28T14:15:00Z
  - file: login.integration.test.ts
    passed_at: 2026-01-28T14:15:00Z
descendants:
  - path: 10-parse-credentials/
    digest: a3f2b7c
  - path: 22-validate-token/
    digest: 9bc4e1d
```

| Field                  | Description                               |
| ---------------------- | ----------------------------------------- |
| `tests[].file`         | Test filename relative to node's `tests/` |
| `tests[].passed_at`    | ISO 8601 timestamp when test last passed  |
| `descendants[].path`   | Child node directory name                 |
| `descendants[].digest` | Git blob SHA of child's `spx-lock.yaml`   |

### Key Properties

- **Incomplete lock files show remaining work** — A node with 2 of 5 tests passing has work left, not problems
- **Never hand-edited** — Generated by `spx test`, verified by `spx verify`
- **Tree coupling** — A parent lock file is valid if and only if the blob hashes it references match and all child lock file digests match
- **States are derived** — Computed from lock file contents, never manually assigned

### Node States

States reflect validation progress. They are mutually exclusive and collectively exhaustive.

| State           | Condition                                                    | Meaning                                      |
| --------------- | ------------------------------------------------------------ | -------------------------------------------- |
| **Unspecified** | No assertions written                                        | Node has a purpose but no testable claims    |
| **Unverified**  | Assertions exist, no tests                                   | Output claims written, tests not yet added   |
| **Pending**     | Tests exist, not all stored in lock file                     | Tests written, not yet passing               |
| **Passing**     | All tests pass, blobs match                                  | Assertions fully validated                   |
| **Stale**       | Spec, test, or child lock file changed since last `spx test` | Lock file no longer reflects current content |
| **Regressed**   | Previously passing test now fails                            | Previously validated assertion now fails     |

Stale and Regressed block merge. Unspecified, Unverified, and Pending are informational — they indicate work in progress, not problems.

### State Computation

**Leaf nodes:** State is computed directly from the node's lock file.

**Non-leaf nodes:** Aggregate state = worst of (local state, descendant states).

State ordering (worst to best):

```text
Regressed > Stale > Pending > Unverified > Unspecified > Passing
```

### Tree Coupling

Parent `spx-lock.yaml` stores a `digest` for each child. When a child's lock file changes:

1. Child's Git blob changes
2. Parent's stored `digest` no longer matches
3. Parent becomes **Stale**
4. Parent must re-run `spx test` to update references

This creates a Merkle tree of validation state, separate from Git's content Merkle tree.

### Merge Conflicts

The lock file tracks content hashes and descendant digests. Same spec + same tests + same code = same lock file. Conflicts arise only when two branches change the same node's tests or spec. Resolution: run `spx test` to regenerate.

### Pipeline Hardening

During development, `spx test` is informational — regenerate the lock file freely as code changes. At commit, pre-commit hooks run `spx verify` to confirm that tests stored in the lock file still pass and flag stale or regressed lock files. In CI, the same checks run again. Each stage adds strictness.

| Scenario                                 | Result                     |
| ---------------------------------------- | -------------------------- |
| Test stored in lock file now fails       | Regression — blocks commit |
| Descendant lock file digest mismatch     | Stale — re-test required   |
| Test file deleted but still in lock file | Phantom — blocks commit    |
| New test file not yet in lock file       | In progress — OK           |

### Commands

```bash
spx test [node [--tree] | --all]      # Run tests, write spx-lock.yaml
spx verify [node [--tree] | --all]    # Verify lock files hold without running tests
spx status [node | --all]             # Show node states without running tests
```

Node addressing uses fractional indices as path: `spx test 31/21/45`

**Test reduction:** Only tests stored in the lock file are run during `spx verify`. Nodes without a lock file (Unspecified/Unverified) are not verified — there is nothing stored to verify.

### Workflow

```bash
# 1. Develop: write spec, write tests, iterate
spx test spx/path/to/node/

# 2. Test subtree: run tests bottom-up, write lock files
spx test spx/path/to/node/ --tree

# 3. Commit
git add spx/ && git commit -m "feat: implement X"

# 4. CI: verify lock files hold
spx verify --all
```

---

## Design Rationale

### Why Not Store Spec or Test Blobs

Early designs stored `spec_blob` and `test_blob` in the lock file to detect staleness:

```yaml
# REJECTED DESIGN
spec_blob: abc123
tests:
  - file: test.ts
    blob: def456
```

**Problems:**

1. **Incomplete coverage**: Cannot store implementation blobs without enumerating all source files each test depends on. Staleness detection misses implementation changes.
2. **Duplicates Git's job**: Git already tracks content. Storing blobs partially duplicates Git's Merkle tree, but incompletely.
3. **False confidence**: If spec_blob and test_blob match, we might skip running tests — but implementation could have changed, causing failures.

Partial blob storage provides unreliable staleness detection. Better to not store content blobs at all and rely on running tests.

### Why Store Descendant Digests

While content blobs are not stored, the lock file DOES store a `digest` for each child:

- `digest` captures the child's ENTIRE validation state (its tests + its descendants)
- One digest reference cascades all the way down
- This builds a Merkle tree of TEST RESULTS, not of CONTENT

### Alternative Designs Considered

| Design                  | Approach                               | Why rejected                                                                 |
| ----------------------- | -------------------------------------- | ---------------------------------------------------------------------------- |
| Content blobs           | Store spec_blob + test_blob            | Ignores implementation changes; duplicates Git                               |
| Separate record commits | Lock file updates in dedicated commits | Any code change invalidates all lock files; helps auditing but not reduction |
| No blobs at all         | Only timestamps, no tree coupling      | Loses the Merkle property; parent can be "Passing" while children changed    |
| **Minimal Merkle**      | **Descendant digests only**            | **Tree coupling without content duplication; clear separation from Git**     |

---

## Principles

1. **Structure follows intent, not progress**
   The tree changes when outcome hypotheses change, not when implementation progresses. When the tree changes, tests change. No new work targets a test that no longer exists.

2. **Failing tests surface remaining work**
   Remaining work is programmatically discoverable — each failing test marks where implementation hasn't yet met spec. The lock file tracks the boundary.

3. **The tree is the decomposition**
   Each node describes its own contribution. Parent nodes never reference child breakdowns — the tree structure encodes the decomposition.

4. **Co-location**
   Each node holds its spec, tests, and lock file together. No parallel trees.

5. **Test harnesses are product code**
   The repository includes everything needed to continuously test its own assertions. Test harnesses live with the source, not as external tooling.

---

## Non-goals

- **Not a CI replacement** — tests run in CI as normal; the lock file tracks what's been validated, not how to run tests
- **Not outcome measurement** — outcomes require real-world evidence (metrics, experiments); the Spec Tree tracks output claim validation, not outcome validation
- **Not a universal ontology** — capability/feature/story is a default decomposition; teams can use different layers
- **Not a discovery or prioritization tool** — discovery happens elsewhere; the Spec Tree takes over once a team decides to build something
- **Not BDD** — no regex translation layer or step definitions; assertions link directly to standard tests (`pytest`, `jest`)
- **Not a test runner** — your tests run as normal; `spx` provides traceability (which specs are validated?) and staleness detection (is that validation still current?)

---

## Glossary

| Term          | Definition                                                                                                                        |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **Outcome**   | A user or business change that requires real-world evidence (metrics, experiments, research) to validate — not locally testable   |
| **Output**    | Software behavior that tests can verify — what the code does                                                                      |
| **Assertion** | A testable output claim, expressed as a Scenario, Mapping, Conformance check, or Property                                         |
| **Spec Tree** | The git-native directory structure (`spx/`) where each node co-locates a spec, its tests, and its lock file                       |
| **Node**      | A point in the Spec Tree — each states what it aims to achieve (Purpose) and has testable assertions                              |
| **Lock file** | `spx-lock.yaml` — a generated file binding spec versions to test results, tracking what's been validated and whether it's current |
| **SPX**       | Short for specs — the directory name (`spx/`) and CLI tooling name (`spx`)                                                        |
