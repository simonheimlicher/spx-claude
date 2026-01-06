# Pre-Submission Verification Checklist

Before declaring "done," confirm:

## Required Checks

- [ ] All tsc errors resolved
- [ ] All eslint errors resolved (auto-fix applied where safe)
- [ ] All tests pass
- [ ] Coverage ≥80% for new code
- [ ] JSDoc present for all public functions
- [ ] No TODO/FIXME comments left unaddressed
- [ ] No console.log statements (use logger)
- [ ] No hardcoded secrets or paths

## Tool Commands

```bash
# Type checking (must report 0 errors)
npx tsc --noEmit

# Linting (must report 0 errors)
npx eslint src/ test/

# Auto-fix style issues
npx eslint src/ test/ --fix

# Run all tests with coverage
npx vitest run --coverage

# Or if npm scripts are defined
npm run typecheck
npm run lint
npm test
```

## Completion Criteria Table

| Criterion                            | Status   |
| ------------------------------------ | -------- |
| Spec fully implemented               | Required |
| All functions have type annotations  | Required |
| All public functions have JSDoc      | Required |
| Tests exist for all public functions | Required |
| tsc passes with zero errors          | Required |
| eslint passes with zero errors       | Required |
| All tests pass                       | Required |
| Coverage ≥80% for new code           | Required |
| No TODOs/FIXMEs unaddressed          | Required |
| No console.log statements            | Required |
| No hardcoded secrets                 | Required |

## What to Check For

### Type Safety

- No `any` without explicit justification
- No `@ts-ignore` without explanation (prefer `@ts-expect-error`)
- All function parameters and returns are typed

### Code Quality

- Public functions have JSDoc with @param/@returns/@throws
- No dead code or commented-out code blocks
- Constants are UPPER_SNAKE_CASE, no magic numbers
- Dependencies injected via parameters

### Testing

- Tests exist for all public functions
- Tests cover edge cases
- Tests use descriptive names
- No mocking - dependency injection only
