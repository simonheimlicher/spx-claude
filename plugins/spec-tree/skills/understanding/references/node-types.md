<overview>
The Spec Tree contains two node types. Every directory in the tree (other than the root and `tests/`) is one of these.
</overview>

<enabler>

**Directory suffix:** `.enabler`
**Spec header:** `## Enables`
**Purpose:** Infrastructure that would be removed if all its dependents were retired.

Enablers exist to serve other nodes. They provide shared infrastructure, utilities, or foundational capabilities that higher-index siblings and their descendants depend on.

**Spec format:**

```markdown
# Node Name

## Enables

[What this enabler provides to its dependents and why they need it]

## Assertions

### Scenarios

- Given context, when action, then result ([test](tests/file.unit.test.ts))

### Compliance

- ALWAYS: behavioral rule — reason
```

**Examples:**

- Test harness that all other nodes use
- Parser that multiple outcome nodes depend on
- State machine that several features build on
- Shared configuration or bootstrap logic

**When to create an enabler:**

- Two or more sibling nodes share a need → factor it into an enabler at a lower index
- Infrastructure that has no direct user-facing value but enables user-facing value
- Removing it would break its dependents

</enabler>

<outcome>

**Directory suffix:** `.outcome`
**Purpose:** Hypothesis connecting a testable output to a measurable change in user behavior and its expected business impact.

The hypothesis has three parts:

- **Output** — what the software does. Assertions specify this. Locally verifiable by tests or review.
- **Outcome** — measurable change in user behavior the output is expected to produce. Requires real users to validate.
- **Impact** — business value: increase revenue, sustain revenue, reduce costs, or avoid costs.

Assertions specify the **output** — not the outcome or impact. You can test what the software does; you can only hypothesize about the user behavior change and business value it leads to.

**Spec format:**

```markdown
# Node Name

WE BELIEVE THAT [output]
WILL [outcome]
CONTRIBUTING TO [impact]

## Assertions

### Scenarios

- Given context, when action, then result ([test](tests/file.unit.test.ts))

### Compliance

- ALWAYS: behavioral rule — reason
```

**When to create an outcome:**

- The behavior has direct or indirect user-facing value
- You can express a hypothesis about what change its output produces
- You can define assertions that specify the output

</outcome>

<common_structure>

**Directory structure:**

```text
NN-slug.{enabler|outcome}/
├── slug.md              # Spec file (no type suffix, no numeric prefix)
├── spx-lock.yaml        # Lock file (written by spx lock when tests pass)
├── tests/               # Co-located test files
│   ├── slug.unit.test.ts
│   └── slug.integration.test.ts
└── NN-child.{enabler|outcome}/   # Nested child nodes (optional)
```

**Spec file naming:**

- The spec file is always `{slug}.md` — no type suffix, no numeric prefix
- The slug matches the directory name without the numeric prefix and type suffix
- Example: `43-status-rollup.outcome/` contains `status-rollup.md`

**Test files:**

- Co-located in `tests/` within the node directory
- Named with test level suffix: `.unit.test.{ext}`, `.integration.test.{ext}`, `.e2e.test.{ext}`
- Assertions specify output, verified by test (`[test]`) or review (`[review]`)

</common_structure>

<lock_file>

`spx-lock.yaml` binds spec content to test evidence via Git blob hashes:

```yaml
schema: spx-lock/v1
blob: a3b7c12
tests:
  - path: tests/slug.unit.test.ts
    blob: 9d4e5f2
```

- `blob` is the Git blob hash of the spec file
- Each test entry has the path and blob hash of the test file
- Lock is written only when all tests pass
- Same state always produces the same lock file (deterministic)

**Node states (derived from lock):**

| State          | Condition                                 | Required action         |
| -------------- | ----------------------------------------- | ----------------------- |
| **Needs work** | No lock file exists                       | Write tests, then lock  |
| **Stale**      | Spec or test blob changed since last lock | Re-lock (`spx lock`)    |
| **Valid**      | All blobs match                           | None — evidence current |

</lock_file>

<product_file>

The root of every tree contains `{product-name}.product.md`. This is not a node — it's the tree's anchor. It captures why the product exists and what change in user behavior it aims to achieve.

```text
spx/
├── product-name.product.md       # Product spec (the root)
├── 15-decision.adr.md            # Product-level decision
├── 15-constraint.pdr.md          # Product-level constraint
├── 21-first.enabler/             # First enabler node
└── 32-first.outcome/             # First outcome node
```

</product_file>

<decision_records>

ADRs and PDRs are flat files, not directories. They sit alongside nodes at any directory level:

- `NN-slug.adr.md` — Architecture Decision Record (governs HOW)
- `NN-slug.pdr.md` — Product Decision Record (governs WHAT)

Their numeric prefix encodes dependency scope: a decision at index 15 constrains all siblings at index 16 and above, including their descendants.

</decision_records>
