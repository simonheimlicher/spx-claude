---
name: understanding-specs
description: Read all specs for a story, feature, or capability before starting work. Use when starting implementation to load requirements and context.
allowed-tools: Read, Glob, Grep
---

<essential_principles>
**COMPLETE CONTEXT OR ABORT. NO EXCEPTIONS.**

- Every work item requires complete documentation hierarchy
- Missing documents = ABORT immediately with clear error
- "Optional" PRD/TRD must exist in strict mode (fail if missing)
- Read order: Product → Capability → Feature → Story (top-down)
- All ADRs at all levels must be read and understood
- This skill runs BEFORE any implementation work begins

</essential_principles>

<objective>
Verify and load complete hierarchical context for a work item by reading all specification documents from product level down to the target work item. Fails fast with actionable errors when required documents are missing. Ensures implementation skills have complete context including all constraints (ADRs), requirements (PRD/TRD), and specifications before starting work.
</objective>

<quick_start>
Invoke with work item identifier:

```bash
# By path
/understanding-specs capability-10_cli/feature-20_commands/story-30_build

# By story name (skill locates it)
/understanding-specs story-30_build

# By natural language
/understanding-specs I'm working on the build command story
```

The skill will:

1. Locate the work item in `specs/work/`
2. Read all documents from product level down to target
3. Fail immediately if any required document is missing
4. Output structured context summary when complete

</quick_start>

<intake>
Provide the work item you're working on:

- **Work item path**: `capability-NN_slug/feature-NN_slug/story-NN_slug`
- **Work item name**: Story name (e.g., "story-30_build")
- **Natural language**: "I'm implementing the build command story"

The skill will locate and verify all documents in the hierarchy.
</intake>

<routing>
All inputs route to: `workflows/ingest-context.md`

This skill has a single workflow that handles all context ingestion.
</routing>

<reference_index>
Detailed patterns and error handling:

| File                           | Purpose                                 |
| ------------------------------ | --------------------------------------- |
| `references/abort-protocol.md` | Error messages and remediation guidance |
| `references/document-types.md` | Required documents at each level        |

</reference_index>

<workflows_index>

| Workflow                      | Purpose                            |
| ----------------------------- | ---------------------------------- |
| `workflows/ingest-context.md` | Hierarchical document verification |

</workflows_index>

<success_criteria>
Context ingestion succeeds when:

- [ ] Work item located in `specs/work/`
- [ ] Product guide (`specs/CLAUDE.md`) read
- [ ] All product ADRs read
- [ ] Capability spec exists and read
- [ ] Capability PRD/TRD exists and read (strict mode)
- [ ] All capability ADRs read
- [ ] Feature spec exists and read (if working on feature/story)
- [ ] Feature TRD exists and read (if working on feature/story, strict mode)
- [ ] All feature ADRs read
- [ ] Story spec exists and read (if working on story)
- [ ] Structured context summary generated with document count and ADR list

</success_criteria>
