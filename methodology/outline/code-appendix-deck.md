# Appendix: Customer Outcome Driven Engineering (CODE)

Everything below is implementation detail. It matters, but it should serve the operating system—not become it.

---

## Appendix A — The Product Tree model

### A1. The canonical structure

A Product Tree is a hierarchical structure:

- **Product** (trunk)
- **Capabilities** (major branches)
- **Features** (sub-branches)
- **Stories** (leaves)
- **Scenarios** (behavior contracts)

A strict concreteness constraint applies:

- if a story cannot be expressed as Given/When/Then scenarios, it is not ready for delivery

---

## Appendix B — BSP numbering for dependency order

### B1. Rationale

BSP (Binary Space Partitioning) encodes dependency order while allowing insertion without renumbering everything.

- lower number = dependency
- same number = parallel / independent
- higher number = depends on lower numbers completing

### B2. Rules

| Rule                   | Value                                          |
| ---------------------- | ---------------------------------------------- |
| Range                  | [10, 99] (two digits)                          |
| Meaning                | Lower BSP = dependency, same BSP = independent |
| First item             | Start at 21 (room for ~10 before)              |
| Insert between X and Y | `floor((X + Y) / 2)`                           |
| Append after X         | `floor((X + 99) / 2)`                          |

### B3. Parallel work pattern

Items with the same BSP number can be worked on in parallel:

21-setup.story/
37-auth.story/
37-profile.story/
37-settings.story/
54-integration.story/

### B4. Strategic insertion patterns

| Pattern               | Strategy             | When to use                                           |
| --------------------- | -------------------- | ----------------------------------------------------- |
| Discovered dependency | Insert LOWER number  | You realize work needs a harness/system first         |
| Improvement / polish  | Insert HIGHER number | You want to add quality improvements after core works |

---

## Appendix C — Naming and layout conventions

### C1. Directory naming

{BSP}-{slug}.{type}/

### C2. Spec file naming

Inside each container:

{slug}.{type}.md

### C3. ADR naming

Flat files interleaved with containers:

NN-{slug}.adr.md

### C4. Unified number space

Within a container, capabilities/features/stories/ADRs share one BSP sequence, so sorting shows dependency order without extra metadata.

---

## Appendix D — Example Product Tree layout

```text
spx/
├── product.prd.md
├── 21-core-decision.adr.md
│
├── 21-test-harness.capability/
│   ├── test-harness.capability.md
│   ├── outcomes.yaml
│   └── tests/
│
├── 37-users.capability/
│   ├── users.capability.md
│   ├── outcomes.yaml
│   │
│   ├── 10-bootstrap.feature/
│   │   └── bootstrap.feature.md
│   │
│   ├── 21-auth-strategy.adr.md
│   │
│   ├── 22-login.feature/
│   │   ├── login.feature.md
│   │   ├── outcomes.yaml
│   │   │
│   │   ├── 21-password-hashing.adr.md
│   │   ├── 22-hash-password.story/
│   │   │   ├── hash-password.story.md
│   │   │   ├── outcomes.yaml
│   │   │   └── tests/
│   │   │       └── hash-password.unit.test.ts
│   │   │
│   │   └── 22-verify-password.story/
│   │       ├── verify-password.story.md
│   │       └── tests/
│   │
│   └── 37-profile.feature/
│       └── profile.feature.md
│
├── 37-billing.capability/
│   └── billing.capability.md
│
└── 54-linter.capability/
    └── linter.capability.md
```

Key observations:

- BSP number sorts dependency order visually
- status is derived from outcomes.yaml evidence, not a separate status field
- pruning a branch removes dependent work without orphaning tickets

⸻

## Appendix E — States derived from evidence

### E1. Container states

| State         | Condition                         | Required Action     |
| ------------- | --------------------------------- | ------------------- |
| **Unknown**   | No tests exist                    | Write tests         |
| **Pending**   | Tests exist, not all claimed      | Fix code or claim   |
| **Stale**     | Descendant outcomes_blob mismatch | Re-claim            |
| **Passing**   | All tests pass, blobs match       | None                |
| **Regressed** | Claimed test fails                | Investigate and fix |

States are mutually exclusive. Every container is in exactly one state.

⸻

## Appendix F — Momentum metrics

### F1. Tree health indicators

Metric What it measures
Realization Rate Stories moving from Pending → Passing per week
Drift % of Passing stories that Regressed this week
Potential Energy Count of Pending + Stale stories
Coverage Depth % of capabilities with passing integration tests

### F2. Drift interpretation

Drift Rate Interpretation
< 1% Well-isolated architecture
1–5% Normal coupling, monitor trends
5–10% High coupling, consider refactoring

> 10% Brittle architecture, intervene

⸻

## Appendix G — Spec formats (contracts + analysis)

### G1. Capability / Feature spec format

```markdown
# {Capability|Feature}: {Name}

## Purpose

What this container delivers and why it matters.

## Requirements

Functional and quality requirements. Constraints tests must verify.

## Test Strategy

| Component        | Level | Harness     | Rationale                    |
| ---------------- | ----- | ----------- | ---------------------------- |
| Argument parsing | 1     | -           | Pure function                |
| CLI integration  | 2     | cli-harness | Needs real binary            |
| Full workflow    | 3     | e2e-harness | Needs end-to-end environment |

### Escalation Rationale

- 1 → 2: what confidence does Level 2 add?
- 2 → 3: what confidence does Level 3 add?

## Outcomes

### 1. [Scenario name]

GIVEN ...
WHEN ...
THEN ...

## Architectural Constraints

| ADR | Constraint |
| --- | ---------- |
| ... | ...        |
```

### G2. Story spec format

````markdown
# Story: {Name}

## Purpose

What this story delivers and why it matters.

## Outcomes

### 1. {Outcome name}

```gherkin
GIVEN ...
WHEN ...
THEN ...
AND ...

Test Files

File Level Harness
… … …

Analysis
Implementation may diverge as understanding deepens.

File Intent
… …

Constant Intent
… …

Config Parameter Test Values Expected Behavior
… … …

Architectural Constraints

ADR Constraint
… …
```
````

---

## Appendix H — Evidence ledger: outcomes.yaml

### H1. Format

```yaml
tests:
  - file: parsing.unit.test.ts
    passed_at: 2026-01-27T10:30:00Z
  - file: cli.integration.test.ts
    passed_at: 2026-01-28T14:15:00Z
descendants:
  - path: 10-parse-args.story/
    outcomes_blob: a3f2b7c
  - path: 22-validate-input.story/
    outcomes_blob: 9bc4e1d
```

| Field                         | Description                                    |
| ----------------------------- | ---------------------------------------------- |
| `tests[].file`                | Test filename relative to container's `tests/` |
| `tests[].passed_at`           | ISO 8601 timestamp when test passed            |
| `descendants[].path`          | Child container directory name                 |
| `descendants[].outcomes_blob` | Git blob SHA of child's outcomes.yaml          |

### H2. State derivation rules

| State         | Condition                         | Required Action     |
| ------------- | --------------------------------- | ------------------- |
| **Unknown**   | No tests exist                    | Write tests         |
| **Pending**   | Tests exist, not all claimed      | Fix code or claim   |
| **Stale**     | Descendant outcomes_blob mismatch | Re-claim            |
| **Passing**   | All tests pass, blobs match       | None                |
| **Regressed** | Claimed test fails                | Investigate and fix |

### H3. Tree coupling (Merkle property)

Parent outcomes.yaml stores `outcomes_blob` for each child. When a child's outcomes.yaml changes:

1. Child's Git blob changes
2. Parent's stored `outcomes_blob` no longer matches
3. Parent becomes **Stale**
4. Parent must re-claim to update references

This creates a Merkle tree of verification state, separate from Git's content Merkle tree. Claim order is bottom-up: children first, then parents.

⸻

## Appendix I — Precommit validation loop

Precommit is the primary feedback loop; CI is insurance.

For each container with outcomes.yaml:

1. **Phantom check**
   Every file in outcomes.yaml must exist at `<container>/tests/<file>`

2. **Regression check**
   Run exactly the tests listed in outcomes.yaml
   - If a claimed test fails → **Regressed** (block commit)

3. **Descendant staleness check**
   For each `descendants[].outcomes_blob`:
   - Compute current Git blob of child's outcomes.yaml
   - If mismatch → **Stale** (child changed, re-claim needed)

4. **Progress rule**
   Tests present but not in outcomes.yaml are "in progress" (not an error)

⸻

## Appendix J — Test infrastructure as first-class product work

Harnesses are production code with their own specs and evidence because:

- harness bugs break many dependent tests (high blast radius)
- harness refactors need regression detection
- harness capabilities must be documented as durable contracts

Reference layout:

project/
├── src/
├── tests/
│ └── harness/
└── spx/
└── 13-test-infrastructure.capability/
├── test-infrastructure.capability.md
├── 10-cli-harness.feature/
│ ├── cli-harness.feature.md
│ ├── outcomes.yaml
│ └── tests/
└── 20-e2e-harness.feature/
├── e2e-harness.feature.md
├── outcomes.yaml
└── tests/

⸻

## Appendix K — Recursive decimal BSP (deep insertion)

When no integer space remains between two BSP numbers, insert with @:

- Between 20 and 21:
- 20@54-new-item
- Between 20@54 and 20@55:
- 20@54@50-deeper-insert

Rules of thumb:

- keep hyphen separators
- avoid renumbering; use @ for insertion
- if recursion becomes too deep, rebalance intentionally
