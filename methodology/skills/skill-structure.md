# Skill Structure

## Design principles

- Keep very few entry-point skills (router skills).
- Keep one foundational methodology skill: `understanding-spx`.
- Keep one target-ingestion skill: `contextualizing-spx`.
- Treat tests as part of the product contract, not implementation afterthoughts.
- Make conversational flow explicit and consistent across entry-point skills.
- Keep migration concerns in a separate optional structure document.
- Split context loading into:
  - Understanding methodology and structure (foundation)
  - Ingesting targeted artifacts for the active task (context intake)

## Intent model (use cases)

## 1. Understand Outcome Engineering context

1a. Systematically ingest context to prepare for a discussion with the user.
1b. Systematically ingest context to prepare for autonomous work.

## 2. Author Outcome Engineering artifacts

2a. Author from scratch from user conversation/prompt, including clarifying questions.
2b. Extend existing artifacts with new requirements, outcomes, or decisions.

## 3. Decompose Outcome Engineering artifacts

3a. Deterministically decompose existing higher-level artifacts to lower levels.

## 4. Review and refactor Outcome Engineering artifacts

4a. Review and structurally refactor (move/re-scope content) through user conversation.
4b. Factor common aspects up to higher-level requirement/decision artifacts.
4c. Clarify/augment/align/deconflict artifacts while preserving product truth.

## 5. Contract test lifecycle based on Outcome Engineering context

5a. Create tests from outcome contracts in existing specs.
5b. Refactor tests when outcome contracts or decisions change.
5c. Update spec artifacts with test file references and adjust outcomes when new test evidence reveals gaps.
5d. Invoke `understanding-spx` first, then invoke `/testing-[language]` for language-specific test design and implementation.

## Optional migration extension

For products with legacy `specs/` migration needs, use:

- `methodology/skills/skill-structure-migration.md`

## Entry-point skill map

| Entry-point skill      | Primary intent coverage | Role                                                                                           |
| ---------------------- | ----------------------- | ---------------------------------------------------------------------------------------------- |
| `understanding-spx`    | 1                       | Foundation methodology gateway before any authoring/decomposition/testing work                 |
| `contextualizing-spx`  | Preflight intake        | Thin context-intake router that determines and loads target-specific artifacts                 |
| `authoring-spx`        | 2                       | Conversation-driven authoring and extension of PRD/ADR/PDR/capability/feature/story artifacts  |
| `decomposing-spx`      | 3                       | Deterministic decomposition router (PRD->capability->feature->story)                           |
| `evolving-spx`         | 4                       | Review/refactor/align/deconflict existing artifacts (active change skill, not advisory oracle) |
| `testing-spx-contract` | 5                       | Contract-test creation/refactoring synchronized with spec outcomes and test references         |

## Foundation ownership model

- `understanding-spx` owns all Outcome Engineering foundation content:
  - durable map worldview
  - decomposition semantics
  - structure/numbering rules
- `contextualizing-spx` owns target context ingestion:
  - determines which files are required for the exact target and operation
  - validates artifact existence/read order
  - returns context manifest or abort/remediation
- Other entry-point skills do not duplicate this content.
- Other entry-point skills must invoke:
  - `understanding-spx` preflight if foundation context is absent or stale
  - `contextualizing-spx` preflight if target context is absent or stale

## Preflight state contract

Each entry-point skill checks for a foundation context packet before doing work:

- `foundation_loaded`: boolean
- `foundation_version`: version/hash/timestamp for `understanding-spx`
- `foundation_scope`: what methodology modules were loaded
- `target_context_loaded`: boolean
- `target_path`: full target path
- `operation_type`: author/decompose/evolve/test
- `scope_depth`: capability/feature/story/decision
- `context_version`: version/hash/timestamp for `contextualizing-spx`
- `tree_revision`: freshness marker for spec tree snapshot

Decision rule:

- If `foundation_loaded` is false -> invoke `understanding-spx` foundation mode.
- If `foundation_version` is stale -> re-invoke `understanding-spx` foundation mode.
- If `target_context_loaded` is false -> invoke `contextualizing-spx`.
- If `target_path` or `operation_type` changed -> re-invoke `contextualizing-spx`.
- If `context_version` or `tree_revision` is stale -> re-invoke `contextualizing-spx`.

## Conversational flow contract for entry-point skills

Every entry-point skill should follow this high-level interaction contract:

1. **Intake**
   - Ask for target path/scope and intended operation.
   - Detect mode (discussion, authoring, decomposition, refactor, testing).
2. **Foundation gate**
   - Check foundation context packet.
   - Invoke `understanding-spx` foundation mode only when missing/stale.
3. **Target context gate**
   - Invoke `contextualizing-spx` for active artifacts.
   - Abort with explicit remediation if required artifacts are missing.
4. **Plan**
   - Present concise execution plan and expected outputs.
5. **Execute**
   - Perform workflow steps for the selected mode.
   - Keep user in the loop at major decision points.
6. **Verify**
   - Run mode-specific verification checks.
   - Show pass/fail status and unresolved risks.
7. **Deliver**
   - Summarize changes, decisions, and next actions.

## Mode-specific conversational flows

### `understanding-spx`

1. Intake foundation mode.
2. Load Outcome Engineering methodology + structure semantics.
3. Return structured context package:
   - foundation invariants
   - version/freshness metadata

### `contextualizing-spx`

1. Intake target path/scope + operation type.
2. Determine required artifact set and read order.
3. Validate required artifacts exist and are readable.
4. Return context manifest:
   - artifact-level constraints
   - open decisions
   - readiness/abort status
   - context version/freshness metadata

### `authoring-spx`

1. Intake artifact type + intended level + path/location.
2. Check foundation packet; invoke `understanding-spx` foundation mode if needed.
3. Check target context packet; invoke `contextualizing-spx` for active scope if needed.
4. Clarify user intent and unresolved product decisions.
5. Draft artifact using shared templates and framework rules.
6. Validate atemporal voice, consistency, and testability.
7. Return draft + open decisions + recommended next decomposition/testing steps.

### `decomposing-spx`

1. Intake source artifact and target decomposition depth.
2. Check foundation packet; invoke `understanding-spx` foundation mode if needed.
3. Check target context packet; invoke `contextualizing-spx` for source artifacts if needed.
4. Apply decomposition rules deterministically (scope, outcomes, BSP ordering).
5. Produce child artifacts with explicit boundaries and dependencies.
6. Validate decomposition quality (size constraints, level correctness, no misplaced outcomes).
7. Return decomposition output + rationale for splits/boundaries.

### `evolving-spx`

1. Intake change request against existing artifacts.
2. Check foundation packet; invoke `understanding-spx` foundation mode if needed.
3. Check target context packet; invoke `contextualizing-spx` for affected hierarchy if needed.
4. Analyze impact across hierarchy and decision records.
5. Propose change set (moves, consolidations, rewrites, deconflicts).
6. Apply refactor/clarification updates.
7. Validate cross-artifact consistency and report unresolved conflicts.

### `testing-spx-contract`

1. Intake target work item(s) and language.
2. Check foundation packet; invoke `understanding-spx` foundation mode if needed.
3. Check target context packet; invoke `contextualizing-spx` for active work items if needed.
4. Invoke `/testing-[language]` for test-level design and implementation.
5. Create or refactor tests as contract evidence.
6. Update spec test references and adjust outcomes where evidence reveals mismatch.
7. Return evidence summary (what is now proven, what remains pending).
