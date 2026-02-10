# HN Positioning Review: CODE Framework & Product Tree

## 1. Naming Problems

### "CODE" (Customer Outcome Driven Engineering)

This name has three compounding issues:

**Unsearchable.** "CODE framework" returns codeframework.io — a .NET business application framework from CODE Magazine that has existed for years. Every Google query, every HN search, every Stack Overflow reference will collide with this. You cannot own this term.

**Overloaded.** In a developer community, "code" is the most generic possible word. "CODE Framework" reads as "code framework" — it carries zero signal. Compare with names that actually work on HN: Shape Up, DORA, Accelerate, Impact Mapping. Each is distinct and searchable.

**Acronym-first thinking.** The backronym "Customer Outcome Driven Engineering" was clearly reverse-engineered to produce "CODE." HN readers detect this instantly and it triggers skepticism. The "Customer" in particular feels forced — the framework is really about *engineering team outcomes*, not customer outcomes in the product-management sense.

**Recommendation:** Drop the acronym. Lead with either the *Product Tree* concept (which is vivid and defensible) or the *Outcome Ledger* concept (which is technically novel). If you need a framework name, consider something that doesn't collide: e.g., "Treeline" (product tree + baseline), or even just "SPX Methodology" since you already own `spx.sh`.

---

### "Concrete Ceiling"

**This term must be replaced immediately.**

"Concrete ceiling" is an established term in workplace equity discourse, referring specifically to the barriers faced by women of color in professional advancement. It appears in Stanford Social Innovation Review, PBS, Harvard Business Review, and National Law Review — all in the context of racial and gender discrimination. Using it to mean "if you can't write Gherkin, you can't enter delivery" will read as, at best, tone-deaf, and at worst, appropriative. On HN, someone *will* flag this, and the ensuing discussion will drown out your actual ideas.

**Recommendation:** The underlying concept is strong — it's an admission gate based on concreteness. Consider:

- **"Testability Gate"** — precise, self-explanatory
- **"Scenario Threshold"** — emphasizes the Gherkin requirement
- **"Definition Threshold"** — broader, less tied to Gherkin specifically
- **"Concreteness Requirement"** — plain, accurate

The pitch deck's own line — "if you can't test it, it's not ready for engineering" — is already a better formulation than the name.

---

### "Product Tree"

This is the strongest name in the framework. Trees are intuitive, the metaphor genuinely maps to the structure (trunk/branch/leaf), and the pruning/growth language works well.

**Caution:** "Product Tree" has prior art in innovation games (Luke Hohmann's "Prune the Product Tree" exercise from 2006), and there's conceptual overlap with Impact Mapping (Gojko Adzic). The pitch deck should acknowledge this lineage rather than presenting the concept as wholly novel. HN readers will know these references.

---

### "BSP" (Binary Space Partitioning)

The borrowing of "BSP" from computational geometry is clever but potentially misleading. BSP in game engines / 3D rendering means something very specific (recursive spatial subdivision of a scene). What you're doing is closer to **gap numbering** or **order-preserving insertion** — a well-known technique in database B-trees and list ordering (used by Figma, by fractional indexing in CRDTs, etc.).

Calling it BSP will prompt "that's not what BSP means" responses from anyone who's done graphics programming. The `@` recursion syntax for deep insertion is also going to raise eyebrows — it adds cognitive overhead that a fractional indexing scheme would handle more cleanly.

**Recommendation:** Call it "gap numbering" or "order-preserving numbering." If you want a distinctive term, even "decimal insertion" is more accurate than BSP. The appendix can note the conceptual parallel to space partitioning, but leading with it invites pedantic corrections.

---

## 2. Positioning Problems

### The Deck Reads as Consulting Pitch, Not Show HN

The pitch deck targets "VP Engineering, CTOs, and engineering leaders who are accountable for outcomes" and includes a phased engagement model (Diagnostic → Pilot → Scale). On HN, this framing is kryptonite. It signals:

- **You want to sell me something**, not show me something useful
- **You haven't validated this at scale** — the engagement model *is* the validation plan
- **This is methodology theater** — process frameworks that come with phased consulting engagements are exactly what HN is allergic to

The irony is that you *have* something concrete: a working CLI (`spx`), a Claude Code plugin marketplace, and a novel verification mechanism (the outcome ledger). Those are *artifacts*, not slide decks.

**Recommendation for HN:**

Lead with the artifact, not the methodology. The strongest HN post would be:

> **Show HN: spx — A Merkle tree for test evidence that catches drift before CI**
>
> *I built a spec-driven development CLI that tracks which tests prove which specifications using git blob SHAs. When a child spec's evidence changes, parent containers go stale — like a Merkle tree of verification state, separate from Git's content tree. Here's the repo, here's how it works.*

This gives HN readers something to *inspect*, not something to *believe*.

### Two Audiences, Zero Focus

The materials simultaneously target:

1. **Individual developers using AI coding agents** (the actual spx-claude repo users)
2. **VP/CTO-level engineering leaders at 50–500 person orgs** (the pitch deck audience)

These are incompatible audiences for a single HN post. The individual developer cares about the CLI, the plugin structure, and whether this makes Claude Code more effective. The VP/CTO cares about metrics, governance, and tooling integration with Jira/Linear.

**Recommendation:** Pick one audience for HN. The natural choice is #1 — individual developers using AI coding agents. This is where you have actual traction (the repo exists, it works, people can try it). The VP/CTO pitch is a later-stage asset for when you have case studies and adoption numbers.

---

## 3. Conceptual Strengths to Emphasize

These are the ideas that are genuinely differentiated and will hold up to scrutiny:

**Outcome Ledger as a separate Merkle tree.** The insight that Git gives you content integrity but *not verification state* is real and under-explored. The outcome ledger (outcomes.yaml) creating a parallel tree of "what passed, when" with blob SHA coupling is technically novel. This is the core intellectual contribution. Lead with it.

**Co-location of spec + tests + evidence.** The principle that a container holds its own spec, tests, and outcome ledger together — no parallel trees to synchronize — is a genuinely practical design choice. It's the opposite of the "test folder mirrors src folder" pattern, and it has real advantages for discoverability, especially by AI agents.

**States derived from evidence, not declared.** "Passing" means "all tests pass and blob SHAs match," not "someone moved a card." "Stale" means "a descendant's evidence changed." This is a concrete improvement over status fields in project management tools.

**Precommit as primary, CI as insurance.** This inverts the common assumption and is practically useful — especially for agentic workflows where feedback speed matters.

**"Potential → Reality" framing.** This is the best piece of writing in the entire set. The reframe from "backlog debt" to "creating and realizing potential" is memorable and emotionally resonant. It's worth preserving even as you restructure the rest.

---

## 4. Strongest Counter-Arguments & Preemptive Responses

### Counter: "This is just BDD/Gherkin with extra steps"

**The attack:** "Given/When/Then has been around since 2003. This is Cucumber with a file system convention stapled on."

**Preemptive response:** Acknowledge the lineage explicitly. The innovation isn't Gherkin — it's the *verification tracking*. BDD tells you to write scenarios; it says nothing about how to know whether they're still passing, when they last passed, or whether changes elsewhere broke them. The outcome ledger is the missing piece that BDD never had. *Don't overclaim.*

### Counter: "Trees can't model real product structures"

**The attack:** "Real products have cross-cutting concerns, shared services, and many-to-many relationships. Forcing everything into a tree is a lie."

**Preemptive response:** This is the hardest one. Trees genuinely *are* a simplification. Your BSP numbering partially addresses independent parallel work (same BSP = parallel), but shared infrastructure components that serve multiple capabilities are awkward in a strict tree. The test-infrastructure capability at BSP 13 is a workaround, not a solution. Be honest about this. One possible response: "The tree models the *verification hierarchy*, not the dependency graph. Shared components can be capabilities with their own outcome ledgers; consumers reference them via ADRs." But acknowledge the tension.

### Counter: "This is waterfall wearing agile clothing"

**The attack:** "Spec → Decision → Implementation is literally waterfall. You're just calling it something different."

**Preemptive response:** The HN discussion "Spec-Driven Development: The Waterfall Strikes Back" (Nov 2025) already surfaced this. Your best defense is that specs are *mutable*. They update in place. The outcome ledger handles the fact that reality drifts from intent. The flow isn't Spec → Ship → Forget. It's Spec → Prove → Evolve → Re-prove. The stale/regressed states are the mechanism that prevents waterfall's failure mode (specs that diverge from reality permanently). Point to the "Editing a Spec = Raising the Bar" section — that's your anti-waterfall argument.

### Counter: "No evidence this works beyond one person's projects"

**The attack:** "You built this for yourself. Show me a team of 50 using it."

**Preemptive response:** Don't bluff. Be transparent that this emerged from your own practice and is being validated. The strength is that it's *running* — there's a CLI, there are plugins, there's a repo with 162 commits. Many successful HN frameworks (Shape Up, trunk-based development advocacy, etc.) started as one practitioner's documented approach. The pitch deck's "50–500 engineer" framing actively undermines you here. Drop it for the HN audience.

### Counter: "The @ recursion in BSP is just adding complexity"

**The attack:** "When your numbering scheme needs recursion, you've already lost. Just use strings/UUIDs/fractional indexes."

**Preemptive response:** This is a fair point. The `@` syntax is the weakest part of the BSP design. In practice, if you're hitting depth 3, you should be rebalancing — which the docs acknowledge. For an HN post, de-emphasize the recursion mechanism and present the basic gap numbering as the core idea. The `@` syntax is an implementation detail that can be relegated to the appendix.

### Counter: "Gherkin as a gate excludes valid engineering work"

**The attack:** "Plenty of legitimate engineering work — infrastructure, performance optimization, refactoring — doesn't fit Given/When/Then naturally. Your 'testability gate' would reject it."

**Preemptive response:** This is partially true. The framework handles it via test levels (Level 1/2/3) and the concept of harnesses. Infrastructure work gets its own capability (the test-infrastructure capability). But some work — like "improve cold start latency by 200ms" — fits awkwardly into Gherkin. Acknowledge this. One response: "Performance targets are scenarios: GIVEN a cold start WHEN the application initializes THEN startup completes in under 200ms. But we agree that not everything is naturally behavioral — the testability gate's job is to force you to *think about evidence*, not to force Gherkin on everything."

---

## 5. Wording and Language Fixes

**"Operating system for engineering"** — overused in consulting/startup contexts (Holacracy, EOS, etc.). Use "engineering framework" or simply "development methodology."

**"Momentum" vs. "velocity"** — the distinction is good but the deck labors it. State it once, cleanly, and move on.

**"Debt psychology" / "graveyards of good intentions"** — these are punchy and good. Keep them.

**"Specs become durable artifacts (not disposable)"** — strong. This is the core value proposition in one line.

**"Pruning" instead of "grooming"** — good rename, but "grooming" is already being replaced in the industry ("refinement"). The pruning metaphor works because it maps to the tree.

**"Realization rate + drift"** — these are the two best metric names. They're concrete and measurable. Lean into them.

**"This is not a new agile flavor"** — risky. HN readers will immediately think "...yes it is." Better to not disclaim and let the technical merits speak for themselves.

---

## 6. What to Actually Post on HN

### Option A: Show HN (recommended)

Focus: the `spx-claude` repo as a Claude Code plugin marketplace with a novel verification mechanism.

Title: `Show HN: Outcome Ledger – A Merkle tree of test evidence for spec-driven development`

Content: 2–3 paragraphs explaining the problem (specs drift from reality, "done" doesn't stay done), the mechanism (outcome ledger with blob SHA coupling), and how to try it (`claude plugin marketplace add simonheimlicher/spx-claude`).

Link: the repo.

### Option B: Blog post + HN submission

Write a standalone post on spx.sh explaining the Product Tree and Outcome Ledger concepts. This gives you more room for the philosophy (the "Potential → Reality" framing) while still being grounded in a working implementation.

Title: `The Product Tree: Replacing infinite backlogs with provable outcomes`

---

## Summary of Changes

| Current                                    | Recommended                                                     |
| ------------------------------------------ | --------------------------------------------------------------- |
| CODE (Customer Outcome Driven Engineering) | Drop the acronym. "SPX Methodology" or lead with "Product Tree" |
| Concrete Ceiling                           | Testability Gate or Definition Threshold                        |
| BSP (Binary Space Partitioning)            | Gap numbering / order-preserving numbering                      |
| "Operating system for engineering"         | "Engineering framework" or just "methodology"                   |
| Target: VPs/CTOs of 50–500 eng orgs        | Target: Individual devs using AI coding agents                  |
| Consulting engagement model                | Drop entirely for HN. Show the repo.                            |
| "This is not a new agile flavor"           | Remove. Let the work speak.                                     |
