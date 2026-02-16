# Skill Structure

## Design principles

- Keep very few skills organized in three layers: foundation, action, contract.
- Foundation skills load once per conversation using marker pattern (no persistent state).
- `understanding-spx` is the single shared library: methodology, structure, templates.
- `contextualizing-spx` handles target-specific artifact ingestion.
- `testing-spx` enforces spec-test consistency both as user entry point and as postflight gate.
- Action skills check for foundation markers before working; invoke foundations if absent.
- Make conversational flow explicit and consistent across action skills.
- Keep migration concerns in a separate optional structure document.

## Intent model (use cases)

### 1. Understand Outcome Engineering context

1a. Systematically ingest context to prepare for a discussion with the user.
1b. Systematically ingest context to prepare for autonomous work.

### 2. Author Outcome Engineering artifacts

2a. Author from scratch from user conversation/prompt, including clarifying questions.
2b. Extend existing artifacts with new requirements, outcomes, or decisions.

### 3. Decompose Outcome Engineering artifacts

3a. Systematically decompose existing higher-level artifacts to lower levels.

### 4. Refactor Outcome Engineering artifacts

4a. Review and structurally refactor (move/re-scope content) through user conversation.
4b. Factor common aspects up to higher-level requirement/decision artifacts.

### 5. Align Outcome Engineering artifacts

5a. Clarify/augment/align/deconflict artifacts while preserving product truth.

### 6. Contract test lifecycle based on Outcome Engineering context

6a. Create tests from assertion contracts in existing specs.
6b. Refactor tests when assertion contracts or decisions change.
6c. Update spec artifacts with test file references and adjust outcomes when new test evidence reveals gaps.

### Optional migration extension

For products with legacy `specs/` migration needs, use:

- `methodology/skills/skill-structure-migration.md`

## Skill map

### Foundation layer

Foundation skills load once per conversation. They emit conversation markers so other skills can detect whether foundation context is present.

| Skill                 | Owns                                                                                         | Marker                               |
| --------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------ |
| `understanding-spx`   | Methodology, durable map worldview, decomposition semantics, BSP rules, all shared templates | `<UNDERSTANDING_SPX>`                |
| `contextualizing-spx` | Target artifact ingestion, read order validation, abort/remediation                          | `<CONTEXTUALIZING_SPX target="...">` |

### Action layer

Action skills do the work. Before starting, they check conversation history for foundation markers and invoke missing foundations.

| Skill             | Use case | Scope                                                                |
| ----------------- | -------- | -------------------------------------------------------------------- |
| `authoring-spx`   | 2        | Create/extend PRD/ADR/PDR/capability/feature/story from conversation |
| `decomposing-spx` | 3        | Systematically decompose higher-level artifacts to lower levels      |
| `refactoring-spx` | 4        | Structural moves, re-scoping, factoring common aspects up            |
| `aligning-spx`    | 5        | Clarify, augment, align, deconflict while preserving product truth   |

### Contract layer

The contract skill is both a user-invocable entry point (use case 6) and a postflight gate that action skills trigger after making changes.

| Skill         | Use case | When invoked                                                 |
| ------------- | -------- | ------------------------------------------------------------ |
| `testing-spx` | 6        | User invokes directly OR action skills trigger after changes |

## Foundation ownership model

- **`understanding-spx`** is the single shared library for all Outcome Engineering knowledge:
  - Durable map worldview (specs are permanent product documentation)
  - Decomposition semantics (what belongs at capability/feature/story level)
  - Structure and BSP numbering rules
  - All shared templates (PRD, ADR, PDR, capability, feature, story)
  - Template access instructions
- **`contextualizing-spx`** owns target context ingestion:
  - Determines which files are required for the exact target and operation
  - Validates artifact existence and read order
  - Returns context manifest or abort with remediation
  - Bootstrap mode: returns empty manifest with `bootstrap=true` when authoring into an empty tree (no abort)
- **`testing-spx`** owns spec-test consistency:
  - Contract between outcomes in specs and tests that prove them
  - Invokes `/testing-[language]` for language-specific test design
  - Updates spec artifacts with test file references
  - Adjusts outcomes when test evidence reveals gaps
- **Action skills do not duplicate foundation content.** They reference `understanding-spx` for templates and methodology.

## Marker-based state detection

Foundation skills emit XML markers into the conversation when loaded. Action skills search conversation history for these markers before starting work. This follows the same pattern as `/pickup` emitting `<PICKUP_ID>` for `/handoff` to find.

| Marker                                     | Emitted by            | Checked by                     | Meaning                              |
| ------------------------------------------ | --------------------- | ------------------------------ | ------------------------------------ |
| `<UNDERSTANDING_SPX>`                      | `understanding-spx`   | All action and contract skills | Methodology and templates are loaded |
| `<CONTEXTUALIZING_SPX target="full/path">` | `contextualizing-spx` | All action and contract skills | Target artifacts are loaded          |

**Decision rule:**

- No `<UNDERSTANDING_SPX>` in conversation → invoke `understanding-spx`
- No `<CONTEXTUALIZING_SPX>` matching current target → invoke `contextualizing-spx`
- Target path changed since last `<CONTEXTUALIZING_SPX>` → re-invoke `contextualizing-spx`

## Template ownership

`understanding-spx` owns all templates. Action skills access them via the foundation skill's base directory:

```text
${UNDERSTANDING_SPX_DIR}/
├── SKILL.md
├── references/
│   ├── durable-map.md
│   ├── decomposition-semantics.md
│   ├── assertion-types.md
│   └── ...
└── templates/
    ├── product/
    │   └── product.prd.md
    ├── decisions/
    │   ├── architectural-decision.adr.md
    │   └── product-decision.pdr.md
    └── outcomes/
        ├── capability-name.capability.md
        ├── feature-name.feature.md
        └── story-name.story.md
```

Action skills reference templates with: `Read: ${UNDERSTANDING_SPX_DIR}/templates/outcomes/story-name.story.md`

## Conversational flow contract

Every action skill follows this interaction contract:

1. **Intake** — Ask for target path/scope and intended operation.
2. **Foundation gate** — Check for `<UNDERSTANDING_SPX>` marker; invoke `understanding-spx` if absent.
3. **Target context gate** — Check for `<CONTEXTUALIZING_SPX>` matching target; invoke `contextualizing-spx` if absent or mismatched. Abort with explicit remediation if required artifacts are missing.
4. **Plan** — Present concise execution plan and expected outputs.
5. **Execute** — Perform workflow steps. Keep user in the loop at major decision points.
6. **Contract gate** — Invoke `testing-spx` to synchronize spec-test consistency.
7. **Deliver** — Summarize changes, decisions, and next actions.

## Mode-specific flows

Each flow documents only what is unique to that mode. All action skills share the standard preflight (steps 1-3) and postflight (steps 6-7) from the conversational flow contract above.

### `understanding-spx`

1. Load Outcome Engineering methodology, structure semantics, and template index.
2. Emit `<UNDERSTANDING_SPX>` marker with loaded module summary.

### `contextualizing-spx`

1. Intake target path/scope and operation type.
2. Determine required artifact set and read order.
3. Validate required artifacts exist and are readable.
4. If operation is `author` and no artifacts exist at target level, return empty manifest with `bootstrap=true` instead of aborting.
5. Emit `<CONTEXTUALIZING_SPX target="full/path">` with context manifest: artifact-level constraints, open decisions, readiness status.

### `authoring-spx`

1. Intake artifact type, intended level, and path/location.
2. Clarify user intent and unresolved product decisions.
3. Draft artifact using templates from `understanding-spx` and framework rules.
4. Validate atemporal voice, consistency, and testability.
5. Return draft, open decisions, and recommended next steps (decomposition or testing).

### `decomposing-spx`

1. Intake source artifact and target decomposition depth.
2. Apply decomposition methodology systematically (scope, outcomes, BSP ordering).
3. Produce child artifacts with explicit boundaries and dependencies.
4. Validate decomposition quality (size constraints, level correctness, no misplaced outcomes).
5. Return decomposition output with rationale for splits and boundaries.

### `refactoring-spx`

1. Intake structural change request (move, re-scope, factor up).
2. Analyze impact across hierarchy and decision records.
3. Propose structural change set (moves, consolidations, new parent artifacts).
4. Apply refactoring updates.
5. Validate cross-artifact consistency after structural changes.

### `aligning-spx`

1. Intake alignment request (clarify, augment, deconflict).
2. Analyze contradictions, gaps, or ambiguities across affected artifacts.
3. Propose alignment changes with rationale.
4. Apply clarification or deconfliction updates.
5. Validate cross-artifact consistency and report unresolved conflicts.

### `testing-spx`

When invoked directly (use case 6):

1. Intake target work item(s) and language.
2. Invoke `/testing-[language]` for test-level design and implementation.
3. Create or refactor tests as contract evidence.
4. Update spec test references and adjust outcomes where evidence reveals mismatch.
5. Return evidence summary (what is now proven, what remains pending).

When invoked as postflight by action skills:

1. Receive change summary from calling skill.
2. Determine which specs were affected and whether test references are stale.
3. If tests need creation or update, invoke `/testing-[language]`.
4. Update spec test references and flag outcomes that lack evidence.
5. Return consistency status to calling skill.

## Current skills disposition

| Current skill                           | Disposition                                                                 |
| --------------------------------------- | --------------------------------------------------------------------------- |
| `understanding-durable-map`             | Absorbed into `understanding-spx` as reference material                     |
| `understanding-assertion-decomposition` | Absorbed into `understanding-spx` as reference material                     |
| `managing-spx`                          | Split: templates/structure → `understanding-spx`, workflows → action skills |
| `decomposing-prd-to-capabilities`       | Absorbed into `decomposing-spx`                                             |
| `decomposing-capability-to-features`    | Absorbed into `decomposing-spx`                                             |
| `decomposing-feature-to-stories`        | Absorbed into `decomposing-spx`                                             |
| `writing-prd`                           | Absorbed into `authoring-spx`                                               |
| `migrating-spec-to-spx`                 | Stays, moved to migration extension                                         |
| `understanding-spx` (current)           | Rewritten as foundation skill                                               |
