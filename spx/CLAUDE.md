# spx/ Directory Guide (CODE Framework)

This guide explains WHEN to invoke spx skills. It is a **router** that tells you which skill to use. The skills themselves contain the HOW (detailed procedures, templates, structure definitions).

---

## Structure Overview (CODE Model)

The `spx/` tree is the always-current map of the product. Nothing moves because work is "done"—specs are a durable map, not a backlog.

```text
spx/
  {product}.prd.md                    # Product requirements
  NN-{slug}.adr.md                    # Product-wide decisions (interleaved)
  NN-{slug}.capability/
    {slug}.capability.md
    tests/
      *.{unit,integration,e2e}.test.{ts,py}
    NN-{slug}.adr.md
    NN-{slug}.feature/
      {slug}.feature.md
      tests/
        *.{unit,integration,e2e}.test.{ts,py}
      NN-{slug}.adr.md
      NN-{slug}.story/
        {slug}.story.md
        tests/
          *.{unit,integration,e2e}.test.{ts,py}
```

---

## Key Principles

1. **Durable map**: Specs stay in place. Nothing moves because work is "done."
2. **Co-location**: Tests live with their spec in `tests/`. No graduation.
3. **No TRDs**: Technical details belong in `feature.md`, not separate files.

---

## BSP = Binary Space Partitioning

**Binary Space Partitioning (BSP)** encodes dependency order: lower BSP items are dependencies that higher-BSP items may rely on; same BSP means independent. The "binary" refers to insertion by halving available space.

- Lower BSP → dependency (others may rely on it)
- Same BSP → independent of each other
- Use `@` for recursive insertion when integers exhausted (e.g., `20@54-audit`)

```text
13-test-infrastructure.capability/  ← Built first (harnesses)
15-validation.capability/           ← Depends on harnesses
20-auth.capability/                 ← Can parallel with 20-billing
20-billing.capability/              ← Same BSP = parallel safe
```

**ALWAYS use full path when referencing work items:**

| Wrong                  | Correct                                                          |
| ---------------------- | ---------------------------------------------------------------- |
| "story-54"             | "15-validation.capability/21-testable.feature/47-commands.story" |
| "implement feature-21" | "implement 15-validation.capability/21-testable.feature"         |

---

## When to Invoke Skills

### Before Implementing ANY Work Item → `/spx:understanding-spx`

**BLOCKING REQUIREMENT**

**Trigger conditions:**

- User says "implement story-NN", "work on feature-NN", or "build capability-NN"
- User references a work item file
- You're about to write implementation code

**What it does**: Loads complete context hierarchy (PRD → ADRs → capability → feature → story).

### When Creating/Organizing Specs → `/spx:managing-spx`

**BLOCKING REQUIREMENT**

**Trigger conditions:**

- User says "create a PRD", "add an ADR", "create capability/feature/story"
- User asks "what's next to work on?"
- You need templates or BSP numbering rules

**What it does**: Provides templates, BSP numbering, structure guidance.

---

## Quick Reference: Skill Selection

| User Says...         | Invoke                   | Do NOT                 |
| -------------------- | ------------------------ | ---------------------- |
| "Implement story-21" | `/spx:understanding-spx` | Read story.md directly |
| "Create a PRD"       | `/spx:managing-spx`      | Search for templates   |
| "What's next?"       | `/spx:managing-spx`      | Grep for work items    |
| "Create a feature"   | `/spx:managing-spx`      | Calculate BSP yourself |

---

## Test Naming Convention

Test level is in the filename suffix:

| Level       | Suffix                       | What It Tests                                      |
| ----------- | ---------------------------- | -------------------------------------------------- |
| Unit        | `*.unit.test.{ts,py}`        | Pure logic, no external dependencies               |
| Integration | `*.integration.test.{ts,py}` | Real dependencies (databases, binaries, harnesses) |
| E2E         | `*.e2e.test.{ts,py}`         | Complete user workflows, real credentials          |

**Any test level can exist at any container level.** A capability may have unit tests; a story may have integration tests. The level describes what KIND of test, not where it lives.

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
├── todo/          # Available for /pickup
├── doing/         # Currently claimed
└── archive/       # Completed sessions
```

Use `/handoff` to create, `/pickup` to claim.
