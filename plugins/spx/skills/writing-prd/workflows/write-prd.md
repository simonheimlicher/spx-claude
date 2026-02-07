<required_reading>
Read `references/prd-template-guide.md` for complete template structure.
</required_reading>

<process>
<determine_location>
Based on conversation and project structure:

**Product-level PRD:**

- File: `spx/{product-name}.prd.md`
- For: New product capability at highest level

**Capability-level PRD:**

- File: `spx/NN-{slug}.capability/{topic}.prd.md`
- For: Product change within existing capability (optional catalyst)

**Feature-level PRD:**

- File: `spx/.../NN-{slug}.feature/{topic}.prd.md`
- For: Feature-specific product requirements (rare)

**Verify location with user if unclear.**
</determine_location>

<fill_sections>
Use information gathered in Phases 1-3 to complete all sections:

**Title:** `# PRD: [Market able Increment Name]`

**Product Vision**: Fill with confirmed information from Phase 1

**Expected Outcome**: Copy measurable outcome and Evidence of Success from Phase 2

**Acceptance Tests**: Copy Gherkin scenarios and E2E test code from Phase 2

**Scope Definition**: Copy included/excluded capabilities from Phase 3

**Product Approach**: Fill UX principles, technical approach, ADR/PDR triggers from Phase 3

**Open Decisions**: Document questions and ADR/PDR triggers from Phase 3

**Dependencies, Pre-Mortem, Delivery Strategy**: Fill from conversation context

**Do NOT use placeholders.** Every section must have real content or "None identified".
</fill_sections>

<write_file>
Write the complete PRD file to the determined location.
</write_file>

<verify_template>
Check every section exists per template:

- [ ] Product Vision complete
- [ ] Expected Outcome has measurable targets
- [ ] Acceptance Tests have Gherkin + E2E code
- [ ] Scope Definition has included/excluded + rationale
- [ ] Product Approach identifies ADR/PDR triggers
- [ ] Open Decisions documents unresolved questions
- [ ] Dependencies categorized
- [ ] Pre-Mortem has ≥4 assumptions

</verify_template>

<self_check_readiness>
Execute Readiness Criteria from template:

**1. Product Vision:**

- [ ] User problem articulated
- [ ] User assumptions documented
- [ ] Customer journey shows transformation

**2. Expected Outcome:**

- [ ] Uses quantified format (X% improvement in Y)
- [ ] Evidence of Success has Current → Target
- [ ] Metrics tied to user value

**3. Acceptance Tests:**

- [ ] E2E test in actual code
- [ ] ≥3 Gherkin scenarios
- [ ] Scenarios observable from user perspective

**4-10:** [Check remaining criteria from template]
</self_check_readiness>

<prepare_summary>

```markdown
## PRD Creation Complete

**Location**: `[filepath]`

**Status**: [Ready | Incomplete - see Open Decisions]

**Product Definition:**

- User Problem: [1-sentence summary]
- Measurable Outcome: [X% improvement in Y]
- User Capabilities: [N] capabilities (UC1, UC2, UC3...)
- Scope: [Key included items] | Excluded: [Key exclusions]

**Next Actions:**

[If ready]: Create ADRs/PDRs for [N] decisions, then decompose into work items
[If incomplete]: Resolve [N] open decisions before implementation

**Readiness Assessment:**

[Report pass/fail for each criterion]
```

</prepare_summary>

</process>

<success_criteria>
PRD writing complete when:

- [ ] Complete PRD file written to correct location
- [ ] All sections filled (no placeholders)
- [ ] Readiness Criteria self-check performed
- [ ] Status marked (ready or incomplete)
- [ ] Delivery summary prepared for user
- [ ] Next actions clearly stated

</success_criteria>
