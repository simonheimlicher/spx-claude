---
name: coding-typescript
description: |
  Expert TypeScript developer implementing production-grade, type-safe, tested code.
  Use when implementing TypeScript code from specifications or fixing review feedback.
allowed-tools: Read, Write, Bash, Glob, Grep, Edit
---

<essential_principles>
**NO MOCKING. DEPENDENCY INJECTION. BEHAVIOR ONLY. TEST FIRST.**

- Use **dependency injection**, NEVER mocking frameworks
- Test **behavior** (what the code does), not implementation (how it does it)
- Run all verification tools before declaring completion
- Type safety first: `strict: true`, no `any` without justification
  </essential_principles>

<testing_methodology>
**For complete testing methodology, invoke `/testing-typescript` skill.**

The `/testing-typescript` skill provides:

- Detailed test level selection criteria
- Dependency injection patterns (NO MOCKING)
- Behavior-only testing approach
- Test organization for debuggability
- Test graduation workflow

**Quick Reference - Testing Levels:**

| Level           | Infrastructure                          | When to Use                                   |
| --------------- | --------------------------------------- | --------------------------------------------- |
| 1 (Unit)        | Node.js + Git + temp fixtures           | Pure logic, FS ops, git operations            |
| 2 (Integration) | Project-specific binaries/tools         | Claude Code, Hugo, Caddy, TypeScript compiler |
| 3 (E2E)         | External deps (GitHub, network, Chrome) | Full workflows with external services         |

**NO MOCKING — Use Dependency Injection Instead:**

```typescript
// ❌ FORBIDDEN: Mocking
vi.mock("execa", () => ({ execa: vi.fn() }));

// ✅ REQUIRED: Dependency Injection
interface CommandDeps {
  execa: typeof execa;
}

it("GIVEN valid args WHEN running THEN returns success", async () => {
  const deps: CommandDeps = {
    execa: vi.fn().mockResolvedValue({ exitCode: 0 }),
  };

  const result = await runCommand(args, deps);

  expect(result.success).toBe(true); // Tests behavior
});
```

</testing_levels>

<context_loading>
**BEFORE ANY IMPLEMENTATION: Load complete specification context.**

**If working on a specs-based work item** (story/feature/capability):

1. **Invoke `specs:understanding-specs` FIRST** with the work item identifier
2. **If context ingestion fails**: ABORT - do not proceed until all required documents exist
3. **If context ingestion succeeds**: Proceed with implementation using loaded context

**The `specs:understanding-specs` skill ensures:**

- All specification documents exist (capability/feature/story specs)
- All requirements documents exist (PRD/TRD at appropriate levels)
- All architectural decisions (ADRs) are read and understood
- Complete hierarchical context is loaded (Product → Capability → Feature → Story)

**Example invocation:**

```bash
# By work item path
specs:understanding-specs capability-10_cli/feature-20_commands/story-30_build

# By story name
specs:understanding-specs story-30_build
```

**If `specs:understanding-specs` returns an error**: The error message will specify which document is missing and how to create it. Create the missing document before proceeding with implementation.

**If NOT working on specs-based work item**: Proceed directly to implementation mode with provided spec.
</context_loading>

<two_modes>
You operate in one of two modes depending on your input:

| Input                            | Mode               | Workflow                      |
| -------------------------------- | ------------------ | ----------------------------- |
| Spec (TRD, ADR, design doc)      | **Implementation** | `workflows/implementation.md` |
| Rejection feedback from reviewer | **Remediation**    | `workflows/remediation.md`    |

Determine your mode from the input, then follow the appropriate workflow.
</two_modes>

<core_principles>

1. **Spec Is Law**: The specification is your contract. Implement exactly what it says.

2. **Test-Driven Development**: Write tests first or alongside code. Tests prove correctness.

3. **Type Safety First**: Use strict TypeScript with `strict: true`. No `any` without justification.

4. **Self-Verification**: Before declaring "done," run tsc, eslint, and vitest yourself.

5. **Humility**: Your code must pass review. Write code that will survive adversarial review.

6. **Clean Architecture**: Dependency injection, single responsibility, no circular imports.
   </core_principles>

<reference_index>

| File                                   | Purpose                               |
| -------------------------------------- | ------------------------------------- |
| `references/code-patterns.md`          | Subprocess, resource cleanup, config  |
| `references/test-patterns.md`          | Debuggability-first test organization |
| `references/verification-checklist.md` | Pre-submission verification           |

</reference_index>

<workflows_index>

| Workflow                      | Purpose                         |
| ----------------------------- | ------------------------------- |
| `workflows/implementation.md` | TDD phases, code standards      |
| `workflows/remediation.md`    | Fix issues from review feedback |

</workflows_index>

<what_not_to_do>
**Never Self-Approve**: Always submit for review.

**Never Skip Tests**: Write tests first. No exceptions.

**Never Ignore Type Errors**:

```typescript
// WRONG
const result = someFunction(); // @ts-ignore

// RIGHT
const result: ExpectedType = someFunction();
```

**Never Hardcode Secrets**:

```typescript
// WRONG
const API_KEY = "sk-1234567890abcdef";

// RIGHT
const API_KEY = process.env.API_KEY;
if (!API_KEY) throw new Error("API_KEY required");
```

</what_not_to_do>

<tool_invocation>

```bash
# Type checking
npx tsc --noEmit

# Linting
npx eslint src/ test/
npx eslint src/ test/ --fix

# Testing
npx vitest run --coverage
```

</tool_invocation>

<success_criteria>
Your implementation is ready for review when:

- [ ] Spec fully implemented
- [ ] All functions have type annotations
- [ ] All public functions have JSDoc
- [ ] Tests exist for all public functions
- [ ] tsc passes with zero errors
- [ ] eslint passes with zero errors
- [ ] All tests pass
- [ ] Coverage ≥80% for new code
- [ ] No TODOs/FIXMEs unaddressed
- [ ] No console.log statements
- [ ] No hardcoded secrets

*Your code will face an adversarial reviewer with zero tolerance. Write code that will survive that scrutiny.*
</success_criteria>
