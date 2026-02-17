# From Managing Tasks to Evolving Outcome-Driven Specs

Over the past decade, the thinking of product people moved from outputs to outcomes, helped by Melissa Perri's *Escaping the Build Trap*, Teresa Torres's *Continuous Discovery Habits*, and Josh Seiden's *Outcomes Over Output*. Thinking in outcomes means explicitly allowing iteration in outputs until a measurable outcome—typically a change in user behavior—was achieved or the hypothesis falsified and the opportunity scrapped.

On the engineering side, [spec-driven development](https://en.wikipedia.org/wiki/Spec-driven_development) saw a renaissance with AI coding tools. Böckeler [taxonomizes the emerging approaches](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) as *spec-first* (the spec generates tasks and is discarded), *spec-anchored* (the spec stays in the repo as living documentation), and *spec-as-source* (the spec is the source of truth that code is derived from). Tools like [Spec Kit](https://github.com/github/spec-kit), [Kiro](https://kiro.dev/), and [OpenSpec](https://openspec.dev/) span this spectrum. Yet even when specs are kept, the progress signal remains task completion, and the binding between spec, tests, and evidence of validation decays with every change.

Neither side closes the loop. Specs rot — written once, never updated after implementation has started, divorced from reality within days. Changes are instead specified with the minimal effort possible: one-off tickets or tasks in an endless Markdown file that are closed without regard to whether outcomes have been achieved. If product people deem that a hypothesis did not pan out, its spec in the form of closed tickets or checked off tasks, implementation and tests remain in the codebase — dead code nobody can safely remove.

AI agents amplify the problem: they generate code with awareness of some of the constraints that might be encoded as so-called consitutions or steering documents yet they paper over contradictions and fill in the blanks with what they could find using search. Maintenance becomes a nightmare as the code base grows and new implementations of existing functionality, foundational code and test harnesses abound. Not even 100% test coverage says anything about whether the product is implemented as specified, or tests written by the same agent just verify its understanding of what it should have implemented. While Git diligently tracks every content change, it cannot answer whether the tests that pass still validate what the spec currently says.

There has to be a better way.

What if we relied on the strengths of AI agents to establish a context system that makes them thrive while still being maintained through conversations? Over the last 12 months, I have developed a methodology that would drive human product people and engineers up the walls yet allows agents to thrive. It rests on three principles:

1. **Always be converging.** Iterate on a durable artifact, store all intermediate versions, never risk losing anything. Whatever the result of the latest well-intended but potentially entirely misguided action of an agent, you can go back to the prior iteration and try again with a different prompt, different context, or a different agent.
2. **Never generate what can be derived deterministically.** If you know what you want the agent to base its actions on, provide it as deterministic context and keep it from grepping the codebase. Whenever you observe that it is looking for prior art, ask yourself if you should create or augment a skill so that you control what the agent considers worthy of imitation.
3. **Expose the maximum leverage decisions, let the agent handle the rest.** Any operational system must expose the highest leverage decisions to a human — either ex ante or ex post — and use the enviable rigor and relentlessness of agents to update all dependent decisions when you change them.

The **Spec Tree** is a git-native product structure built on these principles. Every node co-locates a spec, its tests, and a record that binds them — tracking not just what changed, but what's been validated and whether that validation is still current. Remaining work is the set of assertions not yet satisfied, not yet recorded, or no longer current.

```text
spx/
  billing.prd.md                  # Product requirements
  10-tax-compliance.adr.md        # Global constraint: tax rules
  20-invoicing/                   # Broad yet clearly bounded context
    invoicing.md       # Purpose + assertions
    21-rounding-rules.adr.md      # Local constraint: rounding policy
    record.yaml                   # Validation record
    tests/
    20-line-items/                # Smaller bounded context
      line-items.md
      record.yaml
      tests/
```

## Discovery shapes the Spec Tree, delivery iterates inside its nodes

Every node co-locates a spec, its tests, and a record that binds them — tracking not just what changed, but what's been validated and whether that validation is still current. Remaining work is the set of assertions not yet satisfied, not yet recorded, or no longer current.

The Spec Tree captures three layers:

### 1. Outcome Hypotheses indicate the Value of implementing a sub tree

**Outcome hypotheses** from discovery crystallize as the tree's structure — which capabilities exist, how they decompose. The tree evolves slowly and deliberately, constrained by the reality of what the product already is. Adding a capability or deprecating an existing one is a high-cost structural change that forces product people and engineers to reason together about the product surface.

### 2. Output and its assertions via tests specify the current best guess as to how we achidve the Outcome

**Concrete output specifications** — what the product should be and how it should behave — live inside each node as testable assertions. This is what makes remaining work formally identifiable: every assertion without a passing test is a gap between spec and implementation.

### 3. The specification lock file closees the loop

**Verifiable implementation** via tests and `record.yaml` closes the loop. Tests validate assertions; records track which validations are current.

- Outcome hypotheses (WHY)
- Output specification made executable via per-level tests (WHAT)
- Constraints product and architecture decision records.

## Deterministic Context Ingestion (DCI)

As opposed to probabilistic retrieval (RAG), which guesses which spec and code files might be relevant based on their content similarity, the Spec Tree enables *Deterministic Context Ingestion.* More specitifally, the contex is determined by a small command line utitlity (`spx`) that is invoked by the command the user issues to the agent automatically. The `spx` CLI walks the tree from product level down to the target node and loads the PRD, the decision records, and for each level the bounded context comprising a tree of bounded contexts.

The record also serves as a guardrail: if an agent refactors code but breaks the record, it has changed behavior the spec didn't authorize.

## Schema of the Spec Tree

Instead of figuring out what the product is by adding up completed tasks, remaining work is discovered through assertions at every level and tracked as not-yet-passing tests.

- A repo-native **Spec Tree** where each node states what it aims to achieve (the target position)
- **Assertions** that trace specs to tests — reviewable like code (the current position)
- A **record file** that detects staleness and answers "what's validated?" without running tests (the gap)

The Spec Tree is a git-native product structure (the `spx/` directory) where each node holds a spec and its tests. Here's what one looks like:

```text
spx/
  billing.prd.md                  # Product requirements
  10-tax-compliance.adr.md        # Global constraint: tax rules
  20-invoicing/                   # Broad yet clearly bounded context
    invoicing.md       # Purpose + assertions
    21-rounding-rules.adr.md      # Local constraint: rounding policy
    record.yaml                   # Validation record
    tests/
    20-line-items/                # Smaller bounded context
      line-items.md
      record.yaml
      tests/
    37-usage-breakdown/
  37-cancellation/
    cancellation.md
  54-annual-billing/
```

For example, `invoicing.md` might contain:

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

Pruning also acts as garbage collection. Because tests are co-located with specs, deleting a node removes its tests. Implementation code that was only exercised by those tests loses coverage — CI exposes the dead code. Deleting the spec forces you to clean up the implementation.

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
  NN-{slug}/
    {slug}.md
    record.yaml
    tests/
    NN-{slug}.adr.md
    NN-{slug}/
      {slug}.md
      record.yaml
      tests/
      NN-{slug}.adr.md
      NN-{slug}/
        {slug}.md
        record.yaml
        tests/
```

Each node contains its spec (`{slug}.{type}.md`), record (`record.yaml`), and tests (`tests/`). ADRs are interleaved flat files at any level. Index ordering encodes local dependencies — lower indices are dependencies, same index means parallel work.

For spec formats, naming conventions, and test infrastructure details, see [Outcome Engineering Reference](outcome-engineering-reference.md).

---

## Record

Each node MAY have a `record.yaml` — a record of which assertions have been validated by tests. The spec states what should be true; the record tracks what tests have confirmed.

### Purpose

Git tracks **content integrity** — a Merkle tree of blobs that knows when files change. But it cannot track **semantic integrity**: which outputs have been validated, and are those results still current? A spec can change while its tests still pass against the old behavior. The record closes this gap — binding a specific version of a spec to the specific version of the tests that validated it.

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

For example, when `invoicing.md` changes, `spx check` flags the node and every record downstream as Stale — no manual triage needed.

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
