I have developed a spec-anchored development framework that is part of a more comprehensive methodology named "Outcome Engineering".

As a first step, I want to publish a blog post on my public website, simon.heimlicher.com.

Critically review my draft of the part 1 of a series of posts about the Outcome Engineering Methodology I am developing.

1. What would you change in terms of naming, positioning, wording, examples and the underlying rigorous concepts before making it public?
2. What would be the strongest concerns and counter arguments and how would you adjust the post to avoid them?
3. What are your top 5 suggestions to improve the writing style to sound more like me rather than LLM-generated slop?

---

````markdown
# The Spec Tree: A Git-Native Structure for AI-Driven Development

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

While artifacts are visible and I *could* catch drift if I reviewed them, assumptions are difficult to spot. Over time, the assumptions become the real spec — just not one I can read, review, or maintain. This reminds me of organizational culture as the invisible answer to the question "how are things done around here?".

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

Consider this example of a Spec Tree:

```text
spx/
  billing.prd.md                  # Product outcome hypothesis
  10-tax-compliance.pdr.md        # Product constraint as enduring deterministic context
  20-invoicing/                   # Node: Bounded context based on enduring outcome hypothesis
    invoicing.md                  # Spec: Purpose + assertions
    15-rounding-rules.adr.md      # Node-specific constraint: rounding policy
    spx-lock.yaml                 # Per-node lock file binding spec to test results
    tests/                        # Tests that verify the functional assertions of the spec
    20-line-items/                # Nested node: Bounded context based on enduring outcome hypothesis
      line-items.md
      spx-lock.yaml
      tests/
    20-aggregate-statistics/      # Nested node: Bounded context based on enduring outcome hypothesis
      aggregate-statistics.md
      spx-lock.yaml
      tests/
    37-usage-breakdown/
  37-cancellation/
    cancellation.md
  54-annual-billing/
```
````

### Collocated decision records for product (PDR) and architecture (ADR) decisions

The tree interleaves decision records (ADRs/PDRs) as siblings of the specs they constrain. A decision is not a separate artifact filed elsewhere; it is a sibling constraint to the code it governs. When an agent works on `20-invoicing/`, it cannot miss `10-tax-compliance.pdr.md` sitting right next to the spec.

### Fractional insertion prefixes indicate dependency between siblings

Both `20-line-items` and `20-aggregate-statistics` depend on `15-rounding-rules.adr.md`, but they can be completed,—i.e., their test assertions can be met—independently of each other.

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

```
```
