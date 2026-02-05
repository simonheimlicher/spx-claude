# What Goes Where

A clear taxonomy of what content belongs in ADRs, Specs, and Tests.

## The Three Types

| Type     | Purpose                     | Verified by (skill pattern)          |
| -------- | --------------------------- | ------------------------------------ |
| **ADR**  | GOVERNS how                 | `/reviewing-{language}-architecture` |
| **Spec** | DESCRIBES what should exist | `/reviewing-{language}`              |
| **Test** | PROVES it exists            | `/reviewing-{language}-tests`        |

*Replace `{language}` with actual language: `python`, `typescript`, etc.*

---

## ADRs (Decisions) — GOVERN

ADRs say "when you do X, do it THIS way." They constrain implementation.

### ADR Content

| Section      | Content                                  | Verified by                          |
| ------------ | ---------------------------------------- | ------------------------------------ |
| Context      | Why this decision is needed              | `/reviewing-{language}-architecture` |
| Options      | Alternatives considered with trade-offs  | `/reviewing-{language}-architecture` |
| Decision     | What we chose and why                    | `/reviewing-{language}-architecture` |
| Consequences | Trade-offs accepted, constraints imposed | `/reviewing-{language}-architecture` |

### ADRs Do NOT Contain

| ❌ Never in ADR     | Why not                               | Where it belongs  |
| ------------------- | ------------------------------------- | ----------------- |
| Outcomes (Gherkin)  | ADRs don't describe states to achieve | Spec              |
| Test files          | ADRs don't get tested                 | Spec's test table |
| Implementation code | ADRs govern, don't implement          | Story's Analysis  |
| Work to be done     | ADRs create no work                   | Spec outcomes     |

### ADR Verification

```
/reviewing-{language}-architecture checks ADR STRUCTURE:
□ Context explains why decision is needed
□ Options were genuinely considered (not just "we picked X")
□ Decision states what was chosen and why
□ Consequences acknowledge trade-offs
□ No implementation details leaked in (ADRs GOVERN, don't implement)
□ Constraints are clear and enforceable

/reviewing-{language} checks ADR COMPLIANCE:
□ Code follows constraints from applicable ADRs
□ Deviations are documented or ADR is updated
```

---

## Specs (Containers) — DESCRIBE OUTCOMES

Specs define states that should exist. Each level has specific content.

### Spec Content by Level

| Section                   | Capability | Feature | Story | Verified by             |
| ------------------------- | :--------: | :-----: | :---: | ----------------------- |
| Purpose                   |     ✓      |    ✓    |   ✓   | `/reviewing-{language}` |
| Success Metric            |     ✓      |    —    |   —   | `/reviewing-{language}` |
| Requirements              |     ✓      |    ✓    |   —   | `/reviewing-{language}` |
| Test Strategy             |     ✓      |    ✓    |   —   | `/reviewing-{language}` |
| Outcomes (Gherkin)        |     ✓      |    ✓    |   ✓   | `/reviewing-{language}` |
| Test File Table           |     ✓      |    ✓    |   ✓   | `/reviewing-{language}` |
| Analysis                  |     —      |    —    |   ✓   | `/reviewing-{language}` |
| Architectural Constraints |     ✓      |    ✓    |   ✓   | `/reviewing-{language}` |

### Outcome Scoping

| Container  | Outcomes describe...                   | Example                                     | Verified by             |
| ---------- | -------------------------------------- | ------------------------------------------- | ----------------------- |
| Capability | Complete user journeys                 | "User authenticates and accesses dashboard" | `/reviewing-{language}` |
| Feature    | Integration (stories working together) | "Master-slave loopback works"               | `/reviewing-{language}` |
| Story      | Atomic behavior                        | "SPI master transmits in mode 0"            | `/reviewing-{language}` |

### The Integration Test

To determine if an outcome belongs in Feature vs Story:

```
Q: Can this outcome be verified without other stories in this feature?

YES → Story outcome (atomic)
NO  → Feature outcome (integration)
```

### Quality Gates

Quality gates are typically **Feature outcomes** because they verify combined output:

| Quality Gate           | Why Feature-level                    |
| ---------------------- | ------------------------------------ |
| "Lint-clean HDL"       | Verifies all generated code together |
| "All APIs documented"  | Verifies combined API surface        |
| "No security warnings" | Verifies integrated system           |

### Spec Verification

```
/reviewing-{language} checks:
□ Outcomes are states, not tasks
□ Gherkin is specific and testable
□ Test files are referenced correctly
□ Story outcomes are atomic
□ Feature outcomes require integration
□ Analysis section proves examination (stories only)
```

---

## Tests — PROVE

Tests are executable proof that outcomes exist.

### Test Organization

| Aspect             | Value                  | Verified by                   |
| ------------------ | ---------------------- | ----------------------------- |
| Location           | `<container>/tests/`   | `/reviewing-{language}-tests` |
| Unit naming        | `*.unit.test.*`        | `/reviewing-{language}-tests` |
| Integration naming | `*.integration.test.*` | `/reviewing-{language}-tests` |
| E2E naming         | `*.e2e.test.*`         | `/reviewing-{language}-tests` |

### Test Level vs Container Level

**These are ORTHOGONAL concepts.**

| Test Level | Infrastructure needed     | Can be used by...             |
| ---------- | ------------------------- | ----------------------------- |
| 1          | None (DI, temp dirs)      | Story, Feature, OR Capability |
| 2          | Real binaries/DB          | Story, Feature, OR Capability |
| 3          | Real services/credentials | Story, Feature, OR Capability |

A **Story** CAN have Level 2 tests (if it needs real binaries).
A **Capability** CAN have Level 1 tests (for pure logic verification).

### Test Verification

```
/reviewing-{language}-tests checks:
□ Tests prove outcomes from spec
□ Test level matches infrastructure needs
□ No mocking (DI instead)
□ Tests are co-located with spec
□ Naming convention followed
```

---

## The Flow

```
┌─────────────────────────────────────────┐
│                  ADR                     │
│         "When you do X, do it           │
│              THIS way"                   │
│                                         │
│  Verified by: /reviewing-{lang}-arch    │
└────────────────┬────────────────────────┘
                 │ constrains HOW
                 ▼
┌─────────────────────────────────────────┐
│                 SPEC                     │
│       "This state should exist"          │
│         (Gherkin outcomes)               │
│                                         │
│  Verified by: /reviewing-{lang}         │
└────────────────┬────────────────────────┘
                 │ defines WHAT
                 ▼
┌─────────────────────────────────────────┐
│                 TEST                     │
│        "Here's proof it exists"          │
│                                         │
│  Verified by: /reviewing-{lang}-tests   │
└─────────────────────────────────────────┘
```

---

## Quick Reference

| I want to...                      | Put it in...     | Verified by                          |
| --------------------------------- | ---------------- | ------------------------------------ |
| Constrain implementation approach | ADR              | `/reviewing-{language}-architecture` |
| Define a state that should exist  | Spec outcome     | `/reviewing-{language}`              |
| Prove an outcome exists           | Test             | `/reviewing-{language}-tests`        |
| Document trade-offs               | ADR Consequences | `/reviewing-{language}-architecture` |
| Show I examined the codebase      | Story Analysis   | `/reviewing-{language}`              |
| Gate on combined quality          | Feature outcome  | `/reviewing-{language}`              |
