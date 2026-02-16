Top issues to fix

1. Scope confusion: Outcome Engineering vs Spec Tree

Right now, the doc reads like it is about Outcome Engineering, but ~80% is really about the Spec Tree + status.yaml mechanism.
• Early on you say Outcome Engineering “will eventually cover the entire flow of value…”
• Later, you explicitly say the Spec Tree “tracks outputs, not outcomes” and “not analytics.”

That’s fine, but you should make the boundary explicit once and then stick to it:
• Outcome Engineering = the broader methodology (discovery, prioritization, portfolio decisions, measurement, sunsetting).
• Spec Tree = the repo-native delivery substrate that makes output work inferable and reviewable.

Right now the reader has to infer that separation.

1. Internal contradiction: “nodes represent outcome hypotheses” vs “Spec Tree tracks outputs”

In “Node architecture” you state:

Each node … represents an outcome hypothesis …

but later:

the Spec Tree tracks outputs, not outcomes

Those can both be true, but you need the precise distinction:
• Purpose (WHY) is an outcome hypothesis (not locally testable).
• Assertions (WHAT) are output claims (locally testable).
• status.yaml records which output claims are currently validated.

As written, “node represents an outcome hypothesis” sounds like the node itself can be validated, which you then deny.

1. Over-claiming about other tools

This sentence is brittle:

Spec Kit, Kiro, and OpenSpec all follow the same pattern…

Even if directionally true, it invites “actually, tool X does Y” rebuttals. You can keep the point without the claim:
• “Many spec-first workflows follow a pattern…”
• Then cite Böckeler’s “spec-first” framing.

1. Typos and readability debt

I noticed:
• “beiggest” → “biggest”
• “providng” → “providing”
• a few long sentences with multiple claims (hard to parse, easy to attack)

This matters because your audience will be skeptical.

⸻

Conceptual tightening suggestions

A. Use a crisp “thesis” paragraph early

After the first two paragraphs, add a thesis that anchors everything:
• Task lists describe what we did.
• Specs + tests describe what must be true.
• Therefore remaining work is the set of assertions that are not yet passing (or not yet recorded).

This is the core idea; it currently appears, but only after several paragraphs.

B. Rename a few terms for precision

Not mandatory, but consider:
• “Output generation” → “Implementation and output validation” (if you still use the flow framing elsewhere)
• “Assertions” is good, but define once as “testable output claims” and keep repeating that phrase.
• “Status file” is fine; “record” is good; avoid “validation model” if you also talk about outcomes—call it “output validation model”.

C. Clarify what “infer work” means operationally

You have it implicitly (“failing tests surface remaining work”), but make it explicit:
• Remaining work = {assertions without passing tests} ∪ {tests not recorded} ∪ {records stale} ∪ {records regressed}

This makes “infer work” concrete and defensible.

D. States: “Unknown” should not depend on “no tests exist” alone

Currently:

Unknown: No tests exist → Assertions not yet written

But you can have assertions written with no tests yet. Consider splitting:
• Unspecified: no assertions
• Unverified: assertions exist, tests not present
• Pending: tests present, not fully recorded/passing

If you want to keep it minimal, at least adjust the definitions so they don’t imply a false equivalence.

E. “Tree coupling” and staleness propagation needs one crisp invariant

You say:

Parent records reference child records

Good. Add the invariant:
• Parent status is valid iff its own recorded blobs match and all referenced child status digests match.

That makes “stale” mechanically obvious.

⸻

Suggested edits to key passages

1. Opening (reduce over-claiming, sharpen thesis)

Current: names specific tools + “all follow the same pattern”.

Suggested rewrite:

Many “spec-first” workflows treat the spec as a temporary prompt: write a one-off requirements doc, generate tasks, implement, and move on. The product state remains the sum of completed tasks. Böckeler calls this “spec-first” — the spec exists to generate code and is discarded afterward.

Then immediately add the contrast:

Outcome Engineering flips this: the product state is inferred from evolving specs traced to tests. The remaining work is the set of assertions that are not yet passing or not yet current.

1. “Node architecture” (resolve the outcome/output tension)

Replace:

Each node … represents an outcome hypothesis …

With something like:

Each node carries a Purpose (an outcome hypothesis, validated with real-world evidence) and Assertions (output claims, validated by tests). The Spec Tree does not prove outcomes; it proves which output claims are currently satisfied.

1. Non-goals (make them consistent with the above)

Your non-goals are strong. I would only tweak “not analytics” to align with Purpose:
• “Not outcome measurement — outcomes require real-world evidence; the Spec Tree records output claim validation.”

⸻

Mechanical review of structure

What’s working well
• Clear separation of Purpose vs Assertions.
• Co-location + atomic commits is compelling.
• Status file as “answer what’s validated without running tests” is a strong differentiator.
• The state machine is a good selling point (especially “Stale” and “Regressed”).

What’s still risky / easy to attack
• “Priority and dependency encoded by index ordering” will trigger skepticism.
• Readers will ask: “What about cross-cutting dependencies?” “What about cycles?” “What about parallelism?”
• If you keep it, frame it as default local ordering, not a universal dependency solver.
• “Test harnesses ship with the product” is provocative.
• Clarify: do you mean “the repository includes the harness,” not that end-user deployments include test binaries.

⸻

Minimal list of concrete changes to apply now

1. Add a 2–3 sentence thesis after paragraph 2: state inference from assertions/tests.
2. Replace “nodes represent outcome hypotheses” with Purpose is outcome hypothesis; Assertions are output claims.
3. Reduce brittle claims about other tools; keep the “spec-first” framing.
4. Fix state definitions (Unknown/Pending) so they don’t imply “no tests => no assertions”.
5. Tighten the dependency/ordering claim to avoid sounding like a full dependency model.
6. Fix typos and simplify a few long sentences.

If you want, I can provide a redlined rewrite of the first ~40 lines (up through “Spec Tree”) to lock the framing, since that’s where most readers decide whether to keep reading.

---

# Redlined rewrite (first section through “Spec Tree”)

Below is a suggested rewrite of the opening through the “Spec Tree” section.\
It is not a full-document rewrite; it is a framing + terminology tightening to remove contradictions and reduce brittle claims.

---

## From Adding Up Tasks to Inferring Work from Evolving Specs

[Spec-driven development](https://en.wikipedia.org/wiki/Spec-driven_development) has seen a renaissance with AI coding tools. Many “spec-first” workflows follow a pattern: write a one-off requirements document, generate tasks, let an AI implement them, and treat the spec as finished. Böckeler calls this “spec-first” — the spec exists to generate code and is discarded afterward. The product state is still the sum of completed tasks.

Product thinking moved in the opposite direction. Melissa Perri’s *Escaping the Build Trap*, Teresa Torres’s *Continuous Discovery Habits*, and Josh Seiden’s *Outcomes Over Output* shifted teams from outputs to outcomes: iterate on outputs until a measurable outcome (for example, a user behavior change) is achieved, or falsify the hypothesis and scrap the opportunity.

Outcome Engineering connects these two worlds. Over time, it aims to cover the full flow of value — from business goals and customer understanding through delivery, operations, and sunsetting. Today, its biggest value is bridging the gap between **outcome-driven discovery** and **activity-driven delivery**.

### The key shift: infer work from assertions, not from task lists

In Outcome Engineering, delivery does not “add up tasks” to explain what the product is. Instead, the product state is inferred from **evolving specs traced to tests**:

- Specs state **what should be true**.
- Tests show **what is currently true**.
- Remaining work is the set of **assertions that are not yet satisfied, not yet recorded, or no longer current**.

This is exactly the kind of gap-finding work AI agents excel at.

---

## Discovery shapes the Spec Tree, delivery iterates inside its nodes

Discovery produces hypotheses and priorities. When a team decides to build, the Spec Tree captures the concrete-enough results of discovery in a form delivery can execute and verify.

The Spec Tree evolves with business goals and customer understanding, yet is structured to cover the product surface area end-to-end — giving engineers and product leaders a shared, reviewable artifact.

Each node carries three distinct dimensions:

- **WHY (Purpose):** an *outcome hypothesis* — why this node exists and what we believe it contributes to (validated with real-world evidence, not locally testable).
- **WHAT (Assertions):** *output claims* — testable statements about what the software does (validated by tests).
- **HOW (Decisions):** constraints and architecture decisions (validated via review and conformance checks).

The Spec Tree does **not** prove outcomes. It proves which **output claims** are currently satisfied and whether that validation is still current.

---

## Spec Tree

Instead of figuring out what the product is by adding up completed tasks, remaining work is discovered through assertions at every level and tracked as not-yet-passing (or not-yet-recorded) test results.

The Spec Tree has three core ingredients:

- A repo-native **Spec Tree** where each node states what it aims to achieve (the target position)
- **Assertions** that trace specs to tests — reviewable like code (the current position)
- A **status file** that detects staleness and answers “what’s validated?” without running tests (the gap)

The Spec Tree is a git-native product structure (the `spx/` directory) where each node holds a spec and its tests. Here’s what one looks like:

````text
spx/
  billing.prd.md
  20-invoicing.capability/
    invoicing.capability.md
    status.yaml
    tests/
    20-line-items.feature/
      line-items.feature.md
      status.yaml
      tests/
    37-usage-breakdown.feature/
  37-cancellation.capability/
    cancellation.capability.md
  54-annual-billing.capability/


## Example spec

The spec `invoicing.capability.md` might begin as follows:


```markdown
Purpose: Showing itemized charges reduces billing support tickets.

Assertions: GIVEN a multi-service invoice WHEN the user opens it THEN each charge shows service name, quantity, and unit price.

The Purpose states why this node exists — what we believe it achieves. The Assertions are testable output claims that tests can validate.
```

Notes on what changed:
- Removed brittle “tool X/Y/Z all do the same thing” claim while preserving the “spec-first” critique.
- Added an explicit thesis: **infer product state and remaining work from assertions/tests**, not task lists.
- Fixed the conceptual contradiction by explicitly separating:
  - Purpose = outcome hypothesis (not locally testable)
  - Assertions = output claims (locally testable)
  - Status = records output validation currency
- Tightened language and fixed typos (“biggest”, “providing”).
````
