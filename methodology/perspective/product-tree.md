# The Product Tree: From Backlog to Momentum

## The Problem with Backlogs

Traditional backlogs are graveyards of good intentions. They grow infinitely. Anyone can add to them. Items sink to the bottom and rot. The "backlog grooming" ritual is really backlog mourning—acknowledging that most items will never ship.

Backlogs frame development as **debt reduction**. You start each sprint already behind. Progress means shrinking a pile that others keep growing. This is demoralizing by design.

## The Momentum Alternative

Outcome Engineering replaces the backlog with a **Product Tree**—a living structure where ideas must earn their place through concrete definition.

**The fundamental shift:**

| Backlog Mindset    | Momentum Mindset                 |
| ------------------ | -------------------------------- |
| Reducing debt      | Creating and realizing potential |
| Checking off tasks | Growing observable capabilities  |
| Sprint velocity    | Continuous coherent growth       |
| Infinite wishlist  | Structured outcome requirement   |

### Creating Potential

When you write a spec, you aren't adding to a pile. You are **creating potential energy** in the system. The spec defines a state of the world that doesn't yet exist but should. This potential wants to become real.

The work of engineering is converting potential into reality. Tests are the proof that conversion happened. The outcome ledger records when each piece of potential became real.

### Structured Outcome Requirement

> "It is not possible to add an infinite number of ideas that will never ship."

This is Outcome Engineering's most radical constraint. You cannot dump vague ideas into the system. Every idea must be concrete enough to fit the physics of the product:

```
NN-slug.capability/ → NN-slug.feature/ → NN-slug.story/ → structured outcomes
```

Every outcome must be expressed as a typed, structured outcome (Scenario, Mapping, Conformance, or Property), have referenced test files, and pass agentic review. The quality gate is verified coherence between specs, tests, and passing tests — not syntactic constraints.

This naturally filters wishlist items, scope creep, and "wouldn't it be nice if" features. Ideas that can't be expressed as structured outcomes aren't ready for engineering. They belong in discovery, not delivery.

## BSP: Binary Space Partitioning for Dependencies

The `NN` in `NN-slug.capability/`, `NN-slug.feature/`, `NN-slug.story/` isn't arbitrary. It's **BSP numbering**—a scheme that encodes dependency order while allowing insertion at any point.

### The Rules

| Rule                   | Value                                          |
| ---------------------- | ---------------------------------------------- |
| Range                  | [10, 99] (two digits)                          |
| Meaning                | Lower BSP = dependency, same BSP = independent |
| First item             | Start at 21 (room for ~10 before)              |
| Insert between X and Y | `floor((X + Y) / 2)`                           |
| Append after X         | `floor((X + 99) / 2)`                          |

### Why BSP?

Traditional sequential numbering (01, 02, 03) breaks when you discover dependencies late:

```
02-auth.feature/
03-dashboard.feature/
```

You realize auth needs a config system first. With sequential numbering, you must renumber everything. With BSP:

```
21-config.feature/      ← Insert: floor((10 + 37) / 2) = 23... or just use 21 if first
37-auth.feature/        ← Was 02, now has room before it
54-dashboard.feature/   ← Was 03, now has room before and after
```

### Parallel Work: Same Number, Different Names

Items with the **same BSP number** can be worked on in parallel—they all depend on the previous number, but not on each other:

```
21-setup.story/
37-auth.story/        ← All depend on 21
37-profile.story/     ← but NOT on each other
37-settings.story/    ← can work in parallel
54-integration.story/ ← depends on ALL 37s completing
```

This extends to capabilities:

```
21-test-harness.capability/      ← Infrastructure (dependency for others)
37-users.capability/             ← Functional (independent of each other)
37-billing.capability/           ← Functional (independent of each other)
37-reports.capability/           ← Functional (independent of each other)
54-linter.capability/            ← Improvement (depends on functional work)
```

### Strategic Insertion

BSP enables two critical patterns:

| Pattern                   | Strategy             | When to use                                             |
| ------------------------- | -------------------- | ------------------------------------------------------- |
| **Discovered dependency** | Insert LOWER number  | You realize functional work needs a test harness first  |
| **Improvement/polish**    | Insert HIGHER number | You want to add a linter after core functionality works |

This means the same concept can appear at different BSP numbers:

```
21-auth.capability/       ← Core auth (dependency for others)
54-auth.capability/       ← Auth improvements (depends on other features)
```

### Unified Number Space: BSP First, Type Last

Within a container, ALL items share the same BSP number space—ADRs, features, stories. The BSP number comes first (for sorting), the type comes last (for identification).

**Example: Capability with interleaved ADRs and features**

```
37-users.capability/
├── 10-bootstrap.feature/             ← No ADR dependency, can start immediately
├── 15-config.feature/                ← No ADR dependency
├── 21-auth-strategy.adr.md           ← Architectural decision
├── 22-login.feature/                 ← Depends on ADR 21
├── 22-registration.feature/          ← Depends on ADR 21 (parallel with login)
├── 37-oauth.feature/                 ← Depends on features 22
├── 54-session-management.adr.md      ← Later decision (after OAuth works)
├── 55-logout.feature/                ← Depends on ADR 54
└── 55-token-refresh.feature/         ← Depends on ADR 54 (parallel with logout)
```

**Example: Feature with interleaved ADRs and stories**

```
22-login.feature/
├── login.feature.md                  ← Spec file (slug.type.md)
├── 10-parse-credentials.story/       ← No ADR needed
├── 21-password-hashing.adr.md        ← Security decision
├── 22-hash-password.story/           ← Implements ADR 21
├── 22-verify-password.story/         ← Implements ADR 21 (parallel)
├── 37-rate-limiting.adr.md           ← Performance decision
├── 38-throttle-attempts.story/       ← Implements ADR 37
└── 54-login-flow.story/              ← Integration (depends on all above)
```

**What this enables:**

| Pattern             | Example                                            | Meaning                                        |
| ------------------- | -------------------------------------------------- | ---------------------------------------------- |
| Feature before ADR  | `10-bootstrap.feature/` before `21-auth.adr.md`    | Some work doesn't need architectural decisions |
| ADR blocks features | `21-auth.adr.md` blocks `22-login.feature/`        | Decision must be made before dependent work    |
| Parallel after ADR  | `22-login.feature/` and `22-registration.feature/` | Both depend on ADR, not on each other          |
| Later ADR           | `54-session.adr.md` after features 22-37           | Some decisions emerge from implementation      |

**The name tells you both WHEN (BSP prefix) and WHAT (type suffix).**

### Sibling-Unique, Not Global

BSP numbers are only unique among siblings (combined with slug):

```
21-foo.capability/21-bar.feature/54-baz.story/  ← One story-54
21-foo.capability/37-qux.feature/54-baz.story/  ← DIFFERENT story-54
37-other.capability/21-bar.feature/54-baz.story/  ← DIFFERENT story-54
```

**Always use full paths** when referencing work items. "story-54" is ambiguous; the full path is not.

## The Product Tree Structure

```
spx/
├── product.prd.md                        # The trunk: what is this product?
├── 21-core-decision.adr.md               # Product-level ADR
│
├── 21-test-harness.capability/           # Infrastructure (dependency for others)
│   ├── test-harness.capability.md
│   ├── outcomes.yaml
│   └── tests/
│
├── 37-users.capability/                  # Functional (parallel with billing)
│   ├── users.capability.md
│   ├── outcomes.yaml
│   │
│   ├── 10-bootstrap.feature/             # Feature before any ADR
│   │   └── bootstrap.feature.md
│   │
│   ├── 21-auth-strategy.adr.md           # ADR at position 21
│   │
│   ├── 22-login.feature/                 # Feature depends on ADR 21
│   │   ├── login.feature.md
│   │   ├── outcomes.yaml
│   │   │
│   │   ├── 21-password-hashing.adr.md        # Feature-level ADR
│   │   ├── 22-hash-password.story/           # Story depends on ADR 21
│   │   │   ├── hash-password.story.md
│   │   │   ├── outcomes.yaml
│   │   │   └── tests/
│   │   │       └── hash-password.unit.test.ts
│   │   │
│   │   └── 22-verify-password.story/         # Parallel with hash-password
│   │       ├── verify-password.story.md
│   │       └── tests/
│   │
│   └── 37-profile.feature/               # Feature depends on login (22)
│       └── profile.feature.md
│
├── 37-billing.capability/                # Parallel with users (same BSP)
│   └── billing.capability.md
│
└── 54-linter.capability/                 # Improvement (after functional)
    └── linter.capability.md
```

**Key observations:**

- Directory format: `{BSP}-{slug}.{type}/` (BSP first, then slug, type suffix)
- Spec file format: `{slug}.{type}.md` (matches directory naming)
- ADR format: `{BSP}-{slug}.adr.md` (flat files, interleaved with containers)
- Status derived from the outcome ledger, not a separate status file
- Everything sorts by BSP number first—humans see dependency order at a glance

### Why a Tree?

1. **Trees grow coherently.** You can't have a leaf without a branch, a branch without a trunk. Ideas must connect to existing structure.

2. **Trees are prunable.** Removing a capability removes its features and stories. No orphaned tickets.

3. **Trees show relationships.** The path from root to leaf IS the context. No need for "blocked by" or "related to" metadata.

4. **Trees are navigable.** You can zoom to any level—product vision, capability area, specific behavior—and understand where you are.

## Story States: Precision Over Poetry

States must communicate **what needs to happen**, not just describe the situation. Poetry in philosophy, precision in indicators.

| State         | Meaning                            | Required Action                 |
| ------------- | ---------------------------------- | ------------------------------- |
| **Unknown**   | Test Files links don't resolve     | Write tests                     |
| **Pending**   | Tests exist, not all passing       | Fix code or fix tests           |
| **Stale**     | Spec or test blob changed          | Re-commit with `spx spx commit` |
| **Passing**   | All tests pass, blobs unchanged    | None—potential realized         |
| **Regressed** | Was passing, now fails, blobs same | Investigate and fix             |

### Why Not "Aspiration" and "Realized"?

The original proposal suggested renaming these states to feel more positive:

- "Pending" → "Aspiration"
- "Passing" → "Realized"
- "Stale" → "Evolving"

The problem: **"Evolving" sounds like progress when it's actually a problem.** "Stale" is ugly but communicates urgency. When scanning a status board, you need to instantly know what needs attention.

Reserve inspirational language for the philosophy. Keep indicators clinical.

## Momentum Metrics

Traditional metrics (velocity, burndown) measure how fast you're emptying a bucket that keeps refilling. Momentum metrics measure the health of the tree.

### Tree Health Indicators

| Metric               | What it measures                                 |
| -------------------- | ------------------------------------------------ |
| **Realization Rate** | Stories moving from Pending → Passing per week   |
| **Drift**            | % of Passing stories that Regressed this week    |
| **Potential Energy** | Count of Pending + Stale stories (work waiting)  |
| **Coverage Depth**   | % of capabilities with passing integration tests |

### Understanding Drift

Drift measures how often *unrelated* changes break *previously working* stories. A story regresses when its tests fail without anyone touching its spec or tests—something elsewhere in the codebase caused the failure.

| Drift Rate | Interpretation                            |
| ---------- | ----------------------------------------- |
| < 1%       | Well-isolated architecture                |
| 1-5%       | Normal coupling, monitor trends           |
| 5-10%      | High coupling, consider refactoring       |
| > 10%      | Brittle architecture, intervention needed |

**Drift is unavoidable but manageable.** All codebases drift. The goal isn't zero drift—it's understanding your drift rate and keeping it sustainable.

### The Momentum Dashboard

```
Product: spx-cli
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Realized      ████████████████████░░░░  42/52 stories (81%)
In Motion     ████░░░░░░░░░░░░░░░░░░░░   6/52 stories
Potential     ██░░░░░░░░░░░░░░░░░░░░░░   4/52 stories

Drift: 2.4% (1/42 regressed this week)
This week: +3 realized, +2 potential created
```

This dashboard tells a story of growth, not debt.

## The Philosophy in Practice

### Writing a New Spec = Creating Potential

When you create `54-export-csv.story/export-csv.story.md`, you're not adding to a backlog. You're defining a piece of reality that should exist. The system now has potential energy—a gap between what is defined and what is proven.

### Passing Tests = Realizing Potential

When the outcome ledger records that all scenarios pass, potential has become reality. The story isn't "done" (done implies it goes away). The story is **realized**—it describes something true about the product.

### Editing a Spec = Raising the Bar

When you modify a realized story, you're not "breaking" anything. You're creating new potential. The spec now describes a better reality than what exists. Tests become stale because reality needs to catch up to the new vision.

### Deleting a Spec = Pruning the Tree

When you remove a story, you're not "closing a ticket." You're pruning—deciding this branch no longer serves the tree's growth. The product becomes simpler, more focused.

## Migration from Backlog Thinking

If your team uses traditional backlogs, the shift is cultural before it's technical:

1. **Stop saying "tickets."** Say "stories" or "outcomes."
2. **Stop saying "close."** Say "realize" or "prove."
3. **Stop measuring velocity.** Measure realization rate.
4. **Stop grooming.** Start pruning.
5. **Stop estimating.** Start decomposing until stories are obvious.

The tools follow the language. Change how you talk, and the process changes with it.

---

## Summary

The Product Tree replaces the infinite backlog with a coherent, growable structure. The structured outcome requirement ensures only well-defined work enters the system. Momentum metrics measure growth rather than debt reduction.

**The key insight:** Development isn't about emptying a pile. It's about growing a tree—creating potential and converting it to reality, one proven behavior at a time.
