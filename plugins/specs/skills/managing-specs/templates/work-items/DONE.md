# Completion Evidence

> **Work Item**: [work-item-name]
> **Completed**: [YYYY-MM-DD]

This file marks the work item as complete and provides evidence that all requirements are met.

---

## Test Coverage by Level

> See [testing standards](/docs/testing/standards.md) for level definitions.

| Level | Required For | Status | Test Count |
| ----- | ------------ | ------ | ---------- |
| 1     | Stories      | ✓      | [N] tests  |
| 2     | Features     | ✓      | [N] tests  |
| 3     | Capabilities | ✓      | [N] tests  |

## Verified Tests

Tests that prove functional requirements are met. Tests remain co-located with this work item.

| Requirement                       | Test Location                          | Level |
| --------------------------------- | -------------------------------------- | ----- |
| [Requirement text from work item] | `tests/loader.unit.test.ts::test_name` | 1     |
| [Requirement text from work item] | `tests/hugo.integration.test.ts::name` | 2     |
| [Requirement text from work item] | `tests/cli.e2e.test.ts::test_name`     | 3     |

### For Features and Capabilities

In addition to own tests, verify all children are complete:

```bash
# Feature: verify all story DONE.md exist
find . -name "story-*" -type d -exec test -f {}/DONE.md \; -print

# Capability: verify all feature DONE.md exist
find . -name "feature-*" -type d -exec test -f {}/DONE.md \; -print
```

---

## Testing Principles Verified

| Principle            | Evidence                                          |
| -------------------- | ------------------------------------------------- |
| Behavior tested      | Tests verify WHAT happens, not HOW                |
| No mocking           | All tests use DI pattern                          |
| Generated test data  | Uses factories from `tests/fixtures/factories.ts` |
| BDD structure        | All tests follow GIVEN/WHEN/THEN                  |
| Escalation justified | Level 2+ tests document why Level 1 insufficient  |

---

## Non-Functional Verification

Evidence that coding standards and ADR/PDR requirements are met.

| Standard             | Evidence                                    |
| -------------------- | ------------------------------------------- |
| Type annotations     | All functions have TypeScript type hints    |
| Zod validation       | Config validated at load time               |
| Dependency injection | Functions accept deps parameter for testing |
| Error messages       | All errors include actionable context       |

---

## Test Run Output

```bash
$ npm test -- specs/work/doing/.../tests/

 ✓ tests/loader.unit.test.ts (5 tests)
 ✓ tests/lhci.unit.test.ts (3 tests)
 ✓ tests/hugo.integration.test.ts (2 tests)
 ✓ tests/cli.e2e.test.ts (2 tests)

Test Files  4 passed (4)
Tests       12 passed (12)
Coverage:   85%
```

---

## Notes

[Any relevant notes about the implementation, deviations, or follow-up work identified]
