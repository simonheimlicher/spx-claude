---
name: decomposing-prd-to-capabilities
description: |
  Break down a PRD into capability specs. Use when decomposing product requirements into what the product CAN DO.
  Creates cross-cutting vertical slices like "Document Generation" or "Identity Management".
---

<objective>
Transform Product Requirements (PRD) into well-scoped Capability specs. Each capability represents what the product CAN DO—a large, cross-cutting vertical slice of functionality.
</objective>

<prerequisite>
**READ FIRST**: `/understanding-outcome-decomposition`

You must understand:

- Capability = what the product CAN DO (cross-cutting vertical slice)
- Capabilities may span many customer journeys
- The tree can start small (1 capability) and grow organically

</prerequisite>

<quick_start>

1. Read the PRD completely
2. Identify what the product should be CAPABLE of doing
3. Each capability is a cross-cutting area, not a customer journey
4. Create capability specs with BSP numbering based on dependencies
5. Start small—a single capability is valid

</quick_start>

<workflow>

## Step 1: Analyze PRD for Product Abilities

Read the PRD and ask: **"What should this product be CAPABLE of doing?"**

| Look for                 | Examples                                       |
| ------------------------ | ---------------------------------------------- |
| Core functionality areas | "Generate documents", "Manage user identities" |
| Cross-cutting concerns   | "Analytics", "Notifications", "Audit logging"  |
| Infrastructure needs     | "Test harness", "CLI tooling"                  |
| Integration points       | "Third-party API integration", "Export/import" |

## Step 2: Identify Capability Boundaries

**A capability is what the product CAN DO.**

Examples of good capability names:

- "Document Generation"
- "Custody Account Management"
- "Monetization Platform"
- "Identity and Authentication Management"
- "SPI Peripheral Support" (for hardware library)

**NOT capabilities:**

- "User logs in and sees dashboard" (customer journey, not product ability)
- "Parse configuration file" (too small—story or feature level)
- "Sprint 1 work" (timeline, not functionality)

## Step 3: Consider Starting Small

**A single capability is valid.** The tree grows organically.

```
spx/
├── product.prd.md
└── 21-core.capability/         ← Start with one
    └── 21-first-feature.feature/
        └── 21-first-story.story/
```

Later, you might add:

```
spx/
├── product.prd.md
├── 13-test-infrastructure.capability/  ← Discovered dependency
├── 21-core.capability/
└── 54-advanced.capability/             ← Added as product grows
```

## Step 4: Determine BSP Ordering

**Lower BSP = dependency.** Order by what depends on what.

| Pattern                                | BSP Strategy        |
| -------------------------------------- | ------------------- |
| Infrastructure (test harness, tooling) | Low BSP (10-20)     |
| Core functionality                     | Mid BSP (21-50)     |
| Advanced/optional features             | High BSP (54-80)    |
| Improvements/polish                    | Highest BSP (76-99) |

### BSP Calculation

| Scenario                 | Formula               | Example                   |
| ------------------------ | --------------------- | ------------------------- |
| First capability         | 21                    | `21-core.capability/`     |
| Insert before 21         | floor((10+21)/2) = 15 | `15-infra.capability/`    |
| Insert between 21 and 54 | floor((21+54)/2) = 37 | `37-new.capability/`      |
| Append after 54          | floor((54+99)/2) = 76 | `76-advanced.capability/` |

### Parallel Capabilities (Same BSP)

Capabilities with the same BSP can be worked on in parallel:

```
37-users.capability/      ← Same BSP = parallel
37-billing.capability/    ← Same BSP = parallel
37-reports.capability/    ← Same BSP = parallel
```

## Step 5: Write Capability Specs

For each capability, create:

```
spx/NN-{slug}.capability/
├── {slug}.capability.md
└── tests/                 # Created when starting implementation
```

### Capability Spec Structure

Use the template from `/managing-spx`:

```markdown
# Capability: {Name}

## Purpose

{What this product ability delivers and why it matters to users}

## Success Metric

- **Baseline**: {Current state}
- **Target**: {Expected improvement}
- **Measurement**: {How to track}

## Requirements

{Prose from PRD, focused on this capability's scope}

## Test Strategy

{Overview of how this capability will be tested—can span multiple levels}

## Outcomes

{Capability-level outcomes, if any—remember Principle 11}

## Architectural Constraints

{ADRs and PDRs that constrain this capability}
```

**Note on Outcomes:** A capability may have its own outcomes (high-level integration/E2E scenarios) OR it may have no direct outcomes—just features underneath. Both are valid.

## Step 6: Validate Decomposition

For each capability:

```
□ Represents what the product CAN DO
□ Is a coherent cross-cutting area
□ Has clear boundaries with other capabilities
□ BSP reflects actual dependencies
□ Can be decomposed into ≤9 features (if needed now)
```

</workflow>

<examples>

## Example 1: SPI Library PRD → Capabilities

**PRD excerpt:**

> "Hardware engineers need to configure SPI peripherals, generate Verilog, and verify on real FPGAs."

**Analysis:**

| Product ability          | Capability?                                   |
| ------------------------ | --------------------------------------------- |
| "SPI Peripheral Support" | YES - what the library CAN DO                 |
| "Code Generation"        | YES - distinct ability                        |
| "FPGA Verification"      | MAYBE - could be part of code gen or separate |

**Possible structures:**

**Option A: Start small**

```
spx/
├── spi-library.prd.md
└── 21-spi-peripherals.capability/
```

**Option B: Separate concerns**

```
spx/
├── spi-library.prd.md
├── 13-test-infrastructure.capability/
├── 21-spi-peripherals.capability/
└── 54-fpga-verification.capability/
```

Both are valid. Start with A, grow to B as needed.

## Example 2: Auth System PRD → Capabilities

**PRD excerpt:**

> "Users need to register, log in, reset passwords, and manage their profiles. Support OAuth and magic links."

**Analysis:**

This is ONE capability: "Identity and Authentication Management"

```
spx/
├── auth-system.prd.md
└── 21-identity-auth.capability/
    ├── 21-registration.feature/
    ├── 37-login.feature/
    ├── 54-password-reset.feature/
    └── 76-oauth.feature/
```

**NOT** separate capabilities for login, registration, etc.—those are features.

## Example 3: E-commerce Platform PRD → Capabilities

**PRD excerpt:**

> "Customers browse products, add to cart, checkout with payment, track orders."

**Analysis:**

| Product ability    | Capability                   |
| ------------------ | ---------------------------- |
| Product catalog    | `21-catalog.capability/`     |
| Shopping cart      | `37-cart.capability/`        |
| Payment processing | `37-payment.capability/`     |
| Order fulfillment  | `54-fulfillment.capability/` |

Note: cart and payment are parallel (same BSP)—neither depends on the other.

</examples>

<anti_patterns>

## What NOT to Create as Capabilities

| Anti-pattern          | Why wrong                   | Correct level         |
| --------------------- | --------------------------- | --------------------- |
| "User Login"          | Too small                   | Feature               |
| "Parse Config"        | Too small                   | Story                 |
| "Sprint 1"            | Timeline, not functionality | Not a container       |
| "Customer Journey X"  | Journey, not ability        | May span capabilities |
| "API Endpoint /users" | Implementation detail       | Story                 |

## Signals You're at Wrong Level

| Signal                             | Problem                        |
| ---------------------------------- | ------------------------------ |
| Capability name is a verb phrase   | Might be a feature or story    |
| Capability fits in 1-2 features    | Might be too small             |
| Capability name includes "and"     | Might be multiple capabilities |
| Can't identify what product CAN DO | Needs clarification            |

</anti_patterns>

<success_criteria>

Decomposition complete when:

- [ ] Each capability represents what the product CAN DO
- [ ] Capabilities are cross-cutting vertical slices
- [ ] BSP ordering reflects dependencies
- [ ] Structure can grow organically (start small is OK)
- [ ] No capability is really a feature or story in disguise

</success_criteria>
