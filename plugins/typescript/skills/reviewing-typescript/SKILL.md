---
name: reviewing-typescript
description: Review TypeScript code strictly, reject mocking. Use when reviewing TypeScript code, checking if code is ready for merge, or validating implementations.
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

<objective>
Adversarial code review enforcing zero-tolerance on mocking, type safety, and security.
</objective>

<quick_start>

1. Read `/testing-typescript` for methodology and test level decisions
2. Run the project validation command from `CLAUDE.md`/`README.md` (prefer `pnpm validate`)
3. Do NOT run `tsc`, `eslint`, or `semgrep` individually in review
4. Run Phase 2 infrastructure provisioning
5. Run Phase 3 tests with coverage
6. Complete Phase 4 manual checklist (including ADR/PDR compliance)
7. Determine verdict (APPROVED/REJECTED)
8. If APPROVED: commit outcome and commit work item

</quick_start>

<essential_principles>
**TRUST NO ONE. REJECT MOCKING. ZERO TOLERANCE.**

- If you cannot **verify** something is correct, it is **incorrect**
- **REJECT any use of mocking** — only dependency injection is acceptable
- "It works on my machine" is not evidence. Tool output is evidence.
- Project validation output is truth. Your subjective opinion is secondary.
- Absence = Failure. If the project validation command cannot run, the review is REJECTED.

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
**For spx-based work items: Verify context was loaded before reviewing.**

If you're reviewing code for a spec-driven work item (story/feature/capability), verify the implementation has complete context:

1. **Check that `spx:understanding-specs` was invoked** - Look for context loading in implementation
2. **Verify all ADRs/PDRs are referenced** - Implementation should follow decision records (interleaved in containers)
3. **Verify feature spec requirements met** - Code should satisfy documented requirements

**The `spx:understanding-specs` skill provides:**

- Complete ADR/PDR hierarchy (product/capability/feature decisions)
- Feature spec with validation strategy and acceptance criteria
- Story/feature/capability spec with functional requirements

**Review focus:**

- Does implementation honor all ADRs/PDRs in hierarchy?
- Does implementation satisfy feature spec validation strategy?
- Does implementation meet story/feature acceptance criteria?

**If NOT working on spx-based work item**: Proceed directly with code review using provided specification.
</context_loading>

<verdict_definitions>

| Verdict      | Criteria                        | Next Phase         |
| ------------ | ------------------------------- | ------------------ |
| **APPROVED** | All checks pass, no issues      | Commit outcome     |
| **REJECTED** | Any issue or unverifiable state | Coder fixes issues |

**APPROVED Criteria** (all must be true):

1. Project validation command passes (prefer `pnpm validate`)
2. All tests pass
3. Manual review checklist satisfied
4. No security concerns identified
5. Output contains no notes/warnings/caveats sections

**REJECTED Criteria** (any triggers rejection):

- Project validation command fails
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

| Phase | Name                    | Workflow                             |
| ----- | ----------------------- | ------------------------------------ |
| 0-5   | Review Protocol         | `workflows/review-protocol.md`       |
| 6-7   | Verification (APPROVED) | `workflows/verification-protocol.md` |
| 8     | Commit (APPROVED)       | `workflows/commit-protocol.md`       |

**If verdict is APPROVED**: Continue to Phase 6.
**If verdict is REJECTED**: Skip verification and commit, return feedback.
</review_phases>

<reference_index>

| File                                    | Purpose                                    |
| --------------------------------------- | ------------------------------------------ |
| `references/verification-tests.md`      | Test verification rules and patterns       |
| `references/false-positive-handling.md` | When violations are/aren't false positives |
| `references/manual-review-checklist.md` | Phase 4 manual review items                |

</reference_index>

<workflows_index>

| Workflow                             | Purpose                                  |
| ------------------------------------ | ---------------------------------------- |
| `workflows/review-protocol.md`       | Project validation, tests, manual review |
| `workflows/verification-protocol.md` | Verify tests pass, commit outcomes       |
| `workflows/commit-protocol.md`       | Stage and commit approved work           |

</workflows_index>

<tool_resources>

| File                         | Purpose                      |
| ---------------------------- | ---------------------------- |
| `rules/tsconfig.strict.json` | Project validation reference |
| `rules/eslint.config.js`     | Project validation reference |
| `rules/semgrep_sec.yaml`     | Project validation reference |
| `templates/review_report.md` | Report template              |

</tool_resources>

<output_format>

```markdown
## Review: {target}

### Verdict: [APPROVED / REJECTED]

[One-sentence summary]

### Project Validation

| Tool                               | Status    | Issues                   |
| ---------------------------------- | --------- | ------------------------ |
| validate command (`pnpm validate`) | PASS/FAIL | [failure summary if any] |

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

- [ ] Project validation command run (`pnpm validate` when available)
- [ ] All tests executed with coverage measured
- [ ] Manual review checklist completed
- [ ] Verdict determined and documented
- [ ] If APPROVED: tests verified, outcomes committed, changes committed
- [ ] If REJECTED: actionable feedback provided with file:line references

*Your job is to protect the codebase from defects. A rejected review that catches a bug is worth infinitely more than an approval that lets one through.*
</success_criteria>
