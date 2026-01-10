# Capability: Specs Ingestion and Context Verification

## Success Metric

**Quantitative Target:**

- **Baseline**: 0% of implementation skills verify complete specification context before starting
- **Target**: 100% of implementation skills (coding, reviewing) enforce complete context loading
- **Measurement**: Number of skills with context_loading sections / Total implementation skills

## Testing Strategy

> Capabilities require **all three levels** to prove end-to-end value delivery.
> See testing skill for level definitions.

### Level Assignment

| Component                     | Level | Justification                                            |
| ----------------------------- | ----- | -------------------------------------------------------- |
| Document path parsing         | 1     | Pure function logic, tests filename pattern matching     |
| File system verification      | 2     | Requires real file system to verify document existence   |
| Complete workflow integration | 3     | Verifies understanding-specs works with coding/reviewing |

### Escalation Rationale

- **1 → 2**: Unit tests prove path parsing logic is correct, but Level 2 verifies we can actually find and read documents from the real specs/ hierarchy
- **2 → 3**: Integration tests prove we can read documents, but Level 3 verifies the complete workflow where coding-python invokes understanding-specs and proceeds with implementation

## Capability E2E Tests (Level 3)

These tests verify the **complete user journey** delivers value.

### E2E1: Complete implementation workflow with context verification

```typescript
// tests/e2e/specs-ingestion.e2e.test.ts
describe("Capability: Specs Ingestion", () => {
  it("GIVEN story spec hierarchy WHEN coding skill invoked THEN context verified before implementation", async () => {
    // Given: Complete spec hierarchy exists (capability → feature → story)
    //        PRD at capability level
    //        TRD at feature level
    //        ADRs at multiple levels
    // When: User invokes /coding-python for the story
    //       Skill automatically invokes /understanding-specs first
    // Then: understanding-specs verifies all documents exist
    //       Context summary provided with all loaded documents
    //       coding-python proceeds with full context
    //       Implementation references correct ADRs and TRD
  });
});
```

### E2E2: Abort workflow when required document missing

```typescript
describe("Capability: Specs Ingestion - Abort Protocol", () => {
  it("GIVEN incomplete spec hierarchy WHEN coding skill invoked THEN abort with clear error", async () => {
    // Given: Incomplete hierarchy (missing TRD at feature level)
    // When: User invokes /coding-python for story
    //       Skill invokes /understanding-specs
    // Then: understanding-specs aborts immediately
    //       Clear error message indicates missing TRD
    //       Remediation steps provided
    //       coding-python does NOT proceed with partial context
  });
});
```

## System Integration

This capability integrates with the specs plugin and all language-specific plugins (TypeScript, Python). It ensures that any implementation skill has complete specification context before starting work, preventing partial implementations due to missing documentation.

**Integration points:**

- Specs plugin provides understanding-specs skill and templates
- Language plugins (Python/TypeScript) invoke understanding-specs in their coding/reviewing skills
- Handoff system preserves context claims to prevent parallel agent conflicts

## Completion Criteria

- [ ] All Level 1 tests pass (via feature/story completion)
- [ ] All Level 2 tests pass (via feature completion)
- [ ] All Level 3 E2E tests pass
- [ ] Success metric achieved (100% implementation skills have context loading)
