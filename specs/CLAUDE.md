# specs/ Directory Guide

This guide covers navigating, reading status, and editing work items in the `specs/` directory.

---

## Navigating the `specs/` Directory

### Directory Layout

```
specs/
├── CLAUDE.md                      # This file
├── [product-name].prd.md          # Product-wide PRD (optional)
├── decisions/                      # Product-wide ADRs only
└── work/
    ├── backlog/                    # Future work items
    ├── doing/                      # Active work items
    └── done/                       # Completed work items (permanent)
```

### Three-Level Hierarchy (All Levels Work The Same)

```
specs/work/{doing,backlog,done}/
└── capability-NN_{slug}/                    # Level 1 (TOP)
    ├── {slug}.capability.md                 # Work item definition
    ├── {topic}.prd.md                       # Optional: requirements catalyst
    ├── decisions/adr-NNN_{slug}.md          # Architectural decisions
    ├── tests/                               # E2E tests
    │   └── DONE.md                          # Completion marker
    │
    └── feature-NN_{slug}/                   # Level 2
        ├── {slug}.feature.md                # Work item definition
        ├── {topic}.trd.md                   # Optional: requirements catalyst
        ├── decisions/adr-NNN_{slug}.md      # Architectural decisions
        ├── tests/                           # Integration tests
        │   └── DONE.md                      # Completion marker
        │
        └── story-NN_{slug}/                 # Level 3 (BOTTOM)
            ├── {slug}.story.md              # Work item definition
            └── tests/                       # Unit tests
                └── DONE.md                  # Completion marker
```

### What Lives Where

| Level          | Work Item         | Optional Catalyst | Has Decisions? | Test Type   |
| -------------- | ----------------- | ----------------- | -------------- | ----------- |
| 1 (Capability) | `*.capability.md` | `*.prd.md`        | ✅ Yes         | E2E         |
| 2 (Feature)    | `*.feature.md`    | `*.trd.md`        | ✅ Yes         | Integration |
| 3 (Story)      | `*.story.md`      | ❌ None           | ❌ Inherits    | Unit        |

**Key insight**: Capabilities ARE the top level. No `specs/project.prd.md` above them.

**Fractal nature**: PRD at capability level spawns features. TRD at feature level spawns stories. Stories are atomic—no children.

---

## READ: Status and What to Work On Next

### Three States

Status is determined by the `tests/` directory at each level:

| State           | `tests/` Directory           | Meaning          |
| --------------- | ---------------------------- | ---------------- |
| **OPEN**        | Missing OR empty             | Work not started |
| **IN_PROGRESS** | Has `*.test.*`, no `DONE.md` | Work underway    |
| **DONE**        | Has `DONE.md`                | Complete         |

### 🚨 BSP Numbers = Dependency Order

> **Lower BSP number = must complete FIRST.**
>
> You CANNOT work on item N until ALL items with numbers < N are DONE.

This applies at every level:

| If you see...                                   | It means...                                      |
| ----------------------------------------------- | ------------------------------------------------ |
| `feature-48` before `feature-87`                | feature-48 MUST be DONE before feature-87 starts |
| `story-21` before `story-32`                    | story-21 MUST be DONE before story-32 starts     |
| `feature-48 [OPEN]`, `feature-87 [IN_PROGRESS]` | **BUG**: Dependency violation                    |

### Finding the Next Work Item

```
1. List all work items in BSP order (capability → feature → story)
2. Return the FIRST item where status ≠ DONE
3. That item blocks everything after it
```

**Example**:

```text
feature-48_test-harness [OPEN]        ← Was added after feature-87 but blocks it
feature-87_e2e-workflow [IN_PROGRESS] ← Was already started, then dependency discovered
```

**Next work item**: `feature-48_test-harness` → its first OPEN story.

---

## EDIT: Adding or Reordering Work Items

### BSP Numbering

Two-digit prefixes in range **[10, 99]** encode dependency order.

### Creating New Items

#### Case 1: First Item (No Siblings)

Use position **21** (leaves room for ~10 items before/after):

```
# First feature in a new capability
capability-21_foo/
└── feature-21_first-feature/
```

#### Case 2: Insert Between Siblings

Use midpoint: `new = floor((left + right) / 2)`

```
# Insert between feature-21 and feature-54
new = floor((21 + 54) / 2) = 37

feature-21_first/
feature-37_inserted/    ← NEW
feature-54_second/
```

#### Case 3: Append After Last

Use midpoint to upper bound: `new = floor((last + 99) / 2)`

```
# Append after feature-54
new = floor((54 + 99) / 2) = 76

feature-21_first/
feature-54_second/
feature-76_appended/    ← NEW
```

### Creating a Work Item

Every work item needs:

1. **Directory**: `NN_{slug}/`
2. **Definition file**: `{slug}.{capability|feature|story}.md`
3. **Tests directory**: `tests/` (create when starting work)

Optional:

- **Requirements catalyst**: `{topic}.prd.md` (capability) or `{topic}.trd.md` (feature)
- **Decisions**: `decisions/adr-NNN_{slug}.md`

**Templates**: Use `/managing-specs` skill to access templates.

### Marking Complete

1. Ensure all tests pass
2. Create `tests/DONE.md` with:
   - Summary of what was implemented
   - List of graduated tests (moved to production `tests/`)
   - Any notes for future reference

### Test Graduation

When a work item is DONE, its tests graduate from `specs/.../tests/` to the production test suite:

| From                                     | To                   |
| ---------------------------------------- | -------------------- |
| `specs/.../story-NN/tests/*.test.*`      | `tests/unit/`        |
| `specs/.../feature-NN/tests/*.test.*`    | `tests/integration/` |
| `specs/.../capability-NN/tests/*.test.*` | `tests/e2e/`         |

> ⚠️ **Never write tests directly in `tests/`** — this breaks CI until implementation is complete. Always write in `specs/.../tests/` first, then graduate.

---

## Creating Requirements

**For product requirements:**

- Invoke `/writing-prd` skill
- Creates PRDs with user value and measurable outcomes
- Catalyzes capability decomposition

**For technical requirements:**

- Invoke `/writing-trd` skill
- Creates TRDs with architecture and validation strategy
- Catalyzes feature decomposition

**For structure and templates:**

- Invoke `/managing-specs` skill
- Provides templates for PRD/TRD/ADR/work items
- Defines directory structure and conventions

**Before implementing any work item:**

- Invoke `/understanding-specs` skill
- Reads complete context hierarchy (capability → feature → story)
- Verifies all specification documents exist
- Fails fast if context is incomplete

---

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

**For complete workflow methodology**, reference the SPX framework documentation (when available) or consult the `/managing-specs` skill for structure details.
