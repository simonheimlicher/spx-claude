<required_reading>
Read `references/trd-template-guide.md` for complete template structure.
</required_reading>

<process>
## Determine TRD Location

Based on conversation and project structure:

**Capability-level TRD:**
- File: `specs/capabilities/[slug]/[name].trd.md`
- For: New capability or capability-wide technical change

**Feature-level TRD:**
- File: `specs/capabilities/[cap-slug]/features/[feature-slug]/[name].trd.md`
- For: Feature-specific technical change

**Verify location with user if unclear.**

## Fill Required Sections

Use information gathered in Phases 1-3 to complete all sections:

### Title
```markdown
# TRD: [Name of Technical Change]
```

### Purpose Block
Copy from template (already correct).

### Required Sections Table
Copy from template (already correct).

### Testing Methodology
Copy from template (already correct).

### Problem Statement

Fill with confirmed information from Phase 1:

- **Technical Problem**: Use confirmed root cause
- **Current Pain**: Symptom, root cause, impact
- **Validation Challenge**: What makes this hard to test

### Project-Specific Constraints

If constraints discussed:
- Fill table with constraint, implementation impact, testing impact
- Otherwise: Write "None identified"

### Solution Design

Fill with approved solution from Phase 1:

**Technical Solution**: Summary statement

**Technical Architecture**:

- **Components table**: List components and responsibilities
- **Data Flow**: Describe flow through system
- **Key Interfaces**: Define component boundaries

### Validation Strategy

Fill with content from Phase 2:

**Guarantees Required**: Copy guarantees table with IDs, levels, rationales

**BDD Scenarios**: Copy all scenarios with guarantee references

### Test Infrastructure

Fill with content from Phase 3:

**Level 2: Test Harnesses**: Copy harness table

**Level 3: Credentials**: Copy credentials table

**Infrastructure Gaps**: Copy gaps table (or empty if no gaps)

### Reference Test Implementations

Keep template examples or adapt to project's primary language if requested.

### Dependencies

**Work Item Dependencies**: List any prerequisite work items mentioned

**Runtime Dependencies**: List tools, libraries, services required

**Test Infrastructure Dependencies**: Cross-reference Docker, credential tools, etc.

### Pre-Mortem Analysis

Fill three risk categories:

**Technical Risks**: Risks to implementation
**Test Infrastructure Risks**: Risks to testing capability
**Integration Risks**: Risks from external dependencies

### Readiness Criteria

Copy from template (already correct).

## Write Complete File

Write the complete TRD file to the determined location.

**Do NOT use placeholders.** Every section must be filled with real content or marked "None identified".

## Verify Against Template

Check every section exists:

- [ ] Title matches format
- [ ] Problem Statement complete
- [ ] Solution Design has components/data flow/interfaces
- [ ] Validation Strategy has guarantees table and scenarios
- [ ] Test Infrastructure has L2/L3 tables and gaps
- [ ] Dependencies are categorized
- [ ] Pre-Mortem has all three risk categories

## Self-Check Readiness Criteria

Execute the checklist from the template's Readiness Criteria section:

**1. Problem Statement:**
- [ ] Identifies root cause (confirmed by user)
- [ ] Not just symptoms

**2. Validation Strategy:**
- [ ] Every guarantee has unique ID
- [ ] Every guarantee assigned to one test level
- [ ] Level assignment rationale is coherent
- [ ] Every BDD scenario references a guarantee
- [ ] Every guarantee has ≥1 scenario
- [ ] Scenarios use strict Given/When/Then format

**3. Test Infrastructure:**
- [ ] All L2 dependencies have harnesses with setup/reset commands
- [ ] All L3 dependencies have credential sources
- [ ] No unresolved gaps remain (or explicitly in Gaps table)

**4. Dependencies:**
- [ ] Work item dependencies linked
- [ ] Runtime dependencies have version constraints
- [ ] Test infrastructure dependencies cross-referenced

**5. Pre-Mortem:**
- [ ] ≥1 risk per category (technical, test infra, integration)
- [ ] Each risk has mitigation strategy

## Prepare Delivery Summary

Create a summary for the user:

```markdown
## TRD Creation Complete

**Location**: `[filepath]`

**Status**: [Complete | Incomplete - see Infrastructure Gaps]

**Key Decisions:**
- [Decision 1]
- [Decision 2]
- [Decision 3]

**Guarantees**: [N] guarantees across [N] test levels
- Level 1 (Unit): [N] guarantees
- Level 2 (Integration): [N] guarantees
- Level 3 (E2E): [N] guarantees

**Infrastructure Gaps** (if any):
- [Gap 1 blocking: X]
- [Gap 2 blocking: Y]

**Next Actions**:
[If complete]: Ready for decomposition into work items
[If incomplete]: Resolve infrastructure gaps listed above before implementation

**Readiness Assessment:**
[Report pass/fail for each criterion from self-check]
```

</process>

<success_criteria>
TRD writing complete when:

- [ ] Complete TRD file written to correct location
- [ ] All sections filled (no placeholders)
- [ ] Readiness Criteria self-check performed
- [ ] Status marked (complete or incomplete)
- [ ] Delivery summary prepared for user
- [ ] Next actions clearly stated

</success_criteria>
