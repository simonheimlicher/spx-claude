---
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git commit:*), Skill
description: Commit following Conventional Commits
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

Review the status above. If nothing is staged, carefully review all changes and create one or several logical commits.

## Execution Instructions

**IMPORTANT:** When this command is invoked, you MUST:

1. **Invoke the `core:committing-changes` skill** using the Skill tool

## What This Does

Activates the `core:committing-changes` skill. You must follow its instructions to the letter.
