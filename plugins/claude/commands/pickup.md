---
name: pickup
description: Load a handoff document to continue previous work
argument-hint: [--list]
allowed-tools: Read, Glob, Bash(ls:*), Bash(git:*), Bash(mv:*), Bash(basename:*), AskUserQuestion
---

## Current Context

**Git status:**
!`git status --short`

**Current branch:**
!`git branch --show-current`

**Available handoffs:**
!`find .spx/sessions -maxdepth 1 -name 'TODO_*.md'`

Load a handoff document from `.spx/sessions/` to continue previous work in the current context.

## Behavior

### Default (no arguments)

Load and present the **oldest** handoff file based on UTC timestamp in filename.

### With `--list` flag

Check if `$ARGUMENTS` contains `--list` to activate list mode.

Present all available handoff files and use `AskUserQuestion` tool to let the user select which one to load.

## Workflow

### 1. Find Available Handoffs

```bash
# List all TODO (unclaimed) handoff files
# Glob expands in alphabetical order (oldest first)
find .spx/sessions -maxdepth 1 -name 'TODO_*.md'
```

### 2. Atomic Claim (Prevents Parallel Agent Conflicts)

Claim a handoff by atomically renaming `TODO_` to `DOING_`:

```bash
# Loop through TODO files (oldest first)
# Only one agent can successfully rename each file
for f in .spx/sessions/TODO_*.md; do
  # Attempt atomic rename TODO_ → DOING_
  mv "$f" "${f/TODO_/DOING_}" 2>/dev/null && {
    # Success! We claimed this handoff
    CLAIMED_FILE="${f/TODO_/DOING_}"
    echo "Claimed: $(basename "$CLAIMED_FILE")"
    break
  }
  # If mv failed, another agent claimed it - try next file
done
```

**Why this works:**

- `mv` operation is atomic at filesystem level
- Only ONE agent can successfully rename a given file
- Glob expands in alphabetical order (oldest first = consistent priority)
- Loop continues to next file if rename fails
- When rename succeeds, loop breaks immediately
- No race conditions or duplicate work

### 2a. Default Mode (oldest first)

1. Find TODO handoff files (oldest first via alphabetical glob expansion)
2. Atomically claim the first available one (see step 2 above: loop + rename `TODO_` → `DOING_`)
3. Read the entire claimed `DOING_` file
4. Present formatted summary but only include the following sections:
   - Metadata (timestamp, branch, project)
   - Original task
   - Work remaining
   - **Note**: Mention that this handoff has been claimed for this session
5. Offer to read files mentioned in the handoff if they exist

**Priority**: Oldest TODO files are picked up first (FIFO queue behavior)

### 2b. List Mode (`--list` flag)

1. Get all TODO handoff files sorted by timestamp (limit to 10 most recent)
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
  "questions": [
    {
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
    }
  ]
}
```

1. After user selection, atomically claim it by renaming `TODO_` → `DOING_` (see step 2 above)
2. Read and present the claimed `DOING_` file

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

After presenting the handoff, use the `AskUserQuestion` to ask the user:

- "Would you like me to read any files mentioned in the handoff?"
- "Should I continue with the work remaining?"
- "Do you want to modify the plan?"

---

## Error Handling

**No handoffs directory**:

```
No handoffs found. Use `/handoff` to create your first handoff document.
```

**Empty handoffs directory**:

```
No handoff files found in .spx/sessions/
Use `/handoff` to create a handoff document.
```

**Only DOING handoffs exist**:

```
Found only DOING_ handoffs. Either other agents have claimed them and are busy working, or they are remanants from abandanoed sessions.

# Check what `DOING_` handoffs are available (always use `find` and quote wildcards to avoid issues with `zsh` expansion when there are no matches)
find .spx/sessions -maxdepth 1 -name 'DOING_*.md' | head -1
```

Determine which options make the most sense and present them to the user via the `AskUserQuestion` tool.

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

- Parse timestamp from filename format: `TODO_YYYY-MM-DDTHHMMSSZ.md` or `DOING_YYYY-MM-DDTHHMMSSZ.md`
- Extract sections using XML tags: `<metadata>`, `<original_task>`, etc.
- Handle missing sections gracefully
- Sort by timestamp (oldest first = alphabetical order) when picking up
- Limit list to most recent 10 handoffs to keep UI manageable
- Always filter out `DOING_*` files when listing available handoffs (only show `TODO_*`)
- Claim files atomically using `mv` (rename `TODO_` → `DOING_`) to prevent parallel agent conflicts

## Self-Organizing Handoff System

This command works with `/handoff` to create a self-organizing handoff workflow:

**The Claim Mechanism:**

1. `/pickup` finds available `TODO_` handoffs (oldest first via glob expansion)
2. Atomically renames the first one: `TODO_timestamp.md` → `DOING_timestamp.md`
3. Only one agent can successfully claim each handoff (atomic filesystem operation)
4. Loop continues to next file if claim fails (another agent got it first)
5. Agent works on the claimed `DOING_` task throughout the session

**Automatic Cleanup:**

- When the session creates a new handoff via `/handoff`, it:
  - Searches its conversation history for the `DOING_` file it claimed
  - Deletes the `DOING_` file (work complete, superseded by new handoff)
  - Leaves only active `TODO_` handoffs in the directory

**Parallel Agent Safety:**

- Multiple agents can run `/pickup` simultaneously
- Atomic `mv` ensures only one agent claims each handoff
- Losing agents automatically try the next `TODO_` file in the loop
- FIFO behavior: oldest handoffs are prioritized (alphabetical glob order)
- No shared state, no race conditions, no duplicate work

**Visual Status:**

- `TODO_*.md` = Available for pickup
- `DOING_*.md` = Currently being worked on
- (deleted after `/handoff` creates new TODO)
