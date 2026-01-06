---
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git commit:*)
description: Commit with Conventional Commits
argument-hint: [files-to-stage]
---

## Git Context

**Status:**
!`git status --short`

**Staged changes:**
!`git diff --cached --stat`

**Unstaged changes:**
!`git diff --stat`

**Recent commits (for style reference):**
!`git log --oneline -5`

**Current branch:**
!`git branch --show-current`

## Task

Create a git commit following Conventional Commits.

### If files specified: `$ARGUMENTS`

Stage those files, then commit.

### If no files specified

Review the status above. If nothing is staged, ask which files to stage.

## Commit Format

```
<type>[(scope)]: <description>

[optional body]
```

**Types:** feat | fix | docs | style | refactor | perf | test | ci | build | revert | ctx

**Rules:**

- Imperative mood: "add" not "added"
- Subject: 50 chars max, no period
- Body: wrap at 72 chars, explain WHY not HOW
- NO attribution (never include author names)

## Workflow

1. Stage files selectively (never `git add .` unless explicitly requested)
2. Review staged diff: `git diff --cached`
3. Verify single purpose (one logical change)
4. Create commit with HEREDOC:
   ```bash
   git commit -m "$(cat <<'EOF'
   type(scope): subject line

   Body explaining why.
   EOF
   )"
   ```

## Red Flags - Ask Before Committing

- More than 10 files for a simple fix
- Mix of unrelated changes
- Debug code (console.log, print statements)
- Experimental/incomplete work
