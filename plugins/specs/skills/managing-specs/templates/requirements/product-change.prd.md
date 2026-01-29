# PRD: [Marketable Increment Name]

> **Purpose**: Defines a marketable increment of product functionality and serves as an authoritative contract for implementation.
>
> - Written BEFORE capability/feature structure exists—drives folder creation
> - Defines targeted enhancement delivering real user value
> - Excludes fantasies and bells/whistles to accelerate deployment and user feedback
> - Authoritative: Changes to scope or user outcomes require user approval
> - Triggers ADR creation for technical decisions; if ADRs reveal scope too large → split into multiple PRDs
> - Spawns work items (features/stories) AFTER scope validated and ADRs created
> - No size constraints, no state tracking (OPEN/IN PROGRESS/DONE)
> - Can exist at: project level (`specs/work/{backlog,doing}/[Name].prd.md`), capability level (`.../capability/[Name].prd.md`) or feature level (`.../feature/[Name].prd.md`)
>
> **Core Principle**: Requirements describe WHAT should exist (the ideal solution), NOT WHEN it gets implemented. Use "Out of scope" for boundaries, NOT "Defer to Phase 2" or "MVP excludes...". Implementation timing belongs in work items (capability/feature/story breakdown), NOT in requirements.

## Required Sections

A PRD is ready for implementation when all sections below contain complete information. Empty or placeholder content indicates the PRD is not ready.

| Section          | Purpose                                                                    |
| ---------------- | -------------------------------------------------------------------------- |
| Product Vision   | User problem, value proposition, customer journey, and user assumptions    |
| Expected Outcome | Quantified measurable outcome with evidence metrics                        |
| Acceptance Tests | Complete E2E journey test and Gherkin scenarios proving measurable outcome |
| Scope Definition | Explicit boundaries: what's included, what's excluded, and why             |
| Product Approach | Interaction model, UX principles, technical approach (triggers ADRs)       |
| Success Criteria | User outcomes, quality attributes, definition of "done"                    |
| Open Decisions   | Questions for user, ADR triggers, product trade-offs                       |
| Dependencies     | Work items, customer-facing, technical, and performance requirements       |
| Pre-Mortem       | Assumptions to validate (adoption, performance, platform, scope)           |

## Testing Methodology

This PRD follows the three-tier testing methodology for acceptance validation:

- **Level 1 (Unit)**: Component logic with dependency injection. No external systems.
- **Level 2 (Integration)**: Real infrastructure via test harnesses. No mocking.
- **Level 3 (E2E)**: Real credentials and services. Full user workflows validating measurable outcome.

**Build confidence bottom-up**: Level 1 → Level 2 → Level 3.

Before implementation, agents MUST consult:

- `/testing` — Foundational principles, decision protocol, anti-patterns
- `/testing-python` or `/testing-typescript` — Language-specific patterns and fixtures

## Product Vision

### User Problem

**Who** are we building for? Describe the user archetype (role, context, constraints).

[e.g., "Solo developers managing multiple Claude Code projects who switch contexts frequently and lose track of where they left off."]

**What** problem do they face?

```
As [user type], I am frustrated by [core problem]
because [underlying cause], which prevents me from [desired outcome].
```

### Current Customer Pain

Document the current state and its impact:

- **Symptom**: [What users experience or work around today]
- **Root Cause**: [Why the problem exists - systemic reason, not just missing feature]
- **Customer Impact**: [How this affects user workflow, productivity, or satisfaction]
- **Business Impact**: [How this affects product value, user retention, or strategic goals]

### Customer Solution

```
Implement [solution] that enables [user] to [new capability]
through [interaction model], resulting in [improved outcome].
```

### Customer Journey Context

Describe the transformation this product creates:

- **Before**: [Current state - how users operate today without this capability]
- **During**: [Transition state - how users discover, learn, and adopt this capability]
- **After**: [Future state - how users operate with this capability integrated into workflow]

### User Assumptions

Since we cannot interview real users, document explicit assumptions. If no special assumptions apply beyond standard user capabilities, write "None identified" and proceed.

Assumptions worth documenting include:

- User technical capability (CLI comfort, git knowledge, programming expertise)
- User context (workflow patterns, tool usage, environment)
- User goals and priorities (speed vs. control, automation vs. explicitness)
- User constraints (infrastructure access, tooling limitations)

| Assumption Category  | Specific Assumption                             | Impact on Product Design                       |
| -------------------- | ----------------------------------------------- | ---------------------------------------------- |
| [e.g., User Context] | [e.g., "Works on 3-10 projects simultaneously"] | [e.g., "Must support fast context switching"]  |
| [e.g., User Goals]   | [e.g., "Values automation over manual control"] | [e.g., "Default to auto-capture with opt-out"] |

## Expected Outcome

### Measurable Outcome

Use this specific quantified format:

```
Users will [action] leading to [X% improvement in metric A] and [Y% improvement in metric B],
proven by [measurement method] within [timeframe or at delivery].
```

Example:

```
Users will directly manage their career repository data leading to 60% faster resume creation and
80% increase in career data reuse, proven by resume assembly time reduction and variant utilization
metrics within 3 months.
```

### Evidence of Success

List measurable outcomes with **Current → Target** format:

| Metric                       | Current      | Target     | Improvement                                  |
| ---------------------------- | ------------ | ---------- | -------------------------------------------- |
| [e.g., Resume Creation Time] | 45min        | 18min      | 60% reduction in resume assembly time        |
| [e.g., Career Data Reuse]    | 25%          | 80%        | 80% of resume content from existing variants |
| [e.g., User Engagement]      | 0%           | 70%        | Users actively using repository interface    |
| [e.g., Content Quality]      | Inconsistent | Consistent | Professional narrative coherence             |

## Acceptance Tests

### Complete User Journey Test

Provide a complete E2E test proving the measurable outcome. Adapt to your project's language and framework (TypeScript/Playwright, Python/pytest, etc.).

The test should cover the full user journey, include timing measurements for efficiency claims, verify business metrics for impact claims, and use concrete `data-testid` attributes showing UX design.

```typescript
// Feature-level E2E test - adapt to your language/framework
describe("Feature: [Feature Name]", () => {
  test("user achieves [measurable outcome] through [workflow]", async ({ page }) => {
    // Given: [Initial user state]
    const startTime = Date.now();
    await page.goto("/[entry-point]");

    // When: [User performs workflow phases]
    await expect(page.locator("[data-testid=\"phase-1-element\"]")).toBeVisible();
    await page.click("[data-testid=\"user-action-1\"]");

    await page.fill("[data-testid=\"input-field\"]", "test data");
    await page.click("[data-testid=\"user-action-2\"]");

    // Then: [User achieves measurable outcome]
    await expect(page.locator("[data-testid=\"success-indicator\"]")).toBeVisible();

    const completionTime = Date.now() - startTime;
    expect(completionTime).toBeLessThan(TARGET_TIME_MS);

    const metrics = await getBusinessMetrics(testData.userId);
    expect(metrics.efficiencyGain).toBeGreaterThan(0.6); // 60%+ improvement
  });
});
```

For language-specific testing patterns, consult `/testing-typescript` or `/testing-python` for fixture patterns, async handling, and framework-specific best practices.

### User Scenarios (Gherkin Format)

Write scenarios in Gherkin format. Each scenario validates a key aspect of the measurable outcome.

```gherkin
Feature: [Feature Name from Measurable Outcome]

  Scenario: [Primary User Journey - Core Value]
    Given [initial user state and preconditions]
    When [user performs primary workflow]
    Then [user achieves primary outcome]
    And [supporting outcome or verification]
    And [business metric is improved]

  Scenario: [Secondary User Journey - Supporting Value]
    Given [initial user state and preconditions]
    When [user performs secondary workflow]
    Then [user achieves secondary outcome]
    And [supporting outcome or verification]

  Scenario: [Error Case - Graceful Degradation]
    Given [initial user state and preconditions]
    When [user encounters error condition]
    Then [user receives clear error message]
    And [user understands recovery path]
    And [user data is not lost or corrupted]
```

### Scenario Detail (Given/When/Then)

Expand key scenarios with explicit Given/When/Then format for implementation clarity:

**Scenario: [Primary user achieves core outcome]**

- **Given** [Initial state: user context, data state, system state]
- **When** [User action: specific interaction or trigger]
- **Then** [Observable outcome: what user sees, receives, or can verify]

**Scenario: [User handles edge case gracefully]**

- **Given** [Edge case setup: unusual but valid conditions]
- **When** [User action: same interaction under edge conditions]
- **Then** [Graceful outcome: system handles edge case appropriately]

**Scenario: [User recovers from error]**

- **Given** [Error condition: invalid state or user mistake]
- **When** [User attempts action that triggers error]
- **Then** [Clear error message and recovery guidance provided]

## Scope Definition

### What's Included

This deliverable unit includes:

- ✅ [Capability 1: e.g., "Direct viewing and navigation of career repository data"]
- ✅ [Capability 2: e.g., "Create and edit career variants for different contexts"]
- ✅ [Capability 3: e.g., "Assemble custom resumes from repository with variant selection"]

### What's Explicitly Excluded

Document exclusions with rationale. If no exclusions apply (rare for MVP), write "None identified" and proceed.

| Excluded Capability                       | Rationale                                                                            |
| ----------------------------------------- | ------------------------------------------------------------------------------------ |
| [e.g., Multi-user collaboration]          | Defer until single-user workflow validates value; collaboration is different product |
| [e.g., Cloud sync]                        | Local-first reduces complexity and external dependencies; defer until v2             |
| [e.g., Search across historical handoffs] | Adds complexity; defer until usage patterns emerge from v1                           |

### Scope Boundaries Rationale

**Why these boundaries?**

[Explain the core value you're protecting, the complexity you're avoiding, the learning you need before expanding scope, or the MVP validation strategy.]

Example:

```
This release focuses on the individual user's direct career data management workflow. We're explicitly
avoiding multi-user collaboration and cloud sync until we validate that single-user local-first workflow
delivers the efficiency gains. Real-time collaboration is a different product with different complexity.
```

## Product Approach

### Interaction Model

**How do users interact with this product?**

- **Interface Type**: [CLI command, GUI, API, editor extension, web app, mobile app, etc.]
- **Invocation Pattern**: [How users start: command, click, URL, hotkey, etc.]
- **User Mental Model**: [Analogy or metaphor: "Like X but for Y", "Think of it as Z"]

Example:

- **Interface Type**: CLI command within Claude Code environment
- **Invocation Pattern**: User runs `/handoff` command in active conversation
- **User Mental Model**: "Like git stash but for full conversation and project context"

### User Experience Principles

What UX principles guide design decisions? These become constraints for implementation.

1. **[Principle 1]**: [e.g., "Zero configuration by default - should work without setup"]
2. **[Principle 2]**: [e.g., "Fast feedback - show what was captured immediately"]
3. **[Principle 3]**: [e.g., "Non-destructive - never lose user context"]
4. **[Principle 4]**: [e.g., "Obvious recovery - if something fails, user knows what to do"]

### High-Level Technical Approach

**Note**: Technical approaches that mention specific technologies, patterns, or architectural decisions should trigger Architecture Decision Records (ADRs) during implementation planning.

Describe the approach at a high level without prescribing implementation:

**Data Model:**

- [e.g., "Handoffs are structured documents with metadata and captured state"]
- [e.g., "Each project has a local storage location for handoff documents"]
  - ⚠️ **ADR Trigger**: Storage format? (Markdown + YAML vs. JSON vs. SQLite)

**Key Capabilities:**

- [Capability 1: e.g., "Capture git status, recent commits, and conversation context"]
  - ⚠️ **ADR Trigger**: How much conversation context to capture? (Full history vs. summary vs. user selection)
- [Capability 2: e.g., "Restore context by reading handoff document"]
  - ⚠️ **ADR Trigger**: How to present restored context? (Inject into conversation vs. show in UI vs. file display)

**Integration Points:**

- [e.g., "Integrates with Claude Code skill system for command handling"]
- [e.g., "Uses git CLI for repository state capture"]
  - ⚠️ **ADR Trigger**: How to handle projects without git? (Graceful degradation vs. requirement vs. error)

**User Interface:**

- [e.g., "Command-based interaction via `/handoff` and `/pickup` skills"]
- [e.g., "Output shown inline in conversation with structured summary"]
  - ⚠️ **ADR Trigger**: Output format? (Plain text vs. markdown tables vs. rich UI components)

### Product-Specific Constraints

Document constraints beyond standard product development. If no special constraints apply, write "None identified" and proceed.

Constraints worth documenting include:

- External service dependencies with API limits, auth requirements, or availability concerns
- Platform integration constraints affecting UX or feature scope
- User environment limitations (browser compatibility, device constraints)
- Regulatory or accessibility requirements
- Legacy system compatibility requirements

| Constraint                                       | Impact on Product                           | Impact on Testing                             |
| ------------------------------------------------ | ------------------------------------------- | --------------------------------------------- |
| [e.g., Browser localStorage limit: 5-10MB]       | Must implement data archival strategy       | E2E tests must validate graceful degradation  |
| [e.g., Claude Code CLI API surface is read-only] | Cannot modify conversation programmatically | Test context injection through available APIs |

### Technical Assumptions

Document technical assumptions that need validation during implementation:

- **Architecture Assumption**: [e.g., "Existing three-layer backend provides sufficient API surface"]
- **Performance Assumption**: [e.g., "Sub-500ms response time achievable with current infrastructure"]
- **Integration Assumption**: [e.g., "Current authentication system supports career management workflows"]
- **Scalability Assumption**: [e.g., "Pagination handles datasets up to 10,000 career records"]

## Success Criteria

### User Outcomes

How do we know this delivered value for users?

| Outcome                                             | Success Indicator                                                               |
| --------------------------------------------------- | ------------------------------------------------------------------------------- |
| [e.g., Users successfully resume work after switch] | User can answer "What was I working on?" within 30 seconds of `/pickup`         |
| [e.g., Users trust handoff captures all context]    | User doesn't manually document context before switching (measures system trust) |
| [e.g., Users adopt into regular workflow]           | User runs `/handoff` before every context switch (after successful first use)   |

### Quality Attributes

What non-functional requirements must the product meet?

| Attribute       | Target                                              | Measurement Approach                             |
| --------------- | --------------------------------------------------- | ------------------------------------------------ |
| **Usability**   | [e.g., "Zero-config - works immediately"]           | Test with no documentation, observe friction     |
| **Speed**       | [e.g., "Handoff capture completes in <2 seconds"]   | Time from command invocation to confirmation     |
| **Reliability** | [e.g., "Never lose context - graceful degradation"] | Test error cases (no git, large repos, failures) |
| **Clarity**     | [e.g., "User understands what was captured"]        | Show clear summary of captured context           |

### Definition of "Done"

This deliverable unit is complete when:

1. All acceptance criteria scenarios pass (Given/When/Then validation)
2. Complete user journey E2E test passes (code from "Acceptance Tests" section)
3. Quality attributes meet targets (measured per table above)
4. Error cases handled gracefully (user knows what went wrong and how to recover)
5. Measurable outcome is achievable (Evidence of Success metrics can be collected)

## Open Decisions

### Questions Requiring User Input

Decisions that need user clarification before implementation begins. If no open questions remain, write "None identified" and proceed.

| Question                                               | Option A               | Option B              | Trade-offs                                    | Recommendation    |
| ------------------------------------------------------ | ---------------------- | --------------------- | --------------------------------------------- | ----------------- |
| [e.g., "Should handoffs be per-project or global?"]    | Per-project (isolated) | Global (searchable)   | A = simpler, B = powerful but complex         | [If you have one] |
| [e.g., "How handle conflicts if handoff file exists?"] | Overwrite              | Append with timestamp | A = simple, B = preserves history but clutter | [If you have one] |

### Decisions Triggering ADRs

Technical approaches requiring architectural decisions (marked with ⚠️ in "Product Approach" section):

| Decision Topic                        | Key Question                              | Options to Evaluate                        | Triggers                     |
| ------------------------------------- | ----------------------------------------- | ------------------------------------------ | ---------------------------- |
| [e.g., Context capture strategy]      | How much conversation context to capture? | Full history / Last N messages / Summary   | `/architecting-{lang}` skill |
| [e.g., Storage format]                | Markdown + YAML vs. JSON vs. SQLite?      | Human-readable vs. queryable vs. validated | `/architecting-{lang}` skill |
| [e.g., Graceful degradation approach] | How to handle projects without git?       | Capture partial / Show error / Disable     | `/architecting-{lang}` skill |

### Product Trade-offs

Product-level trade-offs affecting scope, UX, or approach:

| Trade-off                        | Option A               | Option B             | Impact                                                    |
| -------------------------------- | ---------------------- | -------------------- | --------------------------------------------------------- |
| [e.g., "Handoff granularity"]    | One per context switch | Multiple per session | A = simpler, B = more flexible but complex                |
| [e.g., "Context presentation"]   | Inline in conversation | Separate file/UI     | A = immediate, B = cleaner but requires navigation        |
| [e.g., "Automatic vs. explicit"] | Auto-capture on events | User runs command    | A = convenient but surprising, B = predictable but manual |

## Dependencies

### Work Item Dependencies

**Note**: Requirements are orthogonal to work items. This section documents prerequisite capabilities that must exist, not specific stories or tasks (those are created after PRD approval).

Capabilities or platform features that must exist before this PRD can be implemented:

| Dependency                         | Status                               | Rationale                                             |
| ---------------------------------- | ------------------------------------ | ----------------------------------------------------- |
| [e.g., Claude Code skill system]   | Complete                             | Required to implement command-based interaction model |
| [e.g., Platform Capability 40]     | In Progress                          | Required for database safety guarantees               |
| [e.g., Git integration capability] | Complete / In Progress / Not Started | Required to capture repository state                  |

### Customer-Facing Dependencies

Non-technical dependencies affecting user experience:

| Dependency Type      | Specific Need                                                | Impact If Missing                                |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| **User Research**    | [e.g., Interface design validation, workflow usability test] | May miss critical usability issues before launch |
| **Design Assets**    | [e.g., Career repository navigation UI mockups]              | Implementation may not match intended UX vision  |
| **Content**          | [e.g., Help documentation, onboarding guides]                | Users struggle to discover and adopt features    |
| **Support Training** | [e.g., Support team training on career repository concepts]  | Support cannot help users with issues            |

### Technical Dependencies

Platform, infrastructure, and technical prerequisites:

| Dependency                     | Version/Constraint | Purpose                    | Availability                       |
| ------------------------------ | ------------------ | -------------------------- | ---------------------------------- |
| [e.g., Claude Code]            | >=0.x.x            | Host environment           | Assumed available                  |
| [e.g., Git]                    | >=2.0              | Repository state capture   | Optional - graceful degradation    |
| [e.g., Node.js]                | >=18.0.0           | Runtime environment        | Assumed available in Claude Code   |
| [e.g., Platform Service Layer] | Complete           | Database safety compliance | Blocks write operations if missing |

### Performance Requirements

Non-functional technical requirements with measurable targets:

| Requirement Area    | Target                                                | Measurement Approach                      |
| ------------------- | ----------------------------------------------------- | ----------------------------------------- |
| **Response Time**   | [e.g., "Sub-500ms for repository navigation"]         | P95 latency under normal load             |
| **Operation Time**  | [e.g., "Sub-2s for variant creation/editing"]         | End-to-end timing from action to feedback |
| **Scalability**     | [e.g., "Handle 10,000 career records without paging"] | Test with large dataset fixtures          |
| **Analytics Setup** | [e.g., "Usage tracking for adoption metrics"]         | Event instrumentation for all key actions |

## Pre-Mortem Analysis

Frame risks as **assumptions to validate** during implementation.

### Assumption: [User adoption/usability assumption]

Example: "Users will find career repository interface intuitive despite data complexity"

- **Likelihood**: [Low/Medium/High] - [Reasoning for likelihood assessment]
- **Impact**: [Low/Medium/High] - [What happens if assumption is wrong]
- **Mitigation**: [Specific actions to validate assumption or reduce impact]

### Assumption: [Performance/technical assumption]

Example: "Performance will be acceptable with large career datasets"

- **Likelihood**: [Low/Medium/High] - [Reasoning for likelihood assessment]
- **Impact**: [Low/Medium/High] - [What happens if assumption is wrong]
- **Mitigation**: [Specific actions to validate assumption or reduce impact]

### Assumption: [Platform/dependency assumption]

Example: "Platform service layer will be ready when this feature needs it"

- **Likelihood**: [Low/Medium/High] - [Reasoning for likelihood assessment]
- **Impact**: [Low/Medium/High] - [What happens if assumption is wrong]
- **Mitigation**: [Specific actions to validate assumption or reduce impact]

### Assumption: [Scope/complexity assumption]

Example: "Feature scope is achievable within complexity budget for single release"

- **Likelihood**: [Low/Medium/High] - [Reasoning for likelihood assessment]
- **Impact**: [Low/Medium/High] - [What happens if assumption is wrong]
- **Mitigation**: [Specific actions to validate assumption or reduce impact]

## Readiness Criteria

A reviewing agent should verify the following before approving this PRD for implementation:

### 1. Product Vision

- User problem is clearly articulated with concrete pain points
- Value proposition is compelling and specific
- User assumptions are documented explicitly (or "None identified" if standard)
- Customer journey context shows before/during/after transformation

### 2. Expected Outcome

- Measurable outcome uses quantified format: "X% improvement in Y within Z timeframe"
- Evidence of Success table lists concrete metrics with Current → Target format
- Metrics are tied to user value, not just technical metrics

### 3. Acceptance Tests

- Complete E2E test exists in actual code (TypeScript/Python/etc.) or pseudocode
- Test covers full user journey from start to finish
- Test includes timing measurements for efficiency claims
- Test verifies business metrics for impact claims
- At least 3 scenarios in Gherkin format (primary journey, supporting journey, error case)
- Key scenarios expanded with explicit Given/When/Then detail
- Scenarios are observable from user perspective (not internal system behavior)

### 4. Scope Definition

- Included capabilities are concrete and deliverable as one unit
- Excluded capabilities table has rationale (or "None identified" if no exclusions)
- Scope boundaries rationale explains value protected and complexity avoided
- No references to specific work items (capabilities, features, stories) - those come AFTER PRD approval

### 5. Product Approach

- Interaction model is clear (how users interact)
- UX principles are defined to guide implementation decisions
- High-level technical approach identifies ADR triggers (marked with ⚠️)
- Product-specific constraints documented (or "None identified" if standard)
- Technical assumptions are documented explicitly

### 6. Success Criteria

- User outcomes table defines success indicators
- Quality attributes table has targets and measurement approaches
- Definition of "Done" is explicit and verifiable

### 7. Open Decisions

- Questions for user table documents options and trade-offs (or "None identified" if resolved)
- Decisions requiring ADRs are listed in table with triggering questions
- Product trade-offs are articulated with impact analysis

### 8. Dependencies

- Work item dependencies describe prerequisite capabilities, NOT specific stories
- Customer-facing dependencies (design, research, content) documented
- Technical dependencies include version constraints and availability
- Performance requirements have measurable targets

### 9. Pre-Mortem

- At least 4 assumptions across categories (adoption, performance, platform, scope)
- Each assumption framed as "Assumption: [statement]" to validate
- Each assumption has likelihood, impact, and specific mitigation
- Mitigations are actionable, not just "monitor" or "communicate"

### 10. Completeness

- No placeholder content (all [brackets] filled with real content or "None identified")
- No section is marked "TBD" or "TODO"
- PRD describes ONE marketable increment (not decomposed into smaller pieces)
- PRD can drive folder creation: clear capability/ or feature/ level placement
- PRD reads coherently from vision → outcome → acceptance → delivery
- A downstream agent can implement this PRD without guessing or making product decisions
