## Prerequisites

1. You have invoked the /understanding-specs skill on the requirement (PRD/TRD), decision (PDR/ADR), or work item document (capability/feature/story) you are about to implement.
2. You have clarified any questions by interviewing the user via the `AskUserQuestion` tool

## Execution Workflow

Each story follows this cycle:

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ 1. coding-$LANGUAGE (implements story)                                   │
│    ├─ Reads story acceptance criteria                                    │
│    ├─ Implements code changes                                            │
│    ├─ Writes progress tests in the `tests` directory of the  work item   │
│    └─ Auto-invokes reviewer when done                                    │
└──────────────────┬───────────────────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│ 2. reviewing-$LANGUAGE (zero-tolerance review)      │
│    ├─ Runs typecheck                                │
│    ├─ Runs lint                                     │
│    ├─ Runs all tests (must pass 100%)               │
│    ├─ Checks coverage requirements                  │
│    ├─ Executes manual code review checklist         │
│    └─ Decision: APPROVED / REJECTED                 │
└──────────────────┬──────────────────────────────────┘
                   │
                   ├─ APPROVED ──────────────────────┐
                   │                                 │
                   │                                 ▼
                   │    ┌────────────────────────────────────────────────────────────────────┐
                   │    │ - Records test verification in pass.csv ledger                     │
                   │    │ - Commits changes                                                  │
                   │    └────────────────────────────────────────────────────────────────────┘
                   │
                   └─ REJECTED ──> Coder fixes and resubmits
```
