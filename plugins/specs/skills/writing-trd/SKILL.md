---
name: writing-trd
description: Write TRDs documenting how to build and test it. Use when writing TRDs or technical requirements.
allowed-tools: Read, Write, Glob, Grep, AskUserQuestion
---

<essential_principles>
**TRDs are authoritative blueprints written AFTER exploration.** They guide decomposition into work items.

**Testing is first-class:** Validation strategy and test infrastructure must be complete before TRD approval.

**Three-tier testing methodology:**

- **Level 1 (Unit)**: Pure logic, dependency injection, no external systems
- **Level 2 (Integration)**: Real infrastructure via documented test harnesses
- **Level 3 (E2E)**: Real credentials and services, full workflows

**Questioning principles:**

- Always ask about test harnesses and credentials (cannot assume)
- Always confirm root cause and solution approach with user
- Agent decides guarantees and test level assignments (applies `/testing` methodology)
- Adaptive: Ask targeted questions based on conversation gaps

**Quality guardrails:**

- Test infrastructure documented OR in Infrastructure Gaps table (no vagueness)
- Every guarantee (G1, G2, etc.) maps to ≥1 BDD scenario
- Readiness Criteria self-check before delivery

**Requirements define WHAT, not WHEN:**

- Requirements describe the ideal solution (timeless vision)
- Define scope boundaries ("Out of scope: X is separate")
- NEVER include implementation timing ("MVP", "Phase 2", "Defer to")
- Avoid: "Not in MVP", "Phase X", "Defer until", "Later"
- Correct: "Out of scope for this TRD", "Future capability", "Separate requirement"

</essential_principles>

<objective>
Create complete, testable Technical Requirements Documents that serve as authoritative blueprints for implementation. Systematically discover problem root causes, design validation strategies with explicit test level assignments, and document test infrastructure requirements before work begins.
</objective>

<quick_start>

1. **Read context** (top-down, front-to-back through specs)
2. **Confirm understanding** (root cause, solution approach)
3. **Design validation** (guarantees → test levels → BDD scenarios)
4. **Discover infrastructure** (harnesses, credentials, gaps)
5. **Write TRD** (complete file with all sections)
6. **Verify readiness** (self-check against criteria)

</quick_start>

<context_reading_protocol>
**Before asking questions, read project context:**

**Top-down, front-to-back algorithm:**

1. Read `specs/CLAUDE.md` for project structure
2. Determine TRD location (capability or feature level)
3. Read all ADRs at and above TRD's level
4. Read all capabilities/features preceding this one at same level
5. Read related docs mentioned in conversation
6. Invoke `/testing` and language-specific testing skill

**TRD location determines context scope:**

- Capability-level TRD: Read all preceding capabilities
- Feature-level TRD: Read containing capability + all preceding features within it

</context_reading_protocol>

<workflow>
**Phase 0: Context Discovery**

Follow `<context_reading_protocol>` to gather project context.

**Phase 1: Critical Understanding**

Read workflow: `workflows/understand-problem.md`

- Deep-think: Problem vs symptom analysis
- Confirm root cause with user (MUST get approval)
- Confirm solution approach with user (MUST get approval)
- Identify gaps in conversation context

**Phase 2: Validation Strategy Design**

Read workflow: `workflows/design-validation.md`

- Deep-think: Guarantee completeness (what are we missing?)
- Propose guarantees with test level assignments
- Draft BDD scenarios linked to guarantees
- Present to user for validation

**Phase 3: Test Infrastructure Discovery**

Read workflow: `workflows/discover-infrastructure.md`

- Deep-think: Infrastructure feasibility
- Ask about Level 2 harnesses (Docker, databases, binaries)
- Ask about Level 3 credentials (sources, rotation)
- Document in tables OR Infrastructure Gaps table

**Phase 4: TRD Composition**

Read workflow: `workflows/write-trd.md`

- Write complete TRD file to correct location
- Fill all sections per template
- Verify guarantee-to-scenario mapping
- Run Readiness Criteria self-check

**Phase 5: Delivery**

- Mark TRD complete or incomplete (if gaps exist)
- Provide summary of decisions
- List next actions if gaps remain
- Report readiness assessment

</workflow>

<deep_thinking_checkpoints>
The skill pauses for ultra-thinking at three critical junctures:

**Checkpoint 1: Problem vs Symptom (Phase 1)**

Is the stated problem the actual root cause, or just a symptom of something deeper?

- What technical limitation is blocking the desired capability?
- Why does this limitation exist?
- What would resolving it enable?

**Checkpoint 2: Guarantee Completeness (Phase 2)**

Have we identified ALL critical behaviors that must work?

- What failure modes aren't covered?
- What edge cases must be handled?
- What assumptions are we making?
- What second-order effects exist?

**Checkpoint 3: Infrastructure Feasibility (Phase 3)**

Do the required test harnesses and credentials actually exist or can they be built?

- What Docker containers/databases are available?
- What credentials exist and where are they stored?
- What's blocking if infrastructure doesn't exist?
- What's the path to creating missing infrastructure?

</deep_thinking_checkpoints>

<mandatory_user_interactions>
**Agent MUST ask (cannot proceed without answers):**

1. Root cause confirmation (propose analysis, user confirms/corrects)
2. Solution approach confirmation (propose architecture, user approves)
3. Level 2 test harnesses (what exists? how to setup/reset?)
4. Level 3 credentials (where stored? rotation schedule?)

**Agent decides (with documented rationale):**

1. Which guarantees to include (based on solution analysis)
2. Test level assignments (per `/testing` methodology)
3. BDD scenarios (derived from approved guarantees)
4. Component architecture (based on solution approach)

</mandatory_user_interactions>

<gap_handling>
When infrastructure information is unknown:

1. Ask user explicitly (cannot assume or guess)
2. Document answer in appropriate table, OR
3. Add to Infrastructure Gaps table with "Blocking" column

**A TRD with unresolved Infrastructure Gaps is marked incomplete but can be delivered.**

User knows exactly what must be resolved before implementation can begin.

</gap_handling>

<accessing_skill_files>
When this skill is invoked, Claude Code provides the base directory in the loading message:

```
Base directory for this skill: {skill_dir}
```

Use this path to access skill files:

- References: `{skill_dir}/references/`
- Workflows: `{skill_dir}/workflows/`

**IMPORTANT**: Do NOT search the project directory for skill files.
</accessing_skill_files>

<workflows_index>
All workflows in `workflows/`:

| Workflow                   | Purpose                                      |
| -------------------------- | -------------------------------------------- |
| understand-problem.md      | Root cause analysis and solution validation  |
| design-validation.md       | Guarantee identification and test assignment |
| discover-infrastructure.md | Test harness and credential documentation    |
| write-trd.md               | TRD composition and section filling          |

</workflows_index>

<references_index>
Domain knowledge in `references/`:

| Reference                | Purpose                                 |
| ------------------------ | --------------------------------------- |
| trd-template-guide.md    | Complete TRD template with annotations  |
| testing-methodology.md   | Three-tier testing summary and rules    |
| question-patterns.md     | Effective questioning strategies        |
| bdd-scenario-patterns.md | Given/When/Then scenario best practices |

</references_index>

<templates_index>
Referenced templates:

| Template                                        | Purpose                        |
| ----------------------------------------------- | ------------------------------ |
| /managing-specs skill `<requirement_templates>` | TRD template from shared skill |

</templates_index>

<success_criteria>
TRD creation complete when:

- [ ] Problem Statement confirmed by user (root cause, not symptom)
- [ ] Solution Design approved by user (components, data flow, interfaces)
- [ ] Every guarantee has unique ID (G1, G2, G3...)
- [ ] Every guarantee assigned to appropriate test level with rationale
- [ ] Every guarantee has ≥1 BDD scenario referencing it
- [ ] All BDD scenarios use strict Given/When/Then format
- [ ] Test infrastructure documented OR in Infrastructure Gaps table
- [ ] All Level 2 harnesses have setup/reset commands
- [ ] All Level 3 credentials have source and rotation schedule
- [ ] TRD file written to correct location (capability/ or feature/)
- [ ] Readiness Criteria self-check passed (or gaps explicitly documented)
- [ ] User receives summary and next actions

</success_criteria>
