# Outcome Engineering framework

**Opportunity Tree** — product discovery (Linear, Notion):

```text
Customer Billing Problems
├── "I can't understand my invoice"
│   ├── Simplify line items
│   └── Add usage breakdown
├── "I get charged after canceling"
│   └── Handle cancellation edge cases
└── "I want to pay annually"
    └── Support annual billing cycle
```

**Outcome Tree** — testable specs (git):

```text
spx/
  billing.prd.md
  20-invoicing.capability/
    invoicing.capability.md
    assertions.yaml
    tests/
    20-line-items.feature/
      line-items.feature.md
      assertions.yaml
      tests/
    37-usage-breakdown.feature/
  37-cancellation.capability/
    cancellation.capability.md
  54-annual-billing.capability/
```

---

## Rationale

Outcome Engineering replaces the backlog with an **Outcome Tree** — a durable, version-controlled product structure where every node is an outcome hypothesis. The tree grows coherently: you cannot have a feature without a capability, or a story without a feature. Ideas earn their place through concrete definition — each must decompose into typed assertions (Scenario, Mapping, Conformance, Property) with referenced test files.

Discovery tools (Linear, Notion) capture the opportunities and goals that drive what gets built. The Outcome Tree lives in the repository as the specification layer — co-location with tests and evidence enables atomic commits, content-addressable staleness detection, and unified code review.

Tests validate that outputs are correct; they do not validate that outcomes were achieved — that requires real users (see [Program Logic Chain](outcome-engineering-plc.md) for the formal distinction). The assertion record tracks which outputs have been validated — something Git's content integrity alone cannot answer.

Specs create potential. Tests realize it. The tree doesn't shrink as work completes; it grows as the product grows. (For the full philosophy, see [Potential → Reality](../perspective/product-tree.md).)

---

## Validation model

### Node architecture: outcome hypothesis with mutable interior

Each node in the Outcome Tree represents an **outcome hypothesis** — a belief that building this capability, feature, or story contributes to a desired outcome. The tree structure encodes priority and dependency between these hypotheses.

Inside each node:

| Dimension | Role                                                        | Stability |
| --------- | ----------------------------------------------------------- | --------- |
| **WHY**   | Outcome hypothesis — what we believe this achieves          | Stable    |
| **WHAT**  | Outputs — locally testable prerequisites for the hypothesis | Mutable   |
| **HOW**   | Decisions (ADR/PDR) — constraints on activities             | Mutable   |

**The tree is stable because it encodes outcome hypotheses, not implementation plans.** WHAT and HOW iterate freely inside stable nodes. The tree only changes structurally when outcome hypotheses are added (new potential) or pruned (hypothesis invalidated).

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

Assertions are structured output specifications — Scenarios, Mappings, Conformance checks, Properties — that constitute testable claims about what the software does. Before tests validate them, assertions represent potential: places where implementation can deliver value against spec. After tests validate them, assertions become results tracked in the assertion record.

---

## Principles

1. **Durable map of problem discovery**
   The tree evolves with business goals and understanding of user needs, not due to implementation progress. When the tree changes, tests change and any in-progress work that no longer serves a failing test aborts immediately.

2. **Failing tests reveal potential**
   Failing tests indicate potential for implementation to deliver value against spec. The assertion record tracks where implementation matches spec.

3. **Assertions harden through the pipeline**
   Tests provide tentative assertions during development and immutable evidence after CI.

## Schema

1. **The tree is the decomposition**
   Each node describes its own contribution. Parent nodes never reference child breakdowns — the tree structure encodes the decomposition.

2. **Co-location**
   Each node holds its spec, tests, and assertion record together. No parallel trees.

3. **The product tests itself**
   The Outcome Tree drives the evolution of the implementation; code follows suit. For this to hold, the product must include everything needed to continuously test its own assertions through test harnesses that are part of the product's implementation.

---

## Structure

```text
spx/
  {product-slug}.prd.md
  NN-{slug}.adr.md
  NN-{slug}.capability/
    {slug}.capability.md
    assertions.yaml
    tests/
    NN-{slug}.adr.md
    NN-{slug}.feature/
      {slug}.feature.md
      assertions.yaml
      tests/
      NN-{slug}.adr.md
      NN-{slug}.story/
        {slug}.story.md
        assertions.yaml
        tests/
```

Each node contains its spec (`{slug}.{type}.md`), assertion record (`assertions.yaml`), and tests (`tests/`). ADRs are interleaved flat files at any level. Index ordering encodes dependencies — lower indices are dependencies, same index means parallel work. See [Fractional Indexing](outcome-engineering-reference.md#fractional-indexing) for insertion algorithms.

For spec formats, naming conventions, and test infrastructure details, see [Outcome Engineering Reference](outcome-engineering-reference.md).

---

## Assertion Record

Each node MAY have an `assertions.yaml` — the record of which assertions have been validated by tests. Where the spec defines potential, the assertion record tracks what has been realized.

### Purpose

Git tracks content integrity (a Merkle tree of blobs) but cannot answer: which outputs have been validated, and are those results still current? The assertion record provides this — tracking which tests pass, when they passed, and how nodes relate to each other.

### Key Properties

- **Incomplete records show remaining potential** — A node with 2 of 5 tests passing has potential, not problems
- **Never hand-edited** — Generated by `spx spx assert`, checked by `spx spx verify`
- **Tree coupling** — Parent records reference child records, creating a status hierarchy
- **States are derived** — Computed from record contents, never manually assigned

### Node States

States trace the path from potential to validated:

| State         | Condition                           | Meaning                             |
| ------------- | ----------------------------------- | ----------------------------------- |
| **Unknown**   | No tests exist                      | Potential not yet defined           |
| **Pending**   | Tests exist, not all asserted       | Potential defined, not yet realized |
| **Passing**   | All tests pass, blobs match         | Assertions fully validated          |
| **Stale**     | Descendant assertions_blob mismatch | Results need re-testing             |
| **Regressed** | Asserted test fails                 | Results contradicted                |

### Commands

```bash
spx spx test <node>     # Run tests, show results
spx spx assert <node>   # Record passing tests in assertions.yaml
spx spx verify <node>   # Check that assertions hold
spx spx status <node>   # Show states without running tests
```

---

## Development Flow

1. **Write spec** — State the outcome hypothesis (Purpose), define assertions as testable output specifications, describe test strategy. The spec creates potential: failing tests that reveal where implementation can deliver value.
2. **Implement** — Write code and tests that realize the potential defined by the spec. Each passing test closes the gap between spec and reality.
3. **Assert** — Run `spx spx assert <node>` to record validated assertions. Tentative during development — assertions harden as they move through the pipeline.
4. **Commit** — Spec, implementation, tests, and assertions.yaml together in a single atomic change. Co-location keeps everything self-contained.
5. **Precommit** — Catches regressions, phantoms, and staleness. The product tests itself before the change leaves the developer's machine.
6. **CI** — Re-runs the same checks as the permanent record. What was tentative during development becomes immutable.

For precommit validation scenarios, see [Outcome Engineering Reference](outcome-engineering-reference.md#precommit-validation).
