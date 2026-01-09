<required_reading>
Read `references/testing-methodology.md` for infrastructure requirements.
</required_reading>

<process>
## Deep Thinking: Infrastructure Feasibility

**Pause and analyze the guarantees table:**

For Level 2 guarantees:
- What binaries/databases do they require?
- Do these exist in the project already?
- Can they be run in Docker?
- What's the setup/teardown procedure?

For Level 3 guarantees:
- What external services do they require?
- What credentials are needed?
- Where would credentials be stored?
- What happens if credentials expire?

**What's BLOCKING implementation if infrastructure doesn't exist?**

## Identify Required Infrastructure

From the guarantees table, list:

**Level 2 requirements:**
- [ ] Database: PostgreSQL, Redis, etc.
- [ ] Binary: Hugo, TypeScript compiler, etc.
- [ ] Service: Local API server, etc.

**Level 3 requirements:**
- [ ] External API: GitHub, Stripe, Trakt.tv, etc.
- [ ] Browser: Chrome, Playwright
- [ ] Test accounts: User credentials, API keys

## Ask About Level 2 Harnesses

**Use AskUserQuestion to discover harness details.**

For EACH Level 2 dependency, ask:

```
I need to document Level 2 test harnesses.

For [dependency], please provide:

1. **Harness type**: Docker container, local install, or other?
2. **Setup command**: How to start it?
3. **Reset command**: How to clean between tests?
4. **Location**: Where is this documented or configured?
```

**If user doesn't know**, add to Infrastructure Gaps table.

## Ask About Level 3 Credentials

**Use AskUserQuestion to discover credential details.**

For EACH Level 3 dependency, ask:

```
I need to document Level 3 credentials.

For [service/API], please provide:

1. **Environment variables**: What env vars hold credentials?
2. **Source**: Where are credentials stored (1Password, .env.test, etc.)?
3. **Rotation schedule**: How often do they expire?
4. **Test accounts**: What test accounts exist?
```

**If user doesn't know**, add to Infrastructure Gaps table.

## Document in Tables

**Level 2: Test Harnesses**

For KNOWN harnesses, fill the table:

| Dependency | Harness Type     | Setup Command                               | Reset Command                     |
| ---------- | ---------------- | ------------------------------------------- | --------------------------------- |
| PostgreSQL | Docker container | `docker-compose -f test.yml up -d postgres` | `docker-compose exec postgres...` |

**Level 3: Credentials and Test Accounts**

For KNOWN credentials, fill the table:

| Credential      | Environment Variable  | Source                      | Notes             |
| --------------- | --------------------- | --------------------------- | ----------------- |
| Stripe test key | `STRIPE_TEST_API_KEY` | 1Password: Engineering/Test | Rotates quarterly |

## Handle Infrastructure Gaps

For UNKNOWN or MISSING infrastructure, document in Infrastructure Gaps table:

| Gap                                               | Blocking       |
| ------------------------------------------------- | -------------- |
| [e.g., Message queue harness—RabbitMQ or in-mem?] | Implementation |
| [e.g., Trakt.tv OAuth test credentials unknown]   | Level 3 tests  |

**Blocking column indicates what cannot proceed:**

- "Implementation": Cannot implement without this
- "Level X tests": Cannot run Level X tests without this

## Verify Completeness

Check that:

- [ ] Every Level 2 guarantee has harness documented OR gap listed
- [ ] Every Level 3 guarantee has credentials documented OR gap listed
- [ ] No vague "we'll figure it out later" - explicit knowledge or explicit gap
- [ ] Setup/reset commands are specific (not placeholder text)
- [ ] Credential sources are specific (not "somewhere")

## Mark TRD Status

If Infrastructure Gaps table is EMPTY:
- TRD is COMPLETE and ready for decomposition

If Infrastructure Gaps table has entries:
- TRD is INCOMPLETE but can be delivered
- User knows exactly what must be resolved before implementation

</process>

<success_criteria>
Phase 3 complete when:

- [ ] All Level 2 harnesses documented with setup/reset commands
- [ ] All Level 3 credentials documented with source and rotation
- [ ] Unknown infrastructure documented in Infrastructure Gaps table
- [ ] No vague placeholders (explicit knowledge or explicit gap)
- [ ] TRD marked complete or incomplete based on gaps
- [ ] Ready to write complete TRD file

</success_criteria>
