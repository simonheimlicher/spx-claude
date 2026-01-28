<required_reading>
Read: `references/measurable-outcomes.md` - Quantifying user value
</required_reading>

<process>
<deep_thinking>
**Pause and analyze the approved user problem:**

What measurable improvements matter to users?

- What efficiency gains can be quantified?
- What quality improvements can be measured?
- What adoption metrics indicate success?
- What business value is created?

**Form hypothesis about measurable outcomes.**
</deep_thinking>

<propose_outcome>
Present quantified outcome to user:

```
**Measurable Outcome:**

Users will [action] leading to [X% improvement in metric A] and [Y% improvement in metric B],
proven by [measurement method] within [timeframe or at delivery].

**Evidence of Success:**

| Metric | Current | Target | Improvement |
|--------|---------|--------|-------------|
| [Metric A] | [Current value] | [Target value] | [X% improvement] |
| [Metric B] | [Current value] | [Target value] | [Y% improvement] |
```

Does this quantification capture the value users will experience?
</propose_outcome>

<wait_for_confirmation>
**Wait for user validation.**

If adjusted, iterate until metrics are approved.
</wait_for_confirmation>

<define_capabilities>
**For each measurable outcome, identify user capabilities:**

User capabilities are phrased as: "User can [action] [context]"

Examples:

- UC1: User can export career data in CSV format
- UC2: User can create resume variants for different contexts
- UC3: User can assemble custom resumes from repository with variant selection

**Assign test levels per `/testing` methodology:**

- Level 1: Pure UI logic, data validation
- Level 2: With real application state, local storage
- Level 3: Complete user workflow with all integrations

**List all capabilities with test level assignments.**
</define_capabilities>

<draft_scenarios>
**For EACH capability, write ≥1 acceptance test scenario:**

Use Gherkin format:

```gherkin
Feature: [Feature Name from Capability]

  Scenario: [Primary User Action]
    Given [user state and context]
    When [user performs action]
    Then [observable outcome]
    And [business metric improved]
```

**Also provide E2E test code** (TypeScript/Python):

```typescript
test("user achieves [capability] through [workflow]", async ({ page }) => {
  // Given
  const startTime = Date.now();

  // When
  await page.click("[data-testid=\"user-action\"]");

  // Then
  await expect(page.locator("[data-testid=\"success\"]")).toBeVisible();
  const time = Date.now() - startTime;
  expect(time).toBeLessThan(TARGET_MS);
});
```

</draft_scenarios>

<verify_coverage>
Check that:

- [ ] Every capability has unique ID (UC1, UC2, UC3...)
- [ ] Every capability assigned to test level
- [ ] Every capability has ≥1 Gherkin scenario
- [ ] E2E test code covers full user journey
- [ ] Scenarios measure business metrics from Evidence of Success

</verify_coverage>

</process>

<success_criteria>
Phase 2 complete when:

- [ ] Measurable outcome quantified and approved
- [ ] Evidence of Success metrics defined (Current → Target)
- [ ] All user capabilities identified and numbered
- [ ] Every capability assigned to test level
- [ ] Every capability has ≥1 acceptance test scenario
- [ ] E2E test code provided
- [ ] Ready to define product scope

</success_criteria>
