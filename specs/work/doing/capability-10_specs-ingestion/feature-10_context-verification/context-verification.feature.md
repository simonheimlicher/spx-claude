# Feature: Context Verification

## Observable Outcome

Implementation skills (coding, reviewing, architecting) verify complete specification hierarchy exists before starting work, aborting with clear error messages if any required document is missing.

## Testing Strategy

> Features require **Level 1 + Level 2** to prove the feature works with real tools.
> See testing skill for level definitions.

### Level Assignment

| Component                | Level | Justification                                       |
| ------------------------ | ----- | --------------------------------------------------- |
| Path pattern matching    | 1     | Pure function, can verify with fixtures             |
| File system verification | 2     | Needs real file system to verify document existence |

### Escalation Rationale

- **1 → 2**: Unit tests prove our path matching logic (e.g., "capability-NN_slug/\*.prd.md"), but Level 2 verifies we can actually find and read documents from a real specs/ directory with the SPX structure

## Feature Integration Tests (Level 2)

These tests verify that **real tools work together** as expected.

### FI1: Verify complete hierarchy with real file system

```typescript
// tests/integration/context-verification.integration.test.ts
describe("Feature: Context Verification", () => {
  it("GIVEN complete spec hierarchy WHEN understanding-specs invoked THEN all documents loaded", async () => {
    // Given: Real specs/ directory with complete hierarchy
    const tempDir = await createTempSpecsHierarchy({
      capability: "capability-10_test",
      feature: "feature-10_auth",
      story: "story-10_login",
      includePRD: true,
      includeTRD: true,
      includeADRs: ["adr-001_oauth.md"],
    });

    // When: Invoke context verification
    const result = await verifyContext(tempDir, "story-10_login");

    // Then: All documents verified
    expect(result.status).toBe("complete");
    expect(result.documents).toContain("CLAUDE.md");
    expect(result.documents).toContain("test.capability.md");
    expect(result.documents).toContain("test.prd.md");
    expect(result.documents).toContain("auth.feature.md");
    expect(result.documents).toContain("auth.trd.md");
    expect(result.documents).toContain("login.story.md");
  });
});
```

### FI2: Abort on missing required document

```typescript
describe("Feature: Context Verification - Error Handling", () => {
  it("GIVEN missing TRD WHEN understanding-specs invoked THEN abort with clear error", async () => {
    // Given: Incomplete hierarchy (missing TRD)
    const tempDir = await createTempSpecsHierarchy({
      capability: "capability-10_test",
      feature: "feature-10_auth",
      story: "story-10_login",
      includePRD: true,
      includeTRD: false, // Missing!
    });

    // When: Invoke context verification
    const result = await verifyContext(tempDir, "story-10_login");

    // Then: Aborted with clear error
    expect(result.status).toBe("abort");
    expect(result.error).toContain("TRD missing");
    expect(result.error).toContain("auth.trd.md");
    expect(result.remediation).toBeDefined();
  });
});
```

## Capability Contribution

This feature provides the core hierarchical verification logic that enables the specs ingestion capability. It ensures implementation skills can trust that all required documents are present before starting work, eliminating the risk of partial context.

**Integration with other capability features:**

- Provides verification for Python integration story
- Provides verification for TypeScript integration story (if added)
- Shared by all implementation skills across language plugins

## Completion Criteria

- [ ] All Level 1 tests pass (via story completion)
- [ ] All Level 2 integration tests pass
- [ ] Escalation rationale documented
- [ ] understanding-specs skill implements this feature
