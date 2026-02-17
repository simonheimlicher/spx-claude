# Driving Work through Evolving Outcomes: The Spec Tree

Over the past decade, product management thinking has shifted from outputs to outcomes, influenced by Melissa Perri’s *Escaping the Build Trap*, Teresa Torres’s *Continuous Discovery Habits*, and Josh Seiden’s *Outcomes Over Output*. Thinking in outcomes means explicitly allowing iteration of outputs until a measurable outcome—typically a change in user behavior—is achieved, or the hypothesis is falsified and the opportunity dropped.

On the engineering side, spec-driven development has resurfaced alongside AI coding tools. Tools like [Spec Kit](https://github.com/github/spec-kit) and [Kiro](https://kiro.dev/) guide developers through a structured pipeline that turns requirements into design artifacts and task lists. Birgitta Böckeler describes this as a spectrum: **spec-first** workflows write a spec primarily to generate code, while **spec-anchored** workflows keep the spec around as an artifact that evolves over time. Böckeler calls the former “spec-first.” ([Böckeler on Martin Fowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)) Some tools explicitly aim for the latter, positioning specs as living documentation in the repo (for example, [OpenSpec](https://openspec.dev/)).

But across product and engineering, the progress signal often collapses to the same thing: the product state becomes the sum of completed tasks.

Neither side closes the loop. Specs rot. They are written once, then left untouched after implementation starts, divorced from reality within days. Changes get specified with the minimum effort possible: one-off tickets or tasks in an endless Markdown file, closed without regard to whether outcomes were achieved. If product people decide a hypothesis did not pan out, the spec—now scattered across closed tickets and checked-off tasks—may be “done,” but the code and tests remain in the codebase: dead behavior nobody can justify and nobody can safely remove.

Even 100% test coverage doesn’t solve this. Tests can only prove that behavior is stable *relative to the tests*—not that the product matches the current spec. If the same agent wrote both the code and the tests, the tests often validate the agent’s interpretation rather than the intended requirements. Git diligently tracks every content change, but it cannot answer the question that matters most: do the tests that pass still validate what the spec currently claims *today*?

There has to be a better way. After enough agent-assisted iterations, I stopped treating this as a prompting problem and started treating it as a systems problem: unless you design for it, drift is the default.

## Context accumulates faster than it converges

Each agent step produces two kinds of output:

- **Artifacts** you can diff and review: code, tests, documents.
- **Assumptions** that are easy to miss: inferred defaults, choices between conflicting cues, and “prior art” copied because it looked similar.

Artifacts are visible. Assumptions compound. Over time, the assumptions become the real spec—just not one you can read, review, or maintain.

If agent work is going to be operable over time, one law must hold. Two corollaries enforce it.

**Law (ABC — Always Be Converging):** Iterate on a durable artifact in small, reviewable steps such that each step is reversible, repeatable, and reduces uncertainty about what the system should do.

From this law follow two corollaries:

1. **Corollary (Determinism):** Never generate what can be derived deterministically. If you know what the agent should base its actions on, provide it as deterministic context. Don’t make it search the codebase for “prior art.” When you catch the agent imitating whatever it found, treat that as a signal: you likely need a skill or tool that produces the right reference material on demand, so you control what the agent considers worth imitating.

2. **Corollary (Leverage):** Expose the maximum-leverage decisions; let the agent handle the rest. Surface the decisions that shape everything downstream to a human—either before execution or in review—and then let the agent propagate those decisions through all dependent artifacts whenever they change.

Taken together, ABC and its corollaries are design constraints. The rest of this post describes a methodology—and a repo structure—that makes them true by construction.

What if the durable artifact in ABC were the spec itself: not a one-off planning document, but a maintained system of record—kept current through conversation, and anchored in artifacts that are versioned, reviewable, and testable? Over the last 12 months, I have developed a methodology that would drive human product people and engineers up the walls, yet allows agents to thrive.

The **Spec Tree** is a Git-native product structure that uses the strengths of AI agents to address their shortcomings directly. Every node co-locates a spec, its tests, and a record that binds them—tracking not just what changed, but what’s been validated and whether that validation is still current. Remaining work is the set of assertions not yet satisfied, not yet recorded, or no longer current.

````text
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

      
The **Spec Tree** is a Git-native product structure that uses the strengths of AI agents to address their shortcomings directly. Every node co-locates a spec, its tests, and a record that binds them—tracking not just what changed, but what’s been validated and whether that validation is still current. Remaining work is the set of assertions not yet satisfied, not yet recorded, or no longer current.

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
````
