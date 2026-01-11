---
name: reviewing-typescript-architecture
description: Review ADRs to check they follow testing principles. Use when reviewing ADRs or architecture decisions.
allowed-tools: Read, Grep
---

<objective>
Review ADRs against testing principles. Point out what violates principles, reference the specific principle, and show what correct architecture looks like.
</objective>

<context_loading>
**For specs-based work items: Load complete ADR hierarchy before reviewing.**

If you're reviewing ADRs for a spec-driven work item (story/feature/capability), ensure complete architectural context is loaded:

1. **Invoke `specs:understanding-specs`** with the work item identifier
2. **Verify all parent ADRs are loaded** - Must check for consistency with architectural hierarchy
3. **Verify ADR references parent decisions** - Feature/capability ADRs should reference relevant product ADRs

**The `specs:understanding-specs` skill provides:**

- Complete ADR hierarchy (product/capability/feature decisions)
- TRD with technical requirements
- Story/feature/capability spec with acceptance criteria

**Review focus:**

- Does ADR contradict any parent decisions?
- Does ADR include testing strategy with level assignments?
- Does ADR honor no-mocking principle?
- Does ADR document trade-offs and consequences?

**If NOT working on specs-based work item**: Proceed directly with ADR review using provided architectural decision.
</context_loading>

<process>
1. **Read the ADR** completely
2. **Identify violations** - what contradicts testing principles
3. **Output decision** - APPROVED or REJECTED with specific violations
4. **Show correct approach** - what the architecture should be
</process>

<principles_to_enforce>
**Level definitions:**

- Level 1: Node.js built-ins (fs, git, etc.) + temp fixtures
  - Includes: All Node.js standard library, Git operations, standard dev tools (cat, grep, curl)
  - Excludes: Project-specific binaries, network, external services

- Level 2: Project-specific binaries/tools running locally
  - Includes: Hugo, Caddy, Claude Code, Docker, TypeScript compiler
  - Excludes: Network calls, GitHub, external services

- Level 3: External dependencies (GitHub, network, Chrome)
  - Includes: Network services, APIs, external repos, browsers
  - Full real-world workflows with external dependencies

**Critical rules:**

- Git is Level 1 (standard dev tool, always available in CI)
- Project-specific tools require installation/setup (Level 2)
- Network dependencies and external services are Level 3

**Mocking prohibition:**

- NO `vi.mock()` or `jest.mock()` for external services
- NO mocking HTTP responses for external APIs
- Use dependency injection with TypeScript interfaces instead

**Reality principle:**

- "Reality is the oracle, not mocks"
- Tests must verify behavior against real systems at appropriate levels

</principles_to_enforce>

<output_format>

````markdown
# ARCHITECTURE REVIEW

**Decision:** [APPROVED | REJECTED]

---

## Violations

### {Violation name}

**Where:** Lines {X-Y}
**Principle violated:** {Specific principle}
**Why this fails:** {Direct explanation}

**Correct approach:**

```typescript
{Show what the architecture should be}
```
````

---

{Repeat for each violation}

---

## Required Changes

{Concise list of what must change}

---

## References

- Testing principles: {specific principle violated}

---

{If REJECTED: "Revise and resubmit"}
{If APPROVED: "Architecture meets standards"}

````
</output_format>

<what_to_avoid>
**Don't:**

- Provide checklists - the architect understands what needs to change
- Explain multiple times - be concise
- Count how many times you've seen this - focus on principles
- Provide grep commands - focus on principles, not commands

**Do:**

- Reference specific principles
- Show correct architecture (code examples)
- Be direct about what violates principles
- Assume the architect will understand and fix

</what_to_avoid>

<example_review>
```markdown
# ARCHITECTURE REVIEW

**Decision:** REJECTED

---

## Violations

### Mocking External Service

**Where:** Lines 45-47
**Principle violated:** NO MOCKING principle

ADR says "mock the execa calls at the boundary" - this violates testing principles which require dependency injection, not mocking.

**Correct approach:**

```typescript
interface BuildDependencies {
  execa: typeof execa;
}

// Level 1: Inject controlled implementation
// Level 2: Use real binary
````

---

### Missing Testing Strategy

**Where:** ADR has no Testing Strategy section
**Principle violated:** Every ADR must include Testing Strategy

All ADRs require a Testing Strategy section with level assignments.

**Correct approach:**

```markdown
## Testing Strategy

### Level Assignments

| Component        | Level | Justification                   |
| ---------------- | ----- | ------------------------------- |
| Command building | 1     | Pure function, no external deps |
| Hugo invocation  | 2     | Needs real Hugo binary          |
```

---

## Required Changes

1. Remove "mock at boundary" language
2. Add DI interface definitions showing how Level 1 works
3. Add Testing Strategy section with level assignments
4. Update escalation rationale

---

## References

- Testing principles: NO MOCKING, Dependency Injection
- Testing principles: Every ADR must include Testing Strategy

---

Revise and resubmit.

```
</example_review>

<success_criteria>
Review is complete when:

- [ ] All testing principle violations identified
- [ ] Correct approach shown for each violation
- [ ] Required changes listed concisely
- [ ] Decision clearly stated (APPROVED/REJECTED)

_Review with authority from expertise. Be concise and direct._
</success_criteria>
```
