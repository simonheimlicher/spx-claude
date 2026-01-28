# Document Types and Requirements

Complete reference of all document types in the CODE framework hierarchy and their requirements.

## Work Item Hierarchy

```
Product
└── Capability (E2E level)
    ├── {slug}.capability.md     REQUIRED
    ├── {topic}.prd.md           OPTIONAL (enrichment)
    ├── NN-{slug}.adr.md         OPTIONAL (interleaved)
    ├── pass.csv                 Test verification ledger
    ├── tests/                   OPTIONAL (co-located tests)
    └── Feature (Integration level)
        ├── {slug}.feature.md    REQUIRED
        ├── NN-{slug}.adr.md     OPTIONAL (interleaved)
        ├── pass.csv             Test verification ledger
        ├── tests/               OPTIONAL (co-located tests)
        └── Story (Unit level)
            ├── {slug}.story.md  REQUIRED
            ├── pass.csv         Test verification ledger
            └── tests/           OPTIONAL (co-located tests)
```

## Document Types

### Product-Level Documents

**Location**: `spx/`

| Document      | Pattern                 | Required? | Purpose                              |
| ------------- | ----------------------- | --------- | ------------------------------------ |
| Project Guide | `CLAUDE.md`             | YES       | Project structure and navigation     |
| Product ADRs  | `NN-{slug}.adr.md`      | NO        | Product-wide architectural decisions |
| Product PRD   | `{product-name}.prd.md` | NO        | Optional product-wide requirements   |

### Capability-Level Documents

**Location**: `spx/NN-{slug}.capability/`

| Document        | Pattern                | Required?     | Purpose                       |
| --------------- | ---------------------- | ------------- | ----------------------------- |
| Capability Spec | `{slug}.capability.md` | YES           | E2E scenario definition       |
| PRD             | `{topic}.prd.md`       | NO (optional) | Product requirements catalyst |
| Capability ADRs | `NN-{slug}.adr.md`     | NO            | Capability-scoped decisions   |
| Pass Ledger     | `pass.csv`             | NO            | Test verification state       |
| Tests           | `tests/`               | NO            | Co-located tests              |

**Note**: PRD is optional enrichment. If PRD exists but spec is missing, offer to create spec from PRD.

### Feature-Level Documents

**Location**: `spx/NN-{slug}.capability/NN-{slug}.feature/`

| Document     | Pattern             | Required? | Purpose                         |
| ------------ | ------------------- | --------- | ------------------------------- |
| Feature Spec | `{slug}.feature.md` | YES       | Integration scenario definition |
| Feature ADRs | `NN-{slug}.adr.md`  | NO        | Feature-scoped decisions        |
| Pass Ledger  | `pass.csv`          | NO        | Test verification state         |
| Tests        | `tests/`            | NO        | Co-located tests                |

**Note**: Technical details belong in feature.md, not separate TRD documents.

### Story-Level Documents

**Location**: `spx/.../NN-{slug}.story/`

| Document    | Pattern           | Required? | Purpose                          |
| ----------- | ----------------- | --------- | -------------------------------- |
| Story Spec  | `{slug}.story.md` | YES       | Atomic implementation definition |
| Pass Ledger | `pass.csv`        | NO        | Test verification state          |
| Tests       | `tests/`          | NO        | Co-located tests                 |

**Note**: Stories do NOT have their own ADRs. They inherit decisions from parent feature/capability.

## Document Content Requirements

### Specification Files (.capability.md, .feature.md, .story.md)

**Must contain**:

- **Purpose**: What this container delivers and why it matters
- **Requirements**: Functional and quality requirements
- **Outcomes**: Numbered Gherkin scenarios with test file tables

**May contain**:

- **Test Strategy**: Component/level/harness/rationale table
- **Architectural Constraints**: References to applicable ADRs
- **Analysis** (stories only): Files, constants, config examined

### Requirements Documents (PRD)

**PRD (Product Requirements Document)**:

- User value proposition
- Customer journey
- Measurable outcomes (X% improvement targets)
- Acceptance tests (BDD scenarios)

### Architectural Decision Records (ADR)

**Must contain**:

- **Context**: Why this decision is needed
- **Decision**: What is being decided
- **Consequences**: Trade-offs and implications
- **Compliance**: How adherence will be verified

## PRD as Optional Enrichment

PRD documents are **optional** at all levels. The spec file (`.capability.md`, `.feature.md`, `.story.md`) is the only required document for each work item.

**When PRD exists without spec file:**

If a PRD exists at a level but the corresponding spec file is missing, offer to create the spec from the requirements document. This handles cases where work was initiated from a requirements document but the spec wasn't yet created.

## Status Determination

**Use CLI commands to check status (do NOT manually inspect directories):**

```bash
# View project status
spx spec status --format table

# Get next work item (respects BSP ordering)
spx spec next
```

Status is derived from `pass.csv` state, not directory location.

## Test Co-location (CODE Framework)

Tests are co-located with their specs in `spx/.../tests/`. The `pass.csv` ledger tracks which tests pass.

| Level      | Location                           | Test Suffix             |
| ---------- | ---------------------------------- | ----------------------- |
| Capability | `spx/NN-{slug}.capability/tests/`  | `*.e2e.test.ts`         |
| Feature    | `spx/.../NN-{slug}.feature/tests/` | `*.integration.test.ts` |
| Story      | `spx/.../NN-{slug}.story/tests/`   | `*.unit.test.ts`        |

## BSP Numbering

**BSP** (Build Sequence Position): Two-digit numbers (10-99) with `-` separator

**Rules**:

- Lower number = must complete first
- Hyphen (`-`) separates BSP from slug: `NN-{slug}.{type}/`
- Use `@` for recursive insertion when no integer space: `20@54-audit.capability/`
- Never rename existing numbers—insert between them

**Examples**:

- `10-core-cli.capability/` must complete before `20-advanced-features.capability/`
- `30-build.story/` must complete before `40-test.story/`
