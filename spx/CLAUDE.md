# spx/ Directory Guide

This guide explains WHEN to invoke specs skills. It is a **router** that tells you which skill to use. The skills themselves contain the HOW (detailed procedures, templates, structure definitions).

---

## Structure Overview (CODE Model)

The `spx/` tree is the always-current map of the product. Nothing moves because work is "done"—status is tracked via `status.yaml`.

```text
spx/
  {product}.prd.md                    # Product requirements
  adr-NN_{slug}.md                    # Product-wide decisions
  capability-NN_{slug}/
    {slug}.capability.md
    status.yaml                       # Test results, outcome status
    tests/
      *.e2e.test.{ts,py}             # Capability-level E2E tests
    adr-NN_{slug}.md
    feature-NN_{slug}/
      {slug}.feature.md
      status.yaml
      tests/
        *.integration.test.{ts,py}   # Feature-level integration tests
      adr-NN_{slug}.md
      story-NN_{slug}/
        {slug}.story.md
        status.yaml
        tests/
          *.unit.test.{ts,py}        # Story-level unit tests
```

---

## Key Principles

1. **Durable map**: Specs stay in place. Nothing moves because work is "done."
2. **Co-location**: Tests live with their spec in `tests/`. No graduation.
3. **Status via YAML**: `status.yaml` records test results. Empty `fail:` = outcome achieved.
4. **No TRDs**: Technical details belong in `feature.md`, not separate files.

---

## Status Determination

Read `status.yaml` to determine work state:

| State                | Condition           |
| -------------------- | ------------------- |
| **Spec only**        | No `status.yaml`    |
| **In progress**      | `fail:` has entries |
| **Outcome achieved** | `fail: []` (empty)  |

```yaml
# Example status.yaml
spec_commit: abc123
ran: 2025-01-27T10:30:00Z
pass:
  - tests/parsing.unit.test.ts
fail: []
```

---

## BSP Numbers Are SIBLING-UNIQUE

**BSP numbers (capability-NN, feature-NN, story-NN) are ONLY unique among siblings at the same level.**

```text
capability-21/feature-32/story-54  ← One story-54
capability-28/feature-32/story-54  ← DIFFERENT story-54
```

**ALWAYS use the FULL PATH when referencing work items:**

| Wrong                  | Correct                              |
| ---------------------- | ------------------------------------ |
| "story-54"             | "capability-21/feature-54/story-54"  |
| "implement feature-32" | "implement capability-21/feature-32" |

---

## When to Invoke Skills

### Before Implementing ANY Work Item → `/understanding-specs`

**BLOCKING REQUIREMENT**

**Trigger conditions:**

- User says "implement story-NN", "work on feature-NN", or "build capability-NN"
- User references a work item file
- You're about to write implementation code

**What it does**: Loads complete context hierarchy (PRD → ADRs → capability → feature → story).

### When Creating/Organizing Specs → `/managing-specs`

**BLOCKING REQUIREMENT**

**Trigger conditions:**

- User says "create a PRD", "add an ADR", "create capability/feature/story"
- User asks "what's next to work on?"
- You need templates or BSP numbering rules

**What it does**: Provides templates, BSP numbering, structure guidance.

---

## Quick Reference: Skill Selection

| User Says...         | Invoke                 | Do NOT                 |
| -------------------- | ---------------------- | ---------------------- |
| "Implement story-21" | `/understanding-specs` | Read story.md directly |
| "Create a PRD"       | `/managing-specs`      | Search for templates   |
| "What's next?"       | `/managing-specs`      | Grep for work items    |
| "Create a feature"   | `/managing-specs`      | Calculate BSP yourself |

---

## Test Naming Convention

Test level is in the filename suffix:

| Level       | Suffix                       | Container  |
| ----------- | ---------------------------- | ---------- |
| Unit        | `*.unit.test.{ts,py}`        | Story      |
| Integration | `*.integration.test.{ts,py}` | Feature    |
| E2E         | `*.e2e.test.{ts,py}`         | Capability |

---

## Spec-Test Contract

Spec files must reference their tests via relative Markdown links:

```markdown
## Tests

- [Unit: flag parsing](tests/parsing.unit.test.ts)
- [Integration: CLI validation](tests/cli.integration.test.ts)
```

This creates a verifiable contract between specs and tests.

---

## Session Management

Claude Code session handoffs are stored in `.spx/sessions/` (separate from spec tree):

```text
.spx/sessions/
├── TODO_*.md      # Available for /pickup
└── DOING_*.md     # Currently claimed
```

Use `/handoff` to create, `/pickup` to claim.
