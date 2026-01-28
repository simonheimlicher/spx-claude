---
name: managing-specs
description: Create and manage specs including capabilities, features, stories, PRDs, TRDs, and ADRs. Use when creating a feature, creating a story, adding specs, or setting up spec structure.
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

<how_to_access>
**All templates are stored within this skill's base directory.**

When this skill is invoked, Claude Code provides the base directory in the loading message:

```
Base directory for this skill: {skill_dir}
```

Use this exact path for all file access. Throughout this documentation, `${SKILL_DIR}` is a placeholder—Claude must substitute it manually from the loading message.

**IMPORTANT**: Do NOT search the project directory for skill files.
</how_to_access>

<skill_directory_structure>

**The skill's base directory path pattern:**

```
.claude/plugins/cache/{marketplace-name}/{plugin-name}/{version}/skills/managing-specs/
```

**Example**: For spx-claude marketplace, spx plugin version 0.1.0:

```
${SKILL_DIR} = .claude/plugins/cache/spx-claude/specs/0.3.3/skills/managing-specs/
```

</skill_directory_structure>

<template_organization>

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

</template_organization>

<how_to_read_templates>

**Always use the skill's base directory, not the user's project directory.**

```bash
# Pattern
Read: ${SKILL_DIR}/templates/{category}/{template-name}

# Example: Read feature template
Read: ${SKILL_DIR}/templates/work-items/feature-name.feature.md

# With actual path (example for spx-claude marketplace, version 0.3.3)
Read: .claude/plugins/cache/spx-claude/specs/0.3.3/skills/managing-specs/templates/work-items/feature-name.feature.md
```

</how_to_read_templates>

<troubleshooting>

If you cannot find a template:

1. ✅ Verify you're using the skill's base directory, NOT the project directory
2. ✅ Ensure path starts with `${SKILL_DIR}/templates/...` or `.claude/plugins/cache/...`
3. ✅ Use Glob to discover: `Glob: .claude/plugins/cache/**/managing-specs/templates/**/*.md`
4. ❌ Do NOT look for templates in the user's project (e.g., `specs/templates/`)

</troubleshooting>

</accessing_templates>

<structure_definition>

<overview>
The specs/ directory follows the SPX framework structure defined in `structure.yaml`.
</overview>

<three_phase_transformation>

1. **Requirements (PRD/TRD)** - Capture vision without implementation constraints
2. **Decisions (ADR)** - Constrain architecture with explicit trade-offs
3. **Work Items (Capability/Feature/Story)** - Sized, testable implementation containers

</three_phase_transformation>

<directory_structure>

```
spx/
├── {product-name}.prd.md             # Product requirements
├── NN-{slug}.adr.md                  # Product-wide ADRs (interleaved)
└── NN-{slug}.capability/
    ├── {slug}.capability.md
    ├── pass.csv                       # Test verification ledger
    ├── tests/
    ├── NN-{slug}.adr.md              # Capability-scoped ADRs (interleaved)
    └── NN-{slug}.feature/
        ├── {slug}.feature.md
        ├── pass.csv
        ├── tests/
        ├── NN-{slug}.adr.md          # Feature-scoped ADRs (interleaved)
        └── NN-{slug}.story/
            ├── {slug}.story.md
            ├── pass.csv
            └── tests/
```

</directory_structure>

<work_item_hierarchy>

- **Capability**: E2E scenario with product-wide impact
  - Tests graduate to `tests/e2e/`
  - May have optional PRD as catalyst/enrichment
  - Contains features

- **Feature**: Integration scenario with specific functionality
  - Tests graduate to `tests/integration/`
  - May have optional TRD as catalyst/enrichment
  - Contains stories

- **Story**: Unit-tested atomic implementation
  - Tests graduate to `tests/unit/`
  - No children
  - Atomic implementation unit

</work_item_hierarchy>

<key_principles>

- **PRD OR TRD** at same scope, never both
- **Requirements immutable** - code adapts to requirements, not vice versa
- **BSP numbering**: Two-digit (10-99), lower number = must complete first
- **BSP numbers are SIBLING-UNIQUE**: Numbers are only unique among siblings, not globally (see `<bsp_sibling_uniqueness>` below)
- **Test graduation**: `specs/.../tests/` → `tests/{unit,integration,e2e}/`
- **Status rules** (use CLI, do NOT check manually):
  - `spx spec status --format table` - View project status
  - `spx spec next` - Get next work item (respects BSP ordering)

</key_principles>

<bsp_sibling_uniqueness>

**🚨 CRITICAL: BSP numbers are ONLY unique among siblings at the same level.**

```text
21-foo.capability/21-bar.feature/54-baz.story/  ← One story-54
21-foo.capability/37-qux.feature/54-baz.story/  ← DIFFERENT story-54
37-other.capability/21-bar.feature/54-baz.story/  ← DIFFERENT story-54
```

**ALWAYS use the FULL PATH when referencing work items:**

| ❌ WRONG (Ambiguous)     | ✅ CORRECT (Unambiguous)                                  |
| ------------------------ | --------------------------------------------------------- |
| "story-54"               | "21-foo.capability/21-bar.feature/54-baz.story/"          |
| "implement feature-21"   | "implement 21-foo.capability/21-bar.feature/"             |
| "Continue with story-54" | "Continue 21-foo.capability/21-bar.feature/54-baz.story/" |

**Why this matters:**

- Different capabilities can have identically-numbered features
- Different features can have identically-numbered stories
- "story-54" could refer to dozens of different stories across the codebase
- Without the full path, agents will access the WRONG work item

**When communicating about work items:**

1. Always include the full hierarchy path
2. Never use bare numbers like "story-54" without context
3. When in doubt, use the absolute path from `spx/`

</bsp_sibling_uniqueness>

</structure_definition>

<workflow>

<reading_status>

<understanding_work_items>

<three_states>
**Use CLI commands to check status (do NOT manually inspect directories):**

```bash
# View project status
spx spec status --format table

# Get next work item (respects BSP ordering)
spx spec next
```

Status values:

| State           | Meaning          |
| --------------- | ---------------- |
| **OPEN**        | Work not started |
| **IN_PROGRESS** | Work underway    |
| **DONE**        | Complete         |

</three_states>

<bsp_dependency_order>

> **Lower BSP number = must complete FIRST.**
>
> You CANNOT work on item N until ALL items with numbers < N are DONE.

This applies at every level:

| If you see...                              | It means...                              |
| ------------------------------------------ | ---------------------------------------- |
| `48-foo.feature/` before `87-bar.feature/` | 48-foo MUST be DONE before 87-bar starts |
| `21-foo.story/` before `32-bar.story/`     | 21-foo MUST be DONE before 32-bar starts |
| `48-foo [OPEN]`, `87-bar [IN_PROGRESS]`    | **BUG**: Dependency violation            |

</bsp_dependency_order>

<finding_next_work_item>

```
1. List all work items in BSP order (capability → feature → story)
2. Return the FIRST item where status ≠ DONE
3. That item blocks everything after it
```

**Example**:

```text
48-test-harness.feature/ [OPEN]        ← Was added after 87 but blocks it
87-e2e-workflow.feature/ [IN_PROGRESS] ← Was already started, then dependency discovered
```

**Next work item**: `48-test-harness.feature/` → its first OPEN story.

</finding_next_work_item>

</understanding_work_items>

</reading_status>

<managing_items>

<managing_work_items>

<numbering_work_items>

<bsp_numbering>
Two-digit prefixes in range **[10, 99]** encode dependency order.
</bsp_numbering>

<creating_new_items>

<case_1_first_item>

Use position **21** (leaves room for ~10 items before/after):

```
# First feature in a new capability
21-foo.capability/
└── 21-first-feature.feature/
```

</case_1_first_item>

<case_2_insert_between>

Use midpoint: `new = floor((left + right) / 2)`

```
# Insert between 21 and 54
new = floor((21 + 54) / 2) = 37

21-first.feature/
37-inserted.feature/    ← NEW
54-second.feature/
```

</case_2_insert_between>

<case_3_append_after>

Use midpoint to upper bound: `new = floor((last + 99) / 2)`

```
# Append after 54
new = floor((54 + 99) / 2) = 76

21-first.feature/
54-second.feature/
76-appended.feature/    ← NEW
```

</case_3_append_after>

</creating_new_items>

</numbering_work_items>

<creating_work_items>
Every work item needs:

1. **Directory**: `NN-{slug}.{type}/` (e.g., `21-auth.capability/`)
2. **Spec file**: `{slug}.{type}.md` (e.g., `auth.capability.md`)
3. **Tests directory**: `tests/` (create when starting work)

Optional:

- **Decision Records**: `NN-{slug}.adr.md` (interleaved with work items)

</creating_work_items>

</managing_work_items>

</managing_items>

</workflow>

<adr_templates>

<overview>
ADRs document technical choices with trade-offs and consequences.
</overview>

<template_location>

```
${SKILL_DIR}/templates/decisions/architectural-decision.adr.md
```

</template_location>

<usage>

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

</usage>

<scope_levels>

ADRs are interleaved with work items at any level:

- **Product**: `spx/NN-{slug}.adr.md`
- **Capability**: `spx/NN-{slug}.capability/NN-{slug}.adr.md`
- **Feature**: `spx/.../NN-{slug}.feature/NN-{slug}.adr.md`

Stories inherit decisions from parent feature/capability.

</scope_levels>

<naming_convention>

Format: `NN-{slug}.adr.md`

- NN: BSP number in range [10, 99]
- slug: Kebab-case description (e.g., `use-postgresql-for-persistence`)

</naming_convention>

<bsp_numbering_for_adrs>

**Lower BSP number = must decide first (within scope).**

ADRs follow the same BSP numbering as work items:

<creating_first_adr>

Use position **21** (leaves room for ~10 items before/after):

```
21-first-decision.adr.md
```

</creating_first_adr>

<inserting_between_adrs>

Use midpoint: `new = floor((left + right) / 2)`

```
# Insert between 21 and 54
new = floor((21 + 54) / 2) = 37

21-type-safety.adr.md
37-inserted-decision.adr.md    ← NEW
54-cli-framework.adr.md
```

</inserting_between_adrs>

<appending_after_last>

Use midpoint to upper bound: `new = floor((last + 99) / 2)`

```
# Append after 54
new = floor((54 + 99) / 2) = 76

21-type-safety.adr.md
54-cli-framework.adr.md
76-appended-decision.adr.md    ← NEW
```

</appending_after_last>

</bsp_numbering_for_adrs>

<dependency_order_within_scope>

**Scope boundaries**: ADRs are scoped to product/capability/feature.

**Within scope**: Lower BSP = must decide first.

| If you see...             | It means...                              |
| ------------------------- | ---------------------------------------- |
| `21-type-safety.adr.md`   | Foundational decision, must decide first |
| `37-validation.adr.md`    | Depends on 21, must come after           |
| `54-cli-framework.adr.md` | May depend on both 21 and 37             |

**Cross-scope dependencies**: Must be documented explicitly in the ADR content.

| Dependency                            | How to Express                           |
| ------------------------------------- | ---------------------------------------- |
| Feature ADR depends on capability ADR | Reference in "Context" section with link |
| Capability ADR depends on product ADR | Reference in "Context" section with link |

</dependency_order_within_scope>

<why_bsp_numbering>

**Problem**: Sequential numbering (01, 02, 03) cannot accommodate discovered dependencies.

**Example**: You have decisions adr-01, adr-02, adr-03. You discover adr-02 needs a prior decision about type safety. With sequential numbering, you must renumber all subsequent ADRs.

**Solution**: BSP numbering allows insertion at any point using midpoint calculation.

</why_bsp_numbering>

<why_no_numbers_in_content>

**Problem**: If ADR file is renumbered (e.g., adr-23 → adr-37), content with embedded numbers becomes stale.

**Examples**:

- Header `# ADR 23: Foo` - wrong after renumbering
- Reference "See ADR-23" - wrong after renumbering

**Solution**:

- Header: `# ADR: Foo` (document type prefix, no number)
- References: `[Foo](37-foo.adr.md)` (markdown link with path)
- Filenames can be renamed, slugs stay stable, markdown links update automatically

</why_no_numbers_in_content>

<why_markdown_links_only>

**Problem**: Plain text references like "ADR-23" or even `` `23-foo.adr.md` `` break when files are renumbered.

**Solution**: Markdown links `[Decision Title](NN-slug.adr.md)`:

- Modern editors update links automatically on file rename
- Links are clickable for navigation
- Slug provides stability even if number changes
- Can search by slug if link breaks

</why_markdown_links_only>

</adr_templates>

<referencing_adrs>

<overview>
**Always use markdown links with descriptive titles.**
</overview>

<in_markdown_documents>

<within_same_directory>

```markdown
See [Type Safety](21-type-safety.adr.md) for validation approach.
```

</within_same_directory>

<from_child_to_parent_scope>

```markdown
<!-- Feature ADR referencing capability ADR -->

This decision builds on [Config Loading](../21-config-loading.adr.md).

<!-- Story referencing feature ADR -->

Implementation follows [CLI Structure](../21-cli-structure.adr.md).
```

</from_child_to_parent_scope>

<from_work_item_to_adr>

```markdown
<!-- From story to capability ADR -->

Architectural constraints: [Commander Pattern](../../21-commander-pattern.adr.md)

<!-- From feature to product ADR -->

Type system: [Type Safety](../../../21-type-safety.adr.md)
```

</from_work_item_to_adr>

</in_markdown_documents>

<never_use_these_formats>

❌ Plain text reference: "See ADR-21"
❌ Code-only reference: `` `21-type-safety.adr.md` ``
❌ Number-only reference: "ADR 21 specifies..."

</never_use_these_formats>

<why_markdown_links>

- **Clickable**: Navigate directly in editors/viewers
- **Stable**: Slug provides stability even if number changes
- **Updatable**: Modern editors can update links on file rename
- **Descriptive**: Title provides context without opening file

</why_markdown_links>

</referencing_adrs>

<requirement_templates>

<overview>
Templates for Product Requirements (PRD) and Technical Requirements (TRD).
</overview>

<prd_template>

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

</prd_template>

<trd_template>

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

</trd_template>

<requirements_rules>

- **PRD OR TRD** at same scope, never both
- **Immutable**: Code adapts to requirements, not vice versa
- **Optional catalyst**: PRD/TRD may exist as enrichment; spec file is always required

</requirements_rules>

</requirement_templates>

<work_item_templates>

<overview>
Templates for capabilities, features, stories, and completion evidence.
</overview>

<template_locations>

```
${SKILL_DIR}/templates/work-items/capability-name.capability.md
${SKILL_DIR}/templates/work-items/feature-name.feature.md
${SKILL_DIR}/templates/work-items/story-name.story.md
${SKILL_DIR}/templates/work-items/DONE.md
```

</template_locations>

<usage_pattern>

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

</usage_pattern>

<file_placement>

Work items follow this pattern:

```
spx/{BSP}-{slug}.{type}/{slug}.{type}.md
```

Examples:

- `spx/21-core-cli.capability/core-cli.capability.md`
- `spx/21-core-cli.capability/10-init.feature/init.feature.md`
- `spx/21-core-cli.capability/15-init.feature/87-parse-flags.story/parse-flags.story.md`

</file_placement>

<test_graduation>

When work is complete, tests graduate:

- Capability tests: `specs/.../tests/` → `tests/e2e/`
- Feature tests: `specs/.../tests/` → `tests/integration/`
- Story tests: `specs/.../tests/` → `tests/unit/`

DONE.md documents this graduation and provides verification evidence.

</test_graduation>

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
