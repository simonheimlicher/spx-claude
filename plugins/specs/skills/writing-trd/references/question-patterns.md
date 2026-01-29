# Effective Questioning Strategies

## Overview

The skill uses AskUserQuestion tool to fill genuine gaps in understanding. Questions must be:

- **Targeted**: Address specific unknowns, not general exploration
- **Informed**: Based on domain expertise embedded in skill
- **Actionable**: Answers enable concrete progress

## When to Ask vs. Decide

### Always Ask (Cannot Proceed Without)

1. **Root cause validation**
   - Purpose: Confirm problem diagnosis
   - Format: Propose analysis, request confirmation/correction

2. **Solution approach approval**
   - Purpose: Ensure architectural alignment
   - Format: Propose components/flow, request approval

3. **Test harness details**
   - Purpose: Document Level 2 infrastructure
   - Format: For each dependency, ask: type, setup, reset commands

4. **Credential information**
   - Purpose: Document Level 3 infrastructure
   - Format: For each service, ask: env vars, source, rotation, test accounts

### Agent Decides (With Rationale)

1. **Guarantees to include**
   - Agent analyzes solution and proposes guarantees
   - Based on: Solution components, integrations, edge cases

2. **Test level assignments**
   - Agent assigns per `/testing` methodology
   - Documents: Why this level, what confidence it provides

3. **BDD scenarios**
   - Agent derives from guarantees
   - Format: Strict Given/When/Then

4. **Component architecture**
   - Agent proposes based on solution discussion
   - User approves overall direction

## Question Timing

### Phase 1: Understanding

- **Root cause**: Early, blocks all downstream work
- **Solution approach**: After root cause confirmed
- **Constraint clarification**: If ambiguous from conversation

### Phase 2: Validation

- Usually NO questions (agent decides based on methodology)
- OPTIONAL: If complex, agent may summarize for user review

### Phase 3: Infrastructure

- **MUST ask** about Level 2 harnesses
- **MUST ask** about Level 3 credentials
- Cannot proceed with vague "we'll figure it out"

## Question Patterns

### Pattern: Root Cause Validation

**Purpose**: Confirm problem diagnosis before proceeding

**Format**:

```
Based on our discussion, I believe the root cause is:

**Problem**: [Technical limitation]
**Because**: [Underlying reason]
**Blocks**: [Desired capability]

**Symptom**: [What users currently work around]

Does this capture the core issue, or should I adjust my understanding?
```

**Wait for**: Confirmation or correction

**Iteration**: Re-propose until user confirms

### Pattern: Solution Approach Approval

**Purpose**: Ensure architectural alignment

**Single approach**:

```
To address this, I propose:

**Components**:
- [Component 1]: [Responsibility]
- [Component 2]: [Responsibility]

**Data Flow**:
[Input] → [Component A] → [Component B] → [Output]

**Key Interfaces**:
- [Interface 1]: [Purpose]

Does this approach align with your vision?
```

**Multiple approaches** (use AskUserQuestion):

- Present 2-3 distinct approaches
- Describe tradeoffs for each
- Let user select or provide alternative

### Pattern: Test Harness Discovery

**Purpose**: Document Level 2 infrastructure

**Use AskUserQuestion with structured questions**:

```
I need to document Level 2 test harnesses for [dependency].

Please provide:

1. **Harness type**: Docker container, local install, or other?
2. **Setup command**: How to start it?
3. **Reset command**: How to clean between tests?
4. **Documentation**: Where is this configured or documented?
```

**If user doesn't know**: Add to Infrastructure Gaps table

### Pattern: Credential Discovery

**Purpose**: Document Level 3 infrastructure

**Use AskUserQuestion with structured questions**:

```
I need to document Level 3 credentials for [service/API].

Please provide:

1. **Environment variables**: What env vars hold credentials?
2. **Source**: Where are credentials stored (1Password, .env.test, etc.)?
3. **Rotation schedule**: How often do they expire?
4. **Test accounts**: What test accounts exist?
5. **Notes**: Any rate limits, sandbox vs production, etc.?
```

**If user doesn't know**: Add to Infrastructure Gaps table

## Adaptive Questioning

### High Context (from conversation)

User has already described problem, hinted at solution.

**Approach**:

- Minimal questioning
- Confirm understanding with proposals
- Ask only about genuine gaps

**Example**:

```
From our discussion, I understand you want to [summarize].
Let me confirm the root cause: [propose analysis]

Is this correct?
```

### Medium Context (some discussion)

User has stated problem but solution approach unclear.

**Approach**:

- Confirm root cause
- Propose or ask about solution direction
- Ask about infrastructure specifics

**Example**:

```
I see the problem is [X]. Two possible approaches:
1. [Approach A]: [Tradeoffs]
2. [Approach B]: [Tradeoffs]

Which aligns better with your vision, or is there another direction?
```

### Low Context (minimal information)

User invoked skill with minimal prior discussion.

**Approach**:

- Ask about problem and constraints first
- Then ask about solution preferences
- Then infrastructure details

**Example**:

```
To write a complete TRD, I need to understand:

1. **The problem**: What technical limitation are users facing?
2. **Constraints**: Any architectural, technology, or timeline constraints?
3. **Solution vision**: Any thoughts on how to address this?
```

## Question Quality

### Good Questions

✅ Specific:

```
For the PostgreSQL test harness, what Docker command starts it?
```

✅ Actionable:

```
Where are Stripe test credentials stored? (1Password, .env.test, etc.)
```

✅ Informed:

```
I see you need Level 2 tests for Hugo. Is the Hugo binary already in CI,
or do we need to add installation steps?
```

### Bad Questions

❌ Vague:

```
How should we test this?
```

❌ Domain-basic:

```
What is BDD testing? (should know from skill expertise)
```

❌ Already answered:

```
What problem are we solving? (if already discussed)
```

## Gap Handling in Questions

### Infrastructure Unknown

**Don't assume or guess.**

If user says "I don't know where credentials are":

- Document in Infrastructure Gaps table
- Mark what's blocking (Implementation / Level X tests)
- Continue with TRD creation

### Multiple Unknowns

If user doesn't know several infrastructure items:

- Ask about each specifically
- Document all gaps in table
- TRD is "incomplete but deliverable"

### Proprietary/Internal Information

Only ask about project-specific details:

- Where are credentials stored in THIS project?
- What Docker setup does THIS project use?
- What test accounts exist for THIS project?

Do NOT ask about general domain knowledge (that's in the skill).

## Verification Questions

Optional questions to confirm understanding:

**After Phase 1**:

```
Before I design the validation strategy, let me confirm:
- Root cause: [X]
- Solution approach: [Y]

Are these correct?
```

**After Phase 2** (if complex):

```
I've identified [N] guarantees across [N] test levels.

Key validation points:
- [Summary]

Shall I proceed to infrastructure discovery?
```

## Question Limits

**Typical TRD creation**: 2-4 question rounds

1. Root cause + solution confirmation (1 round)
2. Infrastructure discovery (1-2 rounds)
3. Optional clarifications (0-1 rounds)

**Avoid**:

- Asking same thing multiple times
- Asking when information already available
- Asking about general domain knowledge
- Open-ended "tell me everything" questions

## Success Indicators

Questions are effective when:

- User answers move TRD creation forward
- No repeated or circular questioning
- Agent demonstrates domain expertise
- Gaps are filled or explicitly documented
