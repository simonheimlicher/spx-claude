I have developed a spec-anchored development framework that is part of a more comprehensive methodology named "Outcome Engineering".

As a first step, I want to publish a blog post on my public website, simon.heimlicher.com.

Critically review my draft of the part 1 of a series of posts about the Outcome Engineering Methodology I am developing.

1. What would you change in terms of naming, positioning, wording, examples and the underlying rigorous concepts before making it public?
2. What would be the strongest concerns and counter arguments and how would you adjust the post to avoid them?
3. What are your top 5 suggestions to improve the writing style to sound more like me rather than LLM-generated slop?

---

# The Spec Tree: A Git-Native Structure for Agentic Development

On a human team, "how things are done around here" is culture — the rationale behind decisions, the constraints everyone "just knows," the conventions picked up at the coffee machine. Nobody writes it all down because nobody has to: humans absorb enough through proximity, conversation, and memory to make reasonable decisions most of the time.

Agents have none of that. They don't attend standups, they don't overhear hallway debates, they don't remember what was tried last quarter. Every piece of tacit knowledge that a human teammate would absorb by osmosis must be written down, or the agent will infer its own version — confidently and wrong. Drift is the default.

Over the past decade, the thinking of product people moved from outputs to outcomes, helped by Melissa Perri's *Escaping the Build Trap*, Teresa Torres's *Continuous Discovery Habits*, and Josh Seiden's *Outcomes Over Output*. Thinking in outcomes means explicitly allowing iteration in outputs until a measurable outcome — typically a change in user behavior — was achieved or the hypothesis falsified and the opportunity scrapped.

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

Agents will get things wrong. ABC ensures every mistake is reversible: go back one step and try again with a different prompt, different context, or a different agent.

**Design implication:** *The system must be Git-native and track validation history as a generated file within the repo like a package lock file, not just code history.*

### Guideline: Determinism

> Never generate what can be derived deterministically.

If you know what you want the agent to base its actions on, provide it as deterministic context and keep it from grepping the codebase. Whenever you observe that it is looking for prior art, ask yourself if you should create or augment a skill so that you control what the agent considers worthy of imitation.

Wherever a deterministic mechanism can replace a probabilistic one, use it: curate context rather than letting agents search the codebase, link traceability explicitly rather than inferring it, let pre-commit hooks catch what agent judgment would miss. Reserve the generative capacity of agents for the parts that require it.

**Design implication:** *Global and local constraints (Product Decision Records and Architecture Decision Records) must be co-located with the specs they apply to.*

### Guideline: Leverage

> Expose the maximum leverage decisions, let the agent handle the rest.

Any operational system must expose the highest leverage decisions to a human — either ex ante or ex post — and use the industriousness of agents to update all dependent decisions when you change them.

**Design implication:** *The structure must separate what the product should do (spec) from whether it does it (tests + lock file).*

These guidelines are design constraints. What if the durable artifact in ABC were the spec itself — not a one-off planning document, but a maintained system of record that captures both what a team decided and why? Define enduring outcomes at higher levels, evolving outputs at lower levels, make tacit knowledge explicit as decision records — and derive remaining work as failing tests. This would be unacceptable overhead for humans. But agents are tireless and precise — they can maintain this structure as a side effect of doing the work.

## The Spec Tree

The **Spec Tree** is a git-native product structure built on these constraints. It is both unnecessary and unmaintainable without AI agents — and that is the point. Every node co-locates a spec, its tests, and a lock file that binds them — tracking not just what changed, but what's been validated and whether that validation is still current. Remaining work is not a list of tasks to complete. It is the set of assertions not yet satisfied, not yet stored in a lock file, or no longer current — progress measured by assertions validated, not tickets closed.

Consider this example of a Spec Tree:

```text
spx/
  billing.product.md              # Product outcome hypothesis
  10-auth.enabler/                # Enabler: authentication serves all outcome nodes
    auth.md                       # Spec: starts with ## Enables
    spx-lock.yaml
    tests/
  10-tax-compliance.pdr.md        # Product constraint as enduring deterministic context
  20-invoicing.outcome/           # Outcome node: customer-facing capability
    invoicing.md                  # Spec: starts with ## Outcome
    15-rounding-rules.adr.md      # Node-specific constraint: rounding policy
    spx-lock.yaml                 # Per-node lock file binding spec to test results
    tests/                        # Tests that verify the functional assertions of the spec
    20-line-items.outcome/        # Nested outcome node
      line-items.md
      spx-lock.yaml
      tests/
    20-aggregate-statistics.enabler/  # Enabler: exists to serve sibling outcomes
      aggregate-statistics.md     # Spec: starts with ## Enables
      spx-lock.yaml
      tests/
    37-usage-breakdown.outcome/
  37-cancellation.outcome/
    cancellation.md
  54-annual-billing.outcome/
```

### Outcome nodes and enabler nodes

Directories carry a type suffix: `.outcome` for customer-facing capabilities, `.enabler` for nodes that exist to serve other nodes. An outcome node answers "what should the product do?"; an enabler node answers "what does this subtree need in order to work?" If all its dependent outcomes were removed, the enabler would go with them. `20-aggregate-statistics.enabler/` exists to serve `20-line-items.outcome/` and `37-usage-breakdown.outcome/`; it has no independent reason to exist. The spec file inside reflects this: outcome specs start with `## Outcome`, enabler specs with `## Enables`.

### Co-located decision records for product (PDR) and architecture (ADR) decisions

The tree interleaves decision records (ADRs/PDRs) as siblings of the specs they constrain. A decision is not a separate artifact filed elsewhere; it is a sibling constraint to the code it governs. When an agent works on `20-invoicing.outcome/`, it cannot miss `10-tax-compliance.pdr.md` sitting right next to the spec.

### Fractional indexing

Numeric prefixes use gaps (10, 15, 20, 37, 54) so new nodes can be inserted between existing ones without renumbering — the same [fractional indexing](https://www.figma.com/blog/realtime-editing-of-ordered-sequences/) scheme Figma uses for layer ordering.

Why gaps matter: suppose you start with sequential numbering and later discover that invoicing needs a tax compliance constraint first. With sequential numbers, you renumber everything. With gaps, you insert:

```text
10-tax-compliance.pdr.md        # Inserted before existing nodes
20-invoicing.outcome/           # Unchanged
37-cancellation.outcome/        # Unchanged
```

Nodes with the same prefix are independent — they all depend on what comes before them, but not on each other. `20-line-items.outcome/` and `20-aggregate-statistics.enabler/` can be completed in any order. This makes parallel work visible in the directory listing.

## Outcome hypotheses shape the tree structure

The Spec Tree captures three layers:

### 1. Outcome hypotheses indicate the value of implementing a subtree

**Outcome hypotheses** from discovery crystallize as the tree's structure — which capabilities exist, how they decompose. The tree evolves slowly and deliberately, constrained by the reality of what the product already is. Adding a capability or deprecating an existing one is a high-cost structural change that forces product people and engineers to reason together about the product surface.

### 2. Output assertions create remaining work

What the product should be and how it should behave lives inside each node as testable assertions. Each assertion must make clear how sufficient evidence can be gathered by local tests. Writing a spec creates remaining work: every assertion without a passing test is a gap between what is defined and what is proven, waiting to be closed.

### 3. The lock file closes the loop

Tests close the gap. `spx-lock.yaml` records the closure. Like a package lock file, it is generated (by `spx lock`), deterministic, and committed to the repo.

To make drift mechanically visible, the lock file directly references Git's internal object model: `blob` hashes for file contents and `tree` hashes for directory contents.

### What a node looks like inside

The spec at `20-invoicing.outcome/invoicing.md`:

```markdown
## Outcome

We believe showing itemized charges will reduce billing support tickets.

### Assertions

- GIVEN a multi-service invoice WHEN the user opens it THEN each charge shows service name, quantity, and unit price
- GIVEN a charge with tax WHEN tax compliance is enabled THEN the tax amount and rate appear on a separate line
- GIVEN an empty invoice WHEN the user opens it THEN a message explains that no charges exist for the period
```

The lock file at `20-invoicing.outcome/spx-lock.yaml`:

```yaml
blob: f4a1b2c
tests:
  - path: tests/invoicing.unit.test.ts
    blob: 8c3d2e1
  - path: tests/invoicing.integration.test.ts
    blob: 2a7f9b4
descendants:
  - path: 20-line-items.outcome/
    tree: a3f2b7c
  - path: 20-aggregate-statistics.enabler/
    tree: 9bc4e1d
  - path: 37-usage-breakdown.outcome/
    tree: e7d1f42
```

Every `blob` field is a Git blob hash — the same content-addressable hash Git computes for file contents. Every `tree` field is a Git tree hash — the hash Git computes for directory contents. `spx lock` computes these before commit; when Git stores the files, the hashes must match.

Remember the failure mode from earlier — a spec changes, tests still pass for the old behavior? Here is exactly how the lock file catches it: edit `invoicing.md` and its Git blob hash changes. The `blob` in the lock file no longer matches. The node is stale. The same applies to test files: change a test and its `blob` breaks. For descendants, if any file inside `20-line-items.outcome/` is modified, Git computes a new tree hash for that directory. The parent's `tree` hash no longer matches. Staleness propagates upward through Git's native data model; validation propagates downward through `spx lock`.

Because the lock file contains only hashes — no timestamps — `spx lock` is reentrant: running it twice on the same state produces the same file. Two agents working on the same node with the same state produce identical lock files.

## Deterministic context injection

In most agentic workflows, context is assembled heuristically — search, embeddings, tool-use defaults. The selection is hard to review and unstable as the repo grows. The Spec Tree enables deterministic context injection: the `spx` CLI walks the tree from the product level down to the target node and injects exactly two types of files:

1. **Decision records** (ADRs/PDRs) at higher levels and lower indices — tacit knowledge made explicit.
2. **Specs** (`.outcome`/`.enabler`) at higher levels and lower indices — the context the node exists within.

Test harnesses are purposefully excluded; the test strategy is already defined within the specs. If an agent is assigned to work on `30-discounts.outcome`, deterministic context injection provides the following:

```text
spx/
  05-data-retention.pdr.md               <-- INJECTED (Root-level decision)
  10-tax-compliance.pdr.md               <-- INJECTED (Root-level decision)
  20-invoicing.outcome/
    invoicing.md                         <-- INJECTED (Parent spec)
    15-rounding-rules.adr.md             <-- INJECTED (Parent decision)
    20-line-items.outcome/
      line-items.md                      <-- INJECTED (Lower-index sibling spec)
      tests/                             -- Ignored
    30-discounts.outcome/                [TARGET NODE]
      discounts.md                       <-- INJECTED (Target spec)
      spx-lock.yaml                      -- Ignored
      tests/                             -- Ignored
  50-reporting.outcome/                  -- Ignored (Higher index)
```

The agent sees exactly the context it needs. It doesn't search the codebase for "prior art"; the tree provides the authoritative context deterministically. Because the tree is traversable top-down, an agent can always resume work even if prior changes haven't fully propagated.

The lock file also serves as a guardrail: if an agent refactors code but breaks the lock file, it has changed behavior the spec didn't authorize.

If the deterministic context payload for a single node exceeds 50,000 tokens, it serves as an architectural warning. If a component requires that much context to be understood, an agent will struggle to operate on it reliably regardless of context window size. The structure forces architectural boundaries.

## What's actually new here

ADRs, specs, and tests are not new. Monorepo conventions and co-location are not new. What the Spec Tree adds:

- **Spec assertions as first-class, test-addressable objects.** Each assertion in a spec has a corresponding test. Progress is measured by assertions validated, not tasks completed.
- **A lock file with invalidation semantics.** The lock file binds spec content to test results via Git blob and tree hashes. When either side changes, the hash no longer matches and the binding breaks visibly. This is the mechanism that makes drift detectable rather than silent.
- **Deterministic context assembly from tree structure.** The path from root to node defines exactly what context an agent receives. No search, no heuristics, no embeddings — just the tree.

The assumption I had to discard to arrive at this was that specs are for planning. They are not. The spec is the product's source of truth — what the product should do, expressed as testable assertions. With this inversion, my conversation with agents shifts from directing implementation to maintaining what the product should do. The agent detects what code needs to change by running the tests; how it addresses the gap is driven by skills. If the gap is large, agents make plans, adjust them, and discard them after execution. The plan is disposable because the spec is durable.

## What this does and does not guarantee

The Spec Tree can prove that the implemented system matches the assertions someone chose to write and test. It cannot prove that those assertions are the right ones, or that shipping them changes user behavior. That requires discovery and instrumentation — concerns outside the repo. What the Spec Tree closes is the loop *inside* the repo: the gap between what the spec currently says and what the tests currently validate. For agentic development, closing this loop is the prerequisite for everything else.

## What's next

This post introduced the Spec Tree as a structure — one that can always be reconstructed lossily from the product level down. The next posts in this series will cover the agent skills that make the structure operational — how agents maintain, review, and remediate code within the constraints the tree defines — and how the methodology extends beyond the repo to connect spec-level assertions to product-level outcomes.
