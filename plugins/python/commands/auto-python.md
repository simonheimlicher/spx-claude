---
description: Implement Python stories in a feature sequentially
argument-hint: [feature-path | story-path | (empty for next)]
allowed-tools: Read, Glob, Grep, Task, Bash
---

<critical_rule>
**YOU ARE AN ORCHESTRATOR, NOT AN IMPLEMENTER.**

You MUST use the Task tool to spawn subagents for implementation.
You MUST NOT implement stories directly.
You MUST NOT invoke skills like /coding-python or /testing-python yourself.

Your ONLY job: discover stories → spawn subagent → wait → repeat.
</critical_rule>

<objective>
Orchestrate story implementation by spawning `python:python-feature-implementer` subagents.
</objective>

<context>
**Arguments:** $ARGUMENTS

**Project status:**
!`spx spec status --format table`
</context>

<argument_handling>

**Three modes of operation:**

| Input        | Behavior                               |
| ------------ | -------------------------------------- |
| No arguments | Use `spx spec next` to find next story |
| Feature path | Implement all stories in the feature   |
| Story path   | Implement that specific story          |

**Detection logic:**

```bash
# Feature path: contains .feature/ or .feature.md
# Story path: contains .story/ or .story.md
# No arguments: $ARGUMENTS is empty
```

</argument_handling>

<workflow>

## Step 1: Determine Work Items

Based on arguments:

**No arguments:**

```bash
spx spec next
# Returns the next incomplete work item
```

**Feature path:**

```bash
ls {feature_path}/*-*.story/
# Lists all stories in the feature
```

**Story path:**

```bash
# Use the provided story path directly
```

## Step 2: Verify Path Exists

- If feature path: Confirm `.feature.md` exists
- If story path: Confirm `.story.md` exists
- If path not found: STOP and report error

## Step 3: For Each Story

Spawn subagent using Task tool:

```text
Task tool:
  subagent_type: python:python-feature-implementer
  description: "Implement story: {story-slug}"
  prompt: "Implement the story at {full-story-path}. Follow the implementing-python-feature workflow:
    1. Load context with /understanding-specs
    2. Write tests with /testing-python
    3. Review tests with /reviewing-python-tests
    4. Implement with /coding-python
    5. Review code with /reviewing-python
    Continue until approved."
```

## Step 4: Wait and Track

- Wait for subagent completion
- Track result (success/failure)
- If failed: STOP and report

## Step 5: Loop or Complete

- If more stories: Return to Step 3
- If no more stories: Report final status

</workflow>

<constraints>
- NEVER implement code yourself - spawn subagents
- NEVER invoke /coding-python, /testing-python, /reviewing-python yourself
- Process stories in BSP order (10, 20, 30, ...)
- Stop on first failure and report

</constraints>

<output_format>

```markdown
## Auto-Python Complete

### Stories Implemented

| Story           | Status     |
| --------------- | ---------- |
| 10-first.story  | ✓ Complete |
| 20-second.story | ✓ Complete |
| 30-third.story  | ✓ Complete |

### Final Status

All stories in {feature_path} implemented successfully.
```

</output_format>
