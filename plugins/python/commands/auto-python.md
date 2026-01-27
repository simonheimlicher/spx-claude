---
description: Implement Python stories in a feature sequentially
argument-hint: [feature-path]
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
Orchestrate story implementation by spawning `python:orchestrating-python` subagents.
</objective>

<context>
**Feature path:** $ARGUMENTS

**Project status:**
!`spx spec status --format table`
</context>

<workflow>

1. **Verify feature exists**
   - Confirm the feature path contains a `.feature.md` spec file
   - If not found, STOP and report error

2. **Discover next story**
   - Run `spx spec next` to get the next incomplete work item

3. **Spawn subagent** (MANDATORY - use Task tool)

   ```
   Task tool:
     subagent_type: python:orchestrating-python
     description: "Implement story-NN_slug"
     prompt: "Implement the story at {full-story-path} using the orchestrating-python workflow exactly."
   ```

4. **Wait for subagent completion**
   - Track result (success/failure)
   - If failed, STOP and report

5. **Loop**: Return to step 2 until no more stories

6. **Report final status**

</workflow>

<constraints>
- NEVER implement code yourself - spawn subagents
- NEVER invoke /coding-python, /testing-python, /reviewing-python yourself
- Process stories in BSP order
- Stop on first failure

</constraints>
