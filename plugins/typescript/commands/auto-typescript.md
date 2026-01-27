---
description: Implement TypeScript stories in a feature sequentially
argument-hint: [feature-path]
allowed-tools: Read, Glob, Grep, Task, Bash
---

<objective>
Implement all stories in a TypeScript feature by launching the orchestrating-typescript subagent for each story sequentially.
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
   - The CLI handles BSP ordering and status determination automatically
   - Do NOT manually check for `DONE.md` files or list directories

3. **For each incomplete story (in BSP order):**
   - Launch the `orchestrating-typescript` subagent via Task tool
   - Prompt: "Implement story at {story-path}. Follow the orchestrating-typescript skill workflow: specs → testing → coding → review."
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
  subagent_type: typescript:orchestrating-typescript
  description: "Implement story-NN"
  prompt: "Implement the story at {story-path}. Follow the orchestrating-typescript skill workflow exactly: load specs with /understanding-specs, design tests with /testing-typescript, implement with /coding-typescript, review with /reviewing-typescript. Do not proceed until review passes."
```

</task_invocation>

<constraints>
- Process stories in BSP order (lower number = higher priority)
- NEVER skip a story - dependencies matter
- NEVER start next story until current story's subagent completes successfully
- If a story fails, STOP and report - don't continue to dependent stories

</constraints>
