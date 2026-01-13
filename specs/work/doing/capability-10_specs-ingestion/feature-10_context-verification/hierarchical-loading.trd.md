# TRD: Hierarchical Context Loading

> **Purpose**: Documents the hierarchical document verification approach for spec-driven development.

## Problem Statement

### Technical Problem

When implementation skills (coding, reviewing, architecting) try to implement spec-driven work items, they encounter incomplete context
because there's no mechanism to verify all required specification documents exist, which blocks confident implementation that honors all requirements and architectural decisions.

### Current Pain

- **Symptom**: Skills start implementing without reading PRDs, TRDs, or ADRs
- **Root Cause**: No enforced verification protocol that checks document existence before implementation
- **Impact**: Implementations miss requirements, violate architectural decisions, require multiple iterations
- **Validation Challenge**: Hard to test that "all documents were considered" without explicit verification step

## Project-Specific Constraints

| Constraint                                      | Impact on Implementation                               | Impact on Testing                         |
| ----------------------------------------------- | ------------------------------------------------------ | ----------------------------------------- |
| SPX framework hierarchy is fixed (3 levels)     | Must verify exactly 3 levels: capability→feature→story | Test data must follow SPX structure       |
| Documents follow strict naming patterns         | Can use pattern matching for discovery                 | Test fixtures must use correct patterns   |
| Some documents optional (ADRs), others required | Strict mode for PRD/TRD, permissive for ADRs           | Test both strict and permissive scenarios |

## Solution Design

### Technical Solution

```
Implement hierarchical document verification that enables implementation skills to verify complete specification hierarchy
through file system traversal and pattern matching, resulting in abort-on-missing or proceed-with-context decisions.
```

### Technical Architecture

#### Components

| Component          | Responsibility                                         |
| ------------------ | ------------------------------------------------------ |
| PathResolver       | Resolves work item identifier to specs/ directory path |
| DocumentVerifier   | Checks if required documents exist at each level       |
| HierarchyTraverser | Walks from story → feature → capability → product      |
| AbortProtocol      | Generates structured error messages with remediation   |
| ContextSummarizer  | Produces summary of all loaded documents               |

#### Data Flow

```
[Work Item ID] → PathResolver → HierarchyTraverser → DocumentVerifier (each level) →
→ [If missing] → AbortProtocol → [Error + Remediation]
→ [If complete] → ContextSummarizer → [Document List]
```

#### Key Interfaces

| Interface                     | Input           | Output                          | Notes                      |
| ----------------------------- | --------------- | ------------------------------- | -------------------------- |
| verifyHierarchy(workItemId)   | "story-10_auth" | VerificationResult              | Main entry point           |
| DocumentVerifier.checkLevel() | Level, Path     | DocumentList or MissingDocError | Verifies one level         |
| AbortProtocol.generate()      | MissingDoc info | Structured error message        | Includes remediation steps |

## Validation Strategy

### Guarantees Required

| #  | Guarantee                                               | Level | Rationale                                       |
| -- | ------------------------------------------------------- | ----- | ----------------------------------------------- |
| G1 | Path pattern matching correctly identifies documents    | 1     | Pure string matching logic, no external deps    |
| G2 | File system verification finds existing documents       | 2     | Requires real file system with SPX structure    |
| G3 | Abort protocol generates actionable error messages      | 1     | Pure string formatting, can test with fixtures  |
| G4 | Complete workflow integrates with implementation skills | 3     | Requires real skill invocation and coordination |

### BDD Scenarios

**Scenario: Complete hierarchy with all required documents [G2]**

- **Given** Complete spec hierarchy exists (capability→feature→story with PRD, TRD, ADRs)
- **When** Hierarchical verification is invoked for story-10_login
- **Then** All documents are found and loaded, context summary lists all files, verification status is "complete"

**Scenario: Missing required document at feature level [G2, G3]**

- **Given** Incomplete hierarchy with missing TRD at feature level
- **When** Hierarchical verification is invoked for story-10_login
- **When** Verification traverses to feature level
- **Then** TRD not found, abort protocol activated, error message specifies "feature-10_auth/auth.trd.md" missing, remediation steps provided

**Scenario: Path matching handles BSP numbers correctly [G1]**

- **Given** Work item identifier "story-42_complex-auth"
- **When** Path resolver processes identifier
- **Then** Correctly resolves to specs/work/doing/.../story-42_complex-auth/complex-auth.story.md

**Scenario: Integration with coding-python skill [G4]**

- **Given** Complete spec hierarchy exists
- **When** User invokes /coding-python for story
- **Then** coding-python invokes understanding-specs first, verification completes successfully, coding-python proceeds with implementation

## Test Infrastructure

### Level 2: Test Harnesses

| Dependency    | Harness Type      | Setup Command                   | Reset Command    |
| ------------- | ----------------- | ------------------------------- | ---------------- |
| File system   | Temp directory    | `mkdtemp` with SPX structure    | `rm -rf tempdir` |
| Test fixtures | Fixture generator | `createSpecsHierarchy(options)` | N/A              |

### Level 3: Credentials and Test Accounts

No external credentials required - all verification uses local file system.

### Infrastructure Gaps

None - all required infrastructure is available.

## Reference Test Implementations

### Level 1 Example (Unit)

```typescript
// Pure logic test - path pattern matching
describe("PathResolver", () => {
  it("GIVEN story identifier WHEN resolving path THEN correct pattern generated", () => {
    // Given
    const storyId = "story-10_auth";

    // When
    const pattern = PathResolver.generatePattern(storyId);

    // Then
    expect(pattern).toBe("**/story-10_auth/*.story.md");
  });
});
```

### Level 2 Example (Integration)

```typescript
// Integration test - requires real file system
describe("DocumentVerifier", () => {
  it("GIVEN real specs directory WHEN verifying hierarchy THEN finds all documents", async () => {
    // Given: Create temp directory with complete hierarchy
    const tempDir = await createTempSpecsHierarchy({
      capability: "capability-10_test",
      feature: "feature-10_auth",
      story: "story-10_login",
      includePRD: true,
      includeTRD: true,
    });

    // When: Verify hierarchy
    const result = await DocumentVerifier.verifyHierarchy(tempDir, "story-10_login");

    // Then: All documents found
    expect(result.status).toBe("complete");
    expect(result.documents).toHaveLength(6); // CLAUDE.md, capability, PRD, feature, TRD, story
  });
});
```

### Level 3 Example (E2E)

```typescript
// E2E test - requires skill invocation
describe("Context Verification E2E", () => {
  it("GIVEN complete hierarchy WHEN coding-python invoked THEN context verified first", async () => {
    // Given: Real spec hierarchy
    const specPath = "specs/work/doing/capability-10_test/feature-10_auth/story-10_login";
    await createCompleteSpecHierarchy(specPath);

    // When: Invoke coding-python skill
    const session = await createClaudeSession();
    const result = await session.invokeSkill("coding-python", {
      workItem: "story-10_login",
    });

    // Then: Verification happened first
    expect(result.logs).toContain("Invoking understanding-specs");
    expect(result.logs).toContain("Context verification: complete");
    expect(result.implementationStarted).toBe(true);
  });
});
```

## Dependencies

### Work Item Dependencies

| Dependency              | Location                            | Status   |
| ----------------------- | ----------------------------------- | -------- |
| managing-specs skill    | plugins/specs/skills/managing-specs | Complete |
| SPX framework structure | specs/CLAUDE.md                     | Complete |

### Runtime Dependencies

| Dependency  | Version Constraint | Purpose                  |
| ----------- | ------------------ | ------------------------ |
| Node.js     | >=18.0.0           | File system operations   |
| fs/promises | Built-in           | Async file system access |

### Test Infrastructure Dependencies

| Dependency     | Required For      | Provided By               |
| -------------- | ----------------- | ------------------------- |
| Temp directory | Level 2 harnesses | os.tmpdir() + mkdtemp     |
| Test fixtures  | All test levels   | Custom fixture generators |

## Pre-Mortem Analysis

### Technical Risks

| Risk                                            | Likelihood | Impact | Mitigation                                          |
| ----------------------------------------------- | ---------- | ------ | --------------------------------------------------- |
| File system traversal slow on large hierarchies | Low        | Low    | Optimization: early exit on first missing document  |
| Pattern matching fails on edge cases            | Medium     | High   | Comprehensive unit tests for all BSP number formats |

### Test Infrastructure Risks

| Risk                                    | Likelihood | Impact | Mitigation                                      |
| --------------------------------------- | ---------- | ------ | ----------------------------------------------- |
| Temp directory cleanup fails            | Low        | Low    | Use try/finally blocks, document cleanup        |
| Test fixtures drift from real structure | Medium     | Medium | Generate fixtures from managing-specs templates |

### Integration Risks

| Risk                                               | Likelihood | Impact | Mitigation                                     |
| -------------------------------------------------- | ---------- | ------ | ---------------------------------------------- |
| Skills fail to invoke understanding-specs properly | Medium     | High   | Document integration pattern, provide examples |
| Error messages unclear to users                    | Medium     | Medium | User testing of error messages, iterate        |

## Readiness Criteria

This TRD is ready for decomposition when:

1. **Problem Statement**: Identifies root cause (no verification mechanism) not just symptoms
2. **Validation Strategy**: All guarantees mapped to test levels with BDD scenarios
3. **Test Infrastructure**: All dependencies documented with no gaps
4. **Dependencies**: All work items and runtime requirements specified
5. **Pre-Mortem**: Risks identified with mitigation strategies
