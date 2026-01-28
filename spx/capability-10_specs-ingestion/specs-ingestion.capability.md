# Capability: Specs Ingestion and Context Verification

## Problem

Solo developers using Claude Code with spec-driven development face incomplete implementations because Claude skills start implementing without reading all required specification documents.

**User story:**

```
As a solo developer using spec-driven development, I am frustrated by incomplete implementations
because Claude skills start implementing without reading all required specification documents,
which prevents me from getting correct implementations that follow all requirements and decisions.
```

**Current pain:**

- Claude coding skills start implementing stories without reading PRDs or ADRs
- Implementations miss requirements or violate architectural decisions
- Users waste time debugging incomplete implementations
- Manual verification that all specs were considered

## Solution

Hierarchical context ingestion that enables Claude implementation skills to verify complete specification context through automatic skill invocation, resulting in correct implementations that honor all requirements and decisions.

**User journey:**

- **Before**: User manually reminds Claude to read specific documents, hoping nothing is missed
- **After**: User trusts Claude to always have complete context, focuses on reviewing outcomes

## Success Metric

- **Baseline**: 0% of implementation skills verify complete specification context before starting
- **Target**: 100% of implementation skills (coding, reviewing) enforce complete context loading
- **Measurement**: Number of skills with context_loading sections / Total implementation skills

## Scope

**Included:**

- understanding-specs skill in specs plugin
- Context loading integration in TypeScript skills (testing, coding, reviewing, architecting)
- Context loading integration in Python skills (testing, coding, reviewing, architecting)
- Abort protocol with clear error messages and remediation
- Hierarchical document verification (product → capability → feature → story)

**Excluded:**

| Excluded                                 | Rationale                                               |
| ---------------------------------------- | ------------------------------------------------------- |
| Automatic spec creation                  | Separate concern; user creates specs via managing-specs |
| Spec validation (content quality)        | Verification checks existence, not content quality      |
| Cross-capability dependency verification | Defer until usage patterns emerge                       |

## User Experience Principles

1. **Fail fast**: Abort immediately on missing document, don't proceed with partial context
2. **Clear errors**: Error messages show exact missing file path and remediation steps
3. **Idempotent**: Multiple invocations safe, doesn't interfere with workflow
4. **Transparent**: Context summary shows exactly what was loaded

## Acceptance Scenarios

```gherkin
Scenario: Complete context available - proceed with implementation
  Given complete spec hierarchy exists (capability → feature → story)
  And ADRs exist at relevant levels
  When user invokes /coding-python for the story
  Then understanding-specs verifies all documents exist
  And context summary shows all loaded documents
  And coding-python proceeds with implementation

Scenario: Required document missing - abort with clear error
  Given incomplete spec hierarchy (missing feature.md)
  When user invokes /coding-python for story
  Then understanding-specs detects missing document
  And clear error message indicates which document is missing
  And coding-python does NOT proceed with partial context

Scenario: Multiple skills integrate context verification
  Given story requires both architecture review and implementation
  When user invokes /architecting-python then /coding-python
  Then understanding-specs loads context for each (idempotent)
  And implementation follows architectural decisions
```

## Testing Strategy

> Capabilities require **all three levels** to prove end-to-end value delivery.

| Component                     | Level | Justification                                            |
| ----------------------------- | ----- | -------------------------------------------------------- |
| Document path parsing         | 1     | Pure function logic, tests filename pattern matching     |
| File system verification      | 2     | Requires real file system to verify document existence   |
| Complete workflow integration | 3     | Verifies understanding-specs works with coding/reviewing |

## Tests

- [E2E: Complete implementation workflow](tests/specs-ingestion.e2e.test.ts)
- [E2E: Abort workflow when document missing](tests/abort-protocol.e2e.test.ts)

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
