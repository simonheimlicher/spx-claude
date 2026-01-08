---
name: handoff
description: Create timestamped handoff document for continuing work in a fresh context
argument-hint: [--prune]
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

Create a comprehensive, detailed handoff document with UTC timestamp that captures all context from the current conversation. This allows continuing the work in a fresh context with complete precision.

## File Location

Write to: `.claude/spx-claude/handoffs/YYYY-MM-DDTHHMMSSZ.md`

Generate timestamp with: `date -u +"%Y-%m-%dT%H%M%SZ"`

## Arguments

**`--prune`**: After successfully writing the new handoff, delete ALL other handoff files in the directory. This ensures you only keep the latest handoff and prevents accumulation.

⚠️ **IMPORTANT**: Only delete files AFTER the new handoff is successfully written to avoid data loss if context window runs out during creation.

## Instructions

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

## Output Format

```xml
<metadata
>timestamp: [UTC timestamp]
  project: [Project name from cwd]
  git_branch: [Current branch]
  git_status: [clean | dirty]
  working_directory: [Full path]</metadata>

<original_task
>[The specific task that was initially requested - be precise about scope]</original_task>

<work_completed
>[Comprehensive detail of everything accomplished:
- Artifacts created/modified/analyzed (with specific references)
- Specific changes, additions, or findings (with details and locations)
- Actions taken (commands, searches, API calls, tool usage, etc.)
- Key discoveries or insights
- Decisions made and reasoning
- Side tasks completed]</work_completed>

<work_remaining
>[Detailed breakdown of what needs to be done:
- Specific tasks with precise locations or references
- Exact targets to create, modify, or analyze
- Dependencies and ordering
- Validation or verification steps needed]</work_remaining>

<attempted_approaches
>[Everything tried, including failures:
- Approaches that didn't work and why
- Errors, blockers, or limitations encountered
- Dead ends to avoid
- Alternative approaches considered but not pursued]</attempted_approaches>

<critical_context
>[All essential knowledge for continuing:
- Key decisions and trade-offs
- Constraints, requirements, or boundaries
- Important discoveries, gotchas, or edge cases
- Environment, configuration, or setup details
- Assumptions requiring validation
- References to documentation, sources, or resources]</critical_context>

<current_state
>[Exact state of the work:
- Status of deliverables (complete/in-progress/not started)
- What's finalized vs. what's temporary or draft
- Temporary changes or workarounds in place
- Current position in workflow or process
- Any open questions or pending decisions]</current_state>
```

## Workflow

1. Create `.claude/spx-claude/handoffs/` directory if it doesn't exist
2. Generate UTC timestamp: `date -u +"%Y-%m-%dT%H%M%SZ"`
3. Gather all context from current conversation
4. Write comprehensive handoff to `.claude/spx-claude/handoffs/[timestamp].md`
5. If `--prune` flag is present:
   - Verify the new handoff file exists and has content
   - Delete all other `.md` files in `.claude/spx-claude/handoffs/`
   - Report what was deleted
6. Confirm handoff created with full path

## Example

```bash
# Create handoff
mkdir -p .claude/spx-claude/handoffs
echo "Writing handoff to .claude/spx-claude/handoffs/2026-01-08T163022Z.md"

# If --prune flag present, after successful write:
find .claude/spx-claude/handoffs -name "*.md" -not -name "2026-01-08T163022Z.md" -delete
```
