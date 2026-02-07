<required_reading>
Read these reference files NOW:

1. `references/document-types.md` - Required documents at each level
2. `references/abort-protocol.md` - Error handling and remediation

</required_reading>

<process>
Execute these phases IN ORDER. ABORT immediately if any required document is missing.

## Phase 0: Locate Work Item

**Goal**: Find exact path to work item in `specs/work/`

**Input formats accepted**:

- Full path: `capability-10_cli/feature-20_commands/story-30_build`
- Story name only: `story-30_build`
- Natural language: "build command story"

**Actions**:

```bash
# Search for work item (try multiple patterns)
Glob: "specs/work/**/story-{NN}_{slug}/"
Glob: "specs/work/**/feature-{NN}_{slug}/"
Glob: "specs/work/**/capability-{NN}_{slug}/"

# Extract work item details
# Determine level: CAPABILITY, FEATURE, or STORY
# Parse BSP number and slug from path
```

**Abort if**: Work item path not found

**Output**:

```
✓ Work Item Located
  Path: specs/work/doing/capability-10_cli/feature-20_commands/story-30_build
  Level: STORY
  BSP: 30
  Slug: build
```

---

## Phase 1: Product-Wide Context

**Goal**: Load product-wide constraints and guidance

**Required Documents**:

- `specs/CLAUDE.md` ✅ MUST EXIST
- `specs/decisions/adr-*.md` ⚠️ May not exist for new projects

**Actions**:

```bash
# Read project guide
Read: specs/CLAUDE.md

# Find and read all product ADRs and PDRs
Glob: "specs/decisions/adr-*.md"
Glob: "specs/decisions/pdr-*.md"
# For each decision record found:
Read: [decision record path]
```

**Abort if**: `specs/CLAUDE.md` missing

**Strict Mode Check**: None (product ADRs/PDRs are truly optional)

**Output**:

```
✓ Product Context Loaded
  - specs/CLAUDE.md
  - Product ADRs: 3
    • [Type Safety](../decisions/adr-21_type-safety.md)
    • [Testing Strategy](../decisions/adr-37_testing-strategy.md)
    • [CLI Framework](../decisions/adr-54_cli-framework.md)
  - Product PDRs: 1
    • [Simulation Lifecycle](../decisions/pdr-10_simulation-lifecycle.md)
```

---

## Phase 2: Capability Context

**Goal**: Load capability specification, requirements, and decisions

**Required Documents**:

- `{capability-path}/{slug}.capability.md` ✅ MUST EXIST
- `{capability-path}/*.prd.md` ⚠️ OPTIONAL (read if present)
- `{capability-path}/decisions/adr-*.md` ⚠️ May not exist
- `{capability-path}/decisions/pdr-*.md` ⚠️ May not exist

**Actions**:

```bash
# Read capability spec
Glob: "{capability-path}/*.capability.md"
# Verify exactly one file found
Read: {capability-path}/{slug}.capability.md

# Read PRD if present (optional enrichment)
Glob: "{capability-path}/*.prd.md"
# If found:
Read: [PRD path]
# If not found:
# Continue without PRD (it's optional)

# Read capability ADRs and PDRs
Glob: "{capability-path}/decisions/adr-*.md"
Glob: "{capability-path}/decisions/pdr-*.md"
# For each decision record found:
Read: [decision record path]
```

**Abort if**:

- Capability spec missing (but PRD exists → offer to create spec from PRD)
- Multiple capability specs found (ambiguous)

**Offer to create spec if**:

- Capability spec missing BUT PRD exists at this level
- Prompt: "Found PRD but no capability.md - create spec from it?"

**Output**:

```
✓ Capability Context Loaded: capability-10_cli
  - cli.capability.md
  - command-architecture.prd.md (optional, found)
  - Capability ADRs: 2
    • [Commander Pattern](decisions/adr-21_commander-pattern.md)
    • [Config Loading](decisions/adr-37_config-loading.md)
  - Capability PDRs: 0
```

---

## Phase 3: Feature Context

**Goal**: Load feature specification, technical requirements, and decisions

**Skip if**: Working on capability-level (no parent feature)

**Required Documents**:

- `{feature-path}/{slug}.feature.md` ✅ MUST EXIST
- `{feature-path}/*.trd.md` ⚠️ OPTIONAL (read if present)
- `{feature-path}/decisions/adr-*.md` ⚠️ May not exist
- `{feature-path}/decisions/pdr-*.md` ⚠️ May not exist

**Actions**:

```bash
# Read feature spec
Glob: "{feature-path}/*.feature.md"
# Verify exactly one file found
Read: {feature-path}/{slug}.feature.md

# Read TRD if present (optional enrichment)
Glob: "{feature-path}/*.trd.md"
# If found:
Read: [TRD path]
# If not found:
# Continue without TRD (it's optional)

# Read feature ADRs and PDRs
Glob: "{feature-path}/decisions/adr-*.md"
Glob: "{feature-path}/decisions/pdr-*.md"
# For each decision record found:
Read: [decision record path]
```

**Abort if**:

- Feature spec missing (but TRD exists → offer to create spec from TRD)
- Multiple feature specs found (ambiguous)

**Offer to create spec if**:

- Feature spec missing BUT TRD exists at this level
- Prompt: "Found TRD but no feature.md - create spec from it?"

**Output**:

```
✓ Feature Context Loaded: feature-20_commands
  - commands.feature.md
  - command-framework.trd.md (optional, found)
  - Feature ADRs: 1
    • [Subcommand Structure](decisions/adr-21_subcommand-structure.md)
  - Feature PDRs: 0
```

---

## Phase 4: Story Context

**Goal**: Load story specification

**Skip if**: Working on feature-level or capability-level (no story)

**Required Documents**:

- `{story-path}/{slug}.story.md` ✅ MUST EXIST
- `{story-path}/tests/` directory ⚠️ Should exist for IN_PROGRESS status

**Actions**:

```bash
# Read story spec
Glob: "{story-path}/*.story.md"
# Verify exactly one file found
Read: {story-path}/{slug}.story.md

# Check for tests directory to determine status
# Status values: OPEN, IN_PROGRESS, DONE
```

**Abort if**:

- Story spec missing
- Multiple story specs found (ambiguous)

**Warning if**:

- Working on DONE story (status = DONE)

**Output**:

```
✓ Story Context Loaded: story-30_build
  - build.story.md
  - Status: IN_PROGRESS
```

---

## Phase 5: Context Summary

**Goal**: Confirm complete context loaded and provide actionable summary

**Generate**:

```markdown
# CONTEXT INGESTION COMPLETE

## Work Item

- **Level**: Story
- **Path**: specs/work/doing/capability-10_cli/feature-20_commands/story-30_build
- **Status**: IN_PROGRESS
- **BSP**: 30 (story), 20 (feature), 10 (capability)

## Documents Loaded

### Product Level

- **Guide**: specs/CLAUDE.md
- **ADRs**: 3 documents
  - [Type Safety](../decisions/adr-21_type-safety.md)
  - [Testing Strategy](../decisions/adr-37_testing-strategy.md)
  - [CLI Framework](../decisions/adr-54_cli-framework.md)
- **PDRs**: 1 document
  - [Simulation Lifecycle](../decisions/pdr-10_simulation-lifecycle.md)

### Capability Level: cli

- **Spec**: capability-10_cli/cli.capability.md
- **PRD**: capability-10_cli/command-architecture.prd.md
- **ADRs**: 2 documents
  - [Commander Pattern](decisions/adr-21_commander-pattern.md)
  - [Config Loading](decisions/adr-37_config-loading.md)
- **PDRs**: 0 documents

### Feature Level: commands

- **Spec**: feature-20_commands/commands.feature.md
- **TRD**: feature-20_commands/command-framework.trd.md
- **ADRs**: 1 document
  - [Subcommand Structure](decisions/adr-21_subcommand-structure.md)
- **PDRs**: 0 documents

### Story Level: build

- **Spec**: story-30_build/build.story.md
- **Tests**: tests/ directory exists

## Constraints Summary

**Total ADRs Applicable**: 6

- Product-wide: 3
- Capability-scoped: 2
- Feature-scoped: 1
- Story-specific: 0 (stories inherit parent ADRs)

**Total PDRs Applicable**: 1

- Product-wide: 1
- Capability-scoped: 0
- Feature-scoped: 0
- Story-specific: 0 (stories inherit parent PDRs)

**Test Location**: `specs/work/doing/.../story-30_build/tests/` (co-located, permanent)

## Hierarchy Chain
```

Product (spx-claude)
└── Capability 10: cli
└── Feature 20: commands
└── Story 30: build ← YOU ARE HERE

```
## Ready for Implementation

✅ All required documents verified and read
✅ Complete hierarchical context loaded
✅ All architectural constraints (ADRs) and product decisions (PDRs) understood

You may now proceed with implementation.
```

</process>

<success_criteria>
Workflow is complete when:

- [ ] All phases executed in order (0 through 5)
- [ ] Every required document located and read
- [ ] All ADRs and PDRs at all levels read and listed
- [ ] Context summary generated with complete document list
- [ ] Clear indication that implementation may proceed
- [ ] No ABORT conditions triggered (or appropriate error shown)

</success_criteria>
