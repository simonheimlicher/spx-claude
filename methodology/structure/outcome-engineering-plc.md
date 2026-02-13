# Program Logic Chain (PLC)

The Program Logic Chain distinguishes what is locally provable from what requires real-world observation:

```text
Resources → Activities → OUTPUTS → OUTCOMES → IMPACT
                            ↑           ↑          ↑
                       local tests   users      long-term
                       (provable)    (observable) (measurable)
```

- **Resources**: What the team invests (time, budget, infrastructure)
- **Activities**: What the team does (design, implementation, testing)
- **Outputs**: What the software produces. Deterministic, locally testable.
- **Outcomes**: What changes for users as a result. Requires real users, real behavior, real measurement.
- **Impact**: Long-term consequences. Requires sustained observation.

## Relevance to Outcome Engineering

Outcome Engineering operates in the **Outputs ↔ Outcomes** zone of the chain:

- **Assertions** are structured output specifications — locally provable
- **Purpose** states the outcome hypothesis — not locally provable
- **Tests** prove outputs, not outcomes

Whether an outcome is achieved can only be measured in the real world; assertions cover what the product does — outputs — not whether those outputs achieve the desired outcome.

Resources and Impact fall outside the framework's scope. The PLC provides formal context for why the distinction matters.
