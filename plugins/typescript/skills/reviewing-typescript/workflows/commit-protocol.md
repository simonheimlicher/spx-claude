# Commit Protocol (Phase 8)

> **APPROVED Only**: Committing is the seal of approval.

After creating DONE.md, commit the completed work item.

## 8.1 Stage Files Selectively

**NEVER use `git add .` or `git add -A`**. Stage only files from this work item:

```bash
# Stage implementation files
git add src/{files modified for this story}

# Stage graduated tests
git add test/unit/{graduated test files}
git add test/integration/{graduated test files}

# Stage completion evidence
git add specs/doing/.../story-XX/tests/DONE.md
```

## 8.2 Verify Staged Changes

Before committing, verify only work item files are staged:

```bash
git diff --cached --name-only
# Must show ONLY files from this work item

# If unrelated files appear, unstage them:
git restore --staged {unrelated_file}
```

## 8.3 Commit with Conventional Message

Use HEREDOC for proper formatting:

```bash
git commit -m "$(cat <<'EOF'
feat({scope}): implement {story-slug}

- {brief description of what was implemented}
- Tests graduated to test/{location}/

Refs: {capability}/{feature}/{story}
EOF
)"
```

**Scope**: Use the feature or module name (e.g., `auth`, `sync`, `cli`)

## 8.4 Verify Commit Succeeded

```bash
git log -1 --oneline
# Should show the commit just created

git status
# Should show clean working directory for committed files
```

**If commit fails** (e.g., pre-commit hook rejection):

1. Fix the issue identified by the hook
2. Re-stage the fixed files
3. Create a NEW commit (do not amend)
4. Return to 8.4

## 8.5 Return APPROVED

After successful commit, return verdict:

```markdown
## Verdict: APPROVED

Commit: {commit_hash}
Files committed: {count}
Tests graduated: {list}

Work item is DONE.
```
