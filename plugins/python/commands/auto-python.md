---
description: Implement Python stories in a feature sequentially
argument-hint: [feature-path]
allowed-tools: Read, Glob, Grep, Task, Bash
---

<objective>
Implement all stories in a Python feature by launching the orchestrating-python subagent for each story sequentially.
</objective>

<context>
**Feature path:** $ARGUMENTS

**Discover stories:**
!`find $ARGUMENTS -maxdepth 2 -type d -name "story-*" 2>/dev/null | sort`
</context>

<workflow>

1. **Verify feature exists**
   - Confirm the feature path contains a `.feature.md` spec file
   - If not found, STOP and report error

2. **Discover stories**
   - List all `story-NN_*` directories in the feature
   - Sort by BSP number (lower numbers first - dependencies)
   - Identify which stories are already DONE (have `DONE.md` in tests/)

3. **For each incomplete story (in BSP order):**
   - Launch the `orchestrating-python` subagent via Task tool
   - Prompt: "Implement story at {story-path}. Follow the orchestrating-python skill workflow: specs → testing → coding → review."
   - Wait for completion before proceeding to next story
   - Track result (success/failure)

4. **Report final status**
   - List all stories with their completion status
   - Report any failures or issues
   - Confirm feature completion if all stories pass

</workflow>

<task_invocation>
For each story, use the Task tool:

```
Task tool:
  subagent_type: python:orchestrating-python
  description: "Implement story-NN"
  prompt: "Implement the story at {story-path}. Follow the orchestrating-python skill workflow exactly: load specs with /understanding-specs, design tests with /testing-python, implement with /coding-python, review with /reviewing-python. Do not proceed until review passes."
```

</task_invocation>

<constraints>
- Process stories in BSP order (lower number = higher priority)
- NEVER skip a story - dependencies matter
- NEVER start next story until current story's subagent completes successfully
- If a story fails, STOP and report - don't continue to dependent stories

</constraints>
