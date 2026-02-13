# Outcome Engineering framework

**Opportunity Tree** — business understanding (Linear, Jira):

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

**Outcome Tree** — engineering truth (git):

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

The Outcome Tree translates business understanding into engineering truth. Discovery tools (Linear, Jira) host the opportunities and goals that drive tree evolution. The Outcome Tree itself lives in the repository — co-location with tests and evidence enables atomic commits, content-addressable staleness detection, and unified code review.

Tests prove that outputs are correct; they do not prove that outcomes were achieved — that requires real users (see [Program Logic Chain](outcome-engineering-plc.md) for the formal distinction). The assertion ledger records which outputs have been proven — a verification layer that Git's content integrity alone cannot provide.

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

### What tests actually prove

The spec section labeled "Assertions" contains **structured output specifications** — GIVEN/WHEN/THEN scenarios, mappings, conformance checks, and property assertions. These are locally provable prerequisites for the outcome hypothesis stated in Purpose.

| Spec section              | Locally provable?       |
| ------------------------- | ----------------------- |
| Purpose                   | No — requires users     |
| Assertions                | Yes — tests prove these |
| Architectural Constraints | Verified through review |

Each assertion is a testable claim about what the software does. Tests prove assertions. The assertion ledger records which assertions have been proven.

---

## Principles

1. **Durable map of problem discovery**
   The tree evolves with business goals and understanding of user needs, not due to implementation progress. When the tree changes, tests change and any in-progress work that no longer serves a failing test aborts immediately.

2. **Failing tests reveal potential**
   Failing tests indicate potential for implementation to deliver value against spec. The assertion ledger tracks where implementation matches spec.

3. **Assertions harden through the pipeline**
   Tests provide tentative assertions during development and immutable evidence after CI.

## Schema

1. **The tree is the decomposition**
   Each node describes its own contribution. Parent nodes never reference child breakdowns — the tree structure encodes the decomposition.

2. **Co-location**
   Each node holds its spec, tests, and assertion ledger together. No parallel trees.

3. **The product verifies itself**
   The Outcome Tree drives the evolution of the implementation; code follows suit. For this to hold, the product must include everything needed to continuously verify its own assertions through test harnesses that are part of the product's implementation.

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

Each node contains its spec (`{slug}.{type}.md`), assertion ledger (`assertions.yaml`), and tests (`tests/`). ADRs are interleaved flat files at any level. Index ordering encodes dependencies — lower indices are dependencies, same index means parallel work. See [Fractional Indexing](outcome-engineering-reference.md#fractional-indexing) for insertion algorithms.

For spec formats, naming conventions, and test infrastructure details, see [Outcome Engineering Reference](outcome-engineering-reference.md).

---

## Assertion Ledger

Each node MAY have an `assertions.yaml` file listing tests that currently pass. This is the **machine-verifiable proof** that outputs are correct—the record of which output specifications have been proven.

### Purpose

The assertion ledger answers a question Git cannot: "Did this content pass tests, and when?"

Git provides cryptographic integrity of content (a Merkle tree of blobs). The assertion ledger provides verification state—a separate Merkle tree tracking which tests pass and how nodes relate to each other.

### Key Properties

- **Incomplete ledgers are valid** - A node with 2 of 5 tests passing is in progress, not broken
- **Never hand-edited** - Generated by `spx spx assert`, verified by `spx spx verify`
- **Tree coupling** - Parent ledgers reference child ledgers, creating a hierarchy of verification state
- **States are derived** - Unknown, Pending, Stale, Passing, Regressed—computed from ledger contents

### Node States

| State         | Condition                           | Required Action     |
| ------------- | ----------------------------------- | ------------------- |
| **Unknown**   | No tests exist                      | Write tests         |
| **Pending**   | Tests exist, not all asserted       | Fix code or assert  |
| **Stale**     | Descendant assertions_blob mismatch | Re-assert           |
| **Passing**   | All tests pass, blobs match         | None                |
| **Regressed** | Asserted test fails                 | Investigate and fix |

### Commands

```bash
spx spx test <node>     # Run tests, show results
spx spx assert <node>   # Record passing tests in assertions.yaml
spx spx verify <node>   # Check that assertions hold
spx spx status <node>   # Show states without running tests
```

For detailed format specification and design rationale, see [outcome-ledger.md](outcome-ledger.md)

---

## Development Flow

1. **Write spec**: state the outcome hypothesis (Purpose), define assertions (testable output specifications), and describe test strategy
2. **Implement**: write code and tests that prove the outputs
3. **Assert outputs**: run `spx spx assert <node>` to update the assertion ledger
4. **Commit**: spec + implementation + tests + assertions.yaml together
5. **Precommit**: validates no regressions, no phantoms, no staleness
6. **CI**: re-runs validation as immutable evidence

For precommit validation scenarios, see [Outcome Engineering Reference](outcome-engineering-reference.md#precommit-validation).
