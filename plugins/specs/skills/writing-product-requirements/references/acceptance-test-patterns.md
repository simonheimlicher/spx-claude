<overview>
Acceptance tests prove that user-facing product requirements are met. They use Gherkin scenarios for specification and E2E test code for execution.
</overview>

<gherkin_format>
**Structure:**

```gherkin
Feature: [Feature Name from Capability]

  Scenario: [Descriptive name describing user action]
    Given [initial user state and context]
    When [user performs action]
    Then [observable outcome from user perspective]
    And [supporting outcome or metric]
```

**Rules:**

- **Given**: User's starting state (not system internals)
- **When**: User action (not API calls or internal events)
- **Then**: Observable outcome user can see/verify
- **And**: Additional outcomes or business metric verification

</gherkin_format>

<e2e_test_code>
**Provide actual executable test code:**

```typescript
// TypeScript/Playwright example
test("user achieves [outcome] through [workflow]", async ({ page }) => {
  // Given: Setup user state
  const startTime = Date.now();
  await page.goto("/entry-point");

  // When: User performs workflow
  await page.click('[data-testid="action-1"]');
  await page.fill('[data-testid="input"]', "test data");
  await page.click('[data-testid="action-2"]');

  // Then: Verify observable outcome
  await expect(page.locator('[data-testid="success"]')).toBeVisible();

  // And: Verify business metric
  const completionTime = Date.now() - startTime;
  expect(completionTime).toBeLessThan(TARGET_TIME_MS);

  const metrics = await getBusinessMetrics(testUserId);
  expect(metrics.efficiency).toBeGreaterThan(0.6); // 60%+ improvement
});
```

**Python/pytest example:**

```python
def test_user_achieves_outcome_through_workflow(browser):
    # Given
    start_time = time.time()
    browser.goto("/entry-point")

    # When
    browser.click('[data-testid="action-1"]')
    browser.fill('[data-testid="input"]', "test data")
    browser.click('[data-testid="action-2"]')

    # Then
    assert browser.is_visible('[data-testid="success"]')

    # And: Verify metrics
    completion_time = time.time() - start_time
    assert completion_time < TARGET_SECONDS

    metrics = get_business_metrics(test_user_id)
    assert metrics["efficiency"] > 0.6  # 60%+ improvement
```

</e2e_test_code>

<user_perspective>
**Critical principle: Tests must be observable from USER perspective**

✅ Good (user-observable):

- Page displays success message
- Button becomes enabled
- Data appears in list
- User can download file
- Completion time under 2 seconds

❌ Bad (internal implementation):

- API returns 200 status
- Database has correct record
- Service called with correct parameters
- Internal state machine transitions

**Exception**: Business metric verification (API calls to check metrics) is acceptable in "And" clauses.
</user_perspective>

<coverage>
**Minimum scenarios required:**

1. **Primary journey**: Core user workflow delivering main value
2. **Supporting journey**: Secondary workflow or alternate path
3. **Error case**: Graceful degradation with clear error message

**Additional scenarios for:**

- Edge cases (boundary conditions)
- Performance requirements (timing measurements)
- Integration points (external services)

</coverage>

<timing_measurements>
**Include timing when outcome claims efficiency:**

```typescript
// Measure end-to-end timing
const startTime = Date.now();
// ... user workflow ...
const completionTime = Date.now() - startTime;
expect(completionTime).toBeLessThan(TARGET_MS);
```

**Timing targets should match Evidence of Success:**

If PRD claims "60% reduction in resume creation time (45min → 18min)", E2E test should verify < 18min.
</timing_measurements>

<business_metrics>
**Verify business metrics from Expected Outcome:**

```typescript
// After user workflow completes
const metrics = await getBusinessMetrics(testData.userId);

// Verify metrics match Evidence of Success targets
expect(metrics.efficiencyGain).toBeGreaterThan(0.6); // 60%+
expect(metrics.reuseRate).toBeGreaterThan(0.8); // 80%+
```

**Business metrics are measured AFTER user workflow, not during.**
</business_metrics>

<anti_patterns>
**❌ Testing implementation details:**

```gherkin
Scenario: API returns correct format
  When service processes request
  Then response has correct schema
```

Fix: Test user-observable outcome

**❌ Vague outcomes:**

```gherkin
Then user sees success
```

Fix: Be specific about what user sees

**❌ No metric verification:**

Test completes but doesn't verify claimed improvements

Fix: Include timing and business metric checks

</anti_patterns>
