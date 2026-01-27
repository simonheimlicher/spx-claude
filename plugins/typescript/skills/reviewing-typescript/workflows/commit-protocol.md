# Commit Protocol (Phase 8)

> **APPROVED Only**: Committing is the reviewer's seal of approval.

After creating DONE.md, commit the completed work item.

**Follow the `committing-changes` skill** for core commit protocol (selective staging, verification, Conventional Commits format).

## Reviewer-Specific Context

When committing as part of review approval, apply these additional guidelines:

### Files to Stage (Work Item Scope)

Stage **only** files from the approved work item:

| Category            | Example Paths                            |
| ------------------- | ---------------------------------------- |
| Implementation      | `src/{modified files for this story}`    |
| Graduated tests     | `test/unit/`, `test/integration/`        |
| Completion evidence | `specs/doing/.../story-XX/tests/DONE.md` |

**Exclude**: Unrelated files, experimental code, files from other work items.

### Commit Message Context

Include work item reference in footer:

```text
feat({scope}): implement {story-slug}

- {brief description of what was implemented}
- Tests graduated to test/{location}/

Refs: {capability}/{feature}/{story}
```

### Return APPROVED

After successful commit:

```markdown
## Verdict: APPROVED

Commit: {commit_hash}
Files committed: {count}
Tests graduated: {list}

Work item is DONE.
```
