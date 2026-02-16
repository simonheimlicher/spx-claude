# From Adding Up Tasks to Inferring Work from Evolving Specs

[Spec-driven development](https://en.wikipedia.org/wiki/Spec-driven_development) saw a renaissance with AI coding tools. [Spec Kit](https://github.com/github/spec-kit), [Kiro](https://kiro.dev/), and [OpenSpec](https://openspec.dev/) share a common pattern: a one-off requirements document generates tasks, AI implements them, and the spec is done. Böckeler [calls this "spec-first"](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) — the spec exists to generate code and is discarded afterward. The product state is still the sum of completed tasks.

Melissa Perri's *Escaping the Build Trap*, Teresa Torres's *Continuous Discovery Habits*, and Josh Seiden's *Outcomes Over Output* moved the thinking of product people from outputs to outcomes, explicitly allowing iteration in outputs until a measurable outcome such as user behavior was achieved or the hypothesis falsified and the opportunity scrapped.

Task lists describe what was done. Specs traced to tests describe what must be true. Remaining work is the set of assertions not yet satisfied, not yet recorded, or no longer current. The **Spec Tree** is the data structure that makes this concrete — a git-native product structure where every node co-locates a spec, its tests, and a record of what's been validated. Outcome Engineering is the broader methodology; this document introduces the Spec Tree, its delivery substrate.

The Spec Tree emerged from years of building and maintaining products where the gap between what product teams intended and what engineers delivered widened with every sprint — not from lack of effort, but from the absence of a shared, testable structure that could evolve with both the business and the code.

## Discovery shapes the Spec Tree, delivery iterates inside its nodes

The Spec Tree captures three layers:

1. **Outcome hypotheses** from discovery crystallize as the tree's structure — which capabilities exist, how they decompose. The DAG evolves slowly and deliberately, constrained by the reality of what the product already is. Adding a capability or deprecating an existing one is a high-cost structural change that forces product people and engineers to reason together about the product surface.

2. **Concrete output specifications** — what the product should be and how it should behave — live inside each node as testable assertions. This is what makes remaining work formally identifiable: every assertion without a passing test is a gap between spec and reality.

3. **Verifiable implementation** via tests and `record.yaml` closes the loop. Tests validate assertions; records track which validations are current.

AI agents thrive in this structure. Instead of ingesting an entire codebase, an agent walks the Spec Tree from product level down to the target node, loading exactly the constraints (architecture decisions) and requirements (specs) at each level — a bounded, hierarchical context with a clear goal (the spec) and a definition of done (all assertions passing).

## Spec Tree

Instead of figuring out what the product is by adding up completed tasks, remaining work is discovered through assertions at every level and tracked as not-yet-passing tests.

- A repo-native **Spec Tree** where each node states what it aims to achieve (the target position)
- **Assertions** that trace specs to tests — reviewable like code (the current position)
- A **record file** that detects staleness and answers "what's validated?" without running tests (the gap)

The Spec Tree is a git-native product structure (the `spx/` directory) where each node holds a spec and its tests. Here's what one looks like:

```text
spx/
  billing.prd.md
  20-invoicing/
    invoicing.capability.md
    record.yaml
    tests/
    20-line-items/
      line-items.feature.md
      record.yaml
      tests/
    37-usage-breakdown/
  37-cancellation/
    cancellation.capability.md
  54-annual-billing/
```

For example, `invoicing.capability.md` might contain:

> **Purpose:** We believe showing itemized charges will reduce billing support tickets.
>
> **Assertions:** GIVEN a multi-service invoice WHEN the user opens it THEN each charge shows service name, quantity, and unit price.

The Purpose states why this node exists — an outcome hypothesis awaiting real-world evidence. The Assertions are testable output claims that tests can validate.

---

## Rationale

Discovery and prioritization happen elsewhere — the Spec Tree takes over once a team decides to build something. It lives in the repository as the specification layer — co-location with tests enables atomic commits, content-addressable staleness detection, and unified code review.

The record tracks which outputs have been validated and whether those results are still current — something Git's content integrity alone cannot answer. Specs state the target. Tests show what's been met and what remains.

### Non-goals

- **Not a CI replacement** — tests run in CI as normal; the record tracks what's been validated, not how to run tests
- **Not outcome measurement** — outcomes require real-world evidence (metrics, experiments); the Spec Tree records output claim validation, not outcome validation
- **Not a universal ontology** — capability/feature/story is a default decomposition; teams can use different layers
- **Not a discovery or prioritization tool** — discovery happens elsewhere; the Spec Tree takes over once a team decides to build something
- **Not BDD** — no regex translation layer or step definitions; assertions link directly to standard tests (`pytest`, `jest`)
- **Not a test runner** — your tests run as normal; `spx` provides traceability (which specs are validated?) and staleness detection (is that validation still current?)

### How it works

1. **Write the spec** — Purpose (what we believe this achieves) + Assertions (testable claims as Gherkin scenarios, mappings, or properties)
2. **Write tests** that validate the assertions — standard test frameworks (`pytest`, `jest`), no special tooling
3. **Commit together** — spec, tests, and record.yaml in a single atomic change — co-located, reviewable, and staleness-tracked

---

## Validation model

### Node architecture: the problem survives all intermediate solutions

Each node carries a **Purpose** (an outcome hypothesis — why this node exists) and **Assertions** (testable output claims — what the software does). The Spec Tree does not prove outcomes; it proves which output claims are currently satisfied and whether that validation is still current.

Inside each node:

| Dimension | Role                                                             | Evolves with                 |
| --------- | ---------------------------------------------------------------- | ---------------------------- |
| **WHY**   | Outcome hypothesis — what we believe this achieves               | Which outcomes matter        |
| **WHAT**  | Output claims — testable assertions about what the software does | Iteration toward the outcome |
| **HOW**   | Decisions (ADR/PDR) — constraints on implementation              | Iteration toward the outcome |

**The tree structure changes when understanding changes** — business understanding at higher levels, implementation understanding at lower levels. Content within nodes iterates freely.

This separation is the key to durability:

- Changing outputs or decisions = iterating toward the outcome (frequent, normal)
- Restructuring the tree = changing which outcomes matter (rare, deliberate)
- Pruning a node = admitting this wasn't worth building (healthy, explicit)

### What tests validate

| Spec section              | Locally testable?          |
| ------------------------- | -------------------------- |
| Purpose                   | No — requires real users   |
| Assertions                | Yes — tests validate these |
| Architectural Constraints | Checked through review     |

Assertions are structured output claims — Scenarios, Mappings, Conformance checks, Properties — that constitute testable statements about what the software does. Before tests validate them, assertions represent remaining work: places where implementation hasn't yet met spec. After tests validate them, assertions become results tracked in the record.

---

## Principles

1. **Structure follows intent, not progress**
   The tree changes when outcome hypotheses change, not when implementation progresses. When the tree changes, tests change. No new work targets a test that no longer exists.

2. **Failing tests surface remaining work**
   Remaining work is programmatically discoverable — each failing test marks where implementation hasn't yet met spec. The record tracks the boundary.

3. **Records harden through the pipeline**
   During development, `spx record` is informational — re-record freely as code changes. At commit, precommit hooks verify that recorded tests still pass and flag stale or regressed records. In CI, the same checks run again. Each stage adds strictness: a recorded test that now fails blocks the commit at precommit and the merge at CI.

## Schema

1. **The tree is the decomposition**
   Each node describes its own contribution. Parent nodes never reference child breakdowns — the tree structure encodes the decomposition.

2. **Co-location**
   Each node holds its spec, tests, and record together. No parallel trees.

3. **Test harnesses are product code**
   The repository includes everything needed to continuously test its own assertions. Test harnesses live with the source, not as external tooling.

---

## Structure

```text
spx/
  {product-slug}.prd.md
  NN-{slug}.adr.md
  NN-{slug}.capability/
    {slug}.capability.md
    record.yaml
    tests/
    NN-{slug}.adr.md
    NN-{slug}.feature/
      {slug}.feature.md
      record.yaml
      tests/
      NN-{slug}.adr.md
      NN-{slug}.story/
        {slug}.story.md
        record.yaml
        tests/
```

Each node contains its spec (`{slug}.{type}.md`), record (`record.yaml`), and tests (`tests/`). ADRs are interleaved flat files at any level. Index ordering encodes local dependencies — lower indices are dependencies, same index means parallel work.

For spec formats, naming conventions, and test infrastructure details, see [Outcome Engineering Reference](outcome-engineering-reference.md).

---

## Record

Each node MAY have a `record.yaml` — a record of which assertions have been validated by tests. The spec states what should be true; the record tracks what tests have confirmed.

### Purpose

Git tracks content integrity (a Merkle tree of blobs) but cannot answer: which outputs have been validated, and are those results still current? The record provides this — tracking which tests pass and how nodes relate to each other.

### Key Properties

- **Incomplete records show remaining work** — A node with 2 of 5 tests passing has work left, not problems
- **Never hand-edited** — Generated by `spx record`, checked by `spx check`
- **Tree coupling** — A parent record is valid if and only if its own recorded blobs match and all referenced child record digests match
- **States are derived** — Computed from record contents, never manually assigned

### Node States

States reflect validation progress:

| State           | Condition                                          | Meaning                                    |
| --------------- | -------------------------------------------------- | ------------------------------------------ |
| **Unspecified** | No assertions written                              | Node has a purpose but no testable claims  |
| **Unverified**  | Assertions exist, no tests                         | Output claims written, tests not yet added |
| **Pending**     | Tests exist, not all recorded                      | Tests written, not yet passing             |
| **Passing**     | All tests pass, blobs match                        | Assertions fully validated                 |
| **Stale**       | Spec, test, or child record changed since recorded | Record no longer reflects current content  |
| **Regressed**   | Recorded test fails                                | Previously validated assertion now fails   |

Stale and Regressed states block merge. Unspecified, Unverified, and Pending are informational — they indicate work in progress, not problems.

For example, when `invoicing.capability.md` changes, `spx check` flags the node and every record downstream as Stale — no manual triage needed.

### Merge conflicts

The record is not CI output or a build artifact. It tracks content hashes only — no timestamps, no volatile data. Same spec + same tests + same code = same record. Conflicts arise only when two branches change the same node's tests or spec. Resolution: run `spx record` to regenerate the canonical output.

### Commands

```bash
spx test <node>     # Run tests, show results
spx record <node>   # Record passing tests in record.yaml
spx check <node>    # Check that records hold
spx status <node>   # Show states without running tests
```

---

## Glossary

| Term          | Definition                                                                                                                      |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Outcome**   | A user or business change that requires real-world evidence (metrics, experiments, research) to validate — not locally testable |
| **Output**    | Software behavior that tests can verify — what the code does                                                                    |
| **Assertion** | A testable output claim, expressed as a Scenario, Mapping, Conformance check, or Property                                       |
| **Spec Tree** | The git-native directory structure (`spx/`) where each node co-locates a spec, its tests, and its record                        |
| **Node**      | A point in the Spec Tree — each states what it aims to achieve (Purpose) and has testable assertions                            |
| **Record**    | `record.yaml` — a generated file tracking which assertions have been validated by tests and whether that validation is current  |
| **SPX**       | Short for specs — the directory name (`spx/`) and CLI tooling name (`spx`)                                                      |
