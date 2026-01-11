---
name: managing-specs
description: Set up specs directory with templates for PRDs, TRDs, and ADRs. Use when creating or organizing spec structure.
---

<objective>
Single source of truth for specs/ directory structure and all document templates. Enables creation of requirements (PRD/TRD), decisions (ADR), and work items (capability/feature/story) with consistent structure across all products using the SPX framework.
</objective>

<quick_start>
Different use cases read different sections:

- **Template access** → Read `<accessing_templates>` FIRST to understand where templates are located
- **Structure definition** → Read `<structure_definition>` for specs/ directory hierarchy, BSP numbering, test graduation
- **ADR templates** → Read `<adr_templates>` for Architectural Decision Record patterns
- **PRD/TRD templates** → Read `<requirement_templates>` for Product and Technical Requirements
- **Work item templates** → Read `<work_item_templates>` for capability, feature, story, and DONE.md patterns

Use progressive disclosure - read only what you need.
</quick_start>

<accessing_templates>

## How to Access Templates

**All templates are stored within this skill's base directory.**

### Understanding Skill Directory Structure

When you invoke `/managing-specs`, Claude loads this skill from the skill's base directory. Throughout this documentation, we refer to this as `${SKILL_DIR}`.

**The skill's base directory path pattern:**

```
.claude/plugins/cache/{marketplace-name}/{plugin-name}/{version}/skills/managing-specs/
```

**Example**: For spx-claude marketplace, specs plugin version 0.3.3:

```
${SKILL_DIR} = .claude/plugins/cache/spx-claude/specs/0.3.3/skills/managing-specs/
```

### Template Organization

All templates are under `${SKILL_DIR}/templates/`:

```
${SKILL_DIR}/
├── SKILL.md                                    # This file
└── templates/
    ├── decisions/
    │   └── architectural-decision.adr.md
    ├── requirements/
    │   ├── product-change.prd.md
    │   └── technical-change.trd.md
    └── work-items/
        ├── capability-name.capability.md
        ├── feature-name.feature.md
        ├── story-name.story.md
        └── DONE.md
```

### How to Read Templates

**Always use the skill's base directory, not the user's project directory.**

```bash
# Pattern
Read: ${SKILL_DIR}/templates/{category}/{template-name}

# Example: Read feature template
Read: ${SKILL_DIR}/templates/work-items/feature-name.feature.md

# With actual path (example for spx-claude marketplace, version 0.3.3)
Read: .claude/plugins/cache/spx-claude/specs/0.3.3/skills/managing-specs/templates/work-items/feature-name.feature.md
```

### Troubleshooting

If you cannot find a template:

1. ✅ Verify you're using the skill's base directory, NOT the project directory
2. ✅ Ensure path starts with `${SKILL_DIR}/templates/...` or `.claude/plugins/cache/...`
3. ✅ Use Glob to discover: `Glob: .claude/plugins/cache/**/managing-specs/templates/**/*.md`
4. ❌ Do NOT look for templates in the user's project (e.g., `specs/templates/`)

</accessing_templates>

<structure_definition>

## SPX Framework Structure

The specs/ directory follows the SPX framework structure defined in `structure.yaml`.

### Three-Phase Transformation

1. **Requirements (PRD/TRD)** - Capture vision without implementation constraints
2. **Decisions (ADR)** - Constrain architecture with explicit trade-offs
3. **Work Items (Capability/Feature/Story)** - Sized, testable implementation containers

### Directory Structure

```
specs/
├── [product-name].prd.md          # Product-wide PRD (optional)
├── decisions/                      # Product-wide ADRs
│   └── adr-NNN_{slug}.md
└── work/
    ├── backlog/
    ├── doing/
    │   └── capability-NN_{slug}/
    │       ├── {slug}.capability.md
    │       ├── {topic}.prd.md       # Optional PRD catalyst
    │       ├── decisions/           # Capability-scoped ADRs
    │       ├── tests/
    │       └── feature-NN_{slug}/
    │           ├── {slug}.feature.md
    │           ├── {topic}.trd.md   # Optional TRD catalyst
    │           ├── decisions/       # Feature-scoped ADRs
    │           ├── tests/
    │           └── story-NN_{slug}/
    │               ├── {slug}.story.md
    │               └── tests/
    └── done/
```

### Work Item Hierarchy

- **Capability**: E2E scenario with product-wide impact
  - Tests graduate to `tests/e2e/`
  - Triggered by PRD
  - Contains features

- **Feature**: Integration scenario with specific functionality
  - Tests graduate to `tests/integration/`
  - Triggered by TRD
  - Contains stories

- **Story**: Unit-tested atomic implementation
  - Tests graduate to `tests/unit/`
  - No children
  - Atomic implementation unit

### Key Principles

- **PRD OR TRD** at same scope, never both
- **Capability** triggered by PRD → spawns features
- **Feature** triggered by TRD → spawns stories
- **Requirements immutable** - code adapts to requirements, not vice versa
- **BSP numbering**: Two-digit (10-99), lower number = must complete first
- **Test graduation**: `specs/.../tests/` → `tests/{unit,integration,e2e}/`
- **Status rules**:
  - OPEN: No tests exist
  - IN_PROGRESS: Tests exist, no DONE.md
  - DONE: DONE.md exists

### structure.yaml

`structure.yaml` defines the complete framework:

- Work item hierarchy (capability → feature → story)
- Directory patterns: `{level}-{bsp}_{slug}`
- File patterns: `{slug}.{level}.md`
- Test graduation paths for each level
- BSP numbering rules and status determination

**All products use structure.yaml as-is. No per-product customization.**
</structure_definition>

<adr_templates>

## Architectural Decision Records

ADRs document technical choices with trade-offs and consequences.

### Template Location

```
${SKILL_DIR}/templates/decisions/architectural-decision.adr.md
```

### Usage

Read the template and adapt:

```bash
# Read ADR template
Read: ${SKILL_DIR}/templates/decisions/architectural-decision.adr.md

# Adapt for your decision
- Document decision context and problem
- List options considered with pros/cons
- Document chosen option and rationale
- Specify consequences and trade-offs
```

### Scope Levels

ADRs can exist at three levels:

- **Project**: `specs/decisions/adr-NNN_{slug}.md`
- **Capability**: `specs/work/doing/capability-NN/decisions/adr-NNN_{slug}.md`
- **Feature**: `specs/work/doing/.../feature-NN/decisions/adr-NNN_{slug}.md`

Stories inherit decisions from parent feature/capability.

### Naming Convention

Format: `adr-{NNN}_{slug}.md`

- NNN: Three-digit sequential number (001, 002, ...)
- slug: Kebab-case description (e.g., `use-postgresql-for-persistence`)

</adr_templates>

<requirement_templates>

## Requirements Templates

Templates for Product Requirements (PRD) and Technical Requirements (TRD).

### PRD Template

**Location**: `${SKILL_DIR}/templates/requirements/product-change.prd.md`

**Purpose**: Product requirements - user value, customer journey, measurable outcomes

**Usage**:

```bash
# Read PRD template
Read: ${SKILL_DIR}/templates/requirements/product-change.prd.md

# Adapt for product change
- Define user value proposition
- Document measurable outcomes with targets
- Specify acceptance criteria
- Avoid implementation details
```

**Placement**:

- Product-wide: `specs/{product-name}.prd.md`
- Capability catalyst: `specs/work/doing/capability-NN/{topic}.prd.md`

### TRD Template

**Location**: `${SKILL_DIR}/templates/requirements/technical-change.trd.md`

**Purpose**: Technical requirements - system architecture, validation strategy, test infrastructure

**Usage**:

```bash
# Read TRD template
Read: ${SKILL_DIR}/templates/requirements/technical-change.trd.md

# Adapt for technical change
- Specify technical architecture
- Define testing strategy (Level 1/2/3)
- Document validation approach
- Identify infrastructure needs
```

**Placement**:

- Feature catalyst: `specs/work/doing/.../feature-NN/{topic}.trd.md`

### Requirements Rules

- **PRD OR TRD** at same scope, never both
- **Immutable**: Code adapts to requirements, not vice versa
- **Catalyst pattern**: PRD spawns capability, TRD spawns feature

</requirement_templates>

<work_item_templates>

## Work Item Templates

Templates for capabilities, features, stories, and completion evidence.

### Template Locations

```
${SKILL_DIR}/templates/work-items/capability-name.capability.md
${SKILL_DIR}/templates/work-items/feature-name.feature.md
${SKILL_DIR}/templates/work-items/story-name.story.md
${SKILL_DIR}/templates/work-items/DONE.md
```

### Usage Pattern

```bash
# For capability
Read: ${SKILL_DIR}/templates/work-items/capability-name.capability.md
Adapt: Replace {slug} with kebab-case name
       Fill functional requirements
       Add user value context

# For feature
Read: ${SKILL_DIR}/templates/work-items/feature-name.feature.md
Adapt: Replace {slug} with kebab-case name
       Specify integration scope
       Define component interactions

# For story
Read: ${SKILL_DIR}/templates/work-items/story-name.story.md
Adapt: Replace {slug} with kebab-case name
       Detail atomic implementation
       List specific functions/classes

# For completion
Read: ${SKILL_DIR}/templates/work-items/DONE.md
Adapt: List graduated tests by level
       Document verification steps
       Include evidence of completion
```

### File Placement

Work items follow this pattern:

```
specs/work/{backlog|doing|done}/{level}-{bsp}_{slug}/{slug}.{level}.md
```

Examples:

- `specs/work/doing/capability-21_core-cli/core-cli.capability.md`
- `specs/work/doing/capability-21_core-cli/feature-10_init/init.feature.md`
- `specs/work/doing/capability-21_core-cli/feature-10_init/story-01_parse-flags/parse-flags.story.md`

### Test Graduation

When work is complete, tests graduate:

- Capability tests: `specs/.../tests/` → `tests/e2e/`
- Feature tests: `specs/.../tests/` → `tests/integration/`
- Story tests: `specs/.../tests/` → `tests/unit/`

DONE.md documents this graduation and provides verification evidence.
</work_item_templates>

<success_criteria>
Skill is working correctly when:

- [ ] Templates exist and are readable in `templates/` subdirectories
- [ ] structure.yaml is valid YAML and defines complete hierarchy
- [ ] Other skills can successfully read templates from appropriate sections
- [ ] Progressive disclosure guides readers to relevant sections
- [ ] ADR, requirement, and work item patterns are clearly documented
- [ ] Test graduation paths are correctly specified for each level
- [ ] BSP numbering uses consistent two-digit format (10-99)

</success_criteria>
