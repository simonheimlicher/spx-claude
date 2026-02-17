# Always Be Converging: The Spec Tree

Over the past decade, the thinking of product people moved from outputs to outcomes, helped by Melissa Perri's *Escaping the Build Trap*, Teresa Torres's *Continuous Discovery Habits*, and Josh Seiden's *Outcomes Over Output*. Thinking in outcomes means explicitly allowing iteration in outputs until a measurable outcome—typically a change in user behavior—was achieved or the hypothesis falsified and the opportunity scrapped.

On the engineering side, [spec-driven development](https://en.wikipedia.org/wiki/Spec-driven_development) saw a renaissance with AI coding tools. Böckeler [taxonomizes the emerging approaches](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) as *spec-first* (the spec generates tasks and is discarded), *spec-anchored* (the spec stays in the repo as living documentation), and *spec-as-source* (the spec is the source of truth that code is derived from). Tools like [Spec Kit](https://github.com/github/spec-kit), [Kiro](https://kiro.dev/), and [OpenSpec](https://openspec.dev/) span this spectrum. Yet even when specs are kept, the progress signal remains task completion, and the binding between spec, tests, and evidence of validation decays with every change.

## The Broken Loop

Across product and engineering, the progress signal collapses to the same thing: the product state becomes the sum of completed tasks. The product is defined by "what we did" rather than "what it is."

Neither side closes the loop. Specs rot — written once, never updated after implementation has started, divorced from reality within days. Changes are instead specified with the minimal effort possible: one-off tickets or tasks in an endless Markdown file that are closed without regard to whether outcomes have been achieved. If product people deem that a hypothesis did not pan out, its spec in the form of closed tickets or checked off tasks, implementation and tests remain in the codebase as dead code nobody dares to remove.

AI agents amplify the problem: they generate code with awareness of some of the constraints that might be encoded as a so-called *constitution* (Spec Kit) or *steering documents* (Kiro) yet they paper over contradictions and fill in the blanks with what they could find using their favorite tools: `Glob`ing and `Grep`ing the codebase. Ask an agent to add a feature and it will happily create a new helper function, not knowing that a tested implementation already exists in a harness two directories over. Maintenance becomes a nightmare as the codebase grows and new implementations of existing functionality, foundational code and test harnesses abound.

Not even 100% test coverage says anything about whether the product is implemented as specified. Tests written by the same agent that wrote the implementation merely verify the agent's own understanding of what it should have built. Git diligently tracks every content change, but it cannot answer whether the tests that pass still validate what the spec currently says.

There has to be a better way.

After enough agent-assisted iterations and switching back-and-forth between Claude, Codex and Gemini, I stopped treating this as a prompting or model problem and started treating it as a systems problem: unless you design for it, drift is the default.

## Context accumulates faster than it converges

Each agent step produces two kinds of output:

- Artifacts I should review diligently: documents, tests, code
- Assumptions I usually miss: inferred defaults, choices between clear yet conflicting guidelines, and "prior art" found in the codebase and used as gospel because it happened to match the keyword it searched for.

While artifacts are visible and I *could* catch drift if I reviewed them, assumptions are difficult to spot. Over time, the assumptions become the real spec — just not one I can read, review, or maintain. This reminds me of organizational culture as the invisbile answer to the question “how are things done around here?”.

If agent work is going to be operable over time, one law must hold. Two corollaries enforce it. Over the last 12 months, I have developed a methodology that would drive human product people and engineers up the walls yet allows agents to thrive.

### Law: ABC — Always Be Converging

> Iterate on a durable artifact in small, reviewable steps such that each step is reversible and repeatable, and each reduces uncertainty about what the system should do.

Whatever the result of the latest well-intended but potentially entirely misguided action of an agent, you can go back to the prior iteration and try again with a different prompt, different context, or a different agent.

**Design implication:** *The system must be Git-native and track validation history as a generated file within the repo like a package lock file, not just code history.*

### Corollary: Determinism

> Never generate what can be derived deterministically.

If you know what you want the agent to base its actions on, provide it as deterministic context and keep it from grepping the codebase. Whenever you observe that it is looking for prior art, ask yourself if you should create or augment a skill so that you control what the agent considers worthy of imitation.

Wherever a deterministic mechanism can replace a probabilistic one, use it: curate context rather than letting agents search the codebase, link traceability explicitly rather than inferring it, let pre-commit hooks catch what agent judgment would miss. Reserve the generative capacity of agents for the parts that require it.

**Design implication:** *Global and local constraints (Product Decision Records and Architecture Decision Records) must be co-located with the specs they apply to.*

### Corollary: Leverage

> Expose the maximum leverage decisions, let the agent handle the rest.

Any operational system must expose the highest leverage decisions to a human — either ex ante or ex post — and use the enviable rigor and relentlessness of agents to update all dependent decisions when you change them.

**Design implication:** *The structure must separate what the product should do (spec) from whether it does it (tests + lock file).*

ABC and its corollaries are design constraints. What if the durable artifact in ABC were the spec itself — not a one-off planning document, but a maintained system of record? Explicitly define enduring outcomes at higher levels, evolving outputs at lower levels — and derive remaining work as failing tests. The structure that follows makes these constraints true by construction.

## The Spec Tree

The **Spec Tree** is a git-native product structure built on these constraints. Every node co-locates a spec, its tests, and a lock file that binds them — tracking not just what changed, but what's been validated and whether that validation is still current. Remaining work is not a list of tasks to complete. It is the set of assertions not yet satisfied, not yet stored in a lock file, or no longer current — progress measured by assertions validated, not tickets closed.

```text
spx/
  billing.prd.md                  # Product outcome hypothesis
  10-tax-compliance.adr.md        # Product constraint as enduring deterministic context
  20-invoicing/                   # Bounded context based on enduring outcome hypothesis
    invoicing.md       # Purpose + assertions
    21-rounding-rules.adr.md      # Local constraint: rounding policy
    spx-lock.yaml                # Lock file binding spec to test results
    tests/
    20-line-items/                # Smaller bounded context
      line-items.md
      spx-lock.yaml
      tests/
    37-usage-breakdown/
  37-cancellation/
    cancellation.md
  54-annual-billing/
```

The tree interleaves decision records (ADRs/PDRs) as siblings of the specs they constrain. A decision is not a separate artifact filed elsewhere; it is a sibling constraint to the code it governs. When an agent works on `20-invoicing/`, it cannot miss `21-rounding-rules.adr.md` sitting right next to the spec.

## Outcome hypotheses shape the tree structure

The Spec Tree captures three layers:

### 1. Outcome Hypotheses indicate the Value of implementing a sub tree

**Outcome hypotheses** from discovery crystallize as the tree's structure — which capabilities exist, how they decompose. The tree evolves slowly and deliberately, constrained by the reality of what the product already is. Adding a capability or deprecating an existing one is a high-cost structural change that forces product people and engineers to reason together about the product surface.

### 2. Output assertions make remaining work identifiable

What the product should be and how it should behave lives inside each node as testable assertions. This is what makes remaining work formally identifiable: every assertion without a passing test is a gap between spec and implementation.

### 3. The lock file closes the loop

**Verifiable implementation** via tests and `spx-lock.yaml` closes the loop. Tests validate assertions; lock files track which validations are current.

## Deterministic Context Injection (DCI)

Standard retrieval (RAG) is probabilistic — it guesses which files might be relevant based on content similarity. The Spec Tree enables *Deterministic Context Injection*: the `spx` CLI walks the tree from product level down to the target node and injects exactly three things:

1. **The goal:** the spec at the target node
2. **The constraints:** global and local decision records (ADRs/PDRs) encountered on the path
3. **The tools:** test harnesses co-located with the node

The agent sees exactly the context it needs. It doesn't search the codebase for "prior art"; the tree provides the authoritative context deterministically.

The lock file also serves as a guardrail: if an agent refactors code but breaks the lock file, it has changed behavior the spec didn't authorize.

## Schema of the Spec Tree

Instead of figuring out what the product is by adding up completed tasks, remaining work is discovered through assertions at every level and tracked as not-yet-passing tests.

- A repo-native **Spec Tree** where each node states what it aims to achieve (the target position)
- **Assertions** that trace specs to tests — reviewable like code (the current position)
- A **lock file** that detects staleness and answers "what's validated?" without running tests (the gap)

The Spec Tree is a git-native product structure (the `spx/` directory) where each node holds a spec and its tests.

For example, `invoicing.md` might contain:

> **Purpose:** We believe showing itemized charges will reduce billing support tickets.
>
> **Assertions:** GIVEN a multi-service invoice WHEN the user opens it THEN each charge shows service name, quantity, and unit price.

The Purpose states why this node exists — an outcome hypothesis awaiting real-world evidence. The Assertions are testable output claims that tests can validate.

---

## Rationale

Discovery and prioritization happen elsewhere — the Spec Tree takes over once a team decides to build something. It lives in the repository as the specification layer — co-location with tests enables atomic commits, content-addressable staleness detection, and unified code review.

The lock file tracks which outputs have been validated and whether those results are still current — something Git's content integrity alone cannot answer. Specs state the target. Tests show what's been met and what remains.

### Non-goals

- **Not a CI replacement** — tests run in CI as normal; the lock file tracks what's been validated, not how to run tests
- **Not outcome measurement** — outcomes require real-world evidence (metrics, experiments); the Spec Tree tracks output claim validation, not outcome validation
- **Not a universal ontology** — capability/feature/story is a default decomposition; teams can use different layers
- **Not a discovery or prioritization tool** — discovery happens elsewhere; the Spec Tree takes over once a team decides to build something
- **Not BDD** — no regex translation layer or step definitions; assertions link directly to standard tests (`pytest`, `jest`)
- **Not a test runner** — your tests run as normal; `spx` provides traceability (which specs are validated?) and staleness detection (is that validation still current?)

### How it works

1. **Write the spec** — Purpose (what we believe this achieves) + Assertions (testable claims as Gherkin scenarios, mappings, or properties)
2. **Write tests** that validate the assertions — standard test frameworks (`pytest`, `jest`), no special tooling
3. **Commit together** — spec, tests, and spx-lock.yaml in a single atomic change — co-located, reviewable, and staleness-tracked

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

The hardest thing in software isn't adding features; it's removing them. Pruning acts as garbage collection: delete the node, the co-located tests vanish, implementation code loses coverage, and CI exposes the dead code. Deleting the spec forces you to clean up the implementation.

Each structural element provides a specific guarantee: `spx-lock.yaml` ensures semantic integrity; interleaved ADRs ensure constraint compliance; co-located tests ensure garbage collection.

### What tests validate

| Spec section              | Locally testable?          |
| ------------------------- | -------------------------- |
| Purpose                   | No — requires real users   |
| Assertions                | Yes — tests validate these |
| Architectural Constraints | Checked through review     |

Assertions are structured output claims — Scenarios, Mappings, Conformance checks, Properties — testable statements about what the software does. Before tests validate them, assertions represent remaining work: places where implementation hasn't yet met spec. After tests validate them, assertions become results tracked in the lock file.

---

## Principles

1. **Structure follows intent, not progress**
   The tree changes when outcome hypotheses change, not when implementation progresses. When the tree changes, tests change. No new work targets a test that no longer exists.

2. **Failing tests surface remaining work**
   Remaining work is programmatically discoverable — each failing test marks where implementation hasn't yet met spec. The lock file tracks the boundary.

3. **Lock files harden through the pipeline**
   During development, `spx test` is informational — regenerate the lock file freely as code changes. At commit, precommit hooks run `spx verify` to confirm that tests stored in the lock file still pass and flag stale or regressed lock files. In CI, the same checks run again. Each stage adds strictness: a test stored in the lock file that now fails blocks the commit at precommit and the merge at CI.

## Schema

1. **The tree is the decomposition**
   Each node describes its own contribution. Parent nodes never reference child breakdowns — the tree structure encodes the decomposition.

2. **Co-location**
   Each node holds its spec, tests, and lock file together. No parallel trees.

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
    spx-lock.yaml
    tests/
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

Each node contains its spec (`{slug}.{type}.md`), lock file (`spx-lock.yaml`), and tests (`tests/`). ADRs are interleaved flat files at any level. Index ordering encodes local dependencies — lower indices are dependencies, same index means parallel work.

For spec formats, naming conventions, and test infrastructure details, see [Outcome Engineering Reference](outcome-engineering-reference.md).

---

## Lock File

Each node MAY have a `spx-lock.yaml` — a lock file tracking which assertions have been validated by tests. The spec states what should be true; the lock file tracks what tests have confirmed.

### Purpose

Git tracks **content integrity** — a Merkle tree of blobs that knows when files change. But it cannot track **semantic integrity**: which outputs have been validated, and are those results still current? A spec can change while its tests still pass against the old behavior. The lock file closes this gap — binding a specific version of a spec to the specific version of the tests that validated it.

### Key Properties

- **Incomplete lock files show remaining work** — A node with 2 of 5 tests passing has work left, not problems
- **Never hand-edited** — Generated by `spx test`, verified by `spx verify`
- **Tree coupling** — A parent lock file is valid if and only if the blob hashes it references match and all child lock file digests match
- **States are derived** — Computed from lock file contents, never manually assigned

### Node States

States reflect validation progress:

| State           | Condition                                                    | Meaning                                      |
| --------------- | ------------------------------------------------------------ | -------------------------------------------- |
| **Unspecified** | No assertions written                                        | Node has a purpose but no testable claims    |
| **Unverified**  | Assertions exist, no tests                                   | Output claims written, tests not yet added   |
| **Pending**     | Tests exist, not all stored in lock file                     | Tests written, not yet passing               |
| **Passing**     | All tests pass, blobs match                                  | Assertions fully validated                   |
| **Stale**       | Spec, test, or child lock file changed since last `spx test` | Lock file no longer reflects current content |
| **Regressed**   | Previously passing test now fails                            | Previously validated assertion now fails     |

Stale and Regressed states block merge. Unspecified, Unverified, and Pending are informational — they indicate work in progress, not problems.

For example, when `invoicing.md` changes, `spx verify` flags the node and every lock file downstream as Stale — no manual triage needed.

### Merge conflicts

The lock file is not CI output or a build artifact. It tracks content hashes only — no timestamps, no volatile data. Same spec + same tests + same code = same lock file. Conflicts arise only when two branches change the same node's tests or spec. Resolution: run `spx test` to regenerate the canonical output.

### Commands

```bash
spx test <node>     # Run tests, write spx-lock.yaml
spx verify <node>   # Verify lock files hold without running tests
spx status <node>   # Show node states without running tests
```

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
