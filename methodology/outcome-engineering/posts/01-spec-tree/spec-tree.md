# The Spec Tree: A Spec-as-Source Methodology for Human—Agent Collaboration

On a human team, "how things are done around here" is culture — the rationale behind decisions, the constraints everyone "just knows," the conventions picked up at the coffee machine. Nobody writes it all down because nobody has to: humans absorb enough through proximity, conversation, and memory to make reasonable decisions most of the time.

Agents have none of that. They don't attend standups, they don't overhear hallway debates, they don't remember what was tried last quarter. Every piece of tacit knowledge that a human teammate would absorb by osmosis must be written down, or the agent will infer its own version — confidently and wrong. Drift is the default.

These are not new problems — agents just make them acute. Over the past decade, the thinking of product people moved from outputs to outcomes, helped by Melissa Perri's *Escaping the Build Trap*, Teresa Torres's *Continuous Discovery Habits*, and Josh Seiden's *Outcomes Over Output*. Thinking in outcomes means explicitly allowing iteration in outputs until a measurable outcome — typically a change in user behavior — was achieved or the hypothesis falsified and the opportunity scrapped.

On the engineering side, [spec-driven development](https://en.wikipedia.org/wiki/Spec-driven_development) saw a renaissance with AI coding tools. Böckeler [taxonomizes the emerging approaches](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) as *spec-first* (the spec generates tasks and is discarded), *spec-anchored* (the spec stays in the repo as living documentation), and *spec-as-source* (the spec is the source of truth that code is derived from). Tools like [Spec Kit](https://github.com/github/spec-kit), [Kiro](https://kiro.dev/), and [OpenSpec](https://openspec.dev/) span this spectrum. Yet even when specs are kept, the progress signal remains task completion, and the binding between spec, tests, and evidence of validation decays with every change.

## The Broken Loop

Across product and engineering, the progress signal collapses to the same thing: the product state becomes the sum of completed tasks. The product is defined by "what we did" rather than "what it is."

Neither side closes the loop. Specs rot — written once, never updated after implementation has started, divorced from reality within days. Changes are instead specified with the minimal effort possible: one-off tickets or tasks in an endless Markdown file that are closed without regard to whether outcomes have been achieved. If product people deem that a hypothesis did not pan out, its spec in the form of closed tickets or checked off tasks, implementation and tests remain in the codebase as dead code nobody dares to remove.

AI agents amplify the problem: they generate code with awareness of some of the constraints that might be encoded as a so-called *constitution* (Spec Kit) or *steering documents* (Kiro) yet they paper over contradictions and fill in the blanks with whatever they find by searching the codebase. Ask an agent to add a feature and it will happily create a new helper function, not knowing that a tested implementation already exists in a harness two directories over. Maintenance becomes a nightmare as the codebase grows and new implementations of existing functionality, foundational code and test harnesses abound.

Not even 100% test coverage says anything about whether the product is implemented as specified. Tests written by the same agent that wrote the implementation merely verify the agent's own understanding of what it should have built — a deeper problem that requires more than structure to solve. Git diligently tracks every content change, but it cannot answer whether the tests that pass still validate what the spec currently says.

The failure mode is predictable: a spec changes, tests still pass for the old behavior, and the repo no longer contains a reviewable record of what is currently believed to be true.

## Context accumulates faster than it converges

Each agent step produces two kinds of output:

- Artifacts I should review diligently: documents, tests, code
- Assumptions I usually miss: inferred defaults, choices between clear yet conflicting guidelines, and "prior art" found in the codebase and used as gospel because it happened to match the keyword it searched for.

While artifacts are visible and I *could* catch drift if I reviewed them, assumptions are difficult to spot. Over time, the assumptions become the real spec — just not one I can read, review, or maintain.

Three guidelines follow from this. They are not abstractions; they are practical consequences of the fact that agents cannot participate in the conversations that produce tacit knowledge.

### Guideline: ABC — Always Be Converging

> Iterate on a durable artifact in small, reviewable steps such that each step is reversible and repeatable, and each reduces uncertainty about what the system should do.

Agents will get things wrong. ABC ensures every mistake is reversible: go back one step and try again with a different prompt, different context, or a different agent. The system must be Git-native and track validation state as a generated file within the repo — like a package lock file, not just code history.

### Guideline: Determinism

> Never generate what can be derived deterministically.

If you know what you want the agent to base its actions on, provide it as deterministic context and keep it from grepping the codebase. Whenever you observe that it is looking for prior art, ask yourself if you should create or augment a skill so that you control what the agent considers worthy of imitation.

Wherever a deterministic mechanism can replace a probabilistic one, use it: curate context rather than letting agents search the codebase, link traceability explicitly rather than inferring it, let pre-commit hooks catch what agent judgment would miss. Reserve the generative capacity of agents for the parts that require it. Global and local constraints — Product Decision Records (PDRs capture product constraints such as pricing, compliance, and retention that must hold across a subtree) and Architecture Decision Records (ADRs) — must be co-located with the specs they apply to.

### Guideline: Leverage

> Expose the maximum leverage decisions, let the agent handle the rest.

Any operational system must expose the highest leverage decisions to a human — either ex ante or ex post — and use the industriousness of agents to update all dependent decisions when you change them. The structure must separate what the product should do (spec) from whether it does it (tests + lock file).

These guidelines are design constraints, not abstractions. Together they point toward a specific kind of artifact: a maintained system of record that captures both what a team decided and why — where the spec is not a planning document that rots after sprint one, but the product's source of truth.

What would such a system look like? Define enduring outcomes at higher levels, evolving outputs at lower levels. Make tacit knowledge explicit as decision records. Derive remaining work not from a task list, but from assertions not yet validated.

The overhead would be unacceptable for humans. But agents are cheap to rerun and good at mechanical bookkeeping — they can maintain this structure as a side effect of doing the work.

I had to discard a core assumption to arrive at this: specs are not for planning. The spec is the product's source of truth — what the product should do, expressed as testable assertions. With this inversion, my conversation with agents shifts from directing implementation to maintaining what the product should do. The agent detects what code needs to change by running the tests; how it addresses the gap is driven by skills. If the gap is large, agents make plans, adjust them, and discard them after execution. The plan is disposable because the spec is durable.

## The Spec Tree

The **Spec Tree** is a git-native product structure built on these constraints. Every node co-locates a spec, its tests, and a lock file that binds them — tracking not just what changed, but what has been validated and whether that validation is still current. Remaining work is the set of assertions not yet satisfied or no longer current — progress measured by assertions validated, not tickets closed.

### Building the tree

Start with the product outcome hypothesis:

```text
spx/
└── product.product.md
```

`product.product.md` captures why this product should exist and what change in user behavior it aims to achieve.

Add the first customer-facing capability as an **outcome node** — a directory with the `.outcome` suffix:

```text
spx/
├── product.product.md
└── 37-users.outcome/
    └── users.md
```

An outcome node answers "what should the product do?" The spec file inside is `{slug}.md` — no type suffix, no numeric prefix.

As the tree deepens, decisions crystallize. An authentication strategy needs to be settled before implementing login. The decision record lives right next to the code it constrains:

```text
spx/
├── product.product.md
└── 37-users.outcome/
    ├── users.md
    ├── 21-auth-strategy.adr.md
    └── 22-login.outcome/
        ├── login.md
        ├── 21-password-hashing.adr.md
        ├── spx-lock.yaml
        ├── tests/
        ├── 22-hash-password.outcome/
        │   ├── hash-password.md
        │   ├── spx-lock.yaml
        │   └── tests/
        └── 22-verify-password.outcome/
            ├── verify-password.md
            └── tests/
```

Numeric prefixes use gaps (10, 21, 22, 37) so new nodes can be inserted between existing ones without renumbering — the same [fractional indexing](https://www.figma.com/blog/realtime-editing-of-ordered-sequences/) scheme Figma uses for layer ordering. `22-hash-password.outcome/` and `22-verify-password.outcome/` share a prefix because they are independent — they can be completed in any order.

The product grows. A data retention policy applies across the entire product. A test harness serves all outcome nodes. Billing is a parallel capability, independent of users:

```text
spx/
├── product.product.md
├── 10-data-retention.pdr.md
├── 21-test-harness.enabler/
│   ├── test-harness.md
│   ├── spx-lock.yaml
│   └── tests/
├── 37-users.outcome/
│   ├── users.md
│   ├── 21-auth-strategy.adr.md
│   ├── 22-login.outcome/
│   │   ├── login.md
│   │   ├── 21-password-hashing.adr.md
│   │   ├── spx-lock.yaml
│   │   ├── tests/
│   │   ├── 22-hash-password.outcome/
│   │   │   ├── hash-password.md
│   │   │   ├── spx-lock.yaml
│   │   │   └── tests/
│   │   └── 22-verify-password.outcome/
│   │       ├── verify-password.md
│   │       └── tests/
│   └── 37-profile.outcome/
│       └── profile.md
├── 37-billing.outcome/
│   └── billing.md
└── 54-linter.enabler/
    └── linter.md
```

**Enabler nodes** (`.enabler` suffix) exist to serve other nodes. `21-test-harness.enabler/` at the root level serves all outcome nodes in the product; if the product were retired, the test harness would go with it. Position in the tree implies scope: a root-level enabler serves all outcomes; a nested enabler serves only its siblings within the same subtree. Enabler specs start with `## Enables`; outcome specs with `## Outcome`.

When a behavior spans multiple nodes — "cancelling a subscription voids pending invoices" touches both cancellation and billing — the assertion lives in the lowest common ancestor. The ancestor's spec captures cross-cutting behaviors; child nodes handle their local concerns.

## What a node looks like

The spec at `22-login.outcome/login.md`:

```markdown
## Outcome

Users can log in with their email and password.

### Depends on

- ../../21-test-harness.enabler/

### Assertions

- Valid credentials return a session token ([test](tests/login.unit.test.ts))
- Invalid credentials fail with a clear error message ([test](tests/login.unit.test.ts))
- Expired session tokens require re-authentication ([test](tests/login.integration.test.ts))
```

Every assertion links to the test file meant to prove it. The invariant: every assertion must be covered by at least one test file. Agents must not change spec or tests without re-locking; while a lock file is stale, neither the spec being in sync with the tests nor the tests passing should be assumed.

`### Depends on` declares which nodes this spec depends on. Fractional indexing orders nodes for human scanning and decision scoping — decisions at lower indices constrain everything above them — but spec-to-spec dependencies are declared, not inferred from position.

The lock file at `22-login.outcome/spx-lock.yaml`:

```yaml
schema: spx-lock/v1
blob: f4a1b2c
tests:
  - path: tests/login.unit.test.ts
    blob: 8c3d2e1
  - path: tests/login.integration.test.ts
    blob: 2a7f9b4
```

Every `blob` is a Git blob hash — the same content-addressable hash Git computes for file contents. `spx lock` writes the lock file only when all tests pass — the lock is a seal of trust, not a record of results.

Edit `login.md` and its Git blob hash changes. The `blob` in the lock file no longer matches. The node is stale — visibly, before anyone runs a test. Change a test file and its `blob` breaks the same way. `spx verify` compares hashes without running tests; `spx lock` runs tests and regenerates the seal.

Because the lock file contains only hashes — no timestamps — `spx lock` is reentrant: running it twice on the same state produces the same file. Two agents working on the same node with the same state produce identical lock files. After a branch merge, `spx lock` regenerates lock files the same way `npm install` regenerates a package lock.

Each node's lock file tracks only its own spec and tests. Subtree validity is checked by walking the tree: `spx verify --tree` examines each descendant independently. A leaf change does not invalidate the root.

## Deterministic context injection

In most agentic workflows, context is assembled heuristically — search, embeddings, tool-use defaults. The selection is hard to review and unstable as the repo grows. The Spec Tree enables deterministic context injection: the `spx` CLI walks the tree from the product level down to the target node and applies two rules:

1. **Decision records** (ADRs/PDRs) at ancestor levels and lower indices are injected — tacit knowledge made explicit.
2. **Declared dependencies** — specs that the target node references via `### Depends on` — are injected.

Ancestor specs along the path are always included. Test files are excluded; the test strategy is defined within the specs.

If an agent is assigned `22-login.outcome`, deterministic context injection provides:

```text
spx/
  product.product.md                   <-- INJECTED (Root spec)
  10-data-retention.pdr.md             <-- INJECTED (Root decision)
  21-test-harness.enabler/
    test-harness.md                    <-- INJECTED (Declared dependency)
  37-users.outcome/
    users.md                           <-- INJECTED (Ancestor spec)
    21-auth-strategy.adr.md            <-- INJECTED (Ancestor decision)
    22-login.outcome/                  [TARGET NODE]
      login.md                         <-- INJECTED (Target spec)
      spx-lock.yaml                    -- Ignored
      tests/                           -- Ignored
  37-billing.outcome/                  -- Ignored (Not an ancestor)
  54-linter.enabler/                   -- Ignored (Not declared)
```

The agent sees exactly the context it declared or inherited. It doesn't search the codebase for "prior art"; the tree provides the authoritative context deterministically. Because the tree is traversable top-down, an agent can always resume work even if prior changes haven't fully propagated.

A mismatch between the lock file and the current state of spec or tests does not mean behavior changed — it means the evidence is stale. Tests detect behavioral drift; the lock makes it visible when the evidence needs refreshing.

If the deterministic context payload for a single node routinely exceeds an agent's reliable working set, the tree is telling you the component is doing too much. The structure forces architectural boundaries: when a node requires too much context, it is a signal to decompose further.

## The operational loop

```text
$ spx status --tree 37-users.outcome
37-users.outcome/                  needs work (no lock file)
  22-login.outcome/                stale
    login.md changed               (was f4a1b2c, now 91d0a77)
  22-login.outcome/
    22-hash-password.outcome/      valid
    22-verify-password.outcome/    needs work (no lock file)
  37-profile.outcome/              needs work (no lock file)

$ spx lock 22-login.outcome
Running tests...
  tests/login.unit.test.ts         2 passed
  tests/login.integration.test.ts  1 passed
Lock regenerated: 22-login.outcome/spx-lock.yaml

$ spx verify --tree 37-users.outcome
37-users.outcome/                  needs work
  22-login.outcome/                valid
    22-hash-password.outcome/      valid
    22-verify-password.outcome/    needs work
  37-profile.outcome/              needs work
```

`spx verify` is cheap — it compares hashes without running tests. `spx lock` is authoritative — it runs tests and regenerates the seal. `spx verify --lock` does both: verify first, and if stale, lock. This last form is designed for pre-commit hooks; in CI, `spx lock` must produce zero changes.

## What's actually new here

ADRs, specs, and tests are not new. Monorepo conventions and co-location are not new. The Spec Tree combines two independently valuable mechanisms in a single structure:

**Drift detection via lock file.** Git blob hashes bind spec content to test evidence. When either side changes, the hash breaks and the node is visibly stale — before anyone runs a test. This works regardless of how context is assembled.

**Deterministic context from tree structure.** The path from root to node, combined with explicit dependency declarations, defines what context an agent receives. Decisions scope by index; spec dependencies by declaration. This works regardless of whether a lock file exists.

Each stands on its own. Together they close the loop: the lock file shows *whether* the spec is validated; the tree controls *what context* an agent uses to do the validation. Both are maintained by agents as a side effect of doing the work.

The Spec Tree can prove that the implemented system matches the assertions someone chose to write and test. It cannot prove that those assertions are the right ones — that requires discovery and instrumentation outside the repo. What it closes is the loop *inside* the repo: the gap between what the spec currently says and what the tests currently validate. For agentic development, closing this loop is the prerequisite for everything else.

## What's next

This post introduced the Spec Tree as a structure — one that encodes what the product should do (specs with assertions) and what has been validated (lock files with blob hashes). The tree can always be reconstructed from the product level down because the structure is the specification; what is lost in reconstruction is the validation state, which `spx lock` regenerates by running tests.

The next posts in this series will cover the agent skills that make the structure operational — how agents maintain, review, and remediate code within the constraints the tree defines — and how the methodology extends beyond the repo to connect spec-level assertions to product-level outcomes.
