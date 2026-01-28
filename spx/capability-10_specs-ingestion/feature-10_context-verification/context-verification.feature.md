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

- **1 → 2**: Unit tests prove our path matching logic (e.g., "capability-NN_slug/\*.prd.md"), but Level 2 verifies we can actually find and read documents from a real spx/ directory with the SPX structure

## Tests

- [Integration: Verify complete hierarchy](tests/hierarchy-verification.integration.test.ts)
- [Integration: Abort on missing document](tests/abort-missing.integration.test.ts)

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
