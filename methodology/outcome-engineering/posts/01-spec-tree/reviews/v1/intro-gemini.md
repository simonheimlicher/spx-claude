# Driving Work through Evolving Outcomes: The Spec Tree

Over the past decade, product thinking moved from outputs to outcomes, championed by Melissa Perri (*Escaping the Build Trap*), Teresa Torres (*Continuous Discovery Habits*), and Josh Seiden (*Outcomes Over Output*). Thinking in outcomes means iterating on outputs until a measurable change in user behavior is achieved, or the opportunity is scrapped.

In parallel, spec-driven development saw a renaissance with AI tools like [Spec Kit](https://github.com/github/spec-kit) and [OpenSpec](https://openspec.dev/). However, these tools often treat specs as "one-off" prompts to generate code. Once the code exists, the spec rots.

**The Broken Loop:**
Currently, we explicitly define tasks (tickets) and attempt to infer the product state from the sum of those completed tasks. This is backwards.

- **Manual Spec:** Written once, immediately outdated.
- **Manual Tasks:** Endless lists of tickets.
- **Result:** Dead code, zombie features, and a product defined by "what we did" rather than "what it is."

AI agents amplify this entropy. They generate code based on loose constraints and search results, papering over contradictions. To make agents thrive without losing control, we must invert the model: **Explicitly define the enduring outcome (the Spec) and derive the work required to close the gap.**

## Three Principles for Agent-Driven Development

To achieve this inversion, we need a system grounded in three principles. These design decisions prevent drift and allow us to manage agents effectively.

1. **Always Be Converging (ABC)**
   Iterate on a durable artifact. Unlike chat interfaces where context washes away, we must track the state of the product *and* the validity of that state over time. Whatever the result of an agent's latest action, you must be able to revert to a prior stable state.

- *Design Implication:* The system must be Git-native and track validation history (`record.yaml`), not just code history.

1. **Never generate what can be derived deterministically**
   Do not let agents "grep" the codebase to guess style or constraints. If a constraint exists (e.g., tax logic, coding style), provide it as explicit, deterministic context.

- *Design Implication:* Global and local constraints (`.adr.md`) must be co-located with the requirements they affect.

1. **Expose the maximum leverage decisions; let the agent handle the rest**
   Humans make high-leverage decisions (writing the spec, reviewing the diff). Agents provide the rigor to update all dependent systems.

- *Design Implication:* The structure must separate the *definition* of the feature (`.md`) from the *verification* of the feature (`tests/`).

## The Solution: The Spec Tree

Flowing directly from these principles, we arrive at the **Spec Tree**. It is a directory structure where every node represents a bounded context. It co-locates the requirement (Spec), the constraints (Context), and the proof of reality (Record).

In this model, "tasks" disappear. **The Spec Tree tells us what the product should be and do.** Work is simply the gap between the assertions in the Spec and the reality recorded in the Test Results.

```text
spx/
  billing.prd.md                  # Product requirements (The Outcome)
  10-tax-compliance.adr.md        # Global constraint: tax rules (Deterministic Context)
  20-invoicing/                   # Broad yet clearly bounded context
    invoicing.md       # Purpose + assertions
    21-rounding-rules.adr.md      # Local constraint: rounding policy
    record.yaml                   # Validation record (The Convergence Mechanism)
    tests/
    20-line-items/                # Smaller bounded context
      line-items.md
      record.yaml
      tests/
```
