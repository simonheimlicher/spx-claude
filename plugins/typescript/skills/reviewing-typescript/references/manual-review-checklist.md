# Manual Review Checklist (Phase 4)

Read ALL code under review. Check each item:

## Type Safety (Beyond tsc)

- [ ] No use of `any` without explicit justification
- [ ] No `@ts-ignore` without explanation comment (prefer `@ts-expect-error`)
- [ ] Union types are narrowed before use
- [ ] Generic types are properly constrained
- [ ] `as` assertions are justified (prefer type guards)

## Error Handling

- [ ] No empty `catch` blocks (swallowing all errors)
- [ ] Custom error classes for domain errors
- [ ] Error messages include context (what failed, with what input)
- [ ] Async errors are properly caught (no unhandled rejections)

## Resource Management

- [ ] Streams and connections properly closed
- [ ] `finally` blocks for cleanup where needed
- [ ] Timeouts specified for network/subprocess operations
- [ ] AbortController used for cancellable operations

## Security

- [ ] No hardcoded secrets, API keys, or passwords
- [ ] No `eval()` or `new Function()` usage
- [ ] No `child_process.exec()` with user input (use `execFile`)
- [ ] Input validation present where needed
- [ ] SSL verification enabled for HTTP requests

## Code Quality

- [ ] Public functions have JSDoc with @param/@returns/@throws
- [ ] No dead code or commented-out code blocks
- [ ] No unused imports
- [ ] Function names are verbs (`getUser`, `calculateTotal`)
- [ ] Class names are nouns (`UserRepository`, `PaymentProcessor`)
- [ ] Constants are UPPER_SNAKE_CASE
- [ ] No magic numbers (use named constants)

## Architecture

- [ ] Dependencies injected via parameters, not imported globals
- [ ] No circular imports
- [ ] Single responsibility per module/class
- [ ] Clear separation of concerns (IO vs logic)

## Testing

**Test Existence & Coverage**:

- [ ] Tests exist for public functions
- [ ] Tests cover edge cases (empty inputs, null, large values)
- [ ] Tests use descriptive names that explain the scenario
- [ ] No hardcoded paths or environment-specific values in tests
- [ ] Fixtures clean up after themselves

See `references/verification-tests.md` for test organization rules.
