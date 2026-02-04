---
name: decomposing-feature-to-stories
description: |
  Break down a feature into story specs. Use when decomposing a significant slice into atomic implementation units.
  Stories must be expressible as Gherkin scenarios. Aim for ≤7 stories per feature.
---

<objective>
Transform Feature specs into well-scoped Story specs. Each story is an atomic implementation unit—something that can be understood, implemented, and tested as a single coherent piece, expressible as Gherkin scenarios.
</objective>

<prerequisite>
**READ FIRST**: `/understanding-code-framework`

You must understand:

- Story = atomic implementation unit (expressible as Gherkin)
- Feature should have ≤7 stories
- Stories prove themselves through the outcome ledger
- The tree can start small (1 story) and grow organically

</prerequisite>

<quick_start>

1. Read the feature spec completely
2. Identify atomic pieces—each expressible as Gherkin
3. Each story should be implementable as a single coherent unit
4. Create story specs with BSP numbering based on dependencies
5. Aim for ≤7 stories (if more, feature may need splitting)

</quick_start>

<workflow>

## Step 1: Analyze Feature for Atomic Pieces

Read the feature and ask: **"What atomic behaviors make up this feature?"**

| Look for           | Examples                                          |
| ------------------ | ------------------------------------------------- |
| Distinct behaviors | "Parse input", "Validate data", "Generate output" |
| User actions       | "Submit form", "Click button", "View result"      |
| Processing steps   | "Load file", "Transform data", "Save result"      |
| Error handling     | "Handle invalid input", "Recover from failure"    |

**Key test:** Can each piece be expressed as a Gherkin scenario?

```gherkin
GIVEN [precondition]
WHEN [action]
THEN [expected result]
```

If you can't write the Gherkin, the piece isn't well-defined enough.

## Step 2: Verify Atomicity

**An atomic story can be implemented as a single coherent unit.**

For each potential story, ask:

```
□ Can I write clear Gherkin for this?
□ Can I implement this without partial states?
□ Does it have clear acceptance criteria?
□ Is it a single concern, not multiple behaviors bundled?
```

### Signs a Story is Too Large

- You're describing multiple distinct behaviors
- The Gherkin has many unrelated THEN clauses
- Implementation would require multiple PRs
- You keep saying "and also..."

### Signs a Story is Too Small

- It's really part of another story's implementation
- It has no user-visible behavior
- It's a technical detail, not a behavior
- You can't write meaningful Gherkin for it

## Step 3: Check the 7-Story Limit

**If you identify >7 stories, the feature may be too large.**

```
IF stories > 7:
    Consider: Is this really one feature?
    Options:
    - Split feature into multiple features
    - Combine related stories
    - Re-scope the feature
```

## Step 4: Determine BSP Ordering

**Lower BSP = dependency.** Stories with lower BSP must be done before higher BSP stories.

| Pattern                    | BSP Strategy        |
| -------------------------- | ------------------- |
| Foundation (types, config) | Low BSP (10-20)     |
| Core behavior              | Mid BSP (21-50)     |
| Dependent behavior         | Higher BSP (54-80)  |
| Edge cases, polish         | Highest BSP (76-99) |

### Parallel Stories (Same BSP)

Stories with the same BSP can be worked on in parallel:

```
21-login.feature/
├── 21-parse-credentials.story/    ← Foundation
├── 37-validate-password.story/    ← Same BSP = parallel
├── 37-check-account-status.story/ ← Same BSP = parallel
└── 54-create-session.story/       ← Depends on validation (37)
```

## Step 5: Write Story Specs

For each story, create:

```
spx/.../NN-{slug}.feature/NN-{slug}.story/
├── {slug}.story.md
├── outcomes.yaml          # Created when tests pass
└── tests/                 # Created when starting implementation
```

### Story Spec Structure

Use the template from `/managing-spx`:

```markdown
# Story: {Name}

## Purpose

{What this atomic unit delivers and why it matters}

## Outcomes

### 1. {Outcome name}

\`\`\`gherkin
GIVEN {precondition}
WHEN {action}
THEN {expected result}
AND {additional assertion}
\`\`\`

#### Test Files

| File                                    | Level | Harness |
| --------------------------------------- | ----- | ------- |
| [{slug}.unit](tests/{slug}.unit.test.*) | 1     | -       |

#### Analysis

*Implementation may diverge as understanding deepens.*

| File              | Intent         |
| ----------------- | -------------- |
| `src/path/file.*` | {What changes} |

| Constant                | Intent |
| ----------------------- | ------ |
| `src/constants.*::NAME` | {Why}  |

| Config Parameter | Test Values      | Expected Behavior |
| ---------------- | ---------------- | ----------------- |
| `ENV_VAR`        | `unset`, `value` | {Behavior}        |

---

### 2. {Second outcome if needed}

{Same structure}

---

## Architectural Constraints

{ADRs that constrain this story}
```

**Note:** Stories have an **Analysis** section that proves the agent examined the codebase. This is unique to stories.

## Step 6: Validate Decomposition

For each story:

```
□ Is expressible as Gherkin scenarios
□ Is atomic—implementable as single unit
□ Has clear acceptance criteria
□ BSP reflects dependencies
□ Total stories ≤7 (if more, reconsider feature scope)
```

</workflow>

<examples>

## Example 1: Login Feature → Stories

**Feature:** "Password-based Login"

**Analysis:**

| Atomic behavior                    | Gherkin possible?  | Story?  |
| ---------------------------------- | ------------------ | ------- |
| Parse username/password from input | YES                | YES     |
| Validate password against hash     | YES                | YES     |
| Check account isn't locked         | YES                | YES     |
| Create session token               | YES                | YES     |
| Return auth response               | YES                | YES     |
| Handle invalid credentials         | Part of validation | COMBINE |

**Result (5 stories):**

```
21-login.feature/
├── 21-parse-credentials.story/
├── 37-validate-password.story/
├── 37-check-account-status.story/
├── 54-create-session.story/
└── 76-auth-response.story/
```

## Example 2: SPI Master Feature → Stories (Corrected)

**Feature:** "SPI Master"

**Wrong approach (what the agent did):**
Put all outcomes in the feature spec.

**Correct approach:**

| Atomic behavior                | Story?                           |
| ------------------------------ | -------------------------------- |
| SPI master transmits in mode 0 | YES - `21-mode0-transmit.story/` |
| SPI modes 1-3 timing           | COMBINE with mode 0 OR separate  |
| Clock generation               | Part of mode stories OR separate |

**Result:**

```
21-spi-master.feature/
├── 21-mode0.story/         # Mode 0 transmit/receive
├── 37-mode1.story/         # Mode 1 (or combine modes)
├── 37-mode2.story/
├── 37-mode3.story/
└── 54-clock-config.story/  # Clock divider configuration
```

**OR (if modes are similar):**

```
21-spi-master.feature/
├── 21-basic-transfer.story/  # Core transfer logic
├── 37-mode-config.story/     # Mode 0-3 configuration
└── 54-clock-config.story/    # Clock divider
```

## Example 3: Too Many Stories - Feature Needs Splitting

**Original:** "Document Processing" with 12 potential stories

**Problem:** >7 stories indicates feature too large.

**Solution:** Split feature first:

```
BEFORE:
21-document-processing.feature/
├── (12 stories...)

AFTER:
21-document-import.feature/
├── 21-parse-format.story/
├── 37-validate-schema.story/
└── 54-store-document.story/

37-document-transform.feature/
├── 21-apply-rules.story/
├── 37-validate-output.story/
└── 54-handle-errors.story/

54-document-export.feature/
├── 21-format-output.story/
├── 37-write-file.story/
└── 54-verify-export.story/
```

</examples>

<level_of_abstraction>

## Level of Abstraction Depends on Context

The same behavior can be one story or multiple:

**Simple auth system:**

```
21-login.feature/
└── 21-login-flow.story/  # Everything in one
```

**Complex auth system:**

```
21-login.feature/
├── 21-parse-credentials.story/
├── 37-validate-password.story/
├── 37-check-mfa.story/
├── 54-check-account-status.story/
├── 54-audit-login.story/
├── 76-create-session.story/
└── 87-return-response.story/
```

**How to decide:**

| Factor      | Fewer stories         | More stories          |
| ----------- | --------------------- | --------------------- |
| Complexity  | Simple logic          | Complex logic         |
| Team size   | Solo dev              | Multiple devs         |
| Risk        | Low risk              | High risk             |
| Testability | Easy to test together | Need isolated testing |

**Rule of thumb:** Start with fewer stories. Split when you need to.

</level_of_abstraction>

<anti_patterns>

## What NOT to Create as Stories

| Anti-pattern               | Why wrong               | Fix                      |
| -------------------------- | ----------------------- | ------------------------ |
| "The login feature"        | Feature, not story      | Keep as feature          |
| "Write tests"              | Meta task, not behavior | Part of all stories      |
| "Refactor X"               | Technical task          | Tied to a behavior story |
| "Fix bug in Y"             | Maintenance             | Tied to a behavior story |
| Multiple behaviors bundled | Not atomic              | Split into stories       |

## Signals You're at Wrong Level

| Signal                        | Problem                      |
| ----------------------------- | ---------------------------- |
| Can't write Gherkin           | Not well-defined             |
| Multiple GIVEN/WHEN/THEN sets | Multiple stories             |
| "And also..." in description  | Multiple stories             |
| No user-visible behavior      | May be implementation detail |
| Need >7 stories               | Feature too large            |

</anti_patterns>

<gherkin_quality>

## Writing Good Gherkin

**Good Gherkin is specific and testable:**

```gherkin
# GOOD
GIVEN an SPI master configured for mode 0 (CPOL=0, CPHA=0)
WHEN data 0xAB is transmitted
THEN SCLK idles low before transmission
AND data is sampled on rising edge
AND data is shifted on falling edge
AND MOSI outputs 0xAB MSB-first

# BAD (vague)
GIVEN SPI master
WHEN data is sent
THEN it works correctly
```

**The Gherkin test:**

- Can someone implement tests from this Gherkin alone?
- Is every assertion verifiable?
- Are the preconditions specific enough?

</gherkin_quality>

<success_criteria>

Decomposition complete when:

- [ ] Each story is expressible as Gherkin scenarios
- [ ] Each story is atomic and implementable as single unit
- [ ] Total stories ≤7 (or feature needs splitting)
- [ ] BSP ordering reflects dependencies
- [ ] Analysis section proves codebase examination
- [ ] Stories have clear acceptance criteria

</success_criteria>
