# specs/ Directory Guide

This guide explains WHEN to invoke specs skills. It is a **router** that tells you which skill to use. The skills themselves contain the HOW (detailed procedures, templates, structure definitions).

---

## 🚨 MANDATORY SKILL INVOCATION RULES

**YOU MUST STOP AND INVOKE THE APPROPRIATE SKILL BEFORE PROCEEDING.**

Do NOT grep for templates. Do NOT search for structure definitions. Do NOT guess at requirements patterns. The skills contain everything you need.

---

## When to Invoke Skills

### Before Implementing ANY Work Item → `/understanding-specs`

**⛔ BLOCKING REQUIREMENT: DO NOT PROCEED WITHOUT INVOKING THIS SKILL**

**Trigger conditions** (any of these):

- User says "implement story-NN", "work on feature-NN", or "build capability-NN"
- User references a work item file (e.g., "implement this.story.md")
- User asks you to write code for a story/feature/capability
- You see a work item path in `specs/work/doing/`
- You're about to write implementation code in a feature/story/capability scope

**What it does**: Loads complete context hierarchy (PRD/TRD → ADRs → work item → tests). Verifies all specification documents exist before implementation.

**Why mandatory**: Without this skill, you will miss requirements, violate ADRs, and implement the wrong thing.

**Example**:

```text
User: "Implement story-21_parse-flags"
→ STOP. DO NOT proceed to reading files or writing code.
→ IMMEDIATELY invoke: /understanding-specs specs/work/doing/.../story-21_parse-flags/parse-flags.story.md
→ WAIT for skill to load complete context
→ THEN proceed with implementation
```

---

### When Creating/Organizing Spec Structure → `/managing-specs`

**⛔ BLOCKING REQUIREMENT: DO NOT PROCEED WITHOUT INVOKING THIS SKILL**

**Trigger conditions** (any of these):

- User says "create a PRD", "write a TRD", "add an ADR"
- User says "set up specs directory", "organize specs"
- User asks "what's the next work item to implement?"
- You need to understand BSP numbering or work item hierarchy
- You need ANY template (PRD/TRD/ADR/capability/feature/story)
- User asks about work item status (OPEN/IN_PROGRESS/DONE)

**What it does**: Provides access to all templates in the skill's `templates/` directory. Contains SPX framework structure, BSP numbering rules, status determination rules.

**Why mandatory**: Templates live in the skill directory (`.claude/plugins/cache/.../managing-specs/templates/`), NOT in the project. Searching the project will fail. The skill contains the complete structure definition.

**Example**:

```text
User: "Create a PRD for the new authentication feature"
→ STOP. DO NOT search for template files in the project.
→ DO NOT grep for "*.prd.md" files.
→ IMMEDIATELY invoke: /managing-specs
→ READ template from: ${SKILL_DIR}/templates/requirements/product-change.prd.md
→ ADAPT template with authentication feature content
```

**Example**:

```text
User: "What should I work on next?"
→ STOP. DO NOT search for work items yourself.
→ IMMEDIATELY invoke: /managing-specs
→ ASK skill: "What's the next work item based on BSP ordering?"
→ Skill will identify first OPEN or IN_PROGRESS item
```

---

## Quick Reference: Skill Selection Decision Tree

| User Says...                           | YOU MUST INVOKE                        | DO NOT                                       |
| -------------------------------------- | -------------------------------------- | -------------------------------------------- |
| "Implement story-21"                   | `/understanding-specs` (on story file) | Do NOT read story.md directly                |
| "Work on this feature"                 | `/understanding-specs` (on feature)    | Do NOT start coding without context          |
| "Create a PRD"                         | `/managing-specs`                      | Do NOT search project for PRD templates      |
| "Write an ADR"                         | `/managing-specs`                      | Do NOT search project for ADR templates      |
| "What's next to work on?"              | `/managing-specs`                      | Do NOT grep for work items yourself          |
| "Set up specs structure"               | `/managing-specs`                      | Do NOT create directories without guidance   |
| "What's the status of feature-21?"     | `/managing-specs`                      | Do NOT inspect `tests/` directories yourself |
| "Add a new capability/feature/story"   | `/managing-specs`                      | Do NOT calculate BSP numbers yourself        |
| "Show me the PRD for this capability"  | `/understanding-specs` (on capability) | Do NOT read PRD without loading full context |
| "How do I organize requirements docs?" | `/managing-specs`                      | Do NOT invent your own structure             |

---

## Status and Work Item Rules Summary

**⚠️ For complete details, invoke `/managing-specs` skill. Do NOT rely on this summary alone.**

### Key Principles

**BSP Numbering**: Lower number = must complete FIRST. You CANNOT work on item N until ALL items with numbers < N are DONE.

**Status Determination** (read from `tests/` directory at each level):

- **OPEN**: Missing OR empty `tests/` directory
- **IN_PROGRESS**: Has `*.test.*` files, no `DONE.md`
- **DONE**: Has `DONE.md` file

**Finding Next Work Item**: Invoke `/managing-specs` and ask "What's the next work item?" Do NOT calculate this yourself.

**Test Graduation**:

- Story tests: `specs/.../tests/` → `tests/unit/`
- Feature tests: `specs/.../tests/` → `tests/integration/`
- Capability tests: `specs/.../tests/` → `tests/e2e/`

**Critical Rule**: Never write tests directly in `tests/` — this breaks CI until implementation is complete. Always write in `specs/.../tests/` first, then graduate upon completion.

---

## Session Management

Claude Code session handoffs are stored in:

```text
.spx/sessions/
├── TODO_*.md      # Available for /pickup
└── DOING_*.md     # Currently claimed
```

**Commands**:

- Use `/handoff` to create session context for continuation
- Use `/pickup` to load and claim a handoff

---

## Why These Rules Are Non-Negotiable

### Problem: Template Hallucination

Without invoking `/managing-specs`, agents:

- Search the project for templates that don't exist there
- Find OLD PRDs/TRDs and copy their (possibly wrong) structure
- Invent their own template structure
- Miss critical sections required by the SPX framework

**Result**: Requirements documents that are incomplete or violate the framework.

### Problem: Context Gaps

Without invoking `/understanding-specs`, agents:

- Miss parent PRD/TRD context
- Violate ADR decisions unknowingly
- Implement the wrong requirements
- Skip test requirements

**Result**: Code that doesn't match requirements and fails review.

### Problem: Structure Violations

Without invoking `/managing-specs` for BSP numbering:

- Create work items with wrong numbers
- Violate dependency ordering
- Break the BSP algorithm (items can collide or be out of order)

**Result**: Work items that can't be ordered correctly, blocking downstream work.

---

## Verification Checklist

Before proceeding with ANY specs-related task:

- [ ] Did I invoke the correct skill?
- [ ] Did I wait for skill output before proceeding?
- [ ] Did I read templates from `${SKILL_DIR}/templates/`, NOT from the project?
- [ ] Did I load complete context hierarchy for work items?
- [ ] Did I verify BSP numbering through the skill, not by calculating myself?

**If any checkbox is unchecked, STOP and invoke the skill.**

---

**For complete SPX framework structure, templates, and procedures**: Invoke `/managing-specs` skill.

**For work item context loading and validation**: Invoke `/understanding-specs` skill on the work item file.
