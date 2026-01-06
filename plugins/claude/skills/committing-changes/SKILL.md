---
name: committing-changes
description: |
  Write git commit messages following Conventional Commits with pre-commit verification.
  This skill should be used when users ask to commit changes, write a commit message,
  stage files for commit, or need help with git commit standards.
---

# Commit Message

Write effective git commits with Conventional Commits standard and pre-commit verification.

## What This Skill Does

- Guides selective file staging (never `git add .`)
- Writes commit messages in Conventional Commits format
- Verifies atomic commit principles
- Adapts commit types to project domain

## What This Skill Does NOT Do

- Push commits to remote
- Create pull requests
- Modify git configuration
- Bypass pre-commit hooks

---

## Before Creating Any Commit

Gather context:

| Source           | Gather                                                  |
| ---------------- | ------------------------------------------------------- |
| **git status**   | Staged, unstaged, untracked files                       |
| **git diff**     | Actual changes to commit                                |
| **git log**      | Recent commit style for consistency                     |
| **Project docs** | Custom commit types (CLAUDE.md, CONTRIBUTING.md)        |
| **Conversation** | User's intent - what story/issue does this commit solve |

---

## Pre-Commit Verification Protocol

### Step 1: Selective Staging

```bash
# NEVER do this
git add .

# ALWAYS stage specific files
git add path/to/file1.ts path/to/file2.ts
```

**Rules:**

- One logical change per commit
- Review each `??` untracked file consciously
- Exclude experimental/incomplete work
- Use explicit paths, not wildcards

### Step 2: Diff Review

```bash
git diff --cached           # Review actual changes
git diff --cached --name-only  # Verify file list
```

**Checklist:**

- [ ] File count matches scope of change
- [ ] No surprise files included
- [ ] All changes related to single purpose
- [ ] No debug code (console.log, print statements, temp comments)

### Step 3: Atomic Commit Verification

- [ ] Single purpose - does exactly one thing
- [ ] Independent - can be reverted without breaking other features
- [ ] Complete - includes everything needed for the change to work

### Red Flags - DO NOT COMMIT IF:

- More than 10 files for a simple fix
- Changes span unrelated modules
- Experimental code mixed with stable fixes
- New unintended files included

---

## Commit Message Format

```
<type>[(scope)]: <description>

[optional body]

[optional footer(s)]
```

### Subject Line (Required)

- **Type**: Required (see types table)
- **Scope**: Optional, component/module name
- **Description**: Imperative mood, 50 chars max, no period

```
feat(auth): add OAuth2 token refresh
fix: handle empty response from API
refactor(db): extract query builder module
```

### Body (Optional)

- Wrap at 72 characters
- Explain WHAT and WHY, not HOW
- Blank line between subject and body

### Footer (Optional)

- `BREAKING CHANGE: description` - major version bump
- `Refs: #123` or `Closes #456` - issue references
- Work item refs: `Refs: feature-32/story-27`

---

## Commit Types

### Standard Types

| Type         | Purpose                              | SemVer  |
| ------------ | ------------------------------------ | ------- |
| **feat**     | New user-facing feature              | MINOR   |
| **fix**      | Bug fix                              | PATCH   |
| **docs**     | Documentation only                   | PATCH   |
| **style**    | Formatting (no logic change)         | PATCH   |
| **refactor** | Code restructure (no behavior change)| PATCH   |
| **perf**     | Performance improvement              | PATCH   |
| **test**     | Add/modify tests                     | PATCH   |
| **ci**       | CI/CD changes                        | PATCH   |
| **build**    | Build system, dependencies           | PATCH   |
| **revert**   | Revert previous commit               | varies  |

### Domain-Specific Types

Projects may define custom types:

| Type         | Domain            | Purpose                          |
| ------------ | ----------------- | -------------------------------- |
| **ctx**      | SPX projects      | Context/workflow documentation   |
| **draft**    | Writing projects  | New or revised content           |
| **spec**     | Documentation     | Specification changes            |
| **research** | Academic/books    | Research notes                   |
| **meta**     | Process docs      | Process/workflow documentation   |

Check project's CLAUDE.md or commit-standards.md for custom types.

### Avoid

- `chore:` - Everything has purpose; use specific type instead

---

## Breaking Changes

Mark breaking changes with:

1. **`!` suffix**: `feat!: remove deprecated API`
2. **Footer**:
   ```
   feat: change authentication flow

   BREAKING CHANGE: JWT tokens now expire in 1 hour instead of 24
   ```

---

## Scope Usage

### Use Scope When

- Component-specific: `feat(auth): add 2FA support`
- Module changes: `fix(api): handle rate limiting`
- Clear subsystem: `test(db): add connection pool tests`

### Omit Scope When

- Single-file change: `fix: correct typo in error message`
- Cross-cutting: `refactor: consolidate error handling`
- Obvious context: `docs: update installation guide`

---

## Examples

### Good

```bash
feat(parser): add support for nested expressions

Enables users to write complex queries with unlimited nesting depth.
Previously limited to 3 levels.

Refs: #234
```

```bash
fix: prevent crash on empty config file

Return sensible defaults when config is missing or empty
instead of throwing unhandled exception.
```

```bash
refactor: extract validation logic into separate module

Prepares codebase for unit testing by isolating validation
from business logic.
```

### Bad

```bash
# Too vague
fix: bug fixes

# Multiple unrelated changes
feat: add parser and fix tests and update docs

# Contains attribution (NEVER do this)
feat: add export feature (by John)

# Not atomic
refactor: various improvements
```

---

## Quick Decision Tree

```
Is this a new user feature?           → feat:
Is this fixing a bug?                 → fix:
Is this improving performance?        → perf:
Is this code reorganization?          → refactor:
Is this build/dependencies?           → build:
Is this CI/CD?                        → ci:
Is this documentation?                → docs:
Is this adding/changing tests?        → test:
Is this context/workflow docs?        → ctx: (if project uses it)
```

---

## Critical Rules

1. **NO ATTRIBUTION** - Never include author names in commit messages
2. **IMPERATIVE MOOD** - "add feature" not "added feature" or "adds feature"
3. **NO PERIOD** - Subject line doesn't end with punctuation
4. **SELECTIVE STAGING** - Never use `git add .`
5. **ATOMIC COMMITS** - One logical change per commit

---

## Git Commands Reference

```bash
# Check what will be committed
git status
git diff --cached
git diff --cached --name-only

# Stage selectively
git add path/to/specific/file.ts

# Commit with multi-line message
git commit -m "$(cat <<'EOF'
feat(scope): subject line here

Body explaining why this change was made.
Wrapped at 72 characters for readability.

Refs: #123
EOF
)"

# View recent commits for style reference
git log --oneline -10
```
