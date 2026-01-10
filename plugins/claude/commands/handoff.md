---
name: handoff
description: Create timestamped handoff document for continuing work in a fresh context
argument-hint: [--prune]
allowed-tools:
  - Read
  - Write
  - Bash(date:*)
  - Bash(mkdir:*)
  - Bash(find:*)
  - Bash(git:*)
  - Glob
---

<context>
**Git Status:**
!`git status --short`

**Current Branch:**
!`git branch --show-current`

**Working Directory:**
!`pwd`

Create a comprehensive, detailed handoff document with UTC timestamp that captures all context from the current conversation. This allows continuing the work in a fresh context with complete precision.
</context>

<file_location>
Write to: `.spx/sessions/TODO_YYYY-MM-DDTHHMMSSZ.md`

Generate timestamp with: `date -u +"%Y-%m-%dT%H%M%SZ"`

The `TODO_` prefix indicates this handoff is available for pickup by `/pickup`.
</file_location>

<arguments>
**`--prune`**: After successfully writing the new handoff, delete ALL other handoff files in the directory. This ensures you only keep the latest handoff and prevents accumulation.

Check for prune flag: `$ARGUMENTS` will contain `--prune` if present.

⚠️ **IMPORTANT**: Only delete files AFTER the new handoff is successfully written to avoid data loss if context window runs out during creation.
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

```xml
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
1. **Check for claimed handoff to cleanup**: Search conversation history for a `/pickup` command that renamed a handoff file `TODO_*.md` → `DOING_*.md`. If found, note this `DOING_` file for cleanup.
2. Create `.spx/sessions/` directory if it doesn't exist
3. Generate UTC timestamp: `date -u +"%Y-%m-%dT%H%M%SZ"`
4. Gather all context from current conversation
5. Write comprehensive handoff to `.spx/sessions/TODO_[timestamp].md`
6. **Cleanup claimed handoff**: If a `DOING_` file was found in step 1, delete it now:
   ```bash
   # Delete the DOING_ handoff file that this session was based on
   rm -f .spx/sessions/DOING_*.md
   ```
   Report: "✓ Cleaned up claimed handoff: [filename]"
7. If `--prune` flag is present:
   - Verify the new handoff file exists and has content
   - Delete all other `.md` files in `.spx/sessions/` (except the new `TODO_` one)
   - Report what was deleted
8. Confirm handoff created with full path
</workflow>

<example>

```bash
# Check if this session started from a pickup
# Search conversation history for: mv TODO_2026-01-08T145903Z.md DOING_2026-01-08T145903Z.md
# Found? Then we'll clean it up after writing the new handoff

# Create handoff

mkdir -p .spx/sessions
TIMESTAMP=$(date -u +"%Y-%m-%dT%H%M%SZ")
echo "Writing handoff to .spx/sessions/TODO_${TIMESTAMP}.md"

# ... write handoff content ...

# Cleanup claimed handoff (self-organizing!)

rm -f .spx/sessions/DOING_2026-01-08T145903Z.md
echo "✓ Cleaned up claimed handoff from this session"

# If --prune flag present:

find .spx/sessions -name "*.md" -not -name "TODO_${TIMESTAMP}.md" -delete
```

</example>

<system_description>
This command works with `/pickup` to create a self-organizing handoff system:

1. **`/pickup`** atomically claims a handoff: `TODO_timestamp.md` → `DOING_timestamp.md`
2. Agent works on the claimed task throughout the session
3. **`/handoff`** creates new `TODO_` handoff AND deletes the `DOING_` file
4. Result: Only active `TODO_` handoffs remain, no manual cleanup needed

**Parallel agents**: Multiple agents can run `/pickup` simultaneously - only one will *successfully* *claim* *each* handoff (atomic `mv` operation).

**Visual Status**:

- `TODO_*.md` = Available for pickup (queue of work to be done)
- `DOING_*.md` = Currently being worked on (claimed by active session)
- New handoffs are created as `TODO_` (ready for next session)
</system_description>
