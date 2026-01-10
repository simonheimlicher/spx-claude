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

| Excluded Capability              | Rationale                                                  |
| -------------------------------- | ---------------------------------------------------------- |
| [e.g., Multi-user collaboration] | Defer until single-user validates value; different product |
| [e.g., Cloud sync]               | Local-first reduces complexity; defer until v2             |

**If no exclusions: write "None identified"**
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

<identify_adr_triggers>
**Analyze technical approach and mark ADR triggers:**

For each component or integration mentioned:

- Does it involve technology choice? → ⚠️ ADR
- Does it involve architectural pattern? → ⚠️ ADR
- Does it involve data model design? → ⚠️ ADR

**Example:**

- "Storage format for handoffs" → ⚠️ ADR: Markdown+YAML vs JSON vs SQLite
- "Context capture strategy" → ⚠️ ADR: Full history vs summary vs user selection

**Document all ADR triggers in "Open Decisions" table.**
</identify_adr_triggers>

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
- [ ] ADR triggers identified and documented
- [ ] Product-specific constraints documented
- [ ] Ready to write complete PRD

</success_criteria>
