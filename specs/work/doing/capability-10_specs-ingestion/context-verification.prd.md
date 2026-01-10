# PRD: Context Verification for Spec-Driven Development

> **Purpose**: Ensures all implementation skills have complete specification context before starting work, eliminating partial implementations due to missing documentation.

## Product Vision

### User Problem

**Who** are we building for? Solo developers using Claude Code with the SPX framework for spec-driven development.

**What** problem do they face?

```
As a solo developer using spec-driven development, I am frustrated by incomplete implementations
because Claude skills start implementing without reading all required specification documents,
which prevents me from getting correct implementations that follow all requirements and decisions.
```

### Current Customer Pain

- **Symptom**: Claude coding skills start implementing stories without reading PRDs, TRDs, or ADRs, leading to implementations that miss requirements or violate architectural decisions
- **Root Cause**: No enforcement mechanism ensures skills load complete specification hierarchy before implementation
- **Customer Impact**: Wasted time debugging incomplete implementations, manual verification that all specs were considered
- **Business Impact**: Reduces trust in autonomous Claude development, requires more human oversight

### Customer Solution

```
Implement hierarchical context ingestion that enables Claude implementation skills to verify complete specification context
through automatic skill invocation, resulting in correct implementations that honor all requirements and decisions.
```

### Customer Journey Context

- **Before**: User manually reminds Claude to read specific documents (PRDs, TRDs, ADRs), hoping nothing is missed
- **During**: User adopts understanding-specs integration, Claude automatically verifies context before implementing
- **After**: User trusts Claude to always have complete context, focuses on reviewing outcomes not verifying inputs

### User Assumptions

| Assumption Category | Specific Assumption                                | Impact on Product Design                                   |
| ------------------- | -------------------------------------------------- | ---------------------------------------------------------- |
| User Context        | Uses SPX framework with specs/ directory structure | Must follow SPX hierarchy (capability→feature→story)       |
| User Goals          | Values correctness over speed                      | Fail-fast on missing docs preferred over proceeding        |
| User Workflow       | Works on multiple features/stories simultaneously  | Context verification must not interfere with parallel work |

## Expected Outcome

### Measurable Outcome

```
Users will invoke implementation skills (coding, reviewing) leading to 100% of implementations having complete specification context
and 0% implementations starting with missing required documents, proven by context verification logs and implementation quality
metrics at delivery.
```

### Evidence of Success

| Metric                            | Current | Target | Improvement                                |
| --------------------------------- | ------- | ------ | ------------------------------------------ |
| Implementations with full context | 30%     | 100%   | All implementations verify context first   |
| Missing document errors           | Common  | Zero   | No implementation starts with missing docs |
| Implementation correctness        | 60%     | 95%    | Implementations honor all ADRs and TRDs    |

## Acceptance Tests

### Complete User Journey Test

```typescript
// tests/e2e/context-verification.e2e.test.ts
describe("Feature: Context Verification", () => {
  test("user invokes coding skill and context is verified before implementation", async () => {
    // Given: Complete spec hierarchy exists
    const specPath = "specs/work/doing/capability-10_test/feature-10_auth/story-10_login";
    await createCompleteSpecHierarchy(specPath);

    // When: User invokes /coding-python for the story
    const result = await invokeCodingSkill("story-10_login");

    // Then: Context was verified before implementation started
    expect(result.contextVerification).toEqual({
      status: "complete",
      documentsLoaded: ["specs/CLAUDE.md", "capability-10_test.capability.md", "capability-10_test/auth-system.prd.md", "feature-10_auth.feature.md", "feature-10_auth/authentication.trd.md", "story-10_login.story.md"],
      adrsLoaded: ["adr-001_oauth-strategy.md", "adr-002_session-storage.md"],
    });

    // And: Implementation proceeded with full context
    expect(result.implementationStarted).toBe(true);
  });
});
```

### User Scenarios (Gherkin Format)

```gherkin
Feature: Hierarchical Context Verification

  Scenario: Complete context available - proceed with implementation
    Given complete spec hierarchy exists (capability → feature → story)
    And PRD exists at capability level
    And TRD exists at feature level
    And ADRs exist at relevant levels
    When user invokes /coding-python for the story
    Then understanding-specs verifies all documents exist
    And context summary shows all loaded documents
    And coding-python proceeds with implementation
    And implementation references correct specs and decisions

  Scenario: Required document missing - abort with clear error
    Given incomplete spec hierarchy (missing TRD at feature level)
    When user invokes /coding-python for story
    Then understanding-specs detects missing TRD
    And clear error message indicates which document is missing
    And error message shows expected file path
    And error message provides remediation steps
    And coding-python does NOT proceed with partial context

  Scenario: Multiple skills integrate context verification
    Given story requires both architecture review and implementation
    When user invokes /architecting-python
    Then understanding-specs loads all ADRs in hierarchy
    And architect can reference parent decisions
    When user then invokes /coding-python
    Then understanding-specs loads context again (idempotent)
    And implementation follows architectural decisions
```

## Scope Definition

### What's Included

- ✅ understanding-specs skill in specs plugin
- ✅ Context loading integration in TypeScript skills (testing, coding, reviewing, architecting)
- ✅ Context loading integration in Python skills (testing, coding, reviewing, architecting)
- ✅ Abort protocol with clear error messages and remediation
- ✅ Hierarchical document verification (product → capability → feature → story)

### What's Explicitly Excluded

| Excluded Capability                      | Rationale                                               |
| ---------------------------------------- | ------------------------------------------------------- |
| Automatic spec creation                  | Separate concern; user creates specs via managing-specs |
| Spec validation (content quality)        | Verification checks existence, not content quality      |
| Cross-capability dependency verification | Defer until usage patterns emerge                       |

### Scope Boundaries Rationale

This release focuses on ensuring complete context exists before implementation. We're explicitly avoiding content quality validation until we validate that existence checking alone significantly improves implementation quality.

## Product Approach

### Interaction Model

- **Interface Type**: Skill invocation within Claude Code
- **Invocation Pattern**: Automatic invocation by implementation skills before starting work
- **User Mental Model**: "Like compilation that fails fast if imports are missing, but for specification documents"

### User Experience Principles

1. **Fail fast**: Abort immediately on missing document, don't proceed with partial context
2. **Clear errors**: Error messages show exact missing file path and remediation steps
3. **Idempotent**: Multiple invocations safe, doesn't interfere with workflow
4. **Transparent**: Context summary shows exactly what was loaded

### High-Level Technical Approach

**Data Model:**

- Hierarchical document verification following SPX structure
- Each level has required documents (capability.md, PRD, feature.md, TRD, story.md)

**Key Capabilities:**

- File existence verification via file system checks
- Path pattern matching for document discovery
- Abort protocol with structured error messages

**Integration Points:**

- Invoked by implementation skills via Skill tool
- Reads specs/ directory structure
- Returns structured context summary or abort error

## Success Criteria

### User Outcomes

| Outcome                                         | Success Indicator                                                          |
| ----------------------------------------------- | -------------------------------------------------------------------------- |
| Users trust Claude has complete context         | No manual reminders to "read the TRD" after integration                    |
| Users catch missing specs before implementation | Error messages help user realize spec is missing, not after implementation |
| Users save time on implementation quality       | Fewer iterations fixing implementations that violated requirements         |

### Quality Attributes

| Attribute       | Target                                       | Measurement Approach                          |
| --------------- | -------------------------------------------- | --------------------------------------------- |
| **Correctness** | 100% detection of missing required documents | Test with incomplete hierarchies              |
| **Clarity**     | Error messages lead directly to remediation  | User can fix issue from error message alone   |
| **Speed**       | Context verification completes in <5 seconds | Time from invocation to complete/abort        |
| **Safety**      | Never proceed with partial context           | Verify abort on any missing required document |

### Definition of "Done"

1. All acceptance criteria scenarios pass
2. Complete user journey E2E test passes
3. All TypeScript skills integrate context loading
4. All Python skills integrate context loading
5. Abort protocol tested with all missing document scenarios

## Open Decisions

### Questions Requiring User Input

None identified - approach validated during exploration.

### Decisions Triggering ADRs

None required - uses existing SPX framework structure and patterns.

## Dependencies

### Work Item Dependencies

| Dependency              | Status   | Rationale                                   |
| ----------------------- | -------- | ------------------------------------------- |
| SPX framework structure | Complete | Required for hierarchical verification      |
| managing-specs skill    | Complete | Provides templates and structure definition |

### Technical Dependencies

| Dependency  | Version Constraint | Purpose                |
| ----------- | ------------------ | ---------------------- |
| Claude Code | >=0.4.0            | Skill system support   |
| Node.js     | >=18.0.0           | File system operations |

## Pre-Mortem Analysis

### Assumption: Users will accept fail-fast approach

- **Likelihood**: High - user explicitly requested this behavior
- **Impact**: Medium - if wrong, users frustrated by strict enforcement
- **Mitigation**: Clear error messages with remediation make failures actionable

### Assumption: Hierarchical verification is sufficient

- **Likelihood**: High - covers all required SPX documents
- **Impact**: Low - missing edge cases can be added incrementally
- **Mitigation**: Comprehensive abort protocol testing covers known scenarios

### Assumption: Performance acceptable for large hierarchies

- **Likelihood**: High - reading ~10-20 files is fast
- **Impact**: Low - worst case is slow verification, not incorrect results
- **Mitigation**: If slow, add caching in future iteration

## Delivery Strategy

Single release delivery - all components delivered together for coherent functionality.
