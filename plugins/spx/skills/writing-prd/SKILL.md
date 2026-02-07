---
name: writing-prd
description: Write PRDs documenting what users need and why. Use when writing PRDs or product requirements.
allowed-tools: Read, Write, Glob, Grep, AskUserQuestion
---

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

<essential_principles>
**PRDs are authoritative blueprints written AFTER exploration.** They define product value and guide decomposition into work items.

**User value is first-class:** Measurable outcomes and user capabilities must be clear before technical implementation.

**Three-level testing methodology:**

- **Level 1 (Unit)**: Pure logic, dependency injection, no external systems
- **Level 2 (Integration)**: Real infrastructure via documented test harnesses
- **Level 3 (E2E)**: Real credentials and services, full user workflows

**Questioning principles:**

- Always ask about user problem and customer journey (cannot assume)
- Always confirm measurable outcomes with user
- Agent proposes user capabilities as guarantees (applies `/testing` methodology)
- Adaptive: Ask targeted questions based on conversation gaps

**Quality guardrails:**

- Measurable outcomes must be quantified (X% improvement in Y)
- Every user capability maps to ≥1 acceptance test scenario
- Test infrastructure documented OR in dependencies (no vagueness)
- Readiness Criteria self-check before delivery

**Requirements define WHAT, not WHEN:**

- Requirements describe the ideal solution (timeless vision)
- Define scope boundaries ("Out of scope: X is separate")
- NEVER include implementation timing ("MVP", "Phase 2", "Defer to")
- Avoid: "Not in MVP", "Phase X", "Defer until", "Later"
- Correct: "Out of scope for this PRD", "Future capability", "Separate requirement"

</essential_principles>

<objective>
Create complete, testable Product Requirements Documents that serve as authoritative blueprints for product development. Systematically discover user problems, design measurable outcomes with validation strategies, and document acceptance criteria before work begins.
</objective>

<quick_start>

1. **Read context** (project structure, existing PRDs/capabilities)
2. **Confirm understanding** (user problem, customer journey)
3. **Design outcomes** (measurable goals → user capabilities → acceptance tests)
4. **Define scope** (what's included, what's excluded, why)
5. **Write PRD** (complete file with all sections)
6. **Verify readiness** (self-check against criteria)

</quick_start>

<context_reading_protocol>
**Before asking questions, read project context:**

1. Read `spx/CLAUDE.md` for project structure
2. Determine PRD location (project, capability, or feature level)
3. Read existing PRDs to understand product direction
4. Read related capabilities/features if adding to existing structure
5. Invoke `/testing` and language-specific testing skill

**PRD location determines context scope:**

- Project-level PRD: Read all existing capabilities for context
- Capability-level PRD: Read containing project context + sibling capabilities
- Feature-level PRD: Read containing capability + sibling features

</context_reading_protocol>

<workflow>
**Phase 0: Context Discovery**

Follow `<context_reading_protocol>` to gather project context.

**Phase 1: Problem Understanding**

Read workflow: `workflows/understand-user-problem.md`

- Deep-think: Symptom vs root cause of user pain
- Confirm user problem with user (MUST get approval)
- Confirm customer journey (before/during/after transformation)
- Identify conversation gaps

**Phase 2: Outcome Design**

Read workflow: `workflows/design-measurable-outcomes.md`

- Deep-think: What measurable improvements matter to users?
- Propose quantified outcomes (X% improvement in Y)
- Define user capabilities as guarantees
- Draft acceptance test scenarios (Gherkin + E2E code)

**Phase 3: Scope Definition**

Read workflow: `workflows/define-product-scope.md`

- Deep-think: What's the minimal viable increment?
- Define what's included (capabilities delivering value)
- Define what's excluded (with rationale)
- Identify ADR/PDR triggers in technical approach

**Phase 4: PRD Composition**

Read workflow: `workflows/write-prd.md`

- Write complete PRD file to correct location
- Fill all sections per template
- Verify capability-to-scenario mapping
- Run Readiness Criteria self-check

**Phase 5: Delivery**

- Mark PRD ready or incomplete (if dependencies exist)
- Provide summary of product definition
- List next actions (ADRs/PDRs to write, open decisions)
- Report readiness assessment

</workflow>

<deep_thinking_checkpoints>
The skill pauses for ultra-thinking at three critical junctures:

**Checkpoint 1: User Pain vs Symptom (Phase 1)**

Is the stated problem the actual user pain, or just a symptom of deeper needs?

- What capability do users lack?
- Why does this limitation impact their workflow?
- What would resolving it enable them to do?
- How does this align with product vision?

**Checkpoint 2: Measurable Outcome Clarity (Phase 2)**

Have we defined MEASURABLE success that users will actually care about?

- Can we quantify the improvement (X% better at Y)?
- Are metrics user-focused (not just technical)?
- Can we actually measure these outcomes?
- What acceptance tests prove these outcomes?

**Checkpoint 3: Scope Viability (Phase 3)**

Is this scope achievable as one deliverable unit delivering real user value?

- Is it too large (should split into multiple PRDs)?
- Is it too small (not standalone user value)?
- What's excluded and why?
- What ADRs/PDRs will implementation need?

</deep_thinking_checkpoints>

<mandatory_user_interactions>
**Agent MUST ask (cannot proceed without answers):**

1. User problem confirmation (propose analysis, user confirms/corrects)
2. Customer journey validation (before/during/after states)
3. Measurable outcome approval (quantified improvements)
4. Scope boundaries (what's in, what's out, why)

**Agent decides (with documented rationale):**

1. User capabilities to guarantee (based on outcome analysis)
2. Test level assignments (per `/testing` methodology)
3. Acceptance test scenarios (derived from approved capabilities)
4. Product approach and ADR/PDR triggers

</mandatory_user_interactions>

<gap_handling>
When product decisions are unclear:

1. Use AskUserQuestion to clarify product direction
2. Document open questions in "Open Decisions" section
3. Mark ADR/PDR triggers for technical decisions

**A PRD with open decisions can be delivered** if decisions are explicitly documented with options and trade-offs.

User knows exactly what needs resolution before implementation.
</gap_handling>

<workflows_index>
All workflows in `workflows/`:

| Workflow                      | Purpose                                          |
| ----------------------------- | ------------------------------------------------ |
| understand-user-problem.md    | User pain analysis and journey validation        |
| design-measurable-outcomes.md | Outcome quantification and capability definition |
| define-product-scope.md       | Scope boundaries and ADR/PDR identification      |
| write-prd.md                  | PRD composition and section filling              |

</workflows_index>

<references_index>
Domain knowledge in `references/`:

| Reference                   | Purpose                                    |
| --------------------------- | ------------------------------------------ |
| prd-template-guide.md       | Complete PRD template with annotations     |
| testing-methodology.md      | Three-level testing rules                  |
| acceptance-test-patterns.md | Gherkin and E2E test best practices        |
| measurable-outcomes.md      | Quantifying user value and product success |

</references_index>

<templates_index>
Referenced templates:

| Template                                        | Purpose                        |
| ----------------------------------------------- | ------------------------------ |
| /managing-specs skill `<requirement_templates>` | PRD template from shared skill |

</templates_index>

<success_criteria>
PRD creation complete when:

- [ ] User problem confirmed by user (root pain, not symptom)
- [ ] Customer journey approved (before/during/after transformation)
- [ ] Measurable outcome is quantified (X% improvement in Y)
- [ ] Evidence of Success metrics defined (Current → Target)
- [ ] Every user capability has unique ID (UC1, UC2, UC3...)
- [ ] Every capability assigned to appropriate test level
- [ ] Every capability has ≥1 acceptance test scenario (Gherkin)
- [ ] Complete E2E test code provided (TypeScript/Python)
- [ ] Scope boundaries defined (included, excluded, rationale)
- [ ] Product approach identifies ADR/PDR triggers
- [ ] Open decisions documented with options and trade-offs
- [ ] PRD file written to correct location (project/capability/feature)
- [ ] Readiness Criteria self-check passed
- [ ] User receives summary and next actions

</success_criteria>
