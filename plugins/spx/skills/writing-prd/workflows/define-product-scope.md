<required_reading>
Read: `references/prd-template-guide.md` - Scope definition patterns
</required_reading>

<process>
<deep_thinking>
**Pause and analyze the approved capabilities:**

Is this scope achievable as one deliverable unit?

- Is it too large (should split into multiple PRDs)?
- Is it too small (not standalone value)?
- What's the minimal viable increment?
- What can be deferred without losing core value?

**Form hypothesis about scope boundaries.**
</deep_thinking>

<define_included>
**List capabilities that MUST be included:**

- ✅ [Capability 1: Core value delivery]
- ✅ [Capability 2: Essential for workflow]
- ✅ [Capability 3: Enables measurable outcome]

**These form the minimal viable increment.**
</define_included>

<define_excluded>
**List capabilities explicitly EXCLUDED with rationale:**

| Excluded Capability              | Rationale                                                     |
| -------------------------------- | ------------------------------------------------------------- |
| [e.g., Multi-user collaboration] | Out of scope: Different product concern, separate requirement |
| [e.g., Cloud sync]               | Out of scope: This PRD addresses local-first workflows only   |

**If no exclusions: write "None identified"**

<anti_patterns>❌ **AVOID implementation timing language:**

- "Defer to Phase 2"
- "Not in MVP"
- "Later" or "Future version"
- "v2" or "v3"

✅ **USE scope boundary language:**

- "Out of scope: This PRD addresses [specific boundary]"
- "Separate capability: [explain what makes it distinct]"
- "Different product concern: [explain separation]"

**Rationale:** Requirements describe WHAT should exist, not WHEN it gets built.</anti_patterns>

</define_excluded>

<propose_boundaries>
Present scope to user using AskUserQuestion if clarification needed:

```
**Scope Definition:**

**Included:**
- [List of included capabilities]

**Excluded:**
- [List with rationale]

**Rationale:**
[Explain value protected, complexity avoided, learning needed before expanding]

Does this scope feel right for one deliverable unit?
```

**Wait for user validation.**
</propose_boundaries>

<identify_decision_triggers>
**Analyze approach and mark decision triggers (ADR for technical, PDR for product):**

For each component or integration mentioned:

- Does it involve technology choice? → ⚠️ ADR
- Does it involve architectural pattern? → ⚠️ ADR
- Does it involve data model design? → ⚠️ ADR
- Does it involve product behavior/lifecycle? → ⚠️ PDR
- Does it involve user-facing workflow phases? → ⚠️ PDR

**Examples:**

- "Storage format for handoffs" → ⚠️ ADR: Markdown+YAML vs JSON vs SQLite
- "Context capture strategy" → ⚠️ ADR: Full history vs summary vs user selection
- "Simulation phases" → ⚠️ PDR: What phases exist and their semantics
- "Session lifecycle" → ⚠️ PDR: How sessions progress through states

**Document all ADR/PDR triggers in "Open Decisions" table.**
</identify_decision_triggers>

<document_constraints>
**Identify product-specific constraints:**

| Constraint                         | Impact on Product       | Impact on Testing                     |
| ---------------------------------- | ----------------------- | ------------------------------------- |
| [e.g., Browser localStorage limit] | Must implement archival | E2E tests verify graceful degradation |

**If standard product development: write "None identified"**
</document_constraints>

</process>

<success_criteria>
Phase 3 complete when:

- [ ] Included capabilities defined (delivering core value)
- [ ] Excluded capabilities documented with rationale
- [ ] Scope boundaries validated by user
- [ ] ADR/PDR triggers identified and documented
- [ ] Product-specific constraints documented
- [ ] Ready to write complete PRD

</success_criteria>
