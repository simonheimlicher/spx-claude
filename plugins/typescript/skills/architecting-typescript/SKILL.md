---
name: architecting-typescript
description: Write ADRs for TypeScript architecture decisions. Use when making architecture decisions or writing ADRs.
allowed-tools: Read, Write, Glob, Grep
---

<essential_principles>
**TESTING LEVELS IN EVERY ADR. ARCHITECTURE WITHOUT TESTABILITY IS INCOMPLETE.**

- Every ADR MUST include a Testing Strategy section with level assignments
- No `any` without explicit justification in ADR
- Design for dependency injection (NO MOCKING)
- You produce ADRs (Architecture Decision Records), not implementation code

</essential_principles>

<testing_levels_summary>

| Level | Name        | Infrastructure                          | When to Use                                   |
| ----- | ----------- | --------------------------------------- | --------------------------------------------- |
| 1     | Unit        | Node.js built-ins + Git + temp fixtures | Pure logic, FS operations, git operations     |
| 2     | Integration | Project-specific binaries/tools         | Claude Code, Hugo, Caddy, TypeScript compiler |
| 3     | E2E         | External deps (GitHub, network, Chrome) | Full workflows with network/external services |

**Key distinctions:**

- Git is Level 1 (standard dev tool, always available in CI)
- Project-specific tools require installation/setup (Level 2)
- Network dependencies and external services are Level 3

**Core Testing Principles:**

- **NO MOCKING** — Use dependency injection instead
- **Behavior only** — Test what the code does, not how
- **Escalation requires justification** — Each level adds dependencies
- **Reality is the oracle** — Real systems, not simulations

</testing_levels_summary>

<context_loading>
**For specs-based work items: Load complete context before creating ADRs.**

If you're creating ADRs for a spec-driven work item (story/feature/capability), ensure complete hierarchical context is loaded:

1. **Invoke `specs:understanding-specs`** with the work item identifier
2. **Verify all parent ADRs/PDRs are loaded** - Must understand and honor all decision records in hierarchy
3. **Read the feature spec** - Requirements, Test Strategy, and Outcomes sections

**The `specs:understanding-specs` skill provides:**

- Complete ADR/PDR hierarchy (product/capability/feature decisions)
- Feature spec with requirements, test strategy, and outcomes
- Story/feature/capability spec with Gherkin acceptance criteria

**ADR creation requirements:**

- Must not contradict parent ADRs/PDRs (product → capability → feature hierarchy)
- Must reference relevant parent decisions
- Must include testing strategy with level assignments
- Must document trade-offs and consequences

**If NOT working on specs-based work item**: Proceed directly with ADR creation using provided requirements.
</context_loading>

<input_context>
Before creating ADRs, you must understand:

**1. Feature Specification**

- Functional requirements in `## Requirements` section
- Test strategy in `## Test Strategy` section
- Outcomes with Gherkin in `## Outcomes` section
- Architectural constraints from parent ADRs

**2. Project Context**

Read these files to understand project structure and workflow:

- `spx/CLAUDE.md` - Project navigation, work item status, BSP dependencies

For testing methodology, invoke the `/testing-typescript` skill

**3. Existing Decisions**

Read existing ADRs/PDRs to ensure consistency:

- `spx/{NN}-{slug}.adr.md` - Product-level ADRs (interleaved at root)
- `spx/{NN}-{slug}.pdr.md` - Product-level PDRs (interleaved at root)
- ADRs/PDRs interleaved within capability/feature containers

</input_context>

<adr_scope>
You produce ADRs. The scope depends on what you're deciding:

| Decision Scope      | ADR Location                                     | Example                              |
| ------------------- | ------------------------------------------------ | ------------------------------------ |
| Product-wide        | `spx/{NN}-{slug}.adr.md`                         | "Use Zod for all data validation"    |
| Capability-specific | `spx/{NN}-{slug}.capability/{NN}-{slug}.adr.md`  | "CLI command structure"              |
| Feature-specific    | `spx/.../{NN}-{slug}.feature/{NN}-{slug}.adr.md` | "Use execa for subprocess execution" |

**ADR Numbering:**

- BSP range: [10, 99]
- Lower BSP = dependency (higher-BSP ADRs may rely on it)
- Insert using midpoint calculation: `new = floor((left + right) / 2)`
- Append using: `new = floor((last + 99) / 2)`
- First ADR in scope: use 21

See `specs:managing-specs` skill `<adr_templates>` section for complete BSP numbering rules.

**Within-scope dependency order**:

- Capability ADRs: adr-21 must be decided before adr-37
- Feature ADRs: adr-21 must be decided before adr-37
- Product ADRs: adr-21 must be decided before adr-37

**Cross-scope dependencies**: Must be documented explicitly in ADR "Context" section using markdown links.

</adr_scope>

<adr_creation_protocol>
Execute these phases IN ORDER.

**Phase 0: Read Context**

1. Read the feature spec completely (requirements, test strategy, outcomes)
2. Read project context:
   - `spx/CLAUDE.md` - Project structure, navigation, work item management
3. Invoke `/testing-typescript` to understand testing methodology
4. Read existing ADRs for consistency:
   - `spx/{NN}-{slug}.adr.md` - Product-level ADRs
   - ADRs interleaved within capability/feature containers
5. Read `/managing-specs` skill `<adr_templates>` section for ADR template

**Phase 1: Identify Decisions Needed**

For each TRD section, ask:

- What architectural choices does this imply?
- What patterns or approaches should be mandated?
- What constraints should be imposed?
- What trade-offs are being made?

List decisions needed before writing any ADRs.

**Phase 2: Analyze TypeScript-Specific Implications**

For each decision, consider:

- **Type system**: How will types be designed? What generics needed?
- **Architecture**: Which pattern applies (DDD, hexagonal, etc.)?
- **Security**: What boundaries need protection?
- **Testability**: How will this be tested?

**Phase 3: Write ADRs**

Use the project's template. Each ADR must include:

1. **Title**: Clear, specific decision statement
2. **Status**: Proposed, Accepted, Deprecated, Superseded
3. **Context**: Why is this decision needed?
4. **Decision**: What is the specific choice?
5. **Consequences**: What are the trade-offs?
6. **Compliance**: How will adherence be verified?
7. **Testing Strategy** (MANDATORY): Testing levels for each component

**Phase 4: Verify Consistency**

- No ADR should contradict another
- Capability ADRs must align with project ADRs
- Feature ADRs must align with capability ADRs

</adr_creation_protocol>

<testing_strategy_section>
**Required in Every ADR:**

```markdown
## Testing Strategy

### Level Assignments

| Component     | Level           | Justification               |
| ------------- | --------------- | --------------------------- |
| {component_1} | 1 (Unit)        | {why Level 1 is sufficient} |
| {component_2} | 2 (Integration) | {why Level 2 is needed}     |
| {component_3} | 3 (E2E)         | {why Level 3 is needed}     |

### Escalation Rationale

- Level 1→2: {what confidence Level 2 adds that Level 1 cannot provide}
- Level 2→3: {what confidence Level 3 adds that Level 2 cannot provide}

### Testing Principles

- NO MOCKING: Use dependency injection for all external dependencies
- Behavior only: Test observable outcomes, not implementation details
- Minimum level: Each component tested at lowest level that provides confidence
```

</testing_strategy_section>

<what_you_do_not_do>

1. **Do NOT write implementation code**. You write ADRs that constrain implementation.
2. **Do NOT review code**. That's a separate concern.
3. **Do NOT fix bugs**. That's an implementation concern.
4. **Do NOT create work items**. That's a project management concern.

</what_you_do_not_do>

<accessing_skill_files>
When this skill is invoked, Claude Code provides the base directory in the loading message:

```
Base directory for this skill: {skill_dir}
```

Use this path to access skill files:

- References: `{skill_dir}/references/`

**IMPORTANT**: Do NOT search the project directory for skill files.
</accessing_skill_files>

<reference_index>
Detailed patterns and principles:

| File                                  | Purpose                                   |
| ------------------------------------- | ----------------------------------------- |
| `references/adr-patterns.md`          | Common ADR patterns for TypeScript        |
| `references/typescript-principles.md` | Type safety, clean architecture, security |

</reference_index>

<output_format>
When you complete ADR creation, provide:

```markdown
## Architectural Decisions Created

### ADRs Written

| ADR                                                            | Scope         | Decision Summary                 |
| -------------------------------------------------------------- | ------------- | -------------------------------- |
| [Type Safety](spx/21-type-safety.adr.md)                       | Product       | Use strict TS, Zod at boundaries |
| [CLI Structure](spx/32-cli.capability/21-cli-structure.adr.md) | Capability-32 | Commander.js with subcommands    |

### Key Constraints

1. {constraint from [Type Safety](spx/21-type-safety.adr.md)}
2. {constraint from [CLI Structure](spx/32-cli.capability/21-cli-structure.adr.md)}

### Testing Strategy Summary

| Component | Level | Justification |
| --------- | ----- | ------------- |
| ...       | ...   | ...           |
```

</output_format>

<success_criteria>
ADR is complete when:

- [ ] Testing strategy included with level assignments
- [ ] All architectural choices documented
- [ ] Compliance criteria defined for verification
- [ ] No contradictions with existing ADRs
- [ ] Type safety considerations addressed
- [ ] Security boundaries identified

*Remember: Your decisions shape everything downstream. A well-designed architecture enables clean implementation.*
</success_criteria>
