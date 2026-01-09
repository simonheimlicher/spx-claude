# TRD Template Guide

## Overview

The TRD template is located at: `specs/templates/requirements/technical-change.trd.md`

**Always read the template before writing a TRD.** It contains:
- Complete section structure
- Field descriptions and examples
- Readiness criteria checklist

## Template Structure

### Header Block

```markdown
# TRD: [Name of Technical Change]

> **Purpose**: Documents the explored solution after discovery and serves as an authoritative blueprint for implementation.
>
> - Written AFTER user and agent explore the solution space together
> - Captures the agreed-upon approach to solving a technical problem
> - Authoritative: Changes to solution approach require user approval
> - Guides implementation: Spawns work items (features/stories) with binding acceptance criteria
> - No size constraints, no state tracking (OPEN/IN PROGRESS/DONE)
> - Can exist at: capability level or feature level
```

**Do not modify this block.** It defines the TRD's purpose and role.

### Required Sections Table

Lists all sections that must be present and complete.

**Do not modify this table.** It serves as a completeness checklist.

### Testing Methodology

Condensed summary of three-tier testing with skill references.

**Do not modify this section.** It's consistent across all TRDs.

## Sections to Fill

### Problem Statement

**Required subsections:**
- Technical Problem
- Current Pain (Symptom, Root Cause, Impact, Validation Challenge)

**Source**: Phase 1 (understand-problem workflow)

**Best practices:**
- State root cause, not just symptoms
- Explain WHY the limitation exists
- Connect to what's being blocked

### Project-Specific Constraints

**Format**: Table with three columns:
- Constraint
- Impact on Implementation
- Impact on Testing

**Or**: Write "None identified" if no special constraints

**Source**: Phase 1, conversation context

**Examples**:
- API rate limits
- Cross-service ID matching requirements
- Security restrictions on credential storage
- Legacy system integration constraints

### Solution Design

**Required subsections:**

1. **Technical Solution**: High-level summary statement
2. **Technical Architecture**:
   - **Components**: Table of components and responsibilities
   - **Data Flow**: Diagram showing flow through system
   - **Key Interfaces**: Table defining component boundaries

**Source**: Phase 1 (solution approach confirmation)

**Best practices:**
- Don't prescribe implementation details
- Define responsibilities, not internals
- Key interfaces become Level 2 integration points

### Validation Strategy

**Required subsections:**

1. **Guarantees Required**: Table with columns:
   - # (G1, G2, G3...)
   - Guarantee
   - Level (1, 2, or 3)
   - Rationale

2. **BDD Scenarios**: For each guarantee, ≥1 scenario:
   ```
   **Scenario: [Name] [G#]**

   - **Given** [Precondition]
   - **When** [Action]
   - **Then** [Outcome]
   ```

**Source**: Phase 2 (design-validation workflow)

**Best practices:**
- Every guarantee has unique ID
- Level assignment has clear rationale
- Every scenario references a guarantee
- Strict Given/When/Then format

### Test Infrastructure

**Required subsections:**

1. **Level 2: Test Harnesses**: Table with columns:
   - Dependency
   - Harness Type
   - Setup Command
   - Reset Command

2. **Level 3: Credentials and Test Accounts**: Table with columns:
   - Credential
   - Environment Variable
   - Source
   - Notes

3. **Infrastructure Gaps**: Table with columns:
   - Gap
   - Blocking

**Source**: Phase 3 (discover-infrastructure workflow)

**Best practices:**
- Specific commands, not placeholders
- Exact credential sources
- Gaps explicitly listed if unknown
- No "TBD" or "TODO"

### Reference Test Implementations

**Contains**: Example test code at each level (L1, L2, L3)

**From template**: Keep Python examples OR adapt to project language

**Purpose**: Implementation guidance, not exhaustive coverage

**Note**: Template includes language adaptation guidance for TypeScript projects

### Dependencies

**Required subsections:**

1. **Work Item Dependencies**: Table of prerequisite work items
2. **Runtime Dependencies**: Table with version constraints
3. **Test Infrastructure Dependencies**: Table cross-referencing harnesses/tools

**Source**: Conversation context, solution design

**Best practices:**
- Link to actual spec paths for work items
- Include version constraints for runtime deps
- Cross-reference test infrastructure documented above

### Pre-Mortem Analysis

**Required subsections:**

1. **Technical Risks**: Table with Likelihood, Impact, Mitigation
2. **Test Infrastructure Risks**: Same format
3. **Integration Risks**: Same format

**Source**: Phase 4, solution analysis

**Best practices:**
- At least one risk per category
- Every risk has mitigation strategy
- Be specific about likelihood and impact

### Readiness Criteria

**Format**: Numbered checklist with detailed sub-criteria

**From template**: Do not modify, use for self-check

**Purpose**: Agent verifies TRD completeness before delivery

## File Location Rules

### Capability-Level TRD

**Path**: `specs/capabilities/[slug]/[name].trd.md`

**When**: New capability or capability-wide technical change

**Example**: `specs/capabilities/user-authentication/oauth-integration.trd.md`

### Feature-Level TRD

**Path**: `specs/capabilities/[cap-slug]/features/[feature-slug]/[name].trd.md`

**When**: Feature-specific technical change

**Example**: `specs/capabilities/user-auth/features/google-oauth/token-refresh.trd.md`

## TRD Status

### Complete

All sections filled, no Infrastructure Gaps.

**Ready for**: Decomposition into work items

### Incomplete

All sections filled, BUT Infrastructure Gaps table has entries.

**Ready for**: Partial work (non-blocked items)

**Blocks**: Items listed in "Blocking" column of Infrastructure Gaps table

**User action**: Resolve gaps before full implementation

## Common Mistakes

### ❌ Placeholder Text

Bad:
```
**Technical Solution**: TBD
```

Fix: Actually fill the section, or mark as Infrastructure Gap if unknown.

### ❌ Vague Infrastructure

Bad:
```
| PostgreSQL | Docker | Run it | Clean it |
```

Fix:
```
| PostgreSQL | Docker container | `docker-compose -f test.yml up -d postgres` | `docker-compose exec postgres psql -c "TRUNCATE..."` |
```

### ❌ Missing Guarantee IDs

Bad:
```
| Guarantee | Level | Rationale |
```

Fix:
```
| #  | Guarantee | Level | Rationale |
| G1 | ...       | 1     | ...       |
```

### ❌ Scenarios Without Guarantee References

Bad:
```
**Scenario: Price calculation works**
```

Fix:
```
**Scenario: Price calculation with standard discount [G1]**
```

### ❌ Implementation Details in Guarantees

Bad:
```
G1 | PriceCalculator uses BigDecimal | 1 | ...
```

Fix:
```
G1 | Price calculation handles edge cases | 1 | ...
```

## Template Reading

**Before writing ANY TRD, read the complete template:**

```bash
Read specs/templates/requirements/technical-change.trd.md
```

This ensures:
- You use current structure
- You see all examples
- You understand Readiness Criteria
- You copy exact formatting

## Template Updates

If the template is updated (new sections, changed structure):

**The skill should adapt automatically** by reading the current template before each TRD creation.

Do not hardcode template structure in workflows.
