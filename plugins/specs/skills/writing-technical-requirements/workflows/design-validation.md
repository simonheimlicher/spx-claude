<required_reading>
Read these references:

1. `references/testing-methodology.md` - Three-tier testing rules
2. `references/bdd-scenario-patterns.md` - Given/When/Then best practices

</required_reading>

<process>
## Deep Thinking: Guarantee Completeness

**Pause and analyze the approved solution:**

What behaviors MUST work for this solution to succeed?

- **Happy path**: What must work in the normal case?
- **Edge cases**: What boundary conditions must be handled?
- **Error cases**: What failures must be gracefully handled?
- **Integrations**: What external interactions must work?
- **Performance**: What response time/throughput is required?

**What are we MISSING?**

- What failure modes aren't covered?
- What assumptions are we making?
- What second-order effects exist?
- What could break silently?

## Identify Guarantees

For each critical behavior, state a guarantee:

**Format**: "The system guarantees that [specific behavior] [under specific conditions]"

**Examples**:
- "Price calculation handles zero, negative, and overflow edge cases"
- "Database persists order records with transaction integrity"
- "Payment processing completes with real Stripe API integration"

**List all guarantees** (typically 3-8 for most TRDs).

## Assign Test Levels

For EACH guarantee, determine the MINIMUM test level that can verify it:

**Decision tree:**

```
Can this be verified with pure logic and dependency injection?
├─ YES → Level 1 (Unit)
└─ NO  → Can this be verified with local binaries/databases?
          ├─ YES → Level 2 (Integration)
          └─ NO  → Level 3 (E2E)
```

**Level 1 criteria:**
- Pure arithmetic/logic
- String manipulation
- Data validation
- File operations (temp dirs)
- Standard dev tools (git, node, curl)

**Level 2 criteria:**
- Requires project-specific binary (Hugo, TypeScript compiler)
- Requires database (PostgreSQL, Redis via Docker)
- Requires local service integration

**Level 3 criteria:**
- Requires external API credentials
- Requires network service (GitHub API, Stripe)
- Requires browser (Chrome, Playwright)
- Requires full user workflow

## Create Guarantees Table

Build the table with unique IDs:

| #  | Guarantee             | Level | Rationale                   |
| -- | --------------------- | ----- | --------------------------- |
| G1 | [Behavior guaranteed] | 1     | [Why Level 1 is sufficient] |
| G2 | [Behavior guaranteed] | 2     | [Why Level 2 is needed]     |
| G3 | [Behavior guaranteed] | 3     | [Why Level 3 is needed]     |

**Rationale must explain:**

- What dependencies the guarantee requires
- Why lower level cannot verify it
- What confidence this level provides

## Draft BDD Scenarios

For EACH guarantee, write ≥1 scenario using strict Given/When/Then format:

**Scenario structure:**

```
**Scenario: [Descriptive name] [G1]**

- **Given** [Initial state or precondition - be specific]
- **When** [Action performed - single action]
- **Then** [Observable outcome - measurable/verifiable]
```

**Best practices:**

- Reference guarantee by ID: `[G1]`, `[G2]`, etc.
- Given: Set up preconditions, not actions
- When: Single, clear action
- Then: Observable, testable outcome (not implementation details)
- Use concrete examples, not abstractions

**Example:**

```
**Scenario: Discount calculation handles edge cases [G1]**

- **Given** a base price of $100.00 and a discount of 15%
- **When** the system calculates the final price
- **Then** the final price is $85.00

**Scenario: Discount calculation rejects invalid input [G1]**

- **Given** a base price of $100.00 and a discount of -10%
- **When** the system calculates the final price
- **Then** the system throws a validation error
```

## Verify Coverage

Check that:

- [ ] Every guarantee (G1, G2, G3...) has ≥1 scenario
- [ ] Every scenario references exactly one guarantee
- [ ] All scenarios use strict Given/When/Then format
- [ ] Scenarios describe behavior, not implementation

## Present to User (Optional)

If validation strategy is complex or user requested review, summarize:

```
I've identified [N] guarantees across [N] test levels:

**Level 1 (Unit)**: [List guarantee IDs]
**Level 2 (Integration)**: [List guarantee IDs]
**Level 3 (E2E)**: [List guarantee IDs]

Key validation points:
- [High-level summary of what's being validated]
- [Any notable edge cases covered]

Shall I proceed to documenting test infrastructure?
```

</process>

<success_criteria>
Phase 2 complete when:

- [ ] All critical guarantees identified
- [ ] Every guarantee assigned to minimum appropriate test level
- [ ] Every guarantee has unique ID (G1, G2, G3...)
- [ ] Every guarantee has ≥1 BDD scenario
- [ ] All scenarios use Given/When/Then format
- [ ] Coverage verified (no guarantees without scenarios)
- [ ] Ready to discover test infrastructure

</success_criteria>
