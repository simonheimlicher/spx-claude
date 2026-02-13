# Outcome Engineering framework

Specs that live outside the repository drift from the code that implements them. There's no programmatic way to check whether a spec has been implemented, is partially done, or is stale.

The Spec Tree is a git-native product structure where each node holds a spec and its tests. Here's what one looks like:

```text
spx/
  billing.prd.md
  20-invoicing.capability/
    invoicing.capability.md
    test-record.yaml
    tests/
    20-line-items.feature/
      line-items.feature.md
      test-record.yaml
      tests/
    37-usage-breakdown.feature/
  37-cancellation.capability/
    cancellation.capability.md
  54-annual-billing.capability/
```

---

## Rationale

At the core of Outcome Engineering is a **Spec Tree** — a durable, version-controlled product structure where every node is an outcome hypothesis with testable assertions.

Discovery and backlog prioritization are out of scope — the Spec Tree only handles what drives engineering work. It lives in the repository as the specification layer — co-location with tests enables atomic commits, content-addressable staleness detection, and unified code review.

The test record tracks which outputs have been validated and whether those results are still current — something Git's content integrity alone cannot answer. Specs state the target. Tests measure progress toward it.

---

## Validation model

### Node architecture: outcome hypothesis with mutable interior

Each node in the Spec Tree represents an **outcome hypothesis** — a belief that building this capability, feature, or story contributes to a desired outcome. The tree structure encodes priority and dependency between these hypotheses.

Inside each node:

| Dimension | Role                                                        | Stability |
| --------- | ----------------------------------------------------------- | --------- |
| **WHY**   | Outcome hypothesis — what we believe this achieves          | Stable    |
| **WHAT**  | Outputs — locally testable prerequisites for the hypothesis | Mutable   |
| **HOW**   | Decisions (ADR/PDR) — constraints on activities             | Mutable   |

**The tree is stable because it encodes outcome hypotheses, not implementation plans.** WHAT and HOW iterate freely inside stable nodes. The tree only changes structurally when outcome hypotheses are added or pruned (hypothesis invalidated).

This separation is the key to durability:

- Changing outputs or decisions = iterating toward the outcome (frequent, normal)
- Restructuring the tree = changing which outcomes matter (rare, deliberate)
- Pruning a node = admitting the outcome hypothesis did not pan out (healthy, explicit)

### What tests validate

| Spec section              | Locally testable?          |
| ------------------------- | -------------------------- |
| Purpose                   | No — requires real users   |
| Assertions                | Yes — tests validate these |
| Architectural Constraints | Checked through review     |

Assertions are structured output specifications — Scenarios, Mappings, Conformance checks, Properties — that constitute testable claims about what the software does. Before tests validate them, assertions represent remaining work: places where implementation hasn't yet met spec. After tests validate them, assertions become results tracked in the test record.

---

## Principles

1. **Durable map of problem discovery**
   The tree evolves with business goals and understanding of user needs, not due to implementation progress. When the tree changes, tests change and any in-progress work that no longer serves a failing test aborts immediately.

2. **Failing tests surface remaining work**
   Remaining work is programmatically discoverable — each failing test marks where implementation hasn't yet met spec. The test record tracks the boundary.

3. **Assertions harden through the pipeline**
   Tests provide tentative assertions during development and immutable evidence after CI.

## Schema

1. **The tree is the decomposition**
   Each node describes its own contribution. Parent nodes never reference child breakdowns — the tree structure encodes the decomposition.

2. **Co-location**
   Each node holds its spec, tests, and test record together. No parallel trees.

3. **The product tests itself**
   The Spec Tree drives the evolution of the implementation; code follows suit. For this to hold, the product must include everything needed to continuously test its own assertions through test harnesses that are part of the product's implementation.

---

## Structure

```text
spx/
  {product-slug}.prd.md
  NN-{slug}.adr.md
  NN-{slug}.capability/
    {slug}.capability.md
    test-record.yaml
    tests/
    NN-{slug}.adr.md
    NN-{slug}.feature/
      {slug}.feature.md
      test-record.yaml
      tests/
      NN-{slug}.adr.md
      NN-{slug}.story/
        {slug}.story.md
        test-record.yaml
        tests/
```

Each node contains its spec (`{slug}.{type}.md`), test record (`test-record.yaml`), and tests (`tests/`). ADRs are interleaved flat files at any level. Index ordering encodes dependencies — lower indices are dependencies, same index means parallel work. See [Fractional Indexing](outcome-engineering-reference.md#fractional-indexing) for insertion algorithms.

For spec formats, naming conventions, and test infrastructure details, see [Outcome Engineering Reference](outcome-engineering-reference.md).

---

## Test Record

Each node MAY have an `test-record.yaml` — the record of which assertions have been validated by tests. The spec states what should be true; the test record tracks what tests have confirmed.

### Purpose

Git tracks content integrity (a Merkle tree of blobs) but cannot answer: which outputs have been validated, and are those results still current? The test record provides this — tracking which tests pass, when they passed, and how nodes relate to each other.

### Key Properties

- **Incomplete records show remaining work** — A node with 2 of 5 tests passing has work left, not problems
- **Never hand-edited** — Generated by `spx record`, checked by `spx check`
- **Tree coupling** — Parent records reference child records, creating a status hierarchy
- **States are derived** — Computed from record contents, never manually assigned

### Node States

States reflect validation progress:

| State         | Condition                      | Meaning                             |
| ------------- | ------------------------------ | ----------------------------------- |
| **Unknown**   | No tests exist                 | Assertions not yet written          |
| **Pending**   | Tests exist, not all recorded  | Assertions written, not yet passing |
| **Passing**   | All tests pass, blobs match    | Assertions fully validated          |
| **Stale**     | Descendant test record changed | Results need re-testing             |
| **Regressed** | Asserted test fails            | Results contradicted                |

### Commands

```bash
spx test <node>     # Run tests, show results
spx record <node>   # Record passing tests in test-record.yaml
spx check <node>    # Check that test records hold
spx status <node>   # Show states without running tests
```

---

## Development Flow

1. **Write spec** — State the outcome hypothesis (Purpose), define assertions as testable output specifications, describe test strategy. Failing tests show where implementation hasn't yet met spec.
2. **Implement** — Write code and tests against the spec. Each passing test closes the gap between spec and reality.
3. **Record** — Run `spx record <node>` to save which tests pass. Tentative during development — assertions harden as they move through the pipeline.
4. **Commit** — Spec, implementation, tests, and test-record.yaml together in a single atomic change. Co-location keeps everything self-contained.
5. **Precommit** — Catches regressions, phantoms, and staleness. The product tests itself before the change leaves the developer's machine.
6. **CI** — Re-runs the same checks as the permanent record. What was tentative during development becomes immutable.

For precommit validation scenarios, see [Outcome Engineering Reference](outcome-engineering-reference.md#precommit-validation).
