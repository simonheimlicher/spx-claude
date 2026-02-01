# Commit Protocol (Phase 8)

> **APPROVED Only**: Committing is the reviewer's seal of approval.

After committing outcomes, commit the completed work item.

**Follow the `committing-changes` skill** for core commit protocol (selective staging, verification, Conventional Commits format).

## Reviewer-Specific Context

When committing as part of review approval, apply these additional guidelines:

### Files to Stage (Work Item Scope)

Stage **only** files from the approved work item:

| Category            | Example Paths                             |
| ------------------- | ----------------------------------------- |
| Implementation      | `src/{modified files for this story}`     |
| Co-located tests    | `spx/.../NN-{slug}.story/tests/*.test.ts` |
| Verification ledger | `spx/.../NN-{slug}.story/outcomes.yaml`   |

**Exclude**: Unrelated files, experimental code, files from other work items.

### Commit Message Context

Include work item reference in footer:

```text
feat({scope}): implement {story-slug}

- {brief description of what was implemented}
- Tests verified in outcomes.yaml

Refs: {capability}/{feature}/{story}
```

### Return APPROVED

After successful commit:

```markdown
## Verdict: APPROVED

Commit: {commit_hash}
Files committed: {count}
Tests verified: {list from outcomes.yaml}

Work item is DONE.
```
