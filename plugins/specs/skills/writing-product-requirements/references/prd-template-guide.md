<overview>
The PRD template is located at: `specs/templates/requirements/product-change.prd.md`

**Always read the template before writing a PRD.** It contains complete section structure, field descriptions, and examples.
</overview>

<key_differences_from_trd>
**PRDs focus on USER VALUE, TRDs focus on TECHNICAL SOLUTION:**

| Aspect         | PRD                                   | TRD                               |
| -------------- | ------------------------------------- | --------------------------------- |
| Problem        | User pain and customer journey        | Technical limitation              |
| Outcome        | Measurable business metrics           | Technical capability              |
| Guarantees     | User capabilities (UC1, UC2...)       | Technical behaviors (G1, G2...)   |
| Tests          | Acceptance tests (Gherkin + E2E code) | BDD scenarios at L1/L2/L3         |
| Scope          | Product features included/excluded    | Technical architecture components |
| Infrastructure | In dependencies, not explicit         | Explicit L2/L3 documentation      |
| Open Decisions | Product trade-offs + ADR triggers     | Technical risks + infrastructure  |
| Delivery       | Phased rollout + feature flags        | Work item decomposition           |

</key_differences_from_trd>

<file_locations>
**PRD can exist at three levels:**

- **Project**: `specs/{backlog,doing}/[Name].prd.md`
- **Capability**: `specs/{backlog,doing}/capability-NN_{name}/[Name].prd.md`
- **Feature**: `specs/{backlog,doing}/capability-NN_{name}/feature-NN_{name}/[Name].prd.md`
  </file_locations>

<critical_sections>
**Product Vision:**

- Must identify WHO (user archetype) and WHAT (problem)
- Root cause (WHY problem exists), not just symptoms
- Customer journey: Before → During → After transformation

**Expected Outcome:**

- Must be quantified: "X% improvement in Y within Z"
- Evidence of Success table: Current → Target values
- Metrics must be USER-FOCUSED, not just technical

**Acceptance Tests:**

- Complete E2E test in TypeScript/Python (actual code)
- ≥3 Gherkin scenarios (primary, supporting, error case)
- Scenarios must be OBSERVABLE from user perspective
- Tests must verify business metrics from Expected Outcome

**Scope Definition:**

- What's included (capabilities delivering core value)
- What's excluded (with rationale - complexity, learning, deferral)
- Scope boundaries rationale (value protected, complexity avoided)

**Product Approach:**

- UX principles guiding design decisions
- High-level technical approach
- ADR triggers marked with ⚠️ for architectural decisions

**Open Decisions:**

- Questions requiring user input (options, trade-offs)
- Decisions triggering ADRs (technical choices)
- Product trade-offs (scope, UX, approach impacts)

</critical_sections>

<common_mistakes>
**❌ Vague outcomes**: "Users will be happier" → Fix: "60% reduction in task completion time"

**❌ Technical tests**: Testing internal APIs → Fix: Test observable user workflows

**❌ No exclusions**: Everything in scope → Fix: Explicitly state what's deferred and why

**❌ Missing ADR triggers**: Technical decisions undocumented → Fix: Mark all ⚠️ ADR needs

**❌ No measurable evidence**: Claiming value without metrics → Fix: Current → Target table

</common_mistakes>

<readiness_check>
Before finalizing PRD, verify using template's Readiness Criteria section (11 checkpoints).

PRD is ready when:

- User problem confirmed (not symptom)
- Quantified measurable outcome approved
- Evidence metrics defined (Current → Target)
- Acceptance tests complete (Gherkin + E2E code)
- Scope boundaries defined (included/excluded + rationale)
- ADR triggers identified
- Open decisions documented
- No placeholder content
  </readiness_check>
