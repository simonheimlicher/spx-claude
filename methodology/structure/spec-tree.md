This is a crucial correction. Interleaving the decision records (ADRs/PDRs) as siblings is much more powerful than isolating them in a `decisions/` folder because it enforces **temporal and spatial locality**.

When an agent (or a human) looks at `25-simulation.capability/`, they literally cannot miss `21-process-sequence-boundary.adr.md` sitting right next to it. It forces the constraint into the context window.

Here is the **Final Master Version (V7)**. It unifies the philosophy of Version 4, the clean "Lockfile" mechanics of Version 5, and the "AI Retrieval/Garbage Collection" arguments of Version 6, all wrapped around your specific interleaved structure.

---

# Spec Tree: A Git-Native Structure for AI-Driven Development

[Spec-driven development]() is seeing a renaissance with AI coding tools. But we have a problem: **AI agents suck at finding context.**

Ask an agent to "add a feature to the CLI," and it will hallucinate a new `create_random_directory()` function because it didn't know a robust `TreeFactory` already existed in `tests/harness`. It acts like a junior engineer who never reads the docs, creating bloated, untestable code.

Standard RAG (Retrieval-Augmented Generation) is probabilistic—it *guesses* what code is relevant.

The **Spec Tree** is a git-native product structure that provides **Deterministic Context Injection**. It groups Specs, Tests, and Decisions into Bounded Contexts that agents (and humans) can reason about in isolation.

## 1. The "Task vs. Context" Problem

We used to manage software by adding up completed tasks. But "Tasks Completed" doesn't mean "Outcome Achieved."

- **The Old Problem:** Specs live in PDFs or Confluence. They rot the moment code is written. We measure progress by output, not outcomes.
- **The New Problem:** AI agents amplify this. They generate code without knowing the existing architectural constraints, leading to "context blindness."
- **The Solution:** A git-native data structure—the **Spec Tree**—that bridges Discovery (Product) and Delivery (Engineering/AI).

## 2. The Spec Tree Architecture

The Spec Tree (`spx/`) replaces the flat list of files with a tree of **Outcome Hypotheses**.

Crucially, it interleaves **Decisions** (ADRs/PDRs) with **Capabilities** and **Tests**. A decision is not a separate artifact; it is a sibling constraint to the code it governs.

```text
spx/
├── 10-simulation-lifecycle.pdr.md          # Constraint: How simulation MUST work
├── 15-infrastructure/                      # CAPABILITY: The machinery
│   ├── 15-precommit-harness/               # FEATURE: Self-testing
│   ├── tests/                              # PROOF: The harness itself
│   └── spec.lock                           # INTEGRITY: The lockfile
├── 20-python-tooling.adr.md                # Constraint: "Use uv, not poetry"
├── 21-type-system-unification.adr.md       # Constraint: "Single source of truth"
├── 25-simulation/                          # CAPABILITY: The core domain
│   ├── 21-process-sequence-boundary.adr.md # LOCAL Constraint: "No re-entrant processes"
│   ├── 21-process-execution/               # FEATURE: The implementation
│   │   ├── spec.md
│   │   ├── spec.lock
│   │   └── tests/
│   └── simulation.md                       # Purpose: "Accurate clock cycles"
```

### Why this structure matters for AI

When an agent works on `25-simulation`, we don't ask it to "search the repo." We feed it the **Bounded Context**:

1. **The Goal:** `simulation.md`
2. **The Constraints:** `10-simulation-lifecycle.pdr.md` (Global) + `21-process-sequence-boundary.adr.md` (Local)
3. **The Tools:** `tests/harness` (Local utilities)

The agent sees exactly the constraints and tools it needs. It doesn't hallucinate a new process scheduler; it uses the one defined in the sibling ADR.

## 3. The Lockfile (`spec.lock`)

Git tracks source code changes, but it doesn't track **Semantic Integrity**. If a PM changes the definitions in `simulation.md`, your tests might still pass technically (the code runs), but they are now validating the wrong behavior.

We solve this with a **Lockfile**.

The `spec.lock` binds a specific version of a Spec to the specific version of the Tests that validated it.

- **Staleness Detection:** If you edit the Markdown spec, the hash changes. The lockfile becomes "Stale." CI blocks the merge until the developer runs the tests and updates the lock (`spx record`).
- **Agent Guardrails:** If an agent refactors the code but breaks the lock, you know it hallucinated a requirement change.

## 4. Chain of Custody & Garbage Collection

The hardest thing in software isn't adding features; it's removing them. When you delete a feature from Jira, the code stays. Over time, your repo becomes a graveyard of dead logic.

The Spec Tree solves this through **Collocation + Coverage**:

1. **Delete the Node:** You delete `spx/25-simulation/`.
2. **Tests Vanish:** Because the tests were inside the node, they are gone.
3. **Dead Code Exposed:** The implementation code (which lives in `src/`) is no longer being called.
4. **CI Block:** Your coverage report immediately lights up. "Dangerous Code Detected: 15 files in `src/simulation` have 0% coverage."

You don't need to manually track which source files belong to which spec. The tests maintain that link. **Deleting the Spec forces you to garbage collect the Implementation.**

## 5. Methodology vs. Tooling

**Outcome Engineering** is the methodology:

- We iterate on **Outcome Hypotheses** (Nodes), not just tasks.
- We measure progress by **Assertions Validated** (Tests), not tickets closed.

**Spec Tree** is the infrastructure:

- **Discovery** shapes the Tree (Product/Design).
- **Delivery** iterates inside the Nodes (Engineering/AI).

### Non-goals

- **Not a CI replacement:** Tests run in CI as normal. `spx` just checks the lockfile.
- **Not BDD:** No regex, no Gherkin glue code. Just Markdown and standard unit tests (`pytest`, `jest`).
- **Not a Project Management tool:** Discovery and prioritization happen elsewhere; the Spec Tree takes over once a team decides to build something.

## Summary

- **Problem:** Task lists track activity, not value. AI agents hallucinate because they lack context.
- **Solution:** A git-native Spec Tree where structure = value decomposition.
- **Mechanism:** `spec.lock` ensures integrity; Interleaved ADRs ensure compliance; Colocated tests ensure garbage collection.

This is how we move from "writing code" to "managing autonomous agents who write code."
