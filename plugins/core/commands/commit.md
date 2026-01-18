---
allowed-tools: Skill
description: Commit following Conventional Commits
argument-hint: [files-to-stage]
---

# STOP - Invoke Skill First

**Do NOT proceed manually.** Invoke the skill immediately:

```
Skill: core:committing-changes
```

## Context for the Skill

**Arguments:** `$ARGUMENTS`

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
