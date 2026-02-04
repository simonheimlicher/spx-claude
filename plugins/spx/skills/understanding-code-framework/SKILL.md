---
name: understanding-code-framework
description: |
  Foundational skill teaching CODE framework hierarchy and decomposition. Read FIRST before creating or decomposing specs.
  Use when learning CODE structure, understanding capability/feature/story distinctions, or before breaking down work.
allowed-tools: Read, Glob, Grep
---

<objective>
Teach the human interpretation of CODE framework levels. This skill explains WHAT belongs at each level based on scope, sizing, and vertical slicing—enabling agents to correctly decompose work without conflating levels.
</objective>

<essential_principles>

**OUTCOMES, NOT TASKS.**

This is the most fundamental principle. The Product Tree contains OUTCOMES—states of the world that should exist—not tasks to be done.

| WRONG (Task)                  | RIGHT (Outcome)                           |
| ----------------------------- | ----------------------------------------- |
| "Refactor to comply with ADR" | "System produces correct output"          |
| "Fix bug in X"                | "Edge case Y handled correctly"           |
| "Add error handling"          | "Invalid input returns clear error"       |
| "Write tests"                 | Tests are PROOF of outcomes, not outcomes |

**ADRs GOVERN, they don't implement.**

- ADR = "when you do X, do it THIS way" — no outcomes, no work, no tests
- NEED comes from stories/features/capabilities — they describe outcomes
- ADR compliance is verified through the outcomes that stories deliver

**SHARED NEED → ENABLER:**

When multiple containers share a need, factor it into an ENABLER at the lowest convergence point:

```
54-component-library.capability/
├── 54-fixed-point-format.adr.md       # GOVERNS (no tests here)
├── 21-dsp-foundation.feature/         # ENABLER for shared need
│   └── 21-fixed-point-helpers.story/  # Implements shared helpers
│       └── tests/                     # Tests for the enabler
├── 54-dsp-cordic.feature/             # DEPENDS ON enabler
├── 76-dsp-nco.feature/                # DEPENDS ON enabler
└── 87-dsp-cic.feature/                # DEPENDS ON enabler
```

The ADR doesn't get tests. The ENABLER that satisfies the shared need gets tests.

**The Product Tree replaces the backlog.**

- Writing a spec = creating potential energy (a state that should exist)
- Passing tests = realizing potential (proving the state exists)
- The tree grows coherently—ideas must connect to existing structure

**The Concrete Ceiling:**

> If you can't express it as a Gherkin scenario with Given/When/Then, it doesn't belong in the engineering system.

**Principle 11:**

> Higher levels unaware of lower level breakdown. Features don't list story outcomes. Capabilities don't list feature outcomes. Completion bubbles up through the outcome ledger, not spec references.

</essential_principles>

<quick_start>
This skill provides mental models for CODE decomposition. Read it BEFORE using:

- `/decomposing-prd-to-capabilities`
- `/decomposing-capability-to-features`
- `/decomposing-feature-to-stories`

Key questions for each level:

| Level          | Question                                            | Constraint                   |
| -------------- | --------------------------------------------------- | ---------------------------- |
| **Capability** | "What can the product DO?"                          | Cross-cutting vertical slice |
| **Feature**    | "What significant slice can be done in ≤7 stories?" | At most 7 atomic stories     |
| **Story**      | "What's the atomic unit?"                           | Single Gherkin scenario(s)   |

</quick_start>

<level_semantics>

<capability_level>

## Capability: What the Product CAN DO

A capability is a **large, cross-cutting vertical slice** of the product. It represents something the product is capable of doing, providing, or accomplishing.

**Examples:**

- "Document Generation"
- "Custody Account Management"
- "Monetization Platform"
- "Identity and Authentication Management"

**Key characteristics:**

- May be involved in one, many, or ALL customer journeys
- Represents a coherent area of product functionality
- Can start with a single feature and grow
- Up to 9 sibling capabilities (BSP numbering)

**NOT a capability:**

- A single API endpoint (too small → feature or story)
- A customer journey (journeys may span multiple capabilities)
- An implementation detail (belongs in story)

</capability_level>

<feature_level>

## Feature: Significant Vertical Slice (≤7 Stories)

A feature is a **significant and valuable vertical slice** that can be implemented in at most 7 atomic stories.

**Examples:**

- "Stable Diffusion Generation"
- "User Auth" or more specifically "Password Auth" vs "Magic Link Auth"
- "User Lifecycle Management"
- "Export Documents"
- "Save Files to Cloud Storage"

**The 7-story limit is critical:**

```
IF feature needs > 7 stories
THEN split into multiple features

IF feature has only 1-2 stories
THEN it may be too small (or that's fine for now—it can grow)
```

**Key characteristics:**

- Implementable in at most 7 atomic stories
- Represents significant user value
- Can start with a single story and grow
- Up to 9 sibling features per capability

**NOT a feature:**

- A single function or method (too small → story)
- An entire product area (too large → capability)
- A technical task with no user value (may not belong in the tree at all)

</feature_level>

<story_level>

## Story: Atomic Implementation Unit

A story is the **atomic unit of implementation**—something that can be understood and implemented as a single coherent piece.

**Examples:**

- "Reset password"
- "Send reset password email"
- "Parse SPI configuration"
- "Generate Verilog for SPI master"

**The level of abstraction depends on context:**

- In a simple auth system: "Reset password" might be one story
- In a complex auth system: "Send reset password email" and "Validate reset token" might be separate stories

**Key characteristics:**

- Expressible as Gherkin scenario(s)
- Atomic—can be implemented without partial states
- Has clear acceptance criteria
- Up to 9 sibling stories per feature (but aim for ≤7)

**NOT a story:**

- Multiple unrelated behaviors (split into separate stories)
- Vague requirements (not ready for engineering—needs discovery)
- Technical tasks without user-facing behavior (may be part of a story, not a separate one)

</story_level>

</level_semantics>

<organic_growth>

## The Tree Grows Organically

**Starting small is normal:**

```
21-auth.capability/
└── 21-login.feature/
    └── 21-basic-login.story/
```

This is a valid structure. One capability, one feature, one story.

**Growth happens at any level:**

```
21-auth.capability/
├── 21-login.feature/
│   ├── 21-basic-login.story/
│   ├── 37-remember-me.story/        ← Story added
│   └── 54-login-throttling.story/   ← Story added
├── 37-registration.feature/          ← Feature added
│   └── 21-email-registration.story/
└── 54-password-reset.feature/        ← Feature added
    ├── 21-request-reset.story/
    └── 37-complete-reset.story/
```

**BSP numbering enables insertion:**

- Lower BSP = dependency (must be done first)
- Same BSP = parallel (can be done concurrently)
- Higher BSP = dependent (depends on lower numbers)

</organic_growth>

<decomposition_principles>

## How to Decompose

**Capability → Features:**

Ask: "What significant vertical slices make up this capability?"

Each slice should be:

- Independently valuable
- Implementable in ≤7 stories
- A coherent unit of functionality

**Feature → Stories:**

Ask: "What atomic pieces make up this feature?"

Each piece should be:

- Expressible as Gherkin
- Implementable as a single unit
- Clearly testable

**The key question at every level:**

> Can this be expressed as Gherkin scenarios? If not, decompose further or clarify requirements.

</decomposition_principles>

<common_mistakes>

## Common Mistakes

### Mistake 1: Putting Story-Level Outcomes in Features

**Wrong** (the SPI example):

```markdown
# Feature: Serial SPI

## Outcomes

### 1. SPI master transmits in mode 0 ← Should be a story

### 2. SPI modes 1-3 verified ← Should be a story

### 3. SPI slave responds ← Should be a story

### 4. Master-slave loopback works ← Feature-level OK

### 5. Lint-clean HDL ← Feature-level OK
```

**Correct structure:**

```
22-serial-spi.feature/
├── serial-spi.feature.md       # Outcomes 4 & 5 only (if needed)
├── 10-spi-master.story/        # Outcomes 1 & 2 become stories
├── 20-spi-slave.story/         # Outcome 3 becomes a story
└── ...
```

**Why?** Principle 11—features don't list story outcomes. Stories prove themselves through the outcome ledger.

### Mistake 2: Features with >7 Stories

If you need 12 stories, you have 2 features, not 1.

### Mistake 3: Confusing Test Levels with Container Levels

Test levels (1, 2, 3) are about **infrastructure needed**:

- Level 1: No real infra (DI, temp dirs)
- Level 2: Real binaries/databases
- Level 3: Real services/credentials

Container levels (Story, Feature, Capability) are about **scope of concern**:

- Story: Atomic implementation
- Feature: ≤7 stories as a significant slice
- Capability: Cross-cutting product ability

**These are orthogonal.** A story can have Level 2 tests. A capability can have Level 1 tests.

### Mistake 4: Removing All Outcomes from Features

**WRONG:** Agents removing ALL outcomes from features because "features don't list story outcomes."

**Features MUST have their own outcomes.** Principle 11 means features don't list outcomes that ARE stories (atomic implementation units). Features still have:

- **Integration outcomes**: Scenarios where multiple stories work together
- **Quality gates**: Cross-cutting verification (lint-clean, synthesizable)
- **End-to-end scenarios**: Complete feature workflows

```markdown
# Feature: Serial SPI

## Outcomes

### 1. Master-slave loopback works ← KEEP (integration)

### 2. Generated HDL passes lint ← KEEP (quality gate)
```

What to MOVE to stories:

```markdown
### 3. SPI master mode 0 works ← MOVE TO STORY (atomic)
```

### Mistake 5: Treating ADRs as Implementation

**WRONG:** "Where do we test the ADR?" or "ADR compliance story"

**ADRs GOVERN, they don't implement:**

- ADRs create NO work, NO outcomes, NO tests
- NEED comes from stories/features/capabilities
- ADR compliance is verified through the outcomes stories deliver

**The ENABLER pattern for shared needs:**

```
WRONG:
54-component-library.capability/
├── 54-fixed-point-format.adr.md
└── tests/
    └── test_adr_compliance.py    ← ADRs don't get tests!

RIGHT:
54-component-library.capability/
├── 54-fixed-point-format.adr.md       # GOVERNS (no tests)
├── 21-dsp-foundation.feature/         # ENABLER (has the NEED)
│   └── 21-fixed-point-helpers.story/  # Satisfies shared need
│       └── tests/                     # Tests verify outcomes
├── 54-dsp-cordic.feature/             # Uses enabler
└── 76-dsp-nco.feature/                # Uses enabler
```

</common_mistakes>

<reference_index>

## References

| File                                | Purpose                         |
| ----------------------------------- | ------------------------------- |
| `references/level-decision-tree.md` | Step-by-step decision flowchart |

</reference_index>

<success_criteria>

Skill mastery demonstrated when:

- [ ] Can explain the human interpretation of each level
- [ ] Can identify when a feature has too many stories (>7)
- [ ] Can recognize when story-level outcomes are incorrectly placed in features
- [ ] Understands that test levels and container levels are orthogonal
- [ ] Can decompose a capability into features, features into stories
- [ ] Knows that small trees (1-1-1) are valid starting points

</success_criteria>
