# Document Types and Requirements

Complete reference of all document types in the spec-driven development hierarchy and their requirements.

## Work Item Hierarchy

```
Product
└── Capability (E2E level)
    ├── {slug}.capability.md     ✅ REQUIRED
    ├── {topic}.prd.md           ✅ REQUIRED (strict mode)
    ├── decisions/
    │   └── adr-NNN_{slug}.md    ⚠️ OPTIONAL
    ├── tests/                   ⚠️ OPTIONAL (for progress tests)
    └── Feature (Integration level)
        ├── {slug}.feature.md    ✅ REQUIRED
        ├── {topic}.trd.md       ✅ REQUIRED (strict mode)
        ├── decisions/
        │   └── adr-NNN_{slug}.md ⚠️ OPTIONAL
        ├── tests/               ⚠️ OPTIONAL (for progress tests)
        └── Story (Unit level)
            ├── {slug}.story.md  ✅ REQUIRED
            └── tests/           ⚠️ OPTIONAL (determines status)
```

## Document Types

### Product-Level Documents

**Location**: `specs/`

| Document      | Pattern                 | Required? | Purpose                              |
| ------------- | ----------------------- | --------- | ------------------------------------ |
| Project Guide | `CLAUDE.md`             | ✅ YES    | Project structure and navigation     |
| Product ADRs  | `decisions/adr-*.md`    | ⚠️ NO     | Product-wide architectural decisions |
| Product PRD   | `{product-name}.prd.md` | ⚠️ NO     | Optional product-wide requirements   |

### Capability-Level Documents

**Location**: `specs/work/{status}/capability-NN_{slug}/`

| Document        | Pattern                  | Required?                 | Purpose                          |
| --------------- | ------------------------ | ------------------------- | -------------------------------- |
| Capability Spec | `{slug}.capability.md`   | ✅ YES                    | E2E scenario definition          |
| PRD             | `{topic}.prd.md`         | ✅ YES (strict mode)      | Product requirements catalyst    |
| Capability ADRs | `decisions/adr-NNN_*.md` | ⚠️ NO                     | Capability-scoped decisions      |
| Tests           | `tests/`                 | ⚠️ NO (determines status) | Progress tests before graduation |

**Strict Mode**: PRD must exist at capability level

### Feature-Level Documents

**Location**: `specs/work/{status}/capability-NN_{slug}/feature-NN_{slug}/`

| Document     | Pattern                  | Required?                 | Purpose                          |
| ------------ | ------------------------ | ------------------------- | -------------------------------- |
| Feature Spec | `{slug}.feature.md`      | ✅ YES                    | Integration scenario definition  |
| TRD          | `{topic}.trd.md`         | ✅ YES (strict mode)      | Technical requirements catalyst  |
| Feature ADRs | `decisions/adr-NNN_*.md` | ⚠️ NO                     | Feature-scoped decisions         |
| Tests        | `tests/`                 | ⚠️ NO (determines status) | Progress tests before graduation |

**Strict Mode**: TRD must exist at feature level

### Story-Level Documents

**Location**: `specs/work/{status}/.../story-NN_{slug}/`

| Document   | Pattern           | Required?                 | Purpose                          |
| ---------- | ----------------- | ------------------------- | -------------------------------- |
| Story Spec | `{slug}.story.md` | ✅ YES                    | Atomic implementation definition |
| Tests      | `tests/`          | ⚠️ NO (determines status) | Progress tests before graduation |
| Completion | `tests/DONE.md`   | ⚠️ NO (signals DONE)      | Evidence of completion           |

**Note**: Stories do NOT have their own ADRs. They inherit decisions from parent feature/capability.

## Document Content Requirements

### Specification Files (.capability.md, .feature.md, .story.md)

**Must contain**:

- **Functional Requirements**: What the work item delivers
- **Acceptance Criteria**: How to know it's complete
- **Dependencies**: BSP dependencies on other work items

**May contain**:

- **Context**: Background and rationale
- **Implementation Notes**: Guidance for implementers

### Requirements Documents (PRD/TRD)

**PRD (Product Requirements Document)**:

- User value proposition
- Customer journey
- Measurable outcomes (X% improvement targets)
- Acceptance tests (BDD scenarios)

**TRD (Technical Requirements Document)**:

- System architecture
- Validation strategy with test levels
- Test infrastructure requirements
- BDD scenarios at appropriate levels
- Infrastructure gaps (if any)

### Architectural Decision Records (ADR)

**Must contain**:

- **Context**: Why this decision is needed
- **Decision**: What is being decided
- **Consequences**: Trade-offs and implications
- **Compliance**: How adherence will be verified
- **Testing Strategy**: Test levels for components

## Strict Mode

**Enabled by default** for this skill.

| Level      | Strict Mode Requirement            |
| ---------- | ---------------------------------- |
| Capability | PRD must exist                     |
| Feature    | TRD must exist                     |
| Story      | (inherits from feature/capability) |

**Rationale**: "Optional" PRD/TRD should exist to document the catalyst for capability/feature. Strict mode enforces complete documentation.

## Status Determination

Work item status is determined by presence of tests and DONE.md:

| Condition                     | Status      |
| ----------------------------- | ----------- |
| No `tests/` directory         | OPEN        |
| `tests/` exists, no `DONE.md` | IN_PROGRESS |
| `DONE.md` exists in tests/    | DONE        |

## Test Graduation Paths

Tests migrate from progress location to regression location when work item is DONE:

| Level      | Progress Location                | Regression Location  |
| ---------- | -------------------------------- | -------------------- |
| Capability | `specs/.../capability-NN/tests/` | `tests/e2e/`         |
| Feature    | `specs/.../feature-NN/tests/`    | `tests/integration/` |
| Story      | `specs/.../story-NN/tests/`      | `tests/unit/`        |

## BSP Numbering

**BSP** (Backlog Sequencing Priority): Two-digit numbers (10-99)

**Rules**:

- Lower number = must complete first
- Used for dependency ordering at all levels
- Gaps allowed (10, 20, 30... or 11, 12, 13...)
- Never reuse numbers

**Examples**:

- `capability-10_core-cli` must complete before `capability-20_advanced-features`
- `story-30_build` must complete before `story-40_test`
