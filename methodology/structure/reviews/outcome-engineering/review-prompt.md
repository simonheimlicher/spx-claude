I want to position my Ouctome Engineering framework on Hacker News.

Critically review my draft of the Outcome Engineering Framework

What would you change in terms of naming, positioning, wording and the underlying rigorous concepts before making it public?

What would be the strongest counter arguments and how would you preemptively counter them?

---
# From Adding Up Tasks to Inferring Work from Evolving Specs

[Spec-driven development](https://en.wikipedia.org/wiki/Spec-driven_development) saw a renaissance with AI coding tools. [Spec Kit](https://github.com/github/spec-kit), [Kiro](https://kiro.dev/), and [OpenSpec](https://openspec.dev/) all follow the same pattern: a one-off requirements document generates tasks, AI implements them, and the spec is done. Böckeler [calls this "spec-first"](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) — the spec exists to generate code and is discarded afterward. The product state is still the sum of completed tasks.

Melissa Perri's *Escaping the Build Trap*, Teresa Torres's *Continuous Discovery Habits*, and Josh Seiden's *Outcomes Over Output* moved the thinking of product people from outputs to outcomes, explicitly allowing iteration in outputs until a measurable outcome such as user behavior was achieved or the hypothesis falsified and the opportunity scrapped.

While Outcome Engineering will eventually cover the entire flow of value from business goals, customer understanding to operations and sunsetting, we see its biggest value in bridging the gap between outcome-driven discovery and activity-driven delivery.

## Discovery shapes the Spec Tree, delivery iterates around its nodes

The Spec Tree takes the concrete-enough aspects of discovery and makes them concrete enough for delivery. It evolves with business goals and customer understanding, yet is structured to cover the entire product surface, thus providng engineers to reason with product people.

The **Spec Tree** encodes the current expectation directly — its nodes represent outcome hypotheses at different levels of abstraction, each specifying what the product should be and how it should behave. This allows iterating on output under a given outcome hypothesis.

## Spec Tree

Instead of figuring out what the product is by adding up completed tasks, remaining work is discovered through tests at every level and tracked as not-yet-passing assertions — a task at which AI agents excel.

- A repo-native **Spec Tree** where each node states what it aims to achieve (the target position)
- **Assertions** that trace specs to tests — reviewable like code (the current position)
- A **status file** that detects staleness and answers "what's validated?" without running tests (the gap)

The Spec Tree is a git-native product structure (the `spx/` directory) where each node holds a spec and its tests. Here's what one looks like:

```text
spx/
  billing.prd.md
  20-invoicing.capability/
    invoicing.capability.md
    status.yaml
    tests/
    20-line-items.feature/
      line-items.feature.md
      status.yaml
      tests/
    37-usage-breakdown.feature/
  37-cancellation.capability/
    cancellation.capability.md
  54-annual-billing.capability/
```

For example, `invoicing.capability.md` might contain:

> **Purpose:** Showing itemized charges reduces billing support tickets.
>
> **Assertions:** GIVEN a multi-service invoice WHEN the user opens it THEN each charge shows service name, quantity, and unit price.

The Purpose states why this node exists — what we believe it achieves. The Assertions are testable claims that tests can validate
---

## Rationale

Outcome Engineering is the methodology; the **Spec Tree** is the data structure that covers the delivery — a durable, version-controlled product structure where every node states what it aims to achieve and has testable assertions.

Discovery and prioritization happen elsewhere — the Spec Tree takes over once a team decides to build something. It lives in the repository as the specification layer — co-location with tests enables atomic commits, content-addressable staleness detection, and unified code review.

The status file tracks which outputs have been validated and whether those results are still current — something Git's content integrity alone cannot answer. Specs state the target. Tests show what's been met and what remains.

### Non-goals

- **Not a CI replacement** — tests run in CI as normal; the status file tracks what's been validated, not how to run tests
- **Not analytics or outcome measurement** — outcomes are validated through user evidence (metrics, experiments); the Spec Tree tracks outputs, not outcomes
- **Not a universal ontology** — capability/feature/story is a default decomposition; teams can use different layers
- **Not a discovery or prioritization tool** — discovery happens elsewhere; the Spec Tree takes over once a team decides to build something
- **Not BDD** — no regex translation layer or step definitions; assertions link directly to standard tests (`pytest`, `jest`)
- **Not a test runner** — your tests run as normal; `spx` provides traceability (which specs are validated?) and staleness detection (is that validation still current?)

### How it works

1. **Write the spec** — Purpose (what we believe this achieves) + Assertions (testable claims as Gherkin scenarios, mappings, or properties)
2. **Write tests** that validate the assertions — standard test frameworks (`pytest`, `jest`), no special tooling
3. **Commit together** — spec, tests, and status.yaml in a single atomic change — co-located, reviewable, and staleness-tracked

---

## Validation model

### Node architecture: the problem survives all intermediate solutions

Each node in the Spec Tree represents an **outcome hypothesis** — a belief that building this capability, feature, or story contributes to a desired outcome. The tree structure encodes priority and dependency between these hypotheses.

Inside each node:

| Dimension | Role                                                             | Evolves with                 |
| --------- | ---------------------------------------------------------------- | ---------------------------- |
| **WHY**   | Outcome hypothesis — what we believe this achieves               | Which outcomes matter        |
| **WHAT**  | Functionality — testable assertions about what the software does | Iteration toward the outcome |
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

Assertions are structured output specifications — Scenarios, Mappings, Conformance checks, Properties — that constitute testable claims about what the software does. Before tests validate them, assertions represent remaining work: places where implementation hasn't yet met spec. After tests validate them, assertions become results tracked in the status file.

---

## Principles

1. **Structure follows intent, not progress**
   The tree changes when outcome hypotheses change, not when implementation progresses. When the tree changes, tests change. No new work targets a test that no longer exists; whether in-flight work is cancelled depends on the framework implementation.

2. **Failing tests surface remaining work**
   Remaining work is programmatically discoverable — each failing test marks where implementation hasn't yet met spec. The status file tracks the boundary.

3. **Records harden through the pipeline**
   During development, `spx record` is informational — re-record freely as code changes. After commit, precommit hooks and CI enforce that recorded tests must still pass. A recorded test that now fails blocks the commit.

## Schema

1. **The tree is the decomposition**
   Each node describes its own contribution. Parent nodes never reference child breakdowns — the tree structure encodes the decomposition.

2. **Co-location**
   Each node holds its spec, tests, and status file together. No parallel trees.

3. **Test harnesses are product code**
   The product includes everything needed to continuously test its own assertions. Test harnesses ship with the product, not as external tooling.

---

## Structure

```text
spx/
  {product-slug}.prd.md
  NN-{slug}.adr.md
  NN-{slug}.capability/
    {slug}.capability.md
    status.yaml
    tests/
    NN-{slug}.adr.md
    NN-{slug}.feature/
      {slug}.feature.md
      status.yaml
      tests/
      NN-{slug}.adr.md
      NN-{slug}.story/
        {slug}.story.md
        status.yaml
        tests/
```

Each node contains its spec (`{slug}.{type}.md`), status file (`status.yaml`), and tests (`tests/`). ADRs are interleaved flat files at any level. Index ordering encodes dependencies — lower indices are dependencies, same index means parallel work.

For spec formats, naming conventions, and test infrastructure details, see [Outcome Engineering Reference](outcome-engineering-reference.md).

---

## Status File

Each node MAY have a `status.yaml` — the record of which assertions have been validated by tests. The spec states what should be true; the status file tracks what tests have confirmed.

### Purpose

Git tracks content integrity (a Merkle tree of blobs) but cannot answer: which outputs have been validated, and are those results still current? The status file provides this — tracking which tests pass, when they passed, and how nodes relate to each other.

### Key Properties

- **Incomplete records show remaining work** — A node with 2 of 5 tests passing has work left, not problems
- **Never hand-edited** — Generated by `spx record`, checked by `spx check`
- **Tree coupling** — Parent records reference child records, creating a status hierarchy
- **States are derived** — Computed from record contents, never manually assigned

### Node States

States reflect validation progress:

| State         | Condition                                       | Meaning                             |
| ------------- | ----------------------------------------------- | ----------------------------------- |
| **Unknown**   | No tests exist                                  | Assertions not yet written          |
| **Pending**   | Tests exist, not all recorded                   | Assertions written, not yet passing |
| **Passing**   | All tests pass, blobs match                     | Assertions fully validated          |
| **Stale**     | Child record changed since parent last recorded | Parent must re-record               |
| **Regressed** | Recorded test fails                             | Results contradicted                |

Stale and Regressed states block merge. Unknown and Pending are informational — they indicate work in progress, not problems.

For example, when `invoicing.capability.md` changes, `spx check` flags every status file downstream as Stale — no manual triage needed.

### Merge conflicts

The status file is not CI output or a build artifact. It tracks content hashes only — no timestamps, no volatile data. Same spec + same tests + same code = same record. Conflicts arise only when two branches change the same node's tests or spec. Resolution: run `spx record` to regenerate the canonical output.

### Commands

```bash
spx test <node>     # Run tests, show results
spx record <node>   # Record passing tests in status.yaml
spx check <node>    # Check that status files hold
spx status <node>   # Show states without running tests
```

---

## Development Flow

1. **Write spec** — State what this aims to achieve (Purpose), define assertions as testable output specifications, describe test strategy. Failing tests show where implementation hasn't yet met spec.
2. **Implement** — Write code and tests against the spec. Each passing test closes the gap between spec and reality.
3. **Record** — Run `spx record <node>` to save which tests pass. Informational during development; enforced after commit.
4. **Commit** — Spec, implementation, tests, and status.yaml together in a single atomic change. Co-location keeps everything self-contained.
5. **Precommit** — Catches regressions, phantoms, and staleness before the change leaves the developer's machine.
6. **CI** — Re-runs the same checks. After CI, recorded tests that now fail block the pipeline.

For precommit validation scenarios, see [Outcome Engineering Reference](outcome-engineering-reference.md#precommit-validation).

---

## Glossary

| Term            | Definition                                                                                                                      |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Outcome**     | A user or business change that requires real-world evidence (metrics, experiments, research) to validate — not locally testable |
| **Output**      | Software behavior that tests can verify — what the code does                                                                    |
| **Assertion**   | A testable claim about an output, expressed as a Scenario, Mapping, Conformance check, or Property                              |
| **Spec Tree**   | The git-native directory structure (`spx/`) where each node co-locates a spec, its tests, and its status file                   |
| **Node**        | A capability, feature, or story in the Spec Tree — each states what it aims to achieve (Purpose) and has testable assertions    |
| **Status file** | `status.yaml` — a generated file tracking which assertions have been validated by tests and whether that validation is current  |
