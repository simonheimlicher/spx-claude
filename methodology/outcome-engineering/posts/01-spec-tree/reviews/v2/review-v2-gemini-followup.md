Should we rename the keys in the spec-lock.yaml file to match the git distinction between file blob hash and directory tree hash?

```yaml
blob: f4a1b2c
tests:
  - path: tests/invoicing.unit.test.ts
    blob: 8c3d2e1
    passed_at: 2026-02-14T09:32:00Z
  - path: tests/invoicing.integration.test.ts
    blob: 2a7f9b4
    passed_at: 2026-02-14T09:33:12Z
descendants:
  - path: 20-line-items.outcome/
    tree: a3f2b7c
  - path: 20-aggregate-statistics.provider/
    tree: 9bc4e1d
  - path: 37-usage-breakdown.outcome/
    tree: e7d1f42
```

Yes, absolutely. This is a brilliant refinement.

Renaming it to simply `blob` at the root level is the right move. Since the lock file lives inside the node's directory and is fundamentally bound to that node's spec, `spec_blob` is redundant.

More importantly, this perfectly mirrors the output of `git ls-tree`. Anyone building the `spx` CLI can literally parse the output of `git ls-tree HEAD path/to/node` and map the Git object types directly to your YAML keys:

- `100644 blob f4a1b2c... invoicing.md` `blob: f4a1b2c`
- `040000 tree a3f2b7c... 20-line-items.outcome` `tree: a3f2b7c`

Here is the fully assembled, publication-ready draft incorporating this YAML structure, the DCI inclusion rules, the 50k token architectural warning, and the tightened prose from our previous iterations.

---

````markdown
# The Spec Tree: Making Drift Visible in Agentic Development

Over the past decade, product thinking shifted from outputs to outcomes. Frameworks by Melissa Perri, Teresa Torres, and Josh Seiden taught us to iterate on outputs until a measurable change in user behavior is achieved, or the opportunity is scrapped.

On the engineering side, [spec-driven development](https://en.wikipedia.org/wiki/Spec-driven_development) saw a renaissance with AI coding tools. Böckeler [taxonomizes the emerging approaches](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) as *spec-first*, *spec-anchored*, and *spec-as-source*. Tools like Spec Kit, Kiro, and OpenSpec span this spectrum. Yet even when specs are kept as living documentation, the progress signal remains task completion, and the binding between spec, tests, and evidence of validation decays with every change.

## The Broken Loop

Across product and engineering, the progress signal collapses to the same thing: the product state becomes the sum of completed tasks. The product is defined by "what we did" rather than "what it is."

Neither side closes the loop. Specs ossify — written once, never updated after implementation has started, divorced from reality within days. Changes are specified with one-off tickets or tasks in an endless Markdown file that are closed without regard to whether outcomes have been achieved. If a hypothesis fails, its closed tickets, dead code, and obsolete tests remain in the codebase. Nobody dares to remove them.

AI agents amplify the problem. Ask Claude to add a discount flag, and it will confidently write a new `calculate_discount` helper, oblivious to the `PricingService` harness two directories over. Agents paper over contradictions and fill in the blanks with whatever they find by `grep`ing the codebase. Maintenance becomes a nightmare as new implementations of existing functionality and test harnesses abound.

Not even 100% test coverage says anything about whether the product is implemented as specified. Tests written by the same agent that wrote the implementation merely verify the agent's own understanding of what it should have built.

The failure mode is predictable: a spec changes, tests still pass for the old behavior, and the repo no longer contains a reviewable record of what is currently believed to be true.

## Context accumulates faster than it converges

Each agent step produces two kinds of output:

- **Artifacts I should review:** documents, tests, code.
- **Assumptions I usually miss:** inferred defaults, choices between conflicting guidelines, and "prior art" found in the codebase and used as gospel because it matched a keyword search.

While artifacts are visible, assumptions are difficult to spot. Over time, the assumptions become the real spec — just not one I can read, review, or maintain.

This is the tacit knowledge problem. On a human team, half the spec lives in people's heads — the rationale behind decisions, the constraints everyone "just knows," the conventions picked up at the coffee machine. Culture works because humans share context through proximity and memory.

Agents have none of that. Every piece of tacit knowledge that a human teammate would absorb by osmosis must be written down, or the agent will infer its own version — confidently and wrong. Drift is the default unless you design for it.

Three guidelines follow from this reality:

### Guideline: ABC — Always Be Converging

> Iterate on a durable artifact in small, reviewable steps such that each step is reversible and repeatable, and each reduces uncertainty about what the system should do.

Agents will hallucinate. ABC ensures every hallucination is reversible: go back one step and try again with a different prompt, context, or agent.

**Design implication:** *The system must be Git-native and track validation history as a generated file within the repo like a package lock file, not just code history.*

### Guideline: Determinism

> Never generate what can be derived deterministically.

If you know what you want the agent to base its actions on, provide it as deterministic context and keep it from guessing. Wherever a deterministic mechanism can replace a probabilistic one, use it: curate context rather than letting agents search, link traceability explicitly, and let pre-commit hooks catch what agent judgment misses.

**Design implication:** *Global and local constraints (Product and Architecture Decision Records) must be co-located with the specs they apply to.*

### Guideline: Leverage

> Expose the maximum leverage decisions, let the agent handle the rest.

An operational system must expose the highest leverage decisions to a human and use the relentless rigor of agents to update all dependent decisions when you change them.

**Design implication:** *The structure must separate what the product should do (spec) from whether it does it (tests + lock file).*

What if the durable artifact in ABC were the spec itself — a maintained system of record that captures both what a team decided and why? Define enduring outcomes at higher levels, evolving outputs at lower levels, make tacit knowledge explicit as decision records, and derive remaining work as failing tests. This would be unacceptable overhead for humans, but agents can maintain this structure as a side effect of doing the work.

## The Spec Tree

The **Spec Tree** is a Git-native product structure built on these constraints. Every node co-locates a spec, its tests, and a lock file that binds them. Remaining work is not a list of tasks. It is the set of assertions not yet satisfied, not yet stored in a lock file, or no longer current. Progress is measured by assertions validated, not tickets closed.

The Spec Tree in practice:

```text
spx/
  billing.product.md              # Product outcome hypothesis
  10-tax-compliance.pdr.md        # Product constraint as enduring deterministic context
  20-invoicing.outcome/           # Outcome node: customer-facing bounded context
    invoicing.md                  # Spec: starts with ## Outcome
    15-rounding-rules.adr.md      # Node-specific constraint: rounding policy
    spx-lock.yaml                 # Per-node lock file binding spec to test results
    tests/                        # Tests that verify the functional assertions
    20-line-items.outcome/        # Nested outcome node
      line-items.md
      spx-lock.yaml
      tests/
    20-aggregate-stats.provider/  # Enabler: exists to serve sibling outcomes
      aggregate-stats.md          # Spec: starts with ## Provides
      spx-lock.yaml
      tests/
    37-usage-breakdown.outcome/
  37-cancellation.outcome/
    cancellation.md
  54-annual-billing.outcome/
```
````

### Outcome nodes and provider nodes

Borrowing from bounded contexts, nodes are either customer-facing outcomes (`.outcome`) or internal enablers (`.provider`). An outcome node answers "what should the product do?"; a provider node answers "what does this subtree need in order to work?" If `20-line-items` and `37-usage-breakdown` were removed, `20-aggregate-stats` would go with them.

### Co-located decision records

The tree interleaves Product Decision Records (PDRs) and Architecture Decision Records (ADRs) as siblings of the specs they constrain. When an agent works on `20-invoicing.outcome/`, it cannot miss `10-tax-compliance.pdr.md` sitting right next to it.

### Lexical sequencing prefixes

Numeric prefixes use gaps (10, 15, 20, 37, 54) so new nodes can be inserted without renumbering — the same fractional indexing scheme Figma uses for layer ordering. Nodes with the same prefix depend on what comes before them, but not on each other, making parallel work visible.

## The Three Primitives of the Spec Tree

### 1. Outcome hypotheses dictate structure

Outcome hypotheses from discovery crystallize as the tree's structure. The tree evolves deliberately, constrained by what the product already is. Adding or deprecating a capability is a high-cost structural change that forces reasoning about the product surface.

### 2. Output assertions create measurable deficits

What the product should behave like lives inside each node as testable assertions. Writing a spec creates a measurable deficit: a gap between what is defined and what is proven. Every assertion without a passing test is potential waiting to be converted to reality.

### 3. The Git-native lock file closes the loop

Tests convert potential to reality, and `spx-lock.yaml` records the conversion. Like a package lock file, it is generated (by `spx lock`), deterministic, and committed to the repo.

To make drift mechanically visible, the lock file directly references Git's internal object model: `blob` hashes for files, and `tree` hashes for directories.

```yaml
blob: f4a1b2c
tests:
  - path: tests/invoicing.unit.test.ts
    blob: 8c3d2e1
    passed_at: 2026-02-14T09:32:00Z
  - path: tests/invoicing.integration.test.ts
    blob: 2a7f9b4
    passed_at: 2026-02-14T09:33:12Z
descendants:
  - path: 20-line-items.outcome/
    tree: a3f2b7c
  - path: 20-aggregate-statistics.provider/
    tree: 9bc4e1d
```

**How invalidation works:**

- **Specs:** Edit `invoicing.md` and its Git blob hash changes. The root `blob` in the lock file instantly breaks. The node is stale.
- **Tests:** Edit a test file, and its `blob` hash breaks. The lock is stale.
- **Descendants:** If any file inside `20-line-items.outcome/` is modified, Git automatically computes a new `tree` hash for that directory. The `tree` hash in the parent's `descendants` list no longer matches. Staleness propagates upward automatically via Git's native data model.

Editing a spec raises the bar: the product now has a better definition than what exists, and tests that were current become stale until reality catches up.

## Deterministic Context Injection (DCI)

In most agentic workflows, context is assembled heuristically via vector search and embeddings. It guesses. DCI knows.

The `spx` CLI walks the tree from the root down to the target node and injects exactly two types of files:

1. **Decisions (ADR/PDR)** at higher levels and lower indices.
2. **Specs (.outcome/.provider)** at higher levels and lower indices.

Test harnesses are purposefully excluded; the test strategy is already defined within the injected specs. If an agent is assigned to work on the `30-discounts.outcome` node, DCI explicitly highlights and injects the following:

```text
spx/
  05-global-auth.pdr.md                  <-- INJECTED (Higher level decision)
  10-tax-compliance.pdr.md               <-- INJECTED (Higher level decision)
  20-invoicing.outcome/
    invoicing.md                         <-- INJECTED (Higher level parent spec)
    15-rounding-rules.adr.md             <-- INJECTED (Higher level parent decision)
    20-line-items.outcome/
      line-items.md                      <-- INJECTED (Lower index sibling spec)
      tests/                             -- Ignored
    30-discounts.outcome/                [TARGET NODE]
      discounts.md                       <-- INJECTED (Target spec)
      spx-lock.yaml                      -- Ignored
      tests/                             -- Ignored
  50-reporting.outcome/                  -- Ignored (Higher index)
```

The agent sees exactly what it needs. Because the tree is traversable top-down, an agent with DCI can always resume work even if prior changes haven't fully propagated.

**The Context Window Reality Check:**
If the resulting DCI payload for a single node exceeds 50,000 tokens, it serves as an architectural warning. The product surface has likely become a monolith. If a component requires more than 50k tokens of localized constraints and sibling specs to be understood, an agent with a 200k+ context window will struggle to operate on it reliably anyway. The structure forces architectural boundaries.

## What's next

This post introduced the Spec Tree as a structure. The next posts in this series will cover the agent skills that make the structure operational — how agents produce, review, and remediate code within the constraints the tree defines — and how the methodology extends beyond the repo to connect spec-level assertions to product-level outcomes.
