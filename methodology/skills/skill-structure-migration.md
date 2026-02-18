# Skill Structure - Migration Extension

## Purpose

This document defines the optional migration-specific skill structure for products that need to migrate legacy `specs/` artifacts into `spx/`.

Products that do not require migration should use only:

- `methodology/skills/skill-structure.md`

## Migration intent model

## 6. Migrate legacy artifacts to SPX

6a. Migrate legacy `specs/` artifacts and tests to `spx/` structure.
6b. Verify migration correctness (coverage parity, file movement, corrected migration records).

## Entry-point migration skill map

| Entry-point skill       | Primary intent coverage | Role                                                                               |
| ----------------------- | ----------------------- | ---------------------------------------------------------------------------------- |
| `understanding-spx`     | Foundation preflight    | Load methodology/structure foundation                                              |
| `contextualizing-spx`   | Context preflight       | Load migration-boundary context artifacts and verify required files                |
| `migrating-spec-to-spx` | 6                       | Execute migration workflow with verification gates and corrected migration records |

## Migration preflight contract

1. Check foundation packet:
   - `foundation_loaded`
   - `foundation_version`
   - `foundation_scope`
2. If missing or stale, invoke `understanding-spx` foundation mode.
3. Check target context packet:
   - `target_context_loaded`
   - `target_path`
   - `operation_type`
   - `context_version`
   - `tree_revision`
4. If missing or stale, invoke `contextualizing-spx` for the migration boundary.

## Migration conversational flow

### `migrating-spec-to-spx`

1. Intake migration scope and boundary.
2. Run migration preflight (foundation + contextualized intake).
3. Execute migration with verification gates.
4. Validate parity and structure correctness.
5. Produce corrected migration records and cleanup summary.
