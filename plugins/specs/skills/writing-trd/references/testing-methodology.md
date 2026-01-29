<overview>
All TRDs follow the three-tier testing pyramid from the `/testing` skill:

```
Level 3 (E2E)        ← Real credentials, real services, full workflows
Level 2 (Integration) ← Real infrastructure via test harnesses
Level 1 (Unit)        ← Pure logic, dependency injection, no external systems
```

**Build confidence bottom-up**: Level 1 → Level 2 → Level 3
</overview>

<core_principle>
**No Mocking**

Mocking is prohibited. Use:

- **Level 1**: Dependency injection with controlled implementations
- **Level 2/3**: Real infrastructure and services

</core_principle>

<level_decision_rules>
**Level 1 (Unit)**

Use when the guarantee can be verified with:

- Pure arithmetic/logic
- String manipulation
- Data validation
- File operations (temp directories only)
- Standard dev tools (git, node, curl, python)

Infrastructure allowed:

- Test runner (pytest, vitest, jest)
- Language primitives (fs, path, os, crypto)
- Standard dev tools (available in CI without installation)

Cannot verify:

- Database queries
- HTTP API calls
- Project-specific binaries
- External service integration

---

**Level 2 (Integration)**

Use when the guarantee requires:

- Project-specific binary (Hugo, TypeScript compiler, Caddy)
- Database (PostgreSQL, Redis)
- Local service integration

Requires:

- Documented test harness for EACH dependency
- Setup command (how to start)
- Reset command (how to clean between tests)

Cannot verify:

- External API with credentials
- Network services (GitHub API, Stripe)
- Full user workflows

---

**Level 3 (E2E)**

Use when the guarantee requires:

- External API credentials
- Network service (GitHub, Stripe, Trakt.tv)
- Browser (Chrome, Playwright)
- Complete user workflow

Requires:

- Documented credentials for EACH service
- Environment variable names
- Credential source (1Password, .env.test)
- Rotation schedule
- Test account information

</level_decision_rules>

<test_harness_requirements>
For EACH Level 2 dependency, document:

| Required Info                | Example                                              |
| ---------------------------- | ---------------------------------------------------- |
| Harness Type                 | Docker container / Local install / Embedded service  |
| Setup Command                | `docker-compose -f test.yml up -d postgres`          |
| Reset Command                | `docker-compose exec postgres psql -c "TRUNCATE..."` |
| Verify Command               | `hugo version` (for binaries)                        |
| </test_harness_requirements> |                                                      |

<credential_requirements>
For EACH Level 3 dependency, document:

| Required Info              | Example                                  |
| -------------------------- | ---------------------------------------- |
| Environment Vars           | `STRIPE_TEST_API_KEY`, `TEST_USER_EMAIL` |
| Source                     | 1Password: Engineering/Test Credentials  |
| Rotation                   | Quarterly / Annual / Never               |
| Test Accounts              | <test-automation@example.com>            |
| Notes                      | Sandbox vs production, rate limits       |
| </credential_requirements> |                                          |

<infrastructure_gaps>
When harness or credential information is UNKNOWN:

**Do NOT guess or assume.**

Add to Infrastructure Gaps table:

| Gap                                     | Blocking       |
| --------------------------------------- | -------------- |
| [Description of unknown infrastructure] | Implementation |
| [Description of unknown credentials]    | Level 3 tests  |

**Blocking column values:**

- "Implementation": Cannot implement without resolving this
- "Level N tests": Cannot run Level N tests without resolving this

A TRD with Infrastructure Gaps is **incomplete but deliverable**. User knows exactly what must be resolved.
</infrastructure_gaps>

<test_level_assignment_process>
For each guarantee:

1. **Can it be verified with pure logic?**
   - YES → Level 1
   - NO → Continue to 2

2. **Can it be verified with local binaries/databases?**
   - YES → Level 2
   - NO → Continue to 3

3. **Requires external services or credentials?**
   - YES → Level 3

**Document the rationale** for each assignment.
</test_level_assignment_process>

<common_assignment_examples>

| Guarantee                                | Level | Why                               |
| ---------------------------------------- | ----- | --------------------------------- |
| Price calculation handles edge cases     | 1     | Pure arithmetic, no dependencies  |
| Email validation rejects invalid formats | 1     | Pure logic, regex-based           |
| Hugo builds site with correct structure  | 2     | Requires Hugo binary              |
| PostgreSQL persists order records        | 2     | Requires database harness         |
| Stripe payment completes successfully    | 3     | Requires Stripe test credentials  |
| GitHub API creates repository            | 3     | Requires GitHub token and network |
| Complete checkout workflow succeeds      | 3     | Full user flow with real services |
| </common_assignment_examples>            |       |                                   |

<anti_patterns>
**❌ Assigning to higher level than necessary:**

- "Let's do Level 3 to be safe" → Over-engineering, slow tests
- **Fix**: Use minimum level that can verify the guarantee

**❌ Vague infrastructure documentation:**

- "We'll use Docker" → Which container? How to start it?
- **Fix**: Document specific setup/reset commands

**❌ Assuming credentials exist:**

- "Use the test API key" → Which one? Where is it?
- **Fix**: Ask explicitly, document source and rotation

**❌ Skipping levels:**

- Only Level 3 tests for complex feature
- **Fix**: Build confidence bottom-up with all three levels

</anti_patterns>

<readiness_check>
Before finalizing TRD, verify:

- [ ] Every guarantee assigned to exactly one level
- [ ] Level assignment has documented rationale
- [ ] Level 2 guarantees have harnesses documented OR in gaps table
- [ ] Level 3 guarantees have credentials documented OR in gaps table
- [ ] No "TBD" or "TODO" placeholders for infrastructure

</readiness_check>
