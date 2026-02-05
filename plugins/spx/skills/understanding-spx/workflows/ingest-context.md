<required_reading>
Read these reference files NOW:

1. `references/document-types.md` - Required documents at each level
2. `references/abort-protocol.md` - Error handling and remediation

</required_reading>

<process>
Execute these phases IN ORDER. ABORT immediately if any required document is missing.

## Phase 0: Locate Work Item

**Goal**: Find exact path to work item in `spx/`

**Input formats accepted**:

- Full path: `10-cli.capability/20-commands.feature/30-build.story`
- Story name only: `30-build.story`
- Natural language: "build command story"

**Actions**:

```bash
# Search for work item (try multiple patterns)
Glob: "spx/**/*-{slug}.story/"
Glob: "spx/**/*-{slug}.feature/"
Glob: "spx/**/*-{slug}.capability/"

# Extract work item details
# Determine level: CAPABILITY, FEATURE, or STORY
# Parse BSP number and slug from path
```

**Abort if**: Work item path not found

**Output**:

```
Work Item Located
  Path: spx/10-cli.capability/20-commands.feature/30-build.story
  Level: STORY
  BSP: 30
  Slug: build
```

---

## Phase 1: Product-Wide Context

**Goal**: Load product-wide constraints and guidance

**Required Documents**:

- `spx/CLAUDE.md` MUST EXIST
- `spx/NN-*.adr.md` May not exist for new projects

**Actions**:

```bash
# Read project guide
Read: spx/CLAUDE.md

# Find and read all product ADRs
Glob: "spx/*-*.adr.md"
# For each ADR found:
Read: [ADR path]
```

**Abort if**: `spx/CLAUDE.md` missing

**Strict Mode Check**: None (product ADRs are truly optional)

**Output**:

```
Product Context Loaded
  - spx/CLAUDE.md
  - Product ADRs: 3
    - [Type Safety](21-type-safety.adr.md)
    - [Testing Strategy](37-testing-strategy.adr.md)
    - [CLI Framework](54-cli-framework.adr.md)
```

---

## Phase 2: Capability Context

**Goal**: Load capability specification, requirements, and decisions

**Required Documents**:

- `{capability-path}/{slug}.capability.md` MUST EXIST
- `{capability-path}/*.prd.md` OPTIONAL (read if present)
- `{capability-path}/*-*.adr.md` May not exist

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

# Read capability ADRs (interleaved)
Glob: "{capability-path}/*-*.adr.md"
# For each ADR found:
Read: [ADR path]
```

**Abort if**:

- Capability spec missing (but PRD exists -> offer to create spec from PRD)
- Multiple capability specs found (ambiguous)

**Offer to create spec if**:

- Capability spec missing BUT PRD exists at this level
- Prompt: "Found PRD but no capability.md - create spec from it?"

**Enumerate Features (ALWAYS)**:

```bash
# CRITICAL: Use ls or find for directories, NOT Glob
ls -d {capability-path}/*.feature/ 2>/dev/null || echo "No features found"
```

**Output**:

```
Capability Context Loaded: 10-cli.capability
  - cli.capability.md
  - command-architecture.prd.md (optional, found)
  - Capability ADRs: 2
    - [Commander Pattern](21-commander-pattern.adr.md)
    - [Config Loading](37-config-loading.adr.md)
  - Features: 2
    - 20-commands.feature/
    - 37-plugins.feature/
```

---

## Phase 3: Feature Context

**Goal**: Load feature specification and decisions

**Skip if**: Working on capability-level (no parent feature)

**Required Documents**:

- `{feature-path}/{slug}.feature.md` MUST EXIST
- `{feature-path}/*-*.adr.md` May not exist

**Actions**:

```bash
# Read feature spec
Glob: "{feature-path}/*.feature.md"
# Verify exactly one file found
Read: {feature-path}/{slug}.feature.md

# Read feature ADRs (interleaved)
Glob: "{feature-path}/*-*.adr.md"
# For each ADR found:
Read: [ADR path]
```

**Abort if**:

- Feature spec missing
- Multiple feature specs found (ambiguous)

**Note**: Technical details belong in feature.md. No separate TRD documents.

**Enumerate Stories (ALWAYS, even if not descending to story level)**:

```bash
# CRITICAL: Use ls to find directories, NOT Glob
# Glob matches files, not directories - story directories won't appear!
ls -d {feature-path}/*.story/ 2>/dev/null || echo "No stories found"

# Alternative: find command
find {feature-path} -maxdepth 1 -type d -name "*.story"
```

**If no stories found**: Feature is NOT ready for implementation. Recommend decomposition using `/decomposing-feature-to-stories`.

**Output**:

```
Feature Context Loaded: 20-commands.feature
  - commands.feature.md
  - Feature ADRs: 1
    - [Subcommand Structure](21-subcommand-structure.adr.md)
  - Stories: 3
    - 21-parse-flags.story/
    - 37-execute-command.story/
    - 54-output-format.story/
```

---

## Phase 4: Story Context

**Goal**: Load story specification

**Skip if**: Working on feature-level or capability-level (no story)

**Required Documents**:

- `{story-path}/{slug}.story.md` MUST EXIST
- `{story-path}/tests/` directory Should exist for in-progress work

**Actions**:

```bash
# Read story spec
Glob: "{story-path}/*.story.md"
# Verify exactly one file found
Read: {story-path}/{slug}.story.md

# Check for tests/ directory to understand coverage
```

**Abort if**:

- Story spec missing
- Multiple story specs found (ambiguous)

**Output**:

```
Story Context Loaded: 30-build.story
  - build.story.md
  - tests/ directory: exists
```

---

## Phase 5: Context Summary

**Goal**: Confirm complete context loaded and provide actionable summary

**Generate**:

```markdown
# CONTEXT INGESTION COMPLETE

## Work Item

- **Level**: Story
- **Path**: spx/10-cli.capability/20-commands.feature/30-build.story
- **BSP**: 30 (story), 20 (feature), 10 (capability)

## Documents Loaded

### Product Level

- **Guide**: spx/CLAUDE.md
- **ADRs**: 3 documents
  - [Type Safety](21-type-safety.adr.md)
  - [Testing Strategy](37-testing-strategy.adr.md)
  - [CLI Framework](54-cli-framework.adr.md)

### Capability Level: cli

- **Spec**: 10-cli.capability/cli.capability.md
- **PRD**: 10-cli.capability/command-architecture.prd.md
- **ADRs**: 2 documents
  - [Commander Pattern](21-commander-pattern.adr.md)
  - [Config Loading](37-config-loading.adr.md)

### Feature Level: commands

- **Spec**: 20-commands.feature/commands.feature.md
- **ADRs**: 1 document
  - [Subcommand Structure](21-subcommand-structure.adr.md)

### Story Level: build

- **Spec**: 30-build.story/build.story.md
- **Tests**: tests/ directory exists

## Constraints Summary

**Total ADRs Applicable**: 6

- Product-wide: 3
- Capability-scoped: 2
- Feature-scoped: 1
- Story-specific: 0 (stories inherit parent ADRs)

**Test Location**: `spx/.../30-build.story/tests/`

## Hierarchy Chain
```

Product (spx-claude)
└── Capability 10: cli
└── Feature 20: commands
└── Story 30: build ← YOU ARE HERE

```
## Ready for Implementation

All required documents verified and read
Complete hierarchical context loaded
All architectural constraints understood

You may now proceed with implementation.
```

</process>

<success_criteria>
Workflow is complete when:

- [ ] All phases executed in order (0 through 5)
- [ ] Every required document located and read
- [ ] All ADRs at all levels read and listed
- [ ] Context summary generated with complete document list
- [ ] Clear indication that implementation may proceed
- [ ] No ABORT conditions triggered (or appropriate error shown)

</success_criteria>
