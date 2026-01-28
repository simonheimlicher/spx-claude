---
name: coding-typescript
description: Write TypeScript code that's type-safe and tested. Use when coding TypeScript or implementing features.
allowed-tools: Read, Write, Bash, Glob, Grep, Edit
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
**NO MOCKING. DEPENDENCY INJECTION. BEHAVIOR ONLY. TEST FIRST.**

- Use **dependency injection**, NEVER mocking frameworks
- Test **behavior** (what the code does), not implementation (how it does it)
- Run all verification tools before declaring completion
- Type safety first: `strict: true`, no `any` without justification

</essential_principles>

<hierarchy_of_authority>
**Where to look for guidance, in order of precedence:**

| Priority | Source                    | What It Provides                                      |
| -------- | ------------------------- | ----------------------------------------------------- |
| 1        | `docs/`, `README.md`      | Project architecture, design decisions, intended APIs |
| 2        | `CLAUDE.md`               | Project-specific rules for Claude                     |
| 3        | ADRs, specs               | Documented decisions and requirements                 |
| 4        | This skill (`SKILL.md`)   | Generic TypeScript best practices                     |
| 5        | Existing code (reference) | Evidence of implementation, NOT authority             |

**CRITICAL: Existing code is NOT authoritative.**

- Documentation describes **intent** — what SHOULD be done
- Existing code shows **implementation** — what WAS done (may be legacy, wrong, or outdated)
- When docs and code conflict, **docs win**
- When no docs exist, ASK before copying existing patterns

**Never copy patterns from existing code without verifying they match documented intent.**

</hierarchy_of_authority>

<codebase_discovery>
**BEFORE writing any code, discover what already exists.**

### Phase 0: Discovery (MANDATORY)

Run these searches before implementation:

```bash
# 1. Read project documentation
Read: README.md, docs/, CLAUDE.md, CONTRIBUTING.md

# 2. Check available dependencies (don't add what exists)
Read: package.json → dependencies, devDependencies

# 3. Find prior art for what you're building
Grep: function names, class names, patterns similar to your task
Glob: files in similar directories (src/utils/, src/services/, etc.)

# 4. Detect project conventions
Read: existing files in the same directory you'll write to
```

### What to Discover

| Question                        | How to Find                                 |
| ------------------------------- | ------------------------------------------- |
| What libraries are available?   | `package.json` → dependencies               |
| How does this project handle X? | `Grep` for similar patterns                 |
| What utilities already exist?   | `Glob` for `**/utils/**`, `**/helpers/**`   |
| What's the naming convention?   | Read 3-5 files in the target directory      |
| What error classes exist?       | `Grep` for `extends Error`                  |
| What logging pattern is used?   | `Grep` for `logger`, `console.log`, `debug` |
| How are configs structured?     | `Glob` for `**/*.config.*`, `**/config/**`  |

### Discovery Anti-Patterns

```typescript
// ❌ WRONG: Adding lodash when ramda is already used
import _ from "lodash"; // package.json has ramda, not lodash

// ❌ WRONG: Creating new logger when one exists
const logger = console; // Project has @lib/logger

// ❌ WRONG: Inventing naming convention
function fetch_user_by_id() {} // Project uses camelCase

// ❌ WRONG: New error class when domain errors exist
class MyError extends Error {} // Project has @/errors
```

### Discovery Checklist

Before writing code, confirm:

- [ ] Read `package.json` — know what libraries are available
- [ ] Searched for prior art — found (or confirmed none exists)
- [ ] Identified naming conventions from existing files
- [ ] Found existing utilities to reuse (or confirmed none exist)
- [ ] Checked for existing error classes, loggers, configs

**If discovery reveals existing patterns that conflict with this skill's guidance, follow the project's documented patterns.**

</codebase_discovery>

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
- All requirements documents exist (PRD at product level)
- All architectural decisions (ADRs) are read and understood
- Complete hierarchical context is loaded (Product → Capability → Feature → Story)

**Example invocation:**

```bash
# By work item path
specs:understanding-specs 10-cli.capability/20-commands.feature/30-build.story

# By story name
specs:understanding-specs 30-build.story
```

**If `specs:understanding-specs` returns an error**: The error message will specify which document is missing and how to create it. Create the missing document before proceeding with implementation.

**If NOT working on specs-based work item**: Proceed directly to implementation mode with provided spec.
</context_loading>

<two_modes>
You operate in one of two modes depending on your input:

| Input                            | Mode               | Workflow                      |
| -------------------------------- | ------------------ | ----------------------------- |
| Spec (ADR, feature spec)         | **Implementation** | `workflows/implementation.md` |
| Rejection feedback from reviewer | **Remediation**    | `workflows/remediation.md`    |

Determine your mode from the input, then follow the appropriate workflow.
</two_modes>

<core_principles>

1. **Spec Is Law**: The specification is your contract. Implement exactly what it says.

2. **Test-Driven Development**: Write tests first or alongside code. Tests prove correctness.

3. **Type Safety First**: Use strict TypeScript with `strict: true`. No `any` without justification.

4. **Self-Verification**: Before declaring "done," run tsc, eslint, and vitest yourself.

5. **Humility**: Your code must pass review. Write code that will survive adversarial review.

6. **Clean Architecture**: Dependency injection, single responsibility, no circular imports, **no deep relative imports**.

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

**Never Use Deep Relative Imports**:

Before writing any import, ask: *"Is this a module-internal file (same module, moves together) or infrastructure (lib/, tests/helpers/, shared/)?"*

```typescript
// WRONG: Deep relatives to stable locations — will REJECT in review
import { helper } from "../../../../../../tests/helpers/tree-builder";
import { Logger } from "../../../../lib/logging";
import { Config } from "../../../shared/config";

// RIGHT: Configure path aliases in tsconfig.json
import { Logger } from "@lib/logging";
import { Config } from "@shared/config";
import { helper } from "@test/helpers/tree-builder";
```

**Depth Rules:**

- `./sibling` — ✅ OK (same directory, module-internal)
- `../parent` — ⚠️ Review (is it truly module-internal?)
- `../../` or deeper — ❌ REJECT (use path alias)

**Configure tsconfig.json:**

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@test/*": ["tests/*"],
      "@lib/*": ["lib/*"]
    }
  }
}
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
