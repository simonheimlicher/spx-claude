# ADR: Orchestrator Pattern for Skill Coordination

## Problem

Skills that invoke other skills directly create tight coupling, limit flexibility, and make it impossible to use the coordinated workflow as both a subagent and a command.

## Context

- **Business**: Users want to run multi-skill workflows (specs → test → code → review) either interactively via command or autonomously via subagent. Direct skill-to-skill calls prevent this flexibility.
- **Technical**: Claude Code skills can invoke other skills via `/skill-name`. Without architectural guidance, implementers naturally have skills call each other, creating a tangled dependency graph.

## Decision

**Skills SHALL remain independent units that never invoke other skills; coordination SHALL be handled by dedicated orchestrator skills that can be invoked from both commands and subagents.**

## Rationale

Two approaches were considered:

1. **Skills call each other** - Each skill invokes the next in the workflow. Problem: Creates tight coupling. If testing-typescript calls coding-typescript, you cannot use testing-typescript alone. The workflow cannot be used as a subagent because commands cannot wrap skill chains.

2. **Orchestrator pattern** (chosen) - Skills are independent. A dedicated orchestrator skill coordinates them in sequence. Commands invoke the orchestrator. Subagents can also invoke the orchestrator. Each skill can be used independently.

Option 2 was chosen because:

- Skills remain independently usable
- Same orchestration works as command AND subagent
- Workflow can be modified without changing individual skills
- Clear separation: skills implement, orchestrators coordinate

## Trade-offs Accepted

- **More files**: Orchestrator is a separate skill, not embedded in execution skills. Mitigation: Clear separation makes each file simpler.
- **Orchestrator duplication risk**: Testing rules might appear in both orchestrator and testing skill. Mitigation: Orchestrators reference skills ("see /testing-typescript") rather than duplicating content.

## Validation

### How to Recognize Compliance

You're following this decision if:

- Execution skills (coding-*, testing-*, reviewing-*) never contain `/skill-name` invocations
- Multi-skill workflows have a dedicated orchestrator skill (auto-*, workflow-*)
- Commands invoke orchestrator skills, not execution skill chains

### MUST

- Keep execution skills independent - they receive context, do work, return results
- Create orchestrator skills for multi-step workflows
- Reference other skills by name in orchestrators, don't duplicate their content

### NEVER

- Have execution skills invoke other skills directly
- Embed workflow logic in execution skills
- Duplicate testing/coding/reviewing rules in orchestrators (reference them instead)
