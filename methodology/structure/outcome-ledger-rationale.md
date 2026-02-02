# Outcome Ledger: Design Rationale

This document explains the design decisions behind the outcome ledger. For the specification, see [outcome-ledger.md](outcome-ledger.md).

## The Duality: Git and the Outcome Ledger

Two parallel Merkle trees serve different purposes:

| Aspect            | Git                             | Outcome Ledger                  |
| ----------------- | ------------------------------- | ------------------------------- |
| Question answered | "What is the content?"          | "Did this content pass tests?"  |
| Tracks            | Specs, tests, implementation    | Verification state              |
| Merkle structure  | Content blobs → trees → commits | Descendant outcomes_blobs       |
| Proof             | Cryptographic (hash integrity)  | Empirical (run tests to verify) |

Git provides content integrity. The outcome ledger provides verification state. Neither duplicates the other's job.

## Why We Don't Store Spec/Test/Implementation Blobs

Early designs stored `spec_blob` and `test_blob` in outcomes.yaml to detect staleness:

```yaml
# REJECTED DESIGN
spec_blob: abc123
tests:
  - file: test.ts
    blob: def456
```

**Problems:**

1. **Incomplete coverage**: We cannot store implementation blobs without enumerating all source files each test depends on. The staleness detection would miss implementation changes.

2. **Duplicates Git's job**: Git already tracks content. Storing blobs in outcomes.yaml partially duplicates Git's Merkle tree, but incompletely.

3. **False confidence**: If spec_blob and test_blob match, we might skip running tests—but implementation could have changed, causing failures.

**Conclusion:** Partial blob storage provides unreliable staleness detection. Better to not store content blobs at all and rely on running tests.

## Why We Store Descendant outcomes_blobs

While we don't store content blobs, we DO store `outcomes_blob` for each child:

```yaml
descendants:
  - path: 10-parse.story/
    outcomes_blob: a3f2b7c
```

**This provides tree coupling:**

1. Child claims tests pass → child's outcomes.yaml updated
2. Child's Git blob changes
3. Parent's stored `outcomes_blob` no longer matches
4. Parent is Stale → must re-claim

**Why this works but content blobs don't:**

- `outcomes_blob` captures the child's ENTIRE verification state (its tests + ITS descendants)
- One blob reference cascades all the way down
- We're building a Merkle tree of OUTCOMES, not of CONTENT

## Alternative Designs Considered

### Option 1: Store Content Blobs

```yaml
spec_blob: abc123
test_blobs:
  - file: test.ts
    blob: def456
```

**Rejected because:** Ignores implementation. A test might fail due to src/ changes, but we'd skip it because spec/test blobs match.

### Option 2: Separate Outcome Commits

Structure:

```
A: code + spec + tests (code commit)
B: outcomes.yaml only (outcome commit)
```

Detect staleness via `git diff A..HEAD`.

**Rejected because:** Any code change invalidates all outcomes. This helps for auditing but not for test reduction during development.

### Option 3: Pure Claims (No Blobs at All)

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
    outcomes_blob: abc123
```

Store only descendant outcomes_blobs. Get tree coupling without duplicating Git.

**Chosen because:**

- Tree coupling via outcomes_blob
- No content blob duplication
- Clear separation: Git tracks content, ledger tracks verification

## Command Naming: `test` and `claim`

### Why `test` (not `run`, `check`)

- `test` is unambiguous: run the tests
- `run` is generic (run what?)
- `check` might imply status check without running

### Why `claim` (not `commit`, `record`, `pass`)

- `claim` emphasizes epistemological status: it's an assertion, not proof
- `commit` conflicts with `git commit`
- `record` is neutral but loses the claim semantics
- `pass` is confusing (verb vs test result)

**The word "claim" is intentionally precise:**

outcomes.yaml is a claim that can be verified independently. Timestamps are metadata, not evidence. Anyone can checkout the commit and run the claimed tests to verify the claim.

## Epistemology: Claims vs Proofs

The outcome ledger provides **claims**, not **proofs**.

| Aspect                | Claim                       | Proof                               |
| --------------------- | --------------------------- | ----------------------------------- |
| What outcomes.yaml is | Assertion that tests passed | Would require cryptographic witness |
| Timestamps            | Metadata (when claim made)  | Not evidence                        |
| Verification          | Run tests independently     | Would be redundant if proven        |
| Trust model           | Verify by re-running        | Trust the signature                 |

This is appropriate because:

1. **Tests are repeatable**: Anyone can run them
2. **Proofs are expensive**: Cryptographic witnesses add complexity
3. **Claims are sufficient**: CI verifies claims; if they hold, that's enough

## Test Reduction

The ledger reduces tests via a simple rule:

**Only run tests that are claimed to pass.**

- `verify` runs tests in outcomes.yaml
- Containers without outcomes.yaml are not verified (nothing claimed)
- Unknown/Pending containers don't block verification

This is NOT about skipping tests for "unchanged" code (unreliable). It's about knowing WHICH tests to run: the claimed ones.

## MECE Coverage

The design ensures every container is in exactly one state:

| State     | Detection                             |
| --------- | ------------------------------------- |
| Unknown   | No tests/ directory or empty          |
| Pending   | Tests exist, not all in outcomes.yaml |
| Stale     | Descendant outcomes_blob mismatch     |
| Passing   | verify succeeds                       |
| Regressed | verify fails                          |

These states are mutually exclusive and collectively exhaustive.
