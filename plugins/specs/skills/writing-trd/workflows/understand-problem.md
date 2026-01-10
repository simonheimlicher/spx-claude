<required_reading>
Read these references:

1. `references/testing-methodology.md` - Understand three-tier testing
2. `references/question-patterns.md` - Effective questioning strategies

</required_reading>

<process>
## Deep Thinking: Problem vs Symptom

**Pause and analyze what the user has stated:**

- Is this the ROOT CAUSE or just a SYMPTOM?
- What underlying technical limitation causes this problem?
- Why does this limitation exist (architecture, technology, design)?
- What capability would resolving it enable?

**Form a hypothesis about the root cause.**

## Propose Root Cause Analysis

Present your analysis to the user:

```
Based on our discussion, I believe the root cause is:

**Problem**: [Technical limitation]
**Because**: [Underlying reason]
**Blocks**: [Desired capability]

**Symptom**: [What users currently work around]

Does this capture the core issue, or should I adjust my understanding?
```

**Wait for user confirmation or correction.**

If corrected, update your understanding and re-propose until confirmed.

## Analyze Solution Space

From conversation context and project structure, identify:

- What components would address the root cause?
- What existing patterns/capabilities can be leveraged?
- What new capabilities must be created?
- What interfaces need to be defined?

## Propose Solution Approach

Present architectural direction to user using AskUserQuestion if multiple approaches are viable, otherwise propose single approach:

**Single approach proposal:**

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

**Multiple approaches (use AskUserQuestion):**

Present 2-3 distinct approaches with tradeoffs. Let user select or provide alternative.

**Wait for user approval.**

If user requests changes, iterate until approach is approved.

## Identify Conversation Gaps

Analyze what information is STILL unclear:

- Are there ambiguous requirements?
- Are there unstated constraints?
- Are there technology choices that need confirmation?

**Only ask about genuine gaps.** Don't ask about things already discussed or that you can infer from context.

If gaps exist, use AskUserQuestion to fill them before proceeding to Phase 2.

</process>

<success_criteria>
Phase 1 complete when:

- [ ] Root cause confirmed by user (not just symptom)
- [ ] Solution approach approved by user
- [ ] All critical gaps in understanding filled
- [ ] Ready to design validation strategy

</success_criteria>
