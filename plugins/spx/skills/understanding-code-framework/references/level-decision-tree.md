# Level Decision Tree

Use this flowchart to determine the correct container level for any piece of work.

## Decision Flowchart

```
START: Given work to place in the tree
              │
              ▼
┌─────────────────────────────────────┐
│ Is this a PRODUCT ABILITY?          │
│ (What the product CAN DO)           │
│                                     │
│ Examples:                           │
│ - "Document Generation"             │
│ - "Identity Management"             │
│ - "Monetization Platform"           │
└─────────────────────────────────────┘
              │
        YES   │   NO
              │
        ▼     │
┌───────────┐ │
│CAPABILITY │ │
└───────────┘ │
              │
              ▼
┌─────────────────────────────────────┐
│ Is this a SIGNIFICANT SLICE that    │
│ can be done in ≤7 atomic pieces?    │
│                                     │
│ Examples:                           │
│ - "Password Auth"                   │
│ - "Export Documents"                │
│ - "User Lifecycle Management"       │
└─────────────────────────────────────┘
              │
        YES   │   NO (needs >7 pieces)
              │
        ▼     │      ▼
┌───────────┐ │   ┌─────────────────────┐
│FEATURE    │ │   │ Split into multiple │
└───────────┘ │   │ features            │
              │   └─────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│ Is this ATOMIC?                     │
│ (Expressible as Gherkin scenarios)  │
│                                     │
│ Examples:                           │
│ - "Reset password"                  │
│ - "Send reset email"                │
│ - "Parse SPI configuration"         │
└─────────────────────────────────────┘
              │
        YES   │   NO
              │
        ▼     │      ▼
┌───────────┐ │   ┌─────────────────────┐
│STORY      │ │   │ Decompose further   │
└───────────┘     │ or clarify reqs     │
                  └─────────────────────┘
```

## Quick Reference

| Question                       | Answer                     | Level      |
| ------------------------------ | -------------------------- | ---------- |
| What can the product DO?       | Large cross-cutting area   | Capability |
| What slice fits in ≤7 stories? | Significant vertical slice | Feature    |
| What's the atomic unit?        | Single Gherkin scenario(s) | Story      |

## The 7-Story Test

For features, always ask:

```
Can this be implemented in at most 7 atomic stories?

YES → It's a well-scoped feature
NO  → Split into multiple features
```

## Examples

### Example 1: "User Authentication System"

1. Product ability? → YES, it's what the product CAN DO
2. → **Capability**: `21-auth.capability/`

### Example 2: "Password-based Login"

1. Product ability? → NO, it's part of auth
2. Significant slice in ≤7 stories? → YES (login form, validation, session, etc.)
3. → **Feature**: `21-password-login.feature/`

### Example 3: "Validate Password Hash"

1. Product ability? → NO
2. Significant slice? → NO, it's one piece of login
3. Atomic? → YES, expressible as Gherkin
4. → **Story**: `21-validate-hash.story/`

### Example 4: "Complete E-commerce Checkout" (too big)

1. Product ability? → YES, but very large
2. Better as multiple capabilities? →
   - `21-cart.capability/`
   - `37-payment.capability/`
   - `54-fulfillment.capability/`

### Example 5: Feature with 12 Stories (too many)

Original: "User Management" with 12 stories

Split into:

- `21-user-crud.feature/` (4 stories)
- `37-user-roles.feature/` (4 stories)
- `54-user-settings.feature/` (4 stories)

## Organic Growth Patterns

### Starting Small (Valid)

```
21-auth.capability/
└── 21-login.feature/
    └── 21-basic-login.story/
```

One capability, one feature, one story. This is fine.

### Growing Horizontally

```
21-auth.capability/
├── 21-login.feature/
├── 37-registration.feature/    ← Added
└── 54-password-reset.feature/  ← Added
```

### Growing Vertically

```
21-auth.capability/
└── 21-login.feature/
    ├── 21-basic-login.story/
    ├── 37-remember-me.story/      ← Added
    └── 54-login-throttling.story/ ← Added
```

### Growing in Both Directions

```
21-auth.capability/
├── 21-login.feature/
│   ├── 21-basic-login.story/
│   ├── 37-remember-me.story/
│   └── 54-login-throttling.story/
├── 37-registration.feature/
│   ├── 21-email-signup.story/
│   └── 37-oauth-signup.story/
└── 54-password-reset.feature/
    ├── 21-request-reset.story/
    └── 37-complete-reset.story/
```

## Anti-Pattern Detection

| Signal                                    | Problem               | Fix                               |
| ----------------------------------------- | --------------------- | --------------------------------- |
| Feature with >7 stories                   | Too large             | Split into features               |
| Story that's vague                        | Not ready             | Clarify with Gherkin              |
| Capability with 1 tiny feature            | May be too small      | Consider if it's really a feature |
| Feature outcomes duplicate story behavior | Violates Principle 11 | Remove duplicate outcomes         |
