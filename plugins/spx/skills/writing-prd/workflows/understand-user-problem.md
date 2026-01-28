<required_reading>
Read: `references/measurable-outcomes.md` - Understanding user value
</required_reading>

<process>
<deep_thinking>
**User Pain vs Symptom**

Pause and analyze what the user has stated:

- Is this the ROOT USER PAIN or just a SYMPTOM?
- What capability do users currently lack?
- Why does this limitation impact their workflow or goals?
- What would resolving it enable users to accomplish?
- How does this align with overall product vision?

**Form a hypothesis about the root user problem.**
</deep_thinking>

<propose_analysis>
Present your analysis to the user:

```
Based on our discussion, I believe the user problem is:

**Who**: [User archetype - role, context, constraints]
**Problem**: As [user type], I am frustrated by [core problem]
**Because**: [Underlying cause - why this problem exists]
**Prevents**: [Desired outcome users cannot achieve]

**Current Pain**:
- Symptom: [What users experience today]
- Root Cause: [Why the problem exists systemically]
- Customer Impact: [How this affects workflow/productivity]
- Business Impact: [How this affects product value]

Does this capture the user problem, or should I adjust my understanding?
```

**Wait for user confirmation or correction.**

If corrected, update your understanding and re-propose until confirmed.
</propose_analysis>

<analyze_customer_journey>
Map the transformation this product creates:

**Before** (current state):

- How do users operate today without this capability?
- What workarounds do they use?
- What pain do they experience?

**During** (transition):

- How will users discover this capability?
- What will they need to learn?
- How will they adopt it into their workflow?

**After** (future state):

- How will users operate with this capability?
- What will be different in their workflow?
- What new value will they experience?

</analyze_customer_journey>

<propose_customer_journey>
Present to user using AskUserQuestion if clarification needed:

```
**Customer Journey:**

**Before**: [Current state - specific workflow and pain]
**During**: [Transition - discovery and adoption path]
**After**: [Future state - transformed workflow with capability]

Does this customer journey resonate? Any aspects I'm missing?
```

**Wait for user validation.**
</propose_customer_journey>

<document_assumptions>
Since we cannot interview real users, make assumptions explicit:

- User technical capability (CLI comfort, git knowledge)
- User context (workflow patterns, tool usage)
- User goals and priorities (speed vs control)
- User constraints (infrastructure access, tooling)

If assumptions beyond standard user capabilities apply, document in table format. Otherwise: "None identified".
</document_assumptions>

<identify_gaps>
Analyze what information is STILL unclear:

- Are there ambiguous user needs?
- Are there unstated product goals?
- Are there unclear success criteria?

**Only ask about genuine gaps.** Don't ask about things already discussed.

If gaps exist, use AskUserQuestion to fill them before proceeding to Phase 2.
</identify_gaps>

</process>

<success_criteria>
Phase 1 complete when:

- [ ] User problem confirmed (not just symptom)
- [ ] Customer journey validated (before/during/after)
- [ ] User assumptions documented
- [ ] All critical gaps in understanding filled
- [ ] Ready to design measurable outcomes

</success_criteria>
