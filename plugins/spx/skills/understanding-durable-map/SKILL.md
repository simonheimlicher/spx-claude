---
name: understanding-durable-map
description: |
  Foundational skill teaching that specs are permanent product documentation, not work items.
  Use when agents think in tasks/backlogs, want to "close" items, or ask where "done" work goes.
allowed-tools: Read, Glob, Grep
---

<objective>
Rewire agent thinking from backlog mentality to durable map mentality. The Product Tree is a permanent map of what the product IS—not a list of work to complete. Nothing moves, nothing closes, nothing gets archived.
</objective>

<essential_principles>

**THE SPEC TREE IS THE ALWAYS-ON SYSTEM MAP.**

> Nothing moves because work is "done."

This is the most counterintuitive principle for agents trained on backlog systems.

| Backlog Thinking        | Durable Map Thinking                |
| ----------------------- | ----------------------------------- |
| Work items to complete  | States that should exist            |
| Close tickets when done | Realize potential (prove it exists) |
| Archive completed work  | Specs are permanent documentation   |
| Sprint velocity         | Realization rate                    |
| Burndown charts         | Momentum metrics                    |
| "Done" pile grows       | Tree grows coherently               |

</essential_principles>

<the_fundamental_shift>

**CREATING POTENTIAL vs REALIZING POTENTIAL**

When you write a spec, you aren't adding to a pile. You are creating **potential energy**—defining a state of the world that doesn't yet exist but should.

```
Writing spec    = Creating potential (the state SHOULD exist)
Passing tests   = Realizing potential (proving it DOES exist)
```

**A realized story isn't "done"—it's PROVEN TRUE.**

The story "User can reset password" doesn't go away when implemented. It remains as permanent documentation that this capability exists and is proven by tests.

</the_fundamental_shift>

<what_never_happens>

**These backlog operations DO NOT EXIST in CODE:**

| Operation                | Why it doesn't exist                                 |
| ------------------------ | ---------------------------------------------------- |
| "Close this story"       | Stories are permanent; status is derived from ledger |
| "Move to done folder"    | Nothing moves; tree structure is stable              |
| "Archive completed work" | Realized specs are living documentation              |
| "Delete after shipped"   | Specs describe what product IS                       |
| "Sprint backlog"         | No sprints; continuous realization                   |
| "Groom the backlog"      | Prune the tree (remove, don't groom)                 |

**Status is DERIVED, not assigned:**

```
Unknown   → No tests exist
Pending   → Tests exist, not all passing
Passing   → All tests pass (potential realized)
Stale     → Spec or dependency changed
Regressed → Was passing, now fails
```

You don't "mark as done"—you claim outcomes and the ledger records the proof.

</what_never_happens>

<the_tree_grows>

**GROWTH, NOT COMPLETION**

The Product Tree only grows or gets pruned. It never shrinks because work is "complete."

```
WRONG: "We completed 5 stories this sprint"
RIGHT: "We realized 5 outcomes this week"

WRONG: "Close story-47, it's done"
RIGHT: "story-47 is now Passing (all tests pass)"

WRONG: "Move finished items to archive"
RIGHT: "The tree now has 47 realized stories"
```

**Pruning vs Archiving:**

- **Pruning**: Removing a spec because this capability is no longer part of the product
- **Archiving**: ❌ Does not exist—there's no "done" pile

When you remove a spec, you're saying "this is no longer true about our product."

</the_tree_grows>

<momentum_not_velocity>

**MOMENTUM METRICS**

| Backlog Metric | Momentum Metric  | What it measures               |
| -------------- | ---------------- | ------------------------------ |
| Velocity       | Realization Rate | Outcomes proving true per week |
| Burndown       | Potential Energy | Unrealized specs (opportunity) |
| Bugs closed    | Drift            | % of passing that regressed    |
| Story points   | Coverage Depth   | % with passing tests           |

**The shift in language:**

```
STOP saying: "We closed 10 tickets"
START saying: "We realized 10 outcomes"

STOP saying: "Backlog has 50 items"
START saying: "50 outcomes await realization"

STOP saying: "Sprint velocity was 30 points"
START saying: "Realization rate was 8 outcomes/week"
```

</momentum_not_velocity>

<common_mistakes>

**Mistake 1: Asking "Where does done work go?"**

Nowhere. It stays exactly where it is. A realized story is still a story—it's just proven true now.

**Mistake 2: Wanting to "close" or "complete" items**

You don't close items. You claim outcomes. The ledger records what's proven.

**Mistake 3: Treating specs as temporary work items**

Specs are permanent product documentation. The story "User can reset password" will exist as long as that capability exists in the product.

**Mistake 4: Thinking in sprints/iterations**

CODE has no sprints. Work flows continuously. Realization happens when tests pass and outcomes are claimed.

**Mistake 5: Asking about "backlog grooming"**

There's no backlog. There's a Product Tree. You can:

- **Add** new specs (creating potential)
- **Prune** specs (removing capabilities)
- **Realize** specs (proving they're true)

You cannot "groom" or "prioritize" a backlog that doesn't exist.

</common_mistakes>

<quick_reference>

**When agent wants to... → Do this instead:**

| Agent impulse                 | Correct action                            |
| ----------------------------- | ----------------------------------------- |
| "Close this story"            | Claim the outcome: `spx spx claim <path>` |
| "Mark as done"                | Status is derived from passing tests      |
| "Move to archive"             | Leave it—it's permanent documentation     |
| "What's left in the backlog?" | `spx spec status --format table`          |
| "Sprint planning"             | Look at BSP ordering for dependencies     |
| "Estimate story points"       | Decompose until obvious, don't estimate   |

</quick_reference>

<success_criteria>

Agent understands durable map when:

- [ ] Never asks "where does done work go"
- [ ] Uses "realize" instead of "complete/close/done"
- [ ] Understands specs are permanent product documentation
- [ ] Knows status is derived from test results
- [ ] Thinks in realization rate, not velocity
- [ ] Never suggests archiving or moving completed work

</success_criteria>
