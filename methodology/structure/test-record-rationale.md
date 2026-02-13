# Test Record: Design Rationale

This document explains the design decisions behind the test record. For the specification, see [test-record.md](test-record.md).

## The Duality: Git and the Test Record

Two parallel Merkle trees serve different purposes:

| Aspect            | Git                             | Test Record                    |
| ----------------- | ------------------------------- | ------------------------------ |
| Question answered | "What is the content?"          | "Did this content pass tests?" |
| Tracks            | Specs, tests, implementation    | Validation state               |
| Merkle structure  | Content blobs → trees → commits | Descendant record_blobs        |
| Basis             | Cryptographic (hash integrity)  | Empirical (run tests to check) |

Git provides content integrity. The test record provides validation state. Neither duplicates the other's job.

## Why We Don't Store Spec/Test/Implementation Blobs

Early designs stored `spec_blob` and `test_blob` in test-record.yaml to detect staleness:

```yaml
# REJECTED DESIGN
spec_blob: abc123
tests:
  - file: test.ts
    blob: def456
```

**Problems:**

1. **Incomplete coverage**: We cannot store implementation blobs without enumerating all source files each test depends on. The staleness detection would miss implementation changes.

2. **Duplicates Git's job**: Git already tracks content. Storing blobs in test-record.yaml partially duplicates Git's Merkle tree, but incompletely.

3. **False confidence**: If spec_blob and test_blob match, we might skip running tests—but implementation could have changed, causing failures.

**Conclusion:** Partial blob storage provides unreliable staleness detection. Better to not store content blobs at all and rely on running tests.

## Why We Store Descendant record_blobs

While we don't store content blobs, we DO store `record_blob` for each child:

```yaml
descendants:
  - path: 10-parse.story/
    record_blob: a3f2b7c
```

**This provides tree coupling:**

1. Child records passing tests → child's test-record.yaml updated
2. Child's Git blob changes
3. Parent's stored `record_blob` no longer matches
4. Parent is Stale → must re-record

**Why this works but content blobs don't:**

- `record_blob` captures the child's ENTIRE validation state (its tests + ITS descendants)
- One blob reference cascades all the way down
- We're building a Merkle tree of TEST RESULTS, not of CONTENT

## Alternative Designs Considered

### Option 1: Store Content Blobs

```yaml
spec_blob: abc123
test_blobs:
  - file: test.ts
    blob: def456
```

**Rejected because:** Ignores implementation. A test might fail due to src/ changes, but we'd skip it because spec/test blobs match.

### Option 2: Separate Recording Commits

Structure:

```text
A: code + spec + tests (code commit)
B: test-record.yaml only (recording commit)
```

Detect staleness via `git diff A..HEAD`.

**Rejected because:** Any code change invalidates all records. This helps for auditing but not for test reduction during development.

### Option 3: No Blobs at All

```yaml
tests:
  - file: test.ts
    passed_at: 2026-01-28T14:15:00Z
```

No tree coupling. Parent doesn't know if children changed.

**Rejected because:** Loses the Merkle property. Parent can be "Passing" while children changed underneath.

### Option 4: Minimal Merkle (Chosen)

```yaml
tests:
  - file: test.ts
    passed_at: 2026-01-28T14:15:00Z
descendants:
  - path: child/
    record_blob: abc123
```

Store only descendant record_blobs. Get tree coupling without duplicating Git.

**Chosen because:**

- Tree coupling via record_blob
- No content blob duplication
- Clear separation: Git tracks content, test record tracks validation

## Command Naming

### Why `test`

- Unambiguous: run the tests
- `run` is generic (run what?)

### Why `record` (not `claim`, `commit`, `pass`)

- `record` is plain — it records which tests pass
- `claim` overloads the word with epistemological weight the system doesn't need
- `commit` conflicts with `git commit`
- `pass` is confusing (verb vs test result)

### Why `check` (not `verify`, `validate`)

- `check` is plain — check that recorded results still hold
- `verify` and `validate` sound heavier than the operation warrants

**The test record is a record, not a claim:**

test-record.yaml records which tests passed and when. Anyone can checkout the commit and re-run the tests to confirm. Timestamps are metadata. The record is independently checkable.

## Test Reduction

The test record reduces tests via a simple rule:

**Only run tests that are recorded as passing.**

- `check` runs tests in test-record.yaml
- Nodes without test-record.yaml are not checked (nothing recorded)
- Unknown/Pending nodes don't block checking

This is NOT about skipping tests for "unchanged" code (unreliable). It's about knowing WHICH tests to run: the recorded ones.

## MECE Coverage

The design ensures every node is in exactly one state:

| State     | Detection                                |
| --------- | ---------------------------------------- |
| Unknown   | No tests/ directory or empty             |
| Pending   | Tests exist, not all in test-record.yaml |
| Stale     | Descendant record_blob mismatch          |
| Passing   | check succeeds                           |
| Regressed | check fails                              |

These states are mutually exclusive and collectively exhaustive.
