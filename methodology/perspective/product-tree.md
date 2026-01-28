# The Product Tree: From Backlog to Momentum

## The Problem with Backlogs

Traditional backlogs are graveyards of good intentions. They grow infinitely. Anyone can add to them. Items sink to the bottom and rot. The "backlog grooming" ritual is really backlog mourning—acknowledging that most items will never ship.

Backlogs frame development as **debt reduction**. You start each sprint already behind. Progress means shrinking a pile that others keep growing. This is demoralizing by design.

## The Momentum Alternative

CODE replaces the backlog with a **Product Tree**—a living structure where ideas must earn their place through concrete definition.

**The fundamental shift:**

| Backlog Mindset    | Momentum Mindset                 |
| ------------------ | -------------------------------- |
| Reducing debt      | Creating and realizing potential |
| Checking off tasks | Growing observable capabilities  |
| Sprint velocity    | Continuous coherent growth       |
| Infinite wishlist  | Concrete ceiling                 |

### Creating Potential

When you write a spec, you aren't adding to a pile. You are **creating potential energy** in the system. The spec defines a state of the world that doesn't yet exist but should. This potential wants to become real.

The work of engineering is converting potential into reality. Tests are the proof that conversion happened. `pass.csv` is the ledger recording when each piece of potential became real.

### The Concrete Ceiling

> "It is not possible to add an infinite number of ideas that will never ship."

This is CODE's most radical constraint. You cannot dump vague ideas into the system. Every idea must be concrete enough to fit the physics of the product:

```
capability-NN → feature-NN → story-NN → scenarios (Gherkin)
```

If you can't express it as a Gherkin scenario with Given/When/Then, it doesn't belong in the engineering system. This naturally filters wishlist items, scope creep, and "wouldn't it be nice if" features.

**The Concrete Ceiling is not gatekeeping—it's quality control.** Ideas that can't be expressed concretely aren't ready for engineering. They belong in discovery, not delivery.

## BSP: Binary Space Partitioning for Dependencies

The `NN` in `capability-NN`, `feature-NN`, `story-NN` isn't arbitrary. It's **BSP numbering**—a scheme that encodes dependency order while allowing insertion at any point.

### The Rules

| Rule                   | Value                              |
| ---------------------- | ---------------------------------- |
| Range                  | [10, 99] (two digits)              |
| Meaning                | Lower number = must complete first |
| First item             | Start at 21 (room for ~10 before)  |
| Insert between X and Y | `floor((X + Y) / 2)`               |
| Append after X         | `floor((X + 99) / 2)`              |

### Why BSP?

Traditional sequential numbering (01, 02, 03) breaks when you discover dependencies late:

```
feature-02_auth
feature-03_dashboard
```

You realize auth needs a config system first. With sequential numbering, you must renumber everything. With BSP:

```
feature-21_config      ← Insert: floor((10 + 37) / 2) = 23... or just use 21 if first
feature-37_auth        ← Was 02, now has room before it
feature-54_dashboard   ← Was 03, now has room before and after
```

### Parallel Work: Same Number, Different Names

Items with the **same BSP number** can be worked on in parallel—they all depend on the previous number, but not on each other:

```
story-21_setup
story-37_auth        ← All depend on 21
story-37_profile     ← but NOT on each other
story-37_settings    ← can work in parallel
story-54_integration ← depends on ALL 37s completing
```

This extends to capabilities:

```
capability-21_test-harness      ← Infrastructure (must complete first)
capability-37_users             ← Functional (parallel)
capability-37_billing           ← Functional (parallel)
capability-37_reports           ← Functional (parallel)
capability-54_linter            ← Improvement (after all functional)
```

### Strategic Insertion

BSP enables two critical patterns:

| Pattern                   | Strategy             | When to use                                             |
| ------------------------- | -------------------- | ------------------------------------------------------- |
| **Discovered dependency** | Insert LOWER number  | You realize functional work needs a test harness first  |
| **Improvement/polish**    | Insert HIGHER number | You want to add a linter after core functionality works |

This means the same concept can appear at different BSP numbers:

```
capability-21_auth       ← Core auth (must complete first)
capability-54_auth       ← Auth improvements (after other features work)
```

### Unified Number Space: Type in Extension, Order in Prefix

Within a container, ALL items share the same BSP number space—ADRs, features, stories. The type moves to the file extension; the BSP number encodes pure dependency order.

**Example: Capability with interleaved ADRs and features**

```
capability-21_users/
├── 10_bootstrap.feature.md           ← No ADR dependency, can start immediately
├── 15_config.feature.md              ← No ADR dependency
├── 21_auth-strategy.adr.md           ← Architectural decision
├── 22_login.feature.md               ← Depends on ADR 21
├── 22_registration.feature.md        ← Depends on ADR 21 (parallel with login)
├── 37_oauth.feature.md               ← Depends on features 22
├── 54_session-management.adr.md      ← Later decision (after OAuth works)
├── 55_logout.feature.md              ← Depends on ADR 54
└── 55_token-refresh.feature.md       ← Depends on ADR 54 (parallel with logout)
```

**Example: Feature with interleaved ADRs and stories**

```
feature-22_login/
├── 10_parse-credentials.story.md     ← No ADR needed
├── 21_password-hashing.adr.md        ← Security decision
├── 22_hash-password.story.md         ← Implements ADR 21
├── 22_verify-password.story.md       ← Implements ADR 21 (parallel)
├── 37_rate-limiting.adr.md           ← Performance decision
├── 38_throttle-attempts.story.md     ← Implements ADR 37
└── 54_login-flow.story.md            ← Integration (depends on all above)
```

**What this enables:**

| Pattern             | Example                                                | Meaning                                        |
| ------------------- | ------------------------------------------------------ | ---------------------------------------------- |
| Feature before ADR  | `10_bootstrap.feature.md` before `21_auth.adr.md`      | Some work doesn't need architectural decisions |
| ADR blocks features | `21_auth.adr.md` blocks `22_login.feature.md`          | Decision must be made before dependent work    |
| Parallel after ADR  | `22_login.feature.md` and `22_registration.feature.md` | Both depend on ADR, not on each other          |
| Later ADR           | `54_session.adr.md` after features 22-37               | Some decisions emerge from implementation      |

**The filename tells you both WHEN (BSP prefix) and WHAT (extension).**

### Sibling-Unique, Not Global

BSP numbers are only unique among siblings (combined with slug):

```
capability-21/feature-21/story-54  ← One story-54
capability-21/feature-37/story-54  ← DIFFERENT story-54
capability-37/feature-21/story-54  ← DIFFERENT story-54
```

**Always use full paths** when referencing work items. "story-54" is ambiguous; "capability-21/feature-37/story-54" is not.

## The Product Tree Structure

```
spx/
├── product.prd.md                    # The trunk: what is this product?
├── 21_core-decision.adr.md           # Product-level ADR
│
├── 21_test-harness/                  # Infrastructure capability (BSP 21)
│   ├── test-harness.capability.md
│   └── status.yaml
│
├── 37_users/                         # Functional capability (parallel with billing)
│   ├── users.capability.md
│   ├── status.yaml
│   │
│   ├── 10_bootstrap.feature.md       # Feature before any ADR (flat, no stories)
│   │
│   ├── 21_auth-strategy.adr.md       # ADR at position 21
│   │
│   ├── 22_login/                     # Feature depends on ADR 21
│   │   ├── login.feature.md
│   │   ├── status.yaml
│   │   │
│   │   ├── 21_password-hashing.adr.md    # Story-level ADR
│   │   ├── 22_hash-password/             # Story depends on ADR 21
│   │   │   ├── hash-password.story.md
│   │   │   ├── status.yaml
│   │   │   └── tests/
│   │   │       ├── hash-password.unit.test.ts
│   │   │       └── pass.csv
│   │   │
│   │   └── 22_verify-password/           # Parallel with hash-password
│   │       ├── verify-password.story.md
│   │       └── tests/
│   │
│   └── 37_profile/                   # Feature depends on login (22)
│       └── profile.feature.md
│
├── 37_billing/                       # Parallel with users (same BSP)
│   └── billing.capability.md
│
└── 54_linter/                        # Improvement (after functional capabilities)
    └── linter.capability.md
```

**Key observations:**

- BSP prefix on directories AND files
- Type in extension (`.adr.md`, `.capability.md`, `.feature.md`, `.story.md`)
- ADRs interleaved with features/stories at same level
- Directories for containers (capabilities, features, stories with tests)
- Flat files for leaf ADRs or simple features without stories

### Why a Tree?

1. **Trees grow coherently.** You can't have a leaf without a branch, a branch without a trunk. Ideas must connect to existing structure.

2. **Trees are prunable.** Removing a capability removes its features and stories. No orphaned tickets.

3. **Trees show relationships.** The path from root to leaf IS the context. No need for "blocked by" or "related to" metadata.

4. **Trees are navigable.** You can zoom to any level—product vision, capability area, specific behavior—and understand where you are.

## Story States: Precision Over Poetry

States must communicate **what needs to happen**, not just describe the situation. Poetry in philosophy, precision in indicators.

| State         | Meaning                        | Required Action            |
| ------------- | ------------------------------ | -------------------------- |
| **Pending**   | Spec exists, no tests          | Write tests                |
| **Failing**   | Tests exist, not passing       | Fix code or fix tests      |
| **Stale**     | Spec changed, tests outdated   | Update tests to match spec |
| **Passing**   | All tests pass, spec unchanged | None—potential realized    |
| **Regressed** | Was passing, now failing       | Investigate and fix        |

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
| **Stability Index**  | Days since last regression                       |
| **Potential Energy** | Count of Pending + Stale stories (work waiting)  |
| **Coverage Depth**   | % of capabilities with passing integration tests |

### The Momentum Dashboard

```
Product: spx-cli
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Realized      ████████████████████░░░░  42/52 stories (81%)
In Motion     ████░░░░░░░░░░░░░░░░░░░░   6/52 stories
Potential     ██░░░░░░░░░░░░░░░░░░░░░░   4/52 stories

Stability: 14 days since regression
This week: +3 realized, +2 potential created
```

This dashboard tells a story of growth, not debt.

## The Philosophy in Practice

### Writing a New Spec = Creating Potential

When you create `story-54_export-csv/export-csv.story.md`, you're not adding to a backlog. You're defining a piece of reality that should exist. The system now has potential energy—a gap between what is defined and what is proven.

### Passing Tests = Realizing Potential

When `pass.csv` records that all scenarios pass, potential has become reality. The story isn't "done" (done implies it goes away). The story is **realized**—it describes something true about the product.

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

The Product Tree replaces the infinite backlog with a coherent, growable structure. The Concrete Ceiling ensures only well-defined work enters the system. Momentum metrics measure growth rather than debt reduction.

**The key insight:** Development isn't about emptying a pile. It's about growing a tree—creating potential and converting it to reality, one proven behavior at a time.
