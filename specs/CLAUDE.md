# specs/ Directory

Spec-driven development workspace for this product.

## Directory Structure

```
specs/
├── CLAUDE.md                      # This file
├── [product-name].prd.md          # Product-wide PRD (optional)
├── decisions/                      # Product-wide ADRs
│   └── adr-NNN_{slug}.md
└── work/
    ├── backlog/                    # Future work items
    ├── doing/                      # Active work items
    └── done/                       # Completed work items (permanent)
```

## Work Item Hierarchy

Work items follow a three-level hierarchy:

```
capability-NN_{slug}/              # E2E scenarios, tests→tests/e2e/
├── {slug}.capability.md
├── {topic}.prd.md                  # Optional PRD catalyst
├── decisions/                      # Capability-scoped ADRs
├── tests/
└── feature-NN_{slug}/              # Integration scenarios, tests→tests/integration/
    ├── {slug}.feature.md
    ├── {topic}.trd.md              # Optional TRD catalyst
    ├── decisions/                  # Feature-scoped ADRs
    ├── tests/
    └── story-NN_{slug}/            # Atomic implementation, tests→tests/unit/
        ├── {slug}.story.md
        └── tests/
```

## Key Concepts

**Requirements (vision documents):**

- **PRD**: Product requirements (user value, measurable outcomes) → spawns capabilities
- **TRD**: Technical requirements (system architecture, validation) → spawns features
- Rule: PRD OR TRD at same scope, never both

**Decisions (constraints):**

- **ADR**: Architectural decision records at product/capability/feature scope
- Stories inherit decisions from parent feature/capability

**Work item status** (determined by tests/ directory):

- **OPEN**: tests/ missing or empty
- **IN_PROGRESS**: Has test files, no DONE.md
- **DONE**: DONE.md exists

**BSP numbering** (10-99):

- Lower number = must complete first
- Used for dependency ordering at all levels

**Test graduation:**

- Story tests → tests/unit/
- Feature tests → tests/integration/
- Capability tests → tests/e2e/

## Templates and Structure

For templates and detailed structure information:

- Invoke `/bootstrapping-documents` skill
- Or read: `plugins/specs/skills/bootstrapping-documents/`

Templates include:

- Product/technical requirements (PRD/TRD)
- Architectural decisions (ADR)
- Work items (capability/feature/story)
- Completion evidence (DONE.md)
- Structure definition (structure.yaml)

## Creating Requirements

**For product requirements:**

- Invoke `/writing-product-requirements` skill
- Creates PRDs with user value and measurable outcomes

**For technical requirements:**

- Invoke `/writing-technical-requirements` skill
- Creates TRDs with architecture and validation strategy

## Session Management

Claude Code session handoffs are stored in:

```
.spx/sessions/
├── TODO_*.md      # Available for /pickup
└── DOING_*.md     # Currently claimed
```

Use `/handoff` to create session context for continuation.
Use `/pickup` to load and claim a handoff.

---

**For complete workflow methodology**, reference the SPX framework documentation (when available) or consult the `/bootstrapping-documents` skill for structure details.
