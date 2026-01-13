---
name: reviewing-typescript
description: Review TypeScript code strictly, reject mocking. Use when reviewing TypeScript code or checking if code is ready.
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

<accessing_skill_files>
When this skill is invoked, Claude Code provides the base directory in the loading message:

```
Base directory for this skill: {skill_dir}
```

Use this path to access skill files:

- References: `{skill_dir}/references/`
- Workflows: `{skill_dir}/workflows/`
- Templates: `{skill_dir}/templates/`
- Rules: `{skill_dir}/rules/`

**IMPORTANT**: Do NOT search the project directory for skill files.
</accessing_skill_files>

<essential_principles>
**TRUST NO ONE. REJECT MOCKING. ZERO TOLERANCE.**

- If you cannot **verify** something is correct, it is **incorrect**
- **REJECT any use of mocking** — only dependency injection is acceptable
- "It works on my machine" is not evidence. Tool output is evidence.
- Tool outputs are truth. Your subjective opinion is secondary.
- Absence = Failure. If you cannot run a verification tool, the code fails.

</essential_principles>

<test_verification>
**This reviewer enforces testing principles from `/testing-typescript` skill.**

When reviewing tests, verify:

1. **REJECT any mocking** — `vi.mock()`, `jest.mock()` = REJECTED
2. **Verify dependency injection** — External deps must be injected, not mocked
3. **Verify behavior testing** — Tests must verify outcomes, not implementation
4. **Verify correct test level** — Unit/Integration/E2E assignment per `/testing-typescript`

**Rejection Criteria:**

| Violation            | Example                                    | Verdict  |
| -------------------- | ------------------------------------------ | -------- |
| Uses mocking         | `vi.mock('execa')`                         | REJECTED |
| Tests implementation | `expect(mockFn).toHaveBeenCalledWith(...)` | REJECTED |
| Wrong level          | Unit test for Chrome automation            | REJECTED |
| Arbitrary test data  | `"test@example.com"` hardcoded             | REJECTED |
| Deep relative import | `from "../../../../../../tests/helpers"`   | REJECTED |

```typescript
// ❌ REJECT: Mocking
vi.mock("execa", () => ({ execa: vi.fn() }));

// ✅ ACCEPT: Dependency Injection
const deps: CommandDeps = {
  execa: vi.fn().mockResolvedValue({ exitCode: 0 }),
};
const result = await runCommand(args, deps);
expect(result.success).toBe(true); // Tests behavior
```

</test_verification>

<context_loading>
**For specs-based work items: Verify context was loaded before reviewing.**

If you're reviewing code for a spec-driven work item (story/feature/capability), verify the implementation has complete context:

1. **Check that `specs:understanding-specs` was invoked** - Look for context loading in implementation
2. **Verify all ADRs are referenced** - Implementation should follow architectural decisions
3. **Verify TRD/spec requirements met** - Code should satisfy documented requirements

**The `specs:understanding-specs` skill provides:**

- Complete ADR hierarchy (product/capability/feature decisions)
- TRD with validation strategy and acceptance criteria
- Story/feature/capability spec with functional requirements

**Review focus:**

- Does implementation honor all ADRs in hierarchy?
- Does implementation satisfy TRD validation strategy?
- Does implementation meet story/feature acceptance criteria?

**If NOT working on specs-based work item**: Proceed directly with code review using provided specification.
</context_loading>

<verdict_definitions>

| Verdict         | Criteria                                                   | Next Phase          |
| --------------- | ---------------------------------------------------------- | ------------------- |
| **APPROVED**    | All checks pass, no issues                                 | Graduation + Commit |
| **CONDITIONAL** | Only false-positive violations needing disable comments    | Coder adds comments |
| **REJECTED**    | Real bugs, security issues, test failures, design problems | Coder fixes issues  |
| **BLOCKED**     | Infrastructure cannot be provisioned                       | Fix environment     |

**APPROVED Criteria** (all must be true):

1. tsc reports zero errors
2. eslint reports zero error-level violations
3. Semgrep reports zero findings
4. All tests pass
5. Manual review checklist satisfied
6. No security concerns identified

**REJECTED Criteria** (any triggers rejection):

- Any real type error (tsc)
- Any true-positive security violation
- Any test failure
- Missing type annotations on public functions
- Empty catch blocks
- Hardcoded secrets
- `eval()` or `new Function()` usage
- `child_process.exec()` with untrusted input
- Deep relative imports (2+ levels of `../`) to stable locations

</verdict_definitions>

<review_phases>
Execute these phases IN ORDER. Do not skip phases.

| Phase | Name                  | Workflow                           |
| ----- | --------------------- | ---------------------------------- |
| 0-5   | Review Protocol       | `workflows/review-protocol.md`     |
| 6-7   | Graduation (APPROVED) | `workflows/graduation-protocol.md` |
| 8     | Commit (APPROVED)     | `workflows/commit-protocol.md`     |

**If verdict is APPROVED**: Continue to Phase 6.
**If verdict is NOT APPROVED**: Skip graduation and commit, return feedback.
</review_phases>

<reference_index>

| File                                    | Purpose                                    |
| --------------------------------------- | ------------------------------------------ |
| `references/verification-tests.md`      | Test verification rules and patterns       |
| `references/false-positive-handling.md` | When violations are/aren't false positives |
| `references/manual-review-checklist.md` | Phase 4 manual review items                |

</reference_index>

<workflows_index>

| Workflow                           | Purpose                               |
| ---------------------------------- | ------------------------------------- |
| `workflows/review-protocol.md`     | Static analysis, tests, manual review |
| `workflows/graduation-protocol.md` | Move tests to test/, create DONE.md   |
| `workflows/commit-protocol.md`     | Stage and commit approved work        |

</workflows_index>

<tool_resources>

| File                         | Purpose                         |
| ---------------------------- | ------------------------------- |
| `rules/tsconfig.strict.json` | Strict TypeScript configuration |
| `rules/eslint.config.js`     | ESLint with security rules      |
| `rules/semgrep_sec.yaml`     | Custom security pattern rules   |
| `templates/review_report.md` | Report template                 |

</tool_resources>

<output_format>

```markdown
## Review: {target}

### Verdict: [APPROVED / CONDITIONAL / REJECTED / BLOCKED]

[One-sentence summary]

### Static Analysis

| Tool    | Status    | Issues                           |
| ------- | --------- | -------------------------------- |
| tsc     | PASS/FAIL | [count] errors                   |
| eslint  | PASS/FAIL | [count] errors, [count] warnings |
| Semgrep | PASS/FAIL | [count] findings                 |

### Tests

| Metric   | Value      |
| -------- | ---------- |
| Passed   | [count]    |
| Failed   | [count]    |
| Coverage | [percent]% |

### Blocking Issues (if REJECTED)

1. **[file:line]** - [category] - [description]

### Report Location

Full report: `reports/review_{name}_{timestamp}.md`
```

</output_format>

<success_criteria>
Review is complete when:

- [ ] All static analysis tools run (tsc, eslint, Semgrep)
- [ ] All tests executed with coverage measured
- [ ] Manual review checklist completed
- [ ] Verdict determined and documented
- [ ] If APPROVED: tests graduated, DONE.md created, changes committed
- [ ] If REJECTED: actionable feedback provided with file:line references

*Your job is to protect the codebase from defects. A rejected review that catches a bug is worth infinitely more than an approval that lets one through.*
</success_criteria>
