---
name: bootstrapping-documents
description: |
  Provides templates and structure definitions for spec-driven development following the SPX framework.
  This skill should be used when users ask to create PRDs, write TRDs, generate ADRs,
  scaffold capabilities, create features, write stories, or need document templates for requirements.
---

<objective>
Provides templates and structure.yaml for spec-driven development. Enables creation of requirements (PRD/TRD), decisions (ADR), and work items (capability/feature/story) with consistent structure across all products using the SPX framework.
</objective>

<quick_start>
To use a template, read it from this skill's path and adapt the placeholders:

```bash
# For product requirements
Read: templates/requirements/product-change.prd.md
Adapt: Replace {{placeholders}} with product-specific content

# For technical requirements
Read: templates/requirements/technical-change.trd.md
Adapt: Replace {{placeholders}} with technical specifics

# For architectural decisions
Read: templates/decisions/architectural-decision.adr.md
Adapt: Document decision context, options, and consequences

# For work items
Read: templates/work-items/{capability|feature|story}-name.{level}.md
Adapt: Replace naming patterns and fill functional requirements

# For completion evidence
Read: templates/work-items/DONE.md
Adapt: List graduated tests and verification steps
```

**Target directory structure:**

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

</quick_start>

<context>
**SPX Framework Overview:**

The spec-driven development framework transforms requirements into tested implementations through three phases:

1. **Requirements (PRD/TRD)** - Capture vision without implementation constraints
2. **Decisions (ADR)** - Constrain architecture with explicit trade-offs
3. **Work Items (Capability/Feature/Story)** - Sized, testable implementation containers

**Work item hierarchy:**

- **Capability**: E2E scenario with product-wide impact, tests graduate to `tests/e2e/`
- **Feature**: Integration scenario with specific functionality, tests graduate to `tests/integration/`
- **Story**: Unit-tested atomic implementation, tests graduate to `tests/unit/`

**Requirements types:**

- **PRD**: Product requirements - user value, customer journey, measurable outcomes
- **TRD**: Technical requirements - system architecture, validation strategy, test infrastructure

**Decisions:**

- **ADR**: Architectural decision record - technical choices with trade-offs and consequences
- Can exist at product/capability/feature scope
- Stories inherit decisions from parent feature/capability

**Key principles:**

- PRD OR TRD at same scope, never both
- Capability triggered by PRD, Feature triggered by TRD
- Requirements are immutable - code adapts to requirements, not vice versa
- BSP numbering: Two-digit (10-99), lower numbers = must complete first
- Test graduation: specs/.../tests/ → tests/{unit,integration,e2e}/
- Status rules: OPEN (no tests) → IN_PROGRESS (tests, no DONE.md) → DONE (DONE.md exists)

**structure.yaml defines:**

- Work item hierarchy: capability → feature → story
- Directory patterns: `{level}-{bsp}_{slug}` (e.g., `capability-21_core-cli`)
- File patterns: `{slug}.{level}.md` (e.g., `core-cli.capability.md`)
- Test graduation paths for each level
- BSP numbering rules and status determination

All products use structure.yaml as-is. No per-product customization.
</context>

<usage_with_skills>
**For skill authors:**

Template-generating skills should read templates from this skill:

```bash
# Example: /writing-product-requirements reads:
templates/requirements/product-change.prd.md

# Example: /writing-technical-requirements reads:
templates/requirements/technical-change.trd.md

# Work item creation skills read:
templates/work-items/{capability,feature,story}-name.{level}.md
```

**Skills parse structure.yaml to understand:**

- Valid work item levels and their hierarchy
- Directory and file naming conventions
- Where tests graduate upon completion
- BSP numbering constraints
  </usage_with_skills>

<success_criteria>
Skill is working correctly when:

- [ ] Templates exist and are readable in `templates/` subdirectories
- [ ] structure.yaml is valid YAML and defines complete hierarchy
- [ ] Other skills can successfully read and parse templates
- [ ] Directory structure documented matches target product layout
- [ ] BSP numbering uses consistent two-digit format (10-99)
- [ ] Test graduation paths are correctly specified for each level
      </success_criteria>
