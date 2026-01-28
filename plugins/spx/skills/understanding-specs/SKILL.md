---
name: understanding-specs
description: Read all specs for a story, feature, or capability including PRDs and ADRs. Use when starting implementation, checking progress, or asked to "read the spec".
allowed-tools: Read, Glob, Grep
---

<accessing_skill_files>
When this skill is invoked, Claude Code provides the base directory in the loading message:

```
Base directory for this skill: {skill_dir}
```

Use this path to access skill files:

- References: `{skill_dir}/references/`
- Workflows: `{skill_dir}/workflows/`

**IMPORTANT**: Do NOT search the project directory for skill files.
</accessing_skill_files>

<essential_principles>
**COMPLETE CONTEXT OR ABORT. NO EXCEPTIONS.**

- Every work item requires its spec file (`.capability.md`, `.feature.md`, `.story.md`)
- Missing spec file = ABORT immediately with clear error
- PRD is optional enrichment - read if present, offer to create spec from it if spec is missing
- Read order: Product → Capability → Feature → Story (top-down)
- All ADRs at all levels must be read and understood
- This skill runs BEFORE any implementation work begins

</essential_principles>

<objective>
Verify and load complete hierarchical context for a work item by reading all specification documents from product level down to the target work item. Fails fast with actionable errors when required documents are missing. Ensures implementation skills have complete context including all constraints (ADRs), requirements (PRD), and specifications before starting work.
</objective>

<quick_start>
Invoke with **FULL work item path**:

```bash
# ALWAYS use full path (REQUIRED)
/understanding-specs 10-cli.capability/20-commands.feature/30-build.story
```

**🚨 NEVER use bare story/feature numbers** - BSP numbers are sibling-unique, not globally unique:

```bash
# ❌ WRONG: Ambiguous - which story-30?
/understanding-specs 30-build.story

# ✅ CORRECT: Unambiguous full path
/understanding-specs 10-cli.capability/20-commands.feature/30-build.story
```

The skill will:

1. Locate the work item in `spx/`
2. Read all documents from product level down to target
3. Fail immediately if any required document is missing
4. Output structured context summary when complete

</quick_start>

<intake>
Provide the **FULL work item path** you're working on:

- **Full path** (REQUIRED): `NN-slug.capability/NN-slug.feature/NN-slug.story`

**🚨 BSP numbers are sibling-unique, not globally unique.**

| ❌ WRONG (Ambiguous)   | ✅ CORRECT (Unambiguous)                               |
| ---------------------- | ------------------------------------------------------ |
| "30-build.story"       | "10-cli.capability/20-commands.feature/30-build.story" |
| "implement feature-20" | "implement 10-cli.capability/20-commands.feature"      |

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

- [ ] Work item located in `spx/`
- [ ] Product guide (`spx/CLAUDE.md`) read
- [ ] All product ADRs read
- [ ] Capability spec exists and read
- [ ] Capability PRD read if present (optional enrichment)
- [ ] All capability ADRs read
- [ ] Feature spec exists and read (if working on feature/story)
- [ ] All feature ADRs read
- [ ] Story spec exists and read (if working on story)
- [ ] Structured context summary generated with document count and ADR list

</success_criteria>
