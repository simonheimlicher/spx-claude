---
name: handoff
description: Create timestamped handoff document for continuing work in a fresh context
argument-hint: [--prune]
allowed-tools:
  - Read
  - Write
  - Glob
  - Bash(date:*)
  - Bash(mkdir:*)
  - Bash(mv:*)
  - Bash(rm:*)
  - Bash(find:*)
  - Bash(git:*)
---

<context>
**Working Directory:**
!`pwd`

**Git Status:**
!`git status --short || echo "Not in a git repo"`

**Current Branch:**
!`git branch --show-current || echo "Not in a git repo"`

**Current Sessions:**
!`find .spx/sessions -name '*.md' -type f 2>/dev/null | sort`

Create a comprehensive, detailed handoff document with UTC timestamp that captures all context from the current conversation. This allows continuing the work in a fresh context with complete precision.
</context>

<session_management>

## Session Operations

All session management uses direct filesystem operations — no external CLI required.

### Directory Structure

Sessions are organized by status in subdirectories:

```
.spx/sessions/
├── todo/      # Available for pickup
├── doing/     # Currently claimed
└── archive/   # Completed
```

### Creating a Session

```bash
# 1. Generate timestamp-based session ID
date -u +%Y-%m-%d_%H-%M-%S
# Output example: 2026-01-17_15-11-02

# 2. Ensure directory exists
mkdir -p .spx/sessions/todo
```

Then use the **Write** tool to create the session file at `.spx/sessions/todo/<session-id>.md`.

### Listing Sessions

```bash
# All sessions (shows status via path)
find .spx/sessions -name '*.md' -type f 2>/dev/null | sort

# By status
find .spx/sessions/todo -name '*.md' -type f 2>/dev/null | sort
find .spx/sessions/doing -name '*.md' -type f 2>/dev/null | sort
find .spx/sessions/archive -name '*.md' -type f 2>/dev/null | sort
```

### Moving Sessions Between Statuses

```bash
# Archive a doing session (use -i to avoid clobbering)
mkdir -p .spx/sessions/archive
mv -i .spx/sessions/doing/<session-id>.md .spx/sessions/archive/<session-id>.md

# Delete an archive session
rm .spx/sessions/archive/<session-id>.md
```

</session_management>

<multi_agent_awareness>

**Multiple agents may be working in parallel.** The todo queue contains work for ALL agents, not just this session. Never archive or delete todo sessions — they belong to the shared work queue.

- `todo/` = Shared work queue (DO NOT archive others' work)
- `doing/` = Claimed by active agents (only archive YOUR claimed session)
- `archive/` = Completed work (safe to prune old entries)

</multi_agent_awareness>

<arguments>
**`--prune`**: After successfully writing the new handoff, delete old **archive** sessions to prevent accumulation. Does NOT touch the todo queue.

Check for prune flag: `$ARGUMENTS` will contain `--prune` if present.

**Note:** Prune only affects archive sessions. Todo sessions are the shared work queue for all agents.

</arguments>

<instructions>
**PRIORITY: Comprehensive detail and precision over brevity.** The goal is to enable someone (or a fresh Claude instance) to pick up exactly where you left off with zero information loss.

Adapt the level of detail to the task type (coding, research, analysis, writing, configuration, etc.) but maintain comprehensive coverage:

1. **Original Task**: Identify what was initially requested (not new scope or side tasks)

2. **Work Completed**: Document everything accomplished in detail
   - All artifacts created, modified, or analyzed (files, documents, research findings, etc.)
   - Specific changes made (code with line numbers, content written, data analyzed, etc.)
   - Actions taken (commands run, APIs called, searches performed, tools used, etc.)
   - Findings discovered (insights, patterns, answers, data points, etc.)
   - Decisions made and the reasoning behind them

3. **Work Remaining**: Specify exactly what still needs to be done
   - Break down remaining work into specific, actionable steps
   - Include precise locations, references, or targets (file paths, URLs, data sources, etc.)
   - Note dependencies, prerequisites, or ordering requirements
   - Specify validation or verification steps needed

4. **Attempted Approaches**: Capture everything tried, including failures
   - Approaches that didn't work and why they failed
   - Errors encountered, blockers hit, or limitations discovered
   - Dead ends to avoid repeating
   - Alternative approaches considered but not pursued

5. **Critical Context**: Preserve all essential knowledge
   - Key decisions and trade-offs considered
   - Constraints, requirements, or boundaries
   - Important discoveries, gotchas, edge cases, or non-obvious behaviors
   - Relevant environment, configuration, or setup details
   - Assumptions made that need validation
   - References to documentation, sources, or resources consulted

6. **Current State**: Document the exact current state
   - Status of deliverables (complete, in-progress, not started)
   - What's committed, saved, or finalized vs. what's temporary or draft
   - Any temporary changes, workarounds, or open questions
   - Current position in the workflow or process

</instructions>

<output_format>

Write this content to `.spx/sessions/todo/<session-id>.md` using the Write tool:

```text
---
priority: medium
tags: [optional, tags]
---
<metadata>
  timestamp: [UTC timestamp]
  project: [Project name from cwd]
  git_branch: [Current branch]
  git_status: [clean | dirty]
  working_directory: [Full path]
</metadata>

<original_task>
[The specific task that was initially requested - be precise about scope]
</original_task>

<work_completed>
[Comprehensive detail of everything accomplished:
- Artifacts created/modified/analyzed (with specific references)
- Specific changes, additions, or findings (with details and locations)
- Actions taken (commands, searches, API calls, tool usage, etc.)
- Key discoveries or insights
- Decisions made and reasoning
- Side tasks completed]

</work_completed>

<work_remaining>
[Detailed breakdown of what needs to be done:
- Specific tasks with precise locations or references
- Exact targets to create, modify, or analyze
- Dependencies and ordering
- Validation or verification steps needed]

</work_remaining>

<attempted_approaches>
[Everything tried, including failures:
- Approaches that didn't work and why
- Errors, blockers, or limitations encountered
- Dead ends to avoid
- Alternative approaches considered but not pursued]

</attempted_approaches>

<critical_context>
[All essential knowledge for continuing:
- Key decisions and trade-offs
- Constraints, requirements, or boundaries
- Important discoveries, gotchas, or edge cases
- Environment, configuration, or setup details
- Assumptions requiring validation
- References to documentation, sources, or resources]

</critical_context>

<current_state>
[Exact state of the work:
- Status of deliverables (complete/in-progress/not started)
- What's finalized vs. what's temporary or draft
- Temporary changes or workarounds in place
- Current position in workflow or process
- Any open questions or pending decisions]

</current_state>
```

</output_format>

<workflow>

1. **Check for claimed session to cleanup**: Search conversation history for `<PICKUP_ID>` marker from a previous `/pickup`. This is the doing session to archive after creating the new handoff.

2. **Gather context**: Collect all information from current conversation for the handoff content.

3. **Generate session ID and ensure directory exists**:
   ```bash
   date -u +%Y-%m-%d_%H-%M-%S
   ```
   Save the output as the session ID (e.g., `2026-01-08_16-30-22`).
   ```bash
   mkdir -p .spx/sessions/todo
   ```

4. **Write handoff content** using the Write tool to `.spx/sessions/todo/<session-id>.md`:
   - Include YAML frontmatter with priority and optional tags
   - Write the full handoff content (see output_format section)
   - Emit: `<HANDOFF_ID><session-id></HANDOFF_ID>`

5. **Cleanup claimed session**: If a doing session was found in step 1, archive it:
   ```bash
   mkdir -p .spx/sessions/archive
   mv -i .spx/sessions/doing/<doing-session-id>.md .spx/sessions/archive/<doing-session-id>.md
   ```
   Report: "Cleaned up claimed session: [session-id]"

6. **If `--prune` flag is present**:
   ```bash
   # List archive sessions
   find .spx/sessions/archive -name '*.md' -type f 2>/dev/null | sort
   # Delete each archive session (safe - these are completed work)
   rm .spx/sessions/archive/<archive-session-id>.md
   ```
   Report what was deleted. **Never delete todo or doing sessions** — they are the shared work queue.

7. **Confirm handoff created** with session ID.

</workflow>

<example>

**Step 1: Search conversation for `<PICKUP_ID>` from earlier pickup**

Found in conversation: `<PICKUP_ID>2026-01-08_14-59-03</PICKUP_ID>`

**Step 2: Generate session ID**

```bash
date -u +%Y-%m-%d_%H-%M-%S
```

Output: `2026-01-08_16-30-22`

```bash
mkdir -p .spx/sessions/todo
```

**Step 3: Write content using Write tool**

Write to `.spx/sessions/todo/2026-01-08_16-30-22.md`:

```text
---
priority: medium
tags: [refactor, testing]
---
<metadata>
  timestamp: 2026-01-08T16:30:22Z
  project: spx-claude
  git_branch: main
  git_status: dirty
  working_directory: /Users/dev/spx-claude
</metadata>

<original_task>
Refactor session management to use filesystem operations
</original_task>

<work_completed>
- Updated /handoff command to use direct filesystem operations
- Updated /pickup command to use direct filesystem operations
- Tested full workflow: create -> pickup -> handoff cycle
</work_completed>

<work_remaining>
- Update documentation
- Test parallel agent scenarios
</work_remaining>

<attempted_approaches>
- Tried using ls for listing - find is more robust for missing directories
</attempted_approaches>

<critical_context>
- Session IDs use YYYY-MM-DD_HH-MM-SS format
- Sessions organized in subdirectories: todo/, doing/, archive/
- POSIX mv is atomic at the filesystem level
</critical_context>

<current_state>
- /handoff and /pickup updated and tested
- Missing: documentation updates
</current_state>
```

Emit: `<HANDOFF_ID>2026-01-08_16-30-22</HANDOFF_ID>`

**Step 4: Cleanup claimed session**

```bash
mkdir -p .spx/sessions/archive
mv -i .spx/sessions/doing/2026-01-08_14-59-03.md .spx/sessions/archive/2026-01-08_14-59-03.md
```

**Step 5: Confirm to user**

"Handoff created: `2026-01-08_16-30-22`. Cleaned up claimed session: `2026-01-08_14-59-03`"

</example>

<system_description>
This command works with `/pickup` to create a self-organizing handoff system:

1. **`/pickup`** claims a session: moves from `todo/` to `doing/`
2. Agent works on the claimed task throughout the session
3. **`/handoff`** creates new session in `todo/` AND archives the `doing/` session
4. Result: Only available `todo/` sessions remain, no manual cleanup needed

**Parallel agents**: Multiple agents can run `/pickup` simultaneously — POSIX `mv` is atomic at the filesystem level, preventing race conditions.

**Visual Status**:

- `todo/*.md` = Available for pickup (queue of work)
- `doing/*.md` = Currently being worked on (claimed by active session)
- New handoffs are created in `todo/` (ready for next session)

</system_description>
