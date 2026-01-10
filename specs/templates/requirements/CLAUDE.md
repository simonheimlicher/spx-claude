# Requirements Templates Guide for Claude Agents

This guide explains the purpose, structure, and usage of TRD and PRD templates. Future agents: read this carefully before writing requirements documents.

---

## Purpose and Intent

### What Are TRDs and PRDs?

**TRD (Technical Requirements Document):**

- **Written AFTER technical exploration** with user
- Documents **agreed-upon technical solution** to a problem
- **Authoritative blueprint**: Changes require user approval
- Spawns work items: **features and stories**
- Focuses on: Technical architecture, validation strategy, test infrastructure
- Location: `capability/[name].trd.md` or `feature/[name].trd.md`

**PRD (Product Requirements Document):**

- **Written AFTER product/user problem exploration** with user
- Documents **user value and measurable outcomes**
- **Authoritative blueprint**: Changes require user approval
- Spawns work items: **features and stories**
- Focuses on: User pain, customer journey, acceptance criteria, product scope
- Location: `project/[name].prd.md`, `capability/[name].prd.md`, or `feature/[name].prd.md`

### Critical Distinction

**PRDs answer:** WHAT user value? WHY does user need this? WHAT outcomes?
**TRDs answer:** HOW technically? WHAT architecture? HOW to validate?

**Use PRD when:** Change is driven by user needs, product strategy, market demands
**Use TRD when:** Change is technical (refactoring, architecture, technical debt, infrastructure)

---

## When to Use the Skills

### Invoking Skills

**For TRDs:** Use `/writing-technical-requirements` skill

- Agent will read context, ask about root cause and solution
- Agent will design validation strategy with test level assignments
- Agent will discover test infrastructure (harnesses, credentials)
- Agent will write complete TRD file

**For PRDs:** Use `/writing-product-requirements` skill

- Agent will read context, ask about user problem and journey
- Agent will design measurable outcomes with quantified targets
- Agent will create acceptance tests (Gherkin + E2E code)
- Agent will define product scope with inclusion/exclusion rationale

### When NOT to Use These Templates

**Don't use TRD for:**

- User-facing product features → Use PRD instead
- Simple bug fixes → Write directly to work items
- Documentation updates → Edit docs directly

**Don't use PRD for:**

- Technical refactoring → Use TRD instead
- Infrastructure changes → Use TRD instead
- Internal code improvements → Use TRD instead

---

## Template Structure Rationale

### Why We Removed Checkboxes

**Old approach:** `- [ ] Technical Capability: [description]`

**Problem:** Checkboxes imply binary completion. Agents don't check boxes—they write content. Empty checkboxes with placeholder text don't guide agents toward complete specifications.

**New approach:** "Required Sections" table describing WHAT must be complete

**Benefit:** Agents understand what constitutes "ready" without checkbox ceremonies

### Why We Added Guarantee Numbering

**TRD guarantees:** G1, G2, G3...
**PRD capabilities:** UC1, UC2, UC3...

**Purpose:**

- Every BDD scenario MUST reference a guarantee: `Scenario: [Name] [G1]`
- Enables verification: Does every guarantee have ≥1 scenario?
- Prevents orphaned scenarios that don't map to requirements
- Creates traceability from requirement → test

**In Readiness Criteria:**

- Verify every guarantee has unique ID
- Verify every scenario references exactly one guarantee
- Verify every guarantee has ≥1 scenario

### Why Test Infrastructure Is Explicit

**TRD has dedicated sections:**

- **Level 2: Test Harnesses** - Setup/reset commands for each dependency
- **Level 3: Credentials** - Environment variables, sources, rotation schedules
- **Infrastructure Gaps** - Explicit list of unknowns blocking implementation

**Why explicit?**

Before this structure, agents would write: "Use Docker for PostgreSQL" (vague)

Now agents must write:

```
| PostgreSQL | Docker container | `docker-compose -f test.yml up -d postgres` | `docker exec postgres psql -c "TRUNCATE..."` |
```

**Principle:** Test infrastructure documented OR explicitly in gaps table. No "we'll figure it out later."

### Why BDD Scenarios Reference Guarantees

**Format:** `Scenario: [Name] [G1]` or `Scenario: [Name] [UC1]`

**Why?**

- Guarantees/capabilities are WHAT must be true
- Scenarios are HOW we prove it
- Reference linkage ensures complete coverage
- Prevents writing tests unrelated to requirements
- Enables checklist: "Does G3 have any scenarios? No? Missing test."

### Why PRDs Require Quantified Outcomes

**Format:** "X% improvement in Y within Z timeframe"

**Why?**

- "Users will be happier" is not measurable
- "60% reduction in task completion time" is verifiable
- Current → Target format forces baseline understanding
- Enables acceptance test verification (timing measurements, metrics checks)

**Evidence of Success table:**

| Metric | Current | Target | Improvement |
| ------ | ------- | ------ | ----------- |

Forces agent to think: What IS the current state? What's realistic improvement?

### Why Infrastructure Gaps Table Exists

**Format:**

| Gap                      | Blocking                       |
| ------------------------ | ------------------------------ |
| [Unknown infrastructure] | Implementation / Level N tests |

**Purpose:**

- Agents cannot always know what test infrastructure exists
- Rather than guess or leave vague, document the unknown explicitly
- "Blocking" column shows what cannot proceed without resolving
- TRD/PRD can be "incomplete but deliverable"—user knows exactly what's missing

---

## Testing Methodology Integration

### Why Testing Is First-Class in Requirements

**Philosophy:** Validation strategy is not an afterthought—it's part of the requirement.

**In TRDs/PRDs:**

- Every guarantee/capability assigned to test level (L1/L2/L3)
- Test level assignment has documented rationale
- Test infrastructure requirements explicit
- BDD scenarios prove guarantees

**Agents must consult:**

- `/testing` skill - Foundational principles
- `/testing-python` or `/testing-typescript` - Language patterns

### Three-Tier Testing

**Level 1 (Unit):**

- Pure logic, dependency injection
- No external systems (databases, APIs, browsers)
- Allowed: Test runner, language primitives, temp files, standard dev tools (git, node, curl)

**Level 2 (Integration):**

- Real infrastructure via test harnesses
- Required: Docker containers, local databases, project-specific binaries
- Must document: Setup command, reset command

**Level 3 (E2E):**

- Real credentials, real services, full workflows
- Required: External API credentials, test accounts, browser automation
- Must document: Environment variables, credential sources, rotation schedules

### No Mocking Principle

**Prohibited:** Creating fake/mock implementations of external dependencies

**Instead:**

- Level 1: Dependency injection with controlled real implementations
- Level 2: Real databases/binaries via documented harnesses
- Level 3: Real services with real credentials

**Why:** Mocking creates false confidence. Reality is the oracle.

### Bottom-Up Confidence Building

**Order:** Level 1 → Level 2 → Level 3

**Rationale:**

- Fast tests (L1) catch logic errors quickly
- Medium tests (L2) prove integration with infrastructure
- Slow tests (L3) verify complete user workflows
- Don't skip to L3—build confidence from bottom up

---

## Key Design Decisions

### Agent-Focused Language

**Old:** "A PRD is ready when these boxes are checked..."
**New:** "A PRD is ready when all sections below contain complete information."

**Why:** Agents don't check boxes. Agents write content. Language should describe what constitutes completeness, not ceremonial checkboxes.

### Explicit vs Vague

**Old (vague):**

- "We'll use Docker"
- "Tests will be added"
- "Credentials are in the usual place"

**New (explicit):**

- "Docker container started via `docker-compose -f test.yml up -d postgres`"
- "Level 2 tests verify [specific behavior] using PostgreSQL harness"
- "Credentials in 1Password: Engineering/Test Credentials vault, `STRIPE_TEST_API_KEY` env var"

**Principle:** Either document explicitly OR document as explicit gap. No vagueness.

### Measurable Outcomes Format

**For PRDs, this format is mandatory:**

```
Users will [action] leading to [X% improvement in metric A] and [Y% improvement in metric B],
proven by [measurement method] within [timeframe or at delivery].
```

**Why mandatory:**

- Forces quantification
- Forces baseline understanding (Current → Target)
- Enables verification in acceptance tests
- Prevents vague "better experience" claims

### Infrastructure Gaps as First-Class

**Old assumption:** "If we need it, it exists or we'll build it"

**New reality:** Test infrastructure often doesn't exist yet

**Infrastructure Gaps table makes this explicit:**

- Agent asks user about test infrastructure
- If unknown, goes in gaps table with "Blocking" column
- TRD can be "incomplete" if gaps exist
- User knows exactly what must be resolved before implementation

**This is OK.** Better to have incomplete TRD with explicit gaps than complete TRD with hidden assumptions.

---

## Workflow Context

### TRD Workflow

1. **Exploration phase** (user and agent discuss technical problem)
2. **Agent invokes** `/writing-technical-requirements` skill
3. **Skill reads** project context, ADRs, existing capabilities
4. **Phase 1:** Confirm root cause and solution approach with user
5. **Phase 2:** Design validation strategy (guarantees → test levels → BDD scenarios)
6. **Phase 3:** Discover test infrastructure (harnesses, credentials, gaps)
7. **Phase 4:** Write complete TRD file
8. **Phase 5:** Deliver with readiness assessment and next actions

### PRD Workflow

1. **Exploration phase** (user and agent discuss user problem, product need)
2. **Agent invokes** `/writing-product-requirements` skill
3. **Skill reads** project context, existing PRDs, product direction
4. **Phase 1:** Confirm user problem and customer journey with user
5. **Phase 2:** Design measurable outcomes (metrics → capabilities → acceptance tests)
6. **Phase 3:** Define product scope (included, excluded, ADR triggers)
7. **Phase 4:** Write complete PRD file
8. **Phase 5:** Deliver with readiness assessment and next actions

### What Comes After

**Both TRDs and PRDs spawn work items:**

- Features (mid-level, integration testing)
- Stories (atomic, unit testing)

**TRDs may trigger:**

- ADRs for architectural decisions
- Multiple features if scope is large

**PRDs may trigger:**

- ADRs for technical approach decisions
- Phased delivery if dependencies require it

---

## Common Pitfalls for Future Agents

### DON'T: Write TRD Without Confirmed Root Cause

**Wrong:**

```
User says "dashboard is slow"
Agent writes TRD about dashboard optimization
```

**Right:**

```
User says "dashboard is slow"
Agent investigates: Why is it slow?
Agent proposes: "Root cause is N+1 query pattern loading user data"
User confirms or corrects
THEN agent writes TRD
```

### DON'T: Write PRD Without Quantified Outcomes

**Wrong:**

```
**Expected Outcome:** Users will create resumes faster and more efficiently.
```

**Right:**

```
**Expected Outcome:** Users will create resumes with 60% reduction in time
(45min → 18min) and 80% increase in content reuse (25% → 80%), proven by
resume assembly timing and variant utilization metrics at delivery.
```

### DON'T: Leave Test Infrastructure Vague

**Wrong:**

```
**Level 2 Tests:** We'll use PostgreSQL for database testing.
```

**Right:**

```
**Level 2: Test Harnesses**

| Dependency | Harness Type | Setup Command | Reset Command |
|------------|--------------|---------------|---------------|
| PostgreSQL | Docker container | `docker-compose -f test.yml up -d postgres` | `docker-compose exec postgres psql -c "TRUNCATE users, orders CASCADE;"` |
```

**Or if unknown:**

```
**Infrastructure Gaps**

| Gap | Blocking |
|-----|----------|
| PostgreSQL test harness setup unknown - need Docker configuration | Level 2 tests |
```

### DON'T: Write BDD Scenarios Without Guarantee Links

**Wrong:**

```
Scenario: User can create resume
  Given user is logged in
  When user clicks create
  Then resume is created
```

**Right:**

```
Scenario: User creates resume from repository data [UC2]
  Given user has career data in repository
  When user selects "Create Resume" and chooses template
  Then resume is generated with 80%+ content from existing variants
```

### DON'T: Mix PRD and TRD Concerns

**Wrong PRD (too technical):**

```
**Technical Architecture:**
- Use React with Redux for state management
- PostgreSQL database with Prisma ORM
- Express.js backend with JWT authentication
```

**Right PRD (product-focused):**

```
**Product Approach:**
- Users interact via web interface (responsive for desktop/tablet)
- Direct manipulation: click to edit, auto-save, no modal dialogs
- ⚠️ ADR needed: Client-side state management approach
- ⚠️ ADR needed: Database choice and ORM strategy
```

**Wrong TRD (too product-focused):**

```
**Problem Statement:** Users need to create resumes faster
```

**Right TRD (technical):**

```
**Problem Statement:** Current resume generation queries database N times per
user record due to N+1 pattern, causing 45-second generation time. Need to
implement eager loading strategy to achieve <10-second generation.
```

---

## Template Evolution Notes

### Why We Migrated from Markdown to Pure XML

**Old:** Section headers like `## Overview`, `### Level 1`

**New:** Semantic XML tags like `<overview>`, `<level_decision_rules>`

**Why:**

- Consistent structure across all files
- Better parseability for agents
- Semantic meaning embedded in tags
- Token efficiency (XML tags shorter than markdown headers)

**In templates:** Still use markdown for content structure, but skills use pure XML

### Why "Required Sections" Table Replaced DoR Checklist

**Old DoR (Definition of Ready):**

```
| DoR checkbox | Description |
|--------------|-------------|
| [ ] Outcome  | ... |
| [ ] Tests    | ... |
```

**New Required Sections:**

```
| Section | Purpose |
|---------|---------|
| Validation Strategy | Guarantees mapped to test levels with BDD scenarios |
| Test Infrastructure | Harnesses and credentials documented |
```

**Why:**

- Describes WHAT must be complete, not ceremony
- Focuses on content completeness, not checkbox state
- More natural for agents to understand and verify

---

## Final Guidance for Future Agents

### When Writing a TRD

1. **Invoke** `/writing-technical-requirements` skill—don't write manually
2. **Confirm root cause** with user before proposing solution
3. **Assign every guarantee** to minimum appropriate test level
4. **Ask explicitly** about test harnesses and credentials
5. **Document gaps** if infrastructure unknown—don't guess
6. **Verify coverage**: Every guarantee has ≥1 scenario, every scenario references a guarantee
7. **Run readiness check** before declaring complete

### When Writing a PRD

1. **Invoke** `/writing-product-requirements` skill—don't write manually
2. **Confirm user problem** (pain vs symptom) with user
3. **Quantify outcomes**: X% improvement in Y within Z
4. **Create acceptance tests**: Gherkin + actual E2E test code
5. **Define scope explicitly**: What's in, what's out, WHY
6. **Mark ADR triggers** in technical approach with ⚠️
7. **Run readiness check** before declaring complete

### Trust the Skills

The skills were designed with systematic questioning, deep thinking checkpoints, and quality guardrails. They embody months of learning about what makes good requirements documents.

**Don't bypass the skills.** They ensure:

- User confirmation at critical decision points
- Complete test infrastructure documentation
- Proper guarantee/scenario linkage
- Readiness criteria verification

### Remember the Purpose

**TRDs and PRDs are authoritative blueprints written AFTER exploration.**

They're not:

- ❌ Wish lists
- ❌ Brainstorming documents
- ❌ First drafts to be heavily edited

They ARE:

- ✅ Contracts between user and implementation
- ✅ Complete specifications ready for decomposition
- ✅ Validation strategies proving success
- ✅ Test infrastructure documentation enabling implementation

**Write them with care. They guide all downstream work.**

---

_Last updated: 2026-01-10_
_Skills: writing-technical-requirements v0.1.0, writing-product-requirements v0.1.0_
_Templates: technical-change.trd.md, product-change.prd.md_
