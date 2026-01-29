# BDD Scenario Best Practices

## Overview

BDD (Behavior-Driven Development) scenarios use Given/When/Then format to specify testable behaviors.

**Purpose**: Each scenario defines ONE testable behavior that implements ONE guarantee.

## Structure

```
**Scenario: [Descriptive name] [G#]**

- **Given** [Initial state or precondition]
- **When** [Action performed]
- **Then** [Observable outcome]
```

## Rules

1. **Scenario name**: Describes the behavior being tested
2. **Guarantee reference**: `[G1]`, `[G2]`, etc. links to guarantees table
3. **Given**: Sets up preconditions (state, not actions)
4. **When**: Performs ONE action
5. **Then**: States ONE observable outcome

## Given Clause

**Purpose**: Establish preconditions and initial state

**Best practices:**

- Be specific (not vague)
- Set up state, don't perform actions
- Use concrete examples, not abstractions
- Multiple Given clauses allowed if needed

**Examples:**

✅ Good:

```
- **Given** a base price of $100.00 and a discount of 15%
- **Given** an order with 3 items totaling $150.00
- **Given** a database with 5 user records
```

❌ Bad:

```
- **Given** some prices (vague)
- **Given** the user logs in (action, not state)
- **Given** valid input (not specific)
```

## When Clause

**Purpose**: Perform the action being tested

**Best practices:**

- ONE action only
- Use present tense
- Be specific about what triggers
- Focus on user/system action, not implementation

**Examples:**

✅ Good:

```
- **When** the system calculates the final price
- **When** the user submits the order
- **When** the API receives a POST request to /orders
```

❌ Bad:

```
- **When** the price calculator is instantiated and calls calculate() (implementation detail)
- **When** everything processes (vague)
- **When** the user logs in and submits the form (multiple actions)
```

## Then Clause

**Purpose**: State the observable, verifiable outcome

**Best practices:**

- Describe WHAT happens, not HOW
- Use measurable/verifiable outcomes
- Be specific (not vague)
- Avoid implementation details
- One primary outcome per scenario

**Examples:**

✅ Good:

```
- **Then** the final price is $85.00
- **Then** the system throws a validation error
- **Then** the order appears in the database with status "pending"
- **Then** the response status code is 201
```

❌ Bad:

```
- **Then** the code works correctly (not measurable)
- **Then** the PriceCalculator class returns a BigDecimal (implementation detail)
- **Then** everything succeeds (vague)
```

## Complete Example

### Guarantee

G1: Price calculation handles edge cases (Level 1)

### Scenarios

**Scenario: Discount calculation with standard percentage [G1]**

- **Given** a base price of $100.00 and a discount of 15%
- **When** the system calculates the final price
- **Then** the final price is $85.00

**Scenario: Discount calculation rejects negative discount [G1]**

- **Given** a base price of $100.00 and a discount of -10%
- **When** the system calculates the final price
- **Then** the system throws a validation error with message "Discount cannot be negative"

**Scenario: Discount calculation handles zero discount [G1]**

- **Given** a base price of $100.00 and a discount of 0%
- **When** the system calculates the final price
- **Then** the final price is $100.00

**Scenario: Discount calculation handles 100% discount [G1]**

- **Given** a base price of $100.00 and a discount of 100%
- **When** the system calculates the final price
- **Then** the final price is $0.00

## Multiple Scenarios Per Guarantee

**When to write multiple scenarios:**

- Different edge cases
- Different error conditions
- Different data variations
- Different user paths

**Each scenario should test ONE specific aspect of the guarantee.**

## Linking to Guarantees

**Every scenario MUST reference its guarantee:**

```
**Scenario: [Name] [G#]**
```

This enables verification that every guarantee has test coverage.

## Anti-Patterns

### ❌ Vague Scenarios

Bad:

```
**Scenario: System works correctly [G1]**

- **Given** some input
- **When** something happens
- **Then** it works
```

Fix: Be specific about state, action, and outcome.

### ❌ Testing Implementation

Bad:

```
**Scenario: PriceCalculator uses BigDecimal [G1]**

- **Given** a PriceCalculator instance
- **When** the calculate() method is called
- **Then** the return type is BigDecimal
```

Fix: Test behavior, not implementation details.

### ❌ Multiple Actions

Bad:

```
**Scenario: Complete checkout [G3]**

- **Given** a shopping cart with items
- **When** the user logs in, enters payment info, and confirms order
- **Then** the order is placed
```

Fix: Break into multiple scenarios, each with one action.

### ❌ No Guarantee Reference

Bad:

```
**Scenario: User can log in**

- **Given** valid credentials
- **When** user submits login form
- **Then** user is redirected to dashboard
```

Fix: Add guarantee reference `[G2]` to scenario name.

### ❌ Abstract Inputs

Bad:

```
**Scenario: Validation rejects invalid input [G1]**

- **Given** invalid input
- **When** validation runs
- **Then** error is thrown
```

Fix: Use concrete example of what "invalid" means.

## Coverage Verification

Before completing TRD, verify:

- [ ] Every guarantee (G1, G2, G3...) has ≥1 scenario
- [ ] Every scenario references exactly one guarantee
- [ ] All scenarios use strict Given/When/Then format
- [ ] No vague terms ("some", "valid", "works")
- [ ] No implementation details (class names, methods)
- [ ] Outcomes are measurable/verifiable

## Language-Specific Examples

### Python

```
**Scenario: Repository persists user with valid data [G2]**

- **Given** a PostgreSQL test database with schema applied
- **When** the repository saves a user with email "test@example.com"
- **Then** the database contains a user record with that email
```

### TypeScript

```
**Scenario: API returns 400 for missing required field [G2]**

- **Given** a POST request to /api/users with body missing "email" field
- **When** the API processes the request
- **Then** the response status is 400 and body contains error "email is required"
```

## Scenario Naming

**Good names are descriptive and specific:**

✅ Good:

- "Discount calculation handles zero discount [G1]"
- "API returns 404 for non-existent resource [G2]"
- "Payment processing succeeds with valid Stripe token [G3]"

❌ Bad:

- "Test 1 [G1]" (not descriptive)
- "It works [G2]" (vague)
- "Scenario A [G3]" (meaningless)

## Complex Scenarios

For Level 3 (E2E) scenarios spanning multiple steps, keep Given/When/Then structure but may include more detail:

```
**Scenario: Complete order workflow from cart to confirmation [G3]**

- **Given** a logged-in user with items in cart and saved payment method
- **When** the user proceeds through checkout and confirms the order
- **Then** the order confirmation page displays with order ID and the order appears in Stripe with matching metadata
```

**Note**: Even complex scenarios should have ONE primary action (the complete workflow) and ONE primary outcome (order confirmed and recorded).
