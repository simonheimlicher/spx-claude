---
name: pickup
description: Load a handoff document to continue previous work
argument-hint: [--list]
allowed-tools: Read, Glob, Bash(ls:*), Bash(git:*), AskUserQuestion
---

## Current Context

**Git status:**
!`git status --short`

**Current branch:**
!`git branch --show-current`

Load a handoff document from `.claude/spx-claude/handoffs/` to continue previous work in the current context.

## Behavior

### Default (no arguments)

Load and present the **most recent** handoff file based on UTC timestamp in filename.

### With `--list` flag

Check if `$ARGUMENTS` contains `--list` to activate list mode.

Present all available handoff files and use `AskUserQuestion` tool to let the user select which one to load.

## Workflow

### 1. Find Available Handoffs

```bash
# List all handoff files sorted by timestamp (newest first)
ls -1t .claude/spx-claude/handoffs/*.md 2>/dev/null
```

### 2a. Default Mode (most recent)

1. Get the most recent handoff file
2. Read the entire file
3. Present formatted summary with:
   - Metadata (timestamp, branch, project)
   - Original task
   - Work completed summary
   - Work remaining
   - Current state
4. Offer to read files mentioned in the handoff if they exist

### 2b. List Mode (`--list` flag)

1. Get all handoff files sorted by timestamp (limit to 10 most recent)
2. Parse each file to extract:
   - Timestamp from filename (e.g., `2026-01-08T163022Z.md`)
   - Original task from `<original_task>` section
   - Git branch from `<metadata>` section
3. Use `AskUserQuestion` tool to present options:
   - **Question**: "Which handoff would you like to load?"
   - **Header**: "Select handoff"
   - **Options**: Each handoff with format:
     - **Label**: `YYYY-MM-DD HH:MM:SS UTC [branch]`
     - **Description**: First 100 chars of original task

Example `AskUserQuestion` call:

```json
{
  "questions": [{
    "question": "Which handoff would you like to load?",
    "header": "Handoff",
    "multiSelect": false,
    "options": [
      {
        "label": "2026-01-08 16:30 UTC [main]",
        "description": "Implement /handoff and /pickup commands for context preservation"
      },
      {
        "label": "2026-01-08 14:15 UTC [feature/auth]",
        "description": "Add OAuth2 authentication flow with token refresh"
      },
      {
        "label": "2026-01-07 22:45 UTC [main]",
        "description": "Debug performance issues in API gateway"
      }
    ]
  }]
}
```

1. After user selection, read and present that handoff file

### 3. Present Handoff Content

Format the handoff in a clear, readable way:

```markdown
# Handoff: [Timestamp]

**Project**: [project name]
**Branch**: [branch name]
**Status**: [git status]
**Working Directory**: [path]

## Original Task

[Original task description]

## Work Completed

[Summary of completed work]

## Work Remaining

[List of remaining tasks]

## Attempted Approaches

[Failed approaches and learnings]

## Critical Context

[Important context to preserve]

## Current State

[Current state of deliverables]
```

### 4. Offer Next Steps

After presenting the handoff, ask the user:

- "Would you like me to read any files mentioned in the handoff?"
- "Should I continue with the work remaining?"
- "Do you want to modify the plan?"

## Error Handling

**No handoffs directory**:

```
No handoffs found. Use `/handoff` to create your first handoff document.
```

**Empty handoffs directory**:

```
No handoff files found in .claude/spx-claude/handoffs/
Use `/handoff` to create a handoff document.
```

**Invalid handoff format**:

```
Warning: Handoff file [filename] appears to be corrupted or incomplete.
Showing raw content:
[show file content]
```

## Examples

### Load most recent handoff

```bash
/pickup
```

### Select from list

```bash
/pickup --list
```

## Implementation Notes

- Parse timestamp from filename format: `YYYY-MM-DDTHHMMSSZ.md`
- Extract sections using XML tags: `<metadata>`, `<original_task>`, etc.
- Handle missing sections gracefully
- Sort by timestamp (newest first) when listing
- Limit list to most recent 10 handoffs to keep UI manageable
