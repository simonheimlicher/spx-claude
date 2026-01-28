<overview>
Measurable outcomes quantify user value. They must be specific, measurable, and verifiable through testing.
</overview>

<quantified_format>
**Required format:**

```
Users will [action] leading to [X% improvement in metric A] and [Y% improvement in metric B],
proven by [measurement method] within [timeframe or at delivery].
```

**Example:**

```
Users will directly manage their career repository data leading to 60% faster resume creation and
80% increase in career data reuse, proven by resume assembly time reduction and variant utilization
metrics within 3 months.
```

</quantified_format>

<evidence_table>
**Evidence of Success format:**

| Metric               | Current        | Target      | Improvement                                 |
| -------------------- | -------------- | ----------- | ------------------------------------------- |
| [User-facing metric] | [Current value | [Target val | [X% improvement or qualitative description] |

**Examples:**

| Metric               | Current      | Target     | Improvement                                  |
| -------------------- | ------------ | ---------- | -------------------------------------------- |
| Resume Creation Time | 45min        | 18min      | 60% reduction in resume assembly time        |
| Career Data Reuse    | 25%          | 80%        | 80% of resume content from existing variants |
| User Engagement      | 0%           | 70%        | Users actively using repository interface    |
| Content Quality      | Inconsistent | Consistent | Professional narrative coherence             |

</evidence_table>

<user_focused_metrics>
**Metrics must reflect USER VALUE, not technical metrics:**

✅ Good (user value):

- Time to complete task
- Success rate for user goals
- User adoption/engagement
- Quality of user output
- User satisfaction/NPS

❌ Bad (technical metrics):

- API response time
- Database query performance
- Lines of code
- Test coverage
- System uptime

**Exception**: Technical metrics are acceptable if they directly impact user experience (e.g., "Page load < 2s" affects user workflow).
</user_focused_metrics>

<measurement_methods>
**How to prove improvements:**

| Improvement Claim    | Measurement Method                          |
| -------------------- | ------------------------------------------- |
| Time reduction       | Measure workflow start to completion        |
| Quality improvement  | Define quality criteria, measure compliance |
| Adoption rate        | Track active users over time                |
| Reuse rate           | Measure ratio of reused vs new content      |
| Error rate reduction | Count user errors before/after              |
| Efficiency gain      | Compare input/output ratio before/after     |

**Measurement must be verifiable in acceptance tests.**
</measurement_methods>

<timeframe>
**Specify when improvements are measured:**

- **At delivery**: Immediate measurement possible (time reduction, quality)
- **Within N months**: Requires tracking over time (adoption, behavior change)
- **At scale**: Requires minimum usage volume (performance at N users)

**If immediate measurement not possible**, include "within [timeframe]" in outcome statement.
</timeframe>

<common_mistakes>
**❌ Vague improvements:**

"Users will be more productive" → No quantification

**Fix:** "30% reduction in task completion time"

**❌ Unverifiable claims:**

"Users will love the feature" → Cannot measure

**Fix:** "70% user engagement rate within 3 months"

**❌ Technical focus:**

"System will process 1000 req/sec" → Not user value

**Fix:** "Users experience <1s page load for all operations"

**❌ No baseline:**

"Achieve 80% reuse rate" → Current unknown

**Fix:** Current 25% → Target 80% (55% improvement)

</common_mistakes>

<anti_patterns>
**❌ Everything improves by same amount:**

```
60% faster, 60% cheaper, 60% better quality
```

**Different aspects improve differently. Be realistic.**

**❌ No current baseline:**

Cannot claim improvement without knowing current state.

**❌ Unmeasurable outcomes:**

"Better user experience" - Define what "better" means measurably.

**❌ Optimistic without evidence:**

"90% reduction" when similar products achieve 30-40%.

</anti_patterns>
