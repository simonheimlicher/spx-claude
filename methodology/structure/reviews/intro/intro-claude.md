# Always Be Converging: A Spec-Driven Operating System for AI Agents

Over the past decade, product thinking shifted from outputs to outcomes, helped by Melissa Perri's *Escaping the Build Trap*, Teresa Torres's *Continuous Discovery Habits*, and Josh Seiden's *Outcomes Over Output*. Thinking in outcomes means allowing iteration on what gets built until a measurable outcome — typically a change in user behavior — is achieved, or the hypothesis is falsified and the opportunity scrapped.

On the engineering side, spec-driven development saw a renaissance with AI coding tools. [Spec Kit](https://github.com/github/spec-kit), [Kiro](https://kiro.dev/), and [OpenSpec](https://openspec.dev/) share a common pattern: a one-off requirements document generates tasks, AI implements them, and the spec is done. Böckeler [calls this "spec-first"](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) — the spec exists to generate code and is discarded afterward. The product state is still the sum of completed tasks.

Neither side closes the loop. Specs rot — written once, never updated after implementation has started, divorced from reality within days. Changes are instead specified with minimal effort: one-off tickets or tasks in an endless Markdown file, closed without regard to whether outcomes have been achieved. If product people deem that a hypothesis did not pan out, its spec — in the form of closed tickets or checked-off tasks — along with its implementation and tests, remains in the codebase as dead code nobody can safely remove.

AI agents amplify the problem. They generate code with awareness of some constraints that might be encoded in steering documents, yet they paper over contradictions and fill in the blanks with whatever they find via search. Maintenance becomes a nightmare as the codebase grows: new implementations of existing functionality, foundational code, and test harnesses abound.

Worse, testing provides false confidence. Not even 100% coverage says anything about whether the product matches the spec. Tests written by the same agent that wrote the implementation merely verify the agent's own understanding of what it should have built. Git diligently tracks every content change, but it cannot answer whether the tests that pass still validate what the spec currently says.

There has to be a better way.

## One law and two corollaries

Anyone who has worked extensively with AI agents knows the pattern: context accumulates, confusion increases, and drift is inevitable. The operating system I have built over the past twelve months rests on one law and two corollaries derived from it.

**Law (ABC — Always Be Converging).** Iterate on a durable artifact in small, reviewable steps such that each step is reversible and repeatable, and each reduces uncertainty about what the system should do.

From this law follow two corollaries:

**Corollary (Determinism).** Never generate what can be derived deterministically. Wherever a deterministic mechanism can replace a probabilistic one — curated context instead of codebase search, linked traceability instead of inferred relationships, pre-commit hooks instead of agent judgment, fixed review topologies instead of self-assessment — use it. Reserve the generative capacity of agents for the parts of the problem that genuinely require it.

**Corollary (Leverage).** Expose the maximum-leverage decisions; let the agent handle the rest. Surface the decisions that shape everything downstream to a human — either before execution or in review — and then let the agent propagate those decisions through all dependent artifacts whenever they change.

## The Spec Tree

What if we applied these principles to build a context system that makes agents thrive — one that stays current not through heroic manual effort but as a byproduct of the development process itself?

The **Spec Tree** is a git-native product structure that puts these principles into practice. Every node co-locates a spec, its tests, and a record that binds them — tracking not just what changed, but what has been validated and whether that validation is still current.

Remaining work is not a list of tasks to complete. It is the set of assertions not yet satisfied, not yet recorded, or no longer current.

```text
spx/
  billing.prd.md                  # Product requirements
  10-tax-compliance.adr.md        # Global constraint: tax rules
  20-invoicing/                   # Capability
    invoicing.md                  # Purpose + assertions
    21-rounding-rules.adr.md      # Local constraint: rounding policy
    record.yaml                   # Validation record
    tests/
    20-line-items/                # Feature
      line-items.md
      record.yaml
      tests/
```

*This is the first in a series of posts. Subsequent posts will cover the maker-checker duality of agent skills, operationalizing Outcome Engineering with the `spx` CLI, and a testing strategy focused on the evidence needed to convince a human that the agent's code does what it claims.*
