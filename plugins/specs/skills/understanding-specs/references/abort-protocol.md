# Abort Protocol

Error handling and remediation guidance when required documents are missing.

## When to Abort

ABORT immediately when ANY of these conditions are met:

1. **Work item not found**: Cannot locate work item in `specs/work/`
2. **Product guide missing**: `specs/CLAUDE.md` does not exist
3. **Spec file missing**: Work item spec file (`.capability.md`, `.feature.md`, `.story.md`) missing
4. **PRD missing**: Capability-level PRD missing (strict mode)
5. **TRD missing**: Feature-level TRD missing (strict mode)
6. **Multiple specs found**: Ambiguous - multiple spec files at same level

## Abort Message Format

```markdown
❌ CONTEXT INGESTION FAILED

**Phase**: [Phase name and number]
**Missing Document**: [Document type]
**Expected Path**: [Exact path where file should exist]
**Reason**: [Why this document is required]

**Remediation**:
[Numbered steps to create the missing document]

**Cannot proceed with implementation until this document exists.**
```

## Abort Scenarios

### 1. Work Item Not Found

**Trigger**: Phase 0 - Cannot locate work item

```markdown
❌ CONTEXT INGESTION FAILED

**Phase**: Phase 0 - Locate Work Item
**Missing Document**: Work item directory
**Expected Pattern**: specs/work/**/story-30_build/
**Reason\*\*: Cannot proceed without identifying the work item location

**Remediation**:

1. Verify the work item exists: `ls -R specs/work/`
2. Check work item naming follows pattern: {level}-{BSP}\_{slug}
3. If work item doesn't exist, create it first
4. Re-run `/understanding-specs` with correct work item identifier

**Cannot proceed with implementation until work item exists.**
```

### 2. Product Guide Missing

**Trigger**: Phase 1 - `specs/CLAUDE.md` not found

```markdown
❌ CONTEXT INGESTION FAILED

**Phase**: Phase 1 - Product-Wide Context
**Missing Document**: Product guide
**Expected Path**: specs/CLAUDE.md
**Reason**: Product guide is required for all projects (defines structure and navigation)

**Remediation**:

1. Invoke `/managing-specs` skill to create specs/ structure
2. Or manually create `specs/CLAUDE.md` with project structure documentation
3. Re-run `/understanding-specs` to verify

**Cannot proceed with implementation until product guide exists.**
```

### 3. Capability Spec Missing

**Trigger**: Phase 2 - Capability spec file not found

```markdown
❌ CONTEXT INGESTION FAILED

**Phase**: Phase 2 - Capability Context
**Missing Document**: Capability specification
**Expected Path**: specs/work/doing/capability-10_cli/cli.capability.md
**Reason**: Capability spec defines E2E scenario and is required

**Remediation**:

1. Create capability spec file at expected path
2. Use template from `/managing-specs` skill: `templates/work-items/capability-name.capability.md`
3. Fill in functional requirements and acceptance criteria
4. Re-run `/understanding-specs` to verify

**Cannot proceed with implementation until capability spec exists.**
```

### 4. Capability PRD Missing (Strict Mode)

**Trigger**: Phase 2 - PRD not found at capability level

```markdown
❌ CONTEXT INGESTION FAILED

**Phase**: Phase 2 - Capability Context
**Missing Document**: Product Requirements Document (PRD)
**Expected Path**: specs/work/doing/capability-10_cli/{topic}.prd.md
**Reason**: Capability triggered by PRD (strict mode enforced)

**Remediation**:

1. Invoke `/writing-product-requirements` skill
2. Create PRD at capability level documenting user value and measurable outcomes
3. PRD should explain WHY this capability exists and WHAT value it provides
4. Re-run `/understanding-specs` to verify

**Cannot proceed with implementation until PRD exists.**
```

### 5. Feature Spec Missing

**Trigger**: Phase 3 - Feature spec file not found

```markdown
❌ CONTEXT INGESTION FAILED

**Phase**: Phase 3 - Feature Context
**Missing Document**: Feature specification
**Expected Path**: specs/work/doing/.../feature-20_commands/commands.feature.md
**Reason**: Feature spec defines integration scenario and is required

**Remediation**:

1. Create feature spec file at expected path
2. Use template from `/managing-specs` skill: `templates/work-items/feature-name.feature.md`
3. Fill in functional requirements and acceptance criteria
4. Re-run `/understanding-specs` to verify

**Cannot proceed with implementation until feature spec exists.**
```

### 6. Feature TRD Missing (Strict Mode)

**Trigger**: Phase 3 - TRD not found at feature level

```markdown
❌ CONTEXT INGESTION FAILED

**Phase**: Phase 3 - Feature Context
**Missing Document**: Technical Requirements Document (TRD)
**Expected Path**: specs/work/doing/.../feature-20_commands/{topic}.trd.md
**Reason**: Feature triggered by TRD (strict mode enforced)

**Remediation**:

1. Invoke `/writing-technical-requirements` skill
2. Create TRD at feature level documenting architecture and validation strategy
3. TRD must include:
   - System design
   - Test level assignments (Unit/Integration/E2E)
   - BDD scenarios
   - Test infrastructure requirements
4. Re-run `/understanding-specs` to verify

**Cannot proceed with implementation until TRD exists.**
```

### 7. Story Spec Missing

**Trigger**: Phase 4 - Story spec file not found

```markdown
❌ CONTEXT INGESTION FAILED

**Phase**: Phase 4 - Story Context
**Missing Document**: Story specification
**Expected Path**: specs/work/doing/.../story-30_build/build.story.md
**Reason**: Story spec defines atomic implementation unit and is required

**Remediation**:

1. Create story spec file at expected path
2. Use template from `/managing-specs` skill: `templates/work-items/story-name.story.md`
3. Fill in functional requirements and acceptance criteria
4. Re-run `/understanding-specs` to verify

**Cannot proceed with implementation until story spec exists.**
```

### 8. Multiple Spec Files Found (Ambiguous)

**Trigger**: Any phase - Multiple spec files at same level

```markdown
❌ CONTEXT INGESTION FAILED

**Phase**: Phase 2 - Capability Context
**Problem**: Multiple spec files found
**Found Files**:

- specs/work/doing/capability-10_cli/cli.capability.md
- specs/work/doing/capability-10_cli/old-cli.capability.md
  **Reason**: Ambiguous - cannot determine which spec file is current

**Remediation**:

1. Remove or rename old/duplicate spec files
2. Keep only ONE spec file per work item: {slug}.{level}.md
3. Archive old versions outside specs/ if needed
4. Re-run `/understanding-specs` to verify

**Cannot proceed with implementation until ambiguity is resolved.**
```

## Warning Scenarios (Don't Abort)

These scenarios generate warnings but don't abort:

### Working on DONE Story

```markdown
⚠️ WARNING: Working on completed story

**Story**: story-30_build
**Status**: DONE (DONE.md exists)
**Risk**: This story is marked complete. Changes may require regression testing.

**Recommendations**:

1. If this is new work, create a new story instead
2. If fixing a bug, create a new bug-fix story
3. If truly modifying this story, understand you're changing completed work

Proceeding with context ingestion...
```

### No Product ADRs

```markdown
✓ Product Context Loaded

- specs/CLAUDE.md
- Product ADRs: 0 (none found - acceptable for new projects)
```

### No Capability/Feature ADRs

```markdown
✓ Capability Context Loaded: capability-10_cli

- cli.capability.md
- command-architecture.prd.md
- Capability ADRs: 0 (none found - acceptable)
```

## Success Path (No Errors)

When all documents exist:

```markdown
# CONTEXT INGESTION COMPLETE

✅ All required documents verified and read
✅ Complete hierarchical context loaded
✅ All architectural constraints understood

You may now proceed with implementation.
```

## Strict Mode Toggle

**Current default**: Strict mode ENABLED

To disable strict mode (allow missing PRD/TRD):

- User must explicitly request it
- Not recommended - PRD/TRD document the catalyst for work

**Strict mode OFF** would skip PRD/TRD ABORT and show warnings instead.
