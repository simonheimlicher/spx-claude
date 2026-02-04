---
name: reviewing-python-tests
description: Review Python tests for evidentiary value and spec compliance. Use when reviewing capabilities, features, or stories and their tests in Python.
allowed-tools: Read, Bash, Glob, Grep
---

<objective>
Determine if tests provide genuine evidence that outcomes are fulfilled through adversarial review. Reject tests that can pass while outcomes remain unfulfilled.

**THE ADVERSARIAL QUESTION:**

> How could these tests pass while the outcome remains unfulfilled?

If you can answer that question, the tests are **REJECTED**.
</objective>

<quick_start>
**PREREQUISITE**: Reference these skills when reporting findings:

- `/testing` - Methodology (5 stages, 5 factors, 7 exceptions)
- `/standardizing-python-testing` - Python testing standards

Review protocol has 6 phases - stop at first rejection:

1. **Spec structure validation** - Gherkin format, test links exist, level appropriateness
2. **Evidentiary integrity** - Adversarial test, dependency handling, harness verification
3. **Property-based testing** - MANDATORY for parsers, serializers, math, complex algorithms
4. **Lower-level assumptions** - Check for stories/features, evaluate coverage
5. **ADR compliance** - Identify applicable ADRs, verify constraints
6. **Test quality** - Type annotations, no mocking, magic values

When reporting findings, cite source skills:

- "Per /testing Stage 2 Factor 2, database dependency requires Level 2"
- "Per /standardizing-python-testing, parsers MUST have property-based tests"

Use bash commands to verify:

```bash
# Check test file links exist
ls -la {container}/tests/{linked_file}

# Find silent skips (REJECT if on required deps)
grep -rn "pytest.mark.skipif" {test_dir}

# Find mocking (any = REJECT)
grep -rn "@patch\|Mock()\|MagicMock" {test_dir}

# Find property-based tests (REQUIRED for parsers/serializers)
grep -rn "@given\|hypothesis" {test_dir}
```

</quick_start>

<verdict>
There is no middle ground. No "mostly good." No "acceptable with caveats."

- **APPROVE**: Tests provide genuine evidence for all outcomes at appropriate levels
- **REJECT**: Any deficiency, missing link, silent skip, or evidentiary gap

A missing comma is REJECT. A philosophical disagreement about test structure is REJECT. If it's not APPROVE, it's REJECT.
</verdict>

<context>
This skill protects the outcome ledger from phantom evidence. A single evidentiary gap means CI can go green while promised outcomes remain unfulfilled. The cost of false approval is infinite; the cost of false rejection is rework.
</context>

<review_protocol>
Execute these phases IN ORDER. Stop at first REJECT.

<phase name="spec_structure_validation">
For each outcome in the spec, verify:

**1.1 Outcome Format**

Outcomes MUST be Gherkin only. No code in specs.

````markdown
<!-- ✅ CORRECT: Gherkin only -->

### 1. UART transmitter sends byte correctly

```gherkin
GIVEN a UartTx component configured for 8N1 at 115200 baud
WHEN a byte 0x55 is written to the input stream
THEN the TX line outputs start bit, 8 data bits (LSB first), and stop bit
AND the component signals busy during transmission
```
````

<!-- ❌ REJECT: Code in spec -->

### 1. UART transmitter sends byte correctly

```python
def test_uart_tx():
    uart = UartTx(domain, data_bits=8)
    ...
```

````
**If spec contains code examples**: REJECT. Specs are durable; code drifts.

**1.2 Test File Linkage**

Each outcome MUST have a Test Files table with valid Markdown links:

```markdown
| File                                            | Level | Harness |
| ----------------------------------------------- | ----- | ------- |
| [test_uart_tx.level_1](tests/test_uart_tx.level_1.py) | 1     | -       |
````

**Check:**

1. Link syntax is valid Markdown: `[display](path)`
2. Linked file EXISTS at specified path
3. Level matches filename suffix (`.level_1.py` = Level 1, `.level_2.py` = Level 2, `.level_3.py` = Level 3)

```bash
# Verify linked files exist
ls -la {container}/tests/{linked_file}
```

**If link is broken or file missing**: REJECT.

**1.3 Level Appropriateness**

Evidence lives at specific levels. Verify each outcome is tested at the correct level:

| Evidence Type              | Minimum Level | Example                                  |
| -------------------------- | ------------- | ---------------------------------------- |
| Pure computation/algorithm | 1             | Protocol timing, math correctness        |
| Component interaction      | 2             | TX→RX loopback, multi-entity simulation  |
| Project-specific binary    | 2             | Verilator lint, external tool invocation |
| Real credentials/services  | 3             | Cloud APIs, payment providers            |

**If outcome is tested at wrong level**: REJECT.

**If story-level outcome appears in feature spec**: Note as structural issue (stories should be created), but continue review.

**GATE 1**: Before proceeding to Phase 2, verify:

- [ ] All outcomes use Gherkin format (no code in specs)
- [ ] All test file links are valid markdown AND files exist (ran `ls` for each)
- [ ] All outcomes tested at appropriate level

If any check fails, STOP and REJECT with detailed findings.
</phase>

<phase name="evidentiary_integrity">
For each test file, verify it provides genuine evidence.

**2.1 The Adversarial Test**

Ask: **How could this test pass while the outcome remains unfulfilled?**

| Scenario                                                 | Verdict |
| -------------------------------------------------------- | ------- |
| Test asserts something other than what outcome specifies | REJECT  |
| Test uses hardcoded values that happen to match          | REJECT  |
| Test doesn't actually exercise the code path             | REJECT  |
| Test mocks the thing it's supposed to verify             | REJECT  |
| Test can pass with broken implementation                 | REJECT  |

**2.2 Dependency Availability**

**CRITICAL: Missing dependencies MUST FAIL, not skip.**

Search for silent skip patterns:

```bash
# Find skipif patterns
grep -rn "pytest.mark.skipif" {test_dir}
grep -rn "pytest.skip" {test_dir}
```

**Evaluate each skip:**

| Pattern                                                 | Verdict                                          |
| ------------------------------------------------------- | ------------------------------------------------ |
| `skipif(not verilator_available())` on HDL tests        | **REJECT** - Required dependency must fail       |
| `skipif(not HAS_HYPOTHESIS)` on property tests          | **REJECT** - Test infrastructure must be present |
| `skipif(sys.platform != "linux")` for platform-specific | REVIEW - May be legitimate                       |
| `skipif(os.environ.get("CI"))`                          | REVIEW - What is being skipped?                  |

**The Silent Skip Problem:**

```python
# ❌ REJECT: This allows CI to go green with zero HDL verification
@pytest.mark.skipif(not verilator_available(), reason="Verilator not available")
def test_generates_lint_clean_verilog(): ...


# ✅ CORRECT: Fail loudly if required dependency missing
def test_generates_lint_clean_verilog():
    if not verilator_available():
        pytest.fail("Verilator is REQUIRED. Install with: brew install verilator")
    ...
```

**If tests silently skip on required dependencies**: REJECT. This is evidentiary fraud - CI goes green while providing zero verification.

**2.3 Harness Verification**

If outcome specifies a harness in the Test Files table:

1. Harness must exist and be specified (in `spx/` or `{project}_testing/`)
2. Harness must have its own tests
3. Harness failures must cause test failures, not skips

```bash
# Check if harness is specified and exists
grep -r "Harness" {container}/*.md
ls -la {harness_path}
```

**If harness is referenced but doesn't exist or isn't tested**: REJECT.

**GATE 2**: Before proceeding to Phase 3, verify:

- [ ] Each test file reviewed for adversarial test (can it pass while outcome fails?)
- [ ] Ran grep for skipif patterns, evaluated each found
- [ ] Any harnesses referenced have been verified to exist

If any check fails, STOP and REJECT with detailed findings.
</phase>

<phase name="property_based_testing">
Per `/testing` and `/standardizing-python-testing`, property-based testing is **MANDATORY** for:

| Code Type               | Required Property        | Hypothesis Pattern        |
| ----------------------- | ------------------------ | ------------------------- |
| Parsers                 | `parse(format(x)) == x`  | `@given(st.text())`       |
| Serialization           | `decode(encode(x)) == x` | `@given(valid_objects())` |
| Mathematical operations | Algebraic properties     | `@given(st.integers())`   |
| Complex algorithms      | Invariant preservation   | `@given(valid_inputs())`  |

**3.1 Identify Applicable Code Types**

```bash
# Find parsers and serializers
grep -rn "def parse\|def encode\|def decode\|def serialize\|def deserialize" {src_dir}

# Check if tests have property-based coverage
grep -rn "@given\|from hypothesis" {test_dir}
```

**3.2 Evaluate Coverage**

For each identified parser/serializer/math operation:

| Found                                      | Verdict    |
| ------------------------------------------ | ---------- |
| `@given` decorator with roundtrip property | ✓ PASS     |
| Only example-based tests                   | **REJECT** |
| No tests at all                            | **REJECT** |

**Example rejection:**

```python
# ❌ REJECT: Parser with only example-based tests
def test_parse_json_simple() -> None:
    result = parse('{"key": "value"}')
    assert result == {"key": "value"}


# Missing: @given(st.text()) + roundtrip property
```

**3.3 Verify Property Quality**

Property tests must test meaningful properties, not just "doesn't crash":

```python
# ❌ REJECT: Trivial property (only tests "doesn't crash")
@given(st.text())
def test_parse_doesnt_crash(text: str) -> None:
    try:
        parse(text)
    except ParseError:
        pass  # Expected for invalid input


# ✅ ACCEPT: Meaningful roundtrip property
@given(valid_json_values())
def test_roundtrip(value: JsonValue) -> None:
    assert parse(format(value)) == value
```

**GATE 3**: Before proceeding to Phase 4, verify:

- [ ] Identified all parsers, serializers, math operations, complex algorithms in code under test
- [ ] Ran grep for `@given` patterns in test files
- [ ] Each applicable code type has property-based tests with meaningful properties

If any check fails, STOP and REJECT with detailed findings citing `/standardizing-python-testing`.
</phase>

<phase name="lower_level_assumptions">
Features assume stories have tested what can be tested at story level. Capabilities assume features have done their job.

**4.1 Check for Lower-Level Specs**

```bash
# For a feature, check if stories exist
ls -d {feature_path}/*-*.story/ 2>/dev/null

# For a capability, check if features exist
ls -d {capability_path}/*-*.feature/ 2>/dev/null
```

**4.2 Evaluate Assumptions**

| Scenario                              | Action                                                            |
| ------------------------------------- | ----------------------------------------------------------------- |
| Lower-level specs exist with tests    | Verify assumptions align                                          |
| Lower-level specs exist without tests | Note gap, continue review                                         |
| Lower-level specs don't exist         | Note structural issue, evaluate if tests are appropriately coarse |

**Key principle**: Specs are DURABLE. They DEMAND outcomes. A spec must NEVER say "stories are pending" or "tests will be added later." If lower-level decomposition is needed, those specs should exist.

**If spec contains language about missing/pending specs**: REJECT. Specs are not working documents.

**4.3 Integration Test Assumptions**

For integration tests (Level 2), verify they don't duplicate story-level evidence:

| Integration Test Should    | Integration Test Should NOT              |
| -------------------------- | ---------------------------------------- |
| Verify component contracts | Re-test algorithm correctness            |
| Verify interoperation      | Exhaustively test edge cases             |
| Assume story tests passed  | Provide coarse coverage of unit concerns |

**If integration tests are doing story-level work because stories don't exist**: Note as structural issue. Tests may be legitimately coarse in transitional state, but this should be flagged.

**GATE 4**: Before proceeding to Phase 5, verify:

- [ ] Checked for lower-level specs (stories within features, features within capabilities)
- [ ] No "pending" or "will be added" language in spec
- [ ] Integration tests are not duplicating unit-level work

If any check fails, STOP and REJECT with detailed findings.
</phase>

<phase name="adr_compliance">
Check test code against architectural decisions.

**5.1 Identify Applicable ADRs**

```bash
# Find ADRs referenced in spec
grep -o '\[.*\](.*\.adr\.md)' {spec_file}

# Find ADRs in ancestry
ls {capability_path}/*.adr.md
ls {feature_path}/*.adr.md 2>/dev/null
```

**5.2 Verify Compliance**

For each ADR, check test code follows its constraints:

| ADR Constraint                              | What to Check                 |
| ------------------------------------------- | ----------------------------- |
| "Use `int(signal)` not `_raw()`"            | `grep "_raw()" {test_files}`  |
| "Use `yield`, `yield n`, not string states" | `grep 'yield "' {test_files}` |
| "No mocking"                                | `grep -E "@patch              |

**If tests violate ADR constraints**: REJECT.

**GATE 5**: Before proceeding to Phase 6, verify:

- [ ] Identified all applicable ADRs (spec references + ancestry)
- [ ] Ran grep for each ADR constraint against test files
- [ ] No ADR violations found

If any check fails, STOP and REJECT with detailed findings.
</phase>

<phase name="test_quality">
Verify tests follow Python testing patterns per `/standardizing-python-testing`.

**6.1 Type Annotations**

```bash
# Find test functions missing return type
grep -rn "def test_" {test_dir} | grep -v "-> None"
```

**All test functions MUST have `-> None` return type annotation.**

**6.2 No Mocking**

```bash
# Find mocking patterns
grep -rn "@patch\|Mock()\|MagicMock\|mocker\." {test_dir}
```

**Any mocking = REJECT.** Use dependency injection instead.

**6.3 Magic Values**

```bash
# Find assertions with magic numbers (PLR2004)
grep -rn "assert.*==" {test_dir} | grep -E "[0-9]+"
```

**Magic values in assertions should use named constants.**

**6.4 Test Organization**

- [ ] Test class/function names describe the scenario
- [ ] Assertions verify outcomes, not implementation
- [ ] Fixtures clean up after themselves
- [ ] No hardcoded paths or environment-specific values

**GATE 6 (FINAL)**: Before issuing verdict, verify:

- [ ] All test functions have `-> None` return type
- [ ] No mocking patterns found
- [ ] Magic values use named constants (or are self-documenting)
- [ ] Test organization checklist passes

If all gates passed, issue APPROVE. Otherwise, REJECT with detailed findings.
</phase>

</review_protocol>

<failure_modes>
Failures from actual usage:

**Failure 1: Approved tests with silent skips**

- What happened: Agent saw pytest output with all tests passing, approved
- Why it failed: Tests had `@pytest.mark.skipif` decorators for required dependencies - CI went green with 0 HDL verification
- How to avoid: ALWAYS run grep for skipif patterns in Phase 2.2. Any skipif on a required dependency (verilator, hypothesis, etc.) is automatic REJECT

**Failure 2: Missed broken test links**

- What happened: Agent checked link syntax but didn't verify files exist
- Why it failed: Spec had `[test_foo.level_1](tests/test_foo.level_1.py)` but file was actually named `test_foo_level_1.py`
- How to avoid: Run `ls -la {container}/tests/{file}` for EVERY linked file in Phase 1.2. Don't trust link syntax alone.

**Failure 3: Approved tests that mocked the SUT**

- What happened: Agent searched for `@patch` but tests used `MagicMock()` inline
- Why it failed: Grep pattern didn't catch all mocking variants
- How to avoid: Use complete grep pattern: `grep -rn "@patch\|Mock()\|MagicMock\|mocker\." {test_dir}`

**Failure 4: Missed ADR constraint violation**

- What happened: Agent found ADRs but didn't systematically check each constraint
- Why it failed: ADR said "use `int(signal)` not `_raw()`" but tests used `_raw()` directly
- How to avoid: For EACH ADR constraint, write and run a grep command. Document what you searched for.

**Failure 5: Compared coverage at wrong granularity**

- What happened: Agent saw 39% coverage for one story and flagged as insufficient
- Why it failed: Multiple stories share one implementation file; per-story coverage is meaningless
- How to avoid: Always compare coverage at the implementation file level, not story level

**Failure 6: Approved parser without property-based tests**

- What happened: Agent saw comprehensive example-based tests for a JSON parser, approved
- Why it failed: Example tests don't catch edge cases that property-based tests would find (Unicode, escaping, deeply nested structures)
- How to avoid: Per `/standardizing-python-testing`, parsers MUST have property-based tests. Run `grep -rn "@given" {test_dir}` and verify roundtrip properties exist.

</failure_modes>

<concrete_examples>
**Example 1: APPROVE verdict**

Reviewing `spx/01-uart/03-transmitter.story/`

Phase 1 checks:

````bash
$ grep -A 10 "^### 1\." transmitter.story.md
### 1. UART transmitter sends byte correctly

```gherkin
GIVEN a UartTx component configured for 8N1 at 115200 baud
WHEN a byte 0x55 is written to the input stream
THEN the TX line outputs start bit, 8 data bits (LSB first), and stop bit
````

$ ls -la tests/test_uart_tx.level_1.py
-rw-r--r-- 1 user group 2847 Jan 15 10:23 tests/test_uart_tx.level_1.py
✓ File exists, Level 1 matches .level_1.py suffix

````
Phase 2 checks:

```bash
$ grep -rn "pytest.mark.skipif" tests/
(no results)
✓ No silent skips

$ grep -rn "@patch\|Mock()\|MagicMock" tests/
(no results)
✓ No mocking
````

Phase 5 checks:

```bash
$ grep -rn "def test_" tests/ | grep -v "-> None"
(no results)
✓ All test functions have -> None
```

**Verdict: APPROVE** - All outcomes have genuine evidentiary coverage at appropriate levels.

---

**Example 2: REJECT verdict**

Reviewing `spx/02-hdl/08-verilog-gen.story/`

Phase 2.2 finds silent skip:

```bash
$ grep -rn "pytest.mark.skipif" tests/
tests/test_verilog_gen.level_1.py:15:@pytest.mark.skipif(not verilator_available(), reason="Verilator not available")
```

**Verdict: REJECT**

| # | Category    | Location                       | Issue                         | Required Fix                                     |
| - | ----------- | ------------------------------ | ----------------------------- | ------------------------------------------------ |
| 1 | Silent Skip | test_verilog_gen.level_1.py:15 | skipif on required dependency | Change to pytest.fail() if verilator unavailable |

**How Tests Could Pass While Outcome Fails:**

CI environment doesn't have Verilator installed. Test is silently skipped. CI goes green. The outcome "generates lint-clean Verilog" has zero verification. Users deploy code that produces invalid Verilog.

</concrete_examples>

<output_format>
<approve_template>

```markdown
## Test Review: {container_path}

### Verdict: APPROVE

All outcomes have genuine evidentiary coverage at appropriate levels.

### Outcomes Verified

| # | Outcome | Level | Test File | Evidence Quality |
| - | ------- | ----- | --------- | ---------------- |
| 1 | {name}  | {N}   | {file}    | Genuine          |

### ADR Compliance

| ADR    | Status    |
| ------ | --------- |
| {name} | Compliant |

### Notes

{Any observations about test quality, structural issues for future work, etc.}
```

</approve_template>

<reject_template>

```markdown
## Test Review: {container_path}

### Verdict: REJECT

{One-sentence summary of primary rejection reason}

### Rejection Reasons

| # | Category | Location    | Issue   | Required Fix |
| - | -------- | ----------- | ------- | ------------ |
| 1 | {cat}    | {file:line} | {issue} | {fix}        |

### Detailed Findings

#### {Category}: {Issue Title}

**Location**: `{file}:{line}`

**Problem**: {Detailed explanation of why this is a rejection}

**Evidence**:
```

{Code snippet or grep output showing the issue}

```
**Required Fix**: {Specific action to resolve}

---

### How Tests Could Pass While Outcome Fails

{Explain the evidentiary gap - how could these tests go green while the promised outcome remains unfulfilled?}
```

</reject_template>

</output_format>

<rejection_triggers>
Quick reference for common rejection triggers:

| Category           | Trigger                                    | Verdict |
| ------------------ | ------------------------------------------ | ------- |
| **Spec Structure** | Code examples in spec                      | REJECT  |
| **Spec Structure** | Missing or broken test file links          | REJECT  |
| **Spec Structure** | Language about "pending" specs             | REJECT  |
| **Level**          | Outcome tested at wrong level              | REJECT  |
| **Dependencies**   | `skipif` on required dependency            | REJECT  |
| **Dependencies**   | Harness referenced but missing             | REJECT  |
| **Property-Based** | Parser without `@given` roundtrip test     | REJECT  |
| **Property-Based** | Serializer without `@given` roundtrip test | REJECT  |
| **Property-Based** | Math operation without property tests      | REJECT  |
| **ADR**            | Test violates ADR constraint               | REJECT  |
| **Python**         | Missing `-> None` on test                  | REJECT  |
| **Python**         | Mocking (`@patch`, `Mock()`)               | REJECT  |
| **Evidentiary**    | Test can pass with broken impl             | REJECT  |

</rejection_triggers>

<success_criteria>
Task is complete when:

- [ ] Verdict is APPROVE or REJECT (no middle ground)
- [ ] All 6 phases executed in order (or stopped at first REJECT)
- [ ] All gates passed (or documented why gate failed)
- [ ] Property-based test coverage verified for parsers/serializers/math/algorithms
- [ ] Each rejection reason has file:line location
- [ ] Evidentiary gap explained (how tests could pass while outcome fails)
- [ ] Output follows specified format (APPROVE or REJECT template)

**Verification command**:

```bash
# Your output should contain exactly one of these:
grep -c "### Verdict: APPROVE" review_output.md  # Should be 1 for approve
grep -c "### Verdict: REJECT" review_output.md   # Should be 1 for reject
```

</success_criteria>

<cardinal_rule>
**If you can explain how the tests could pass while the outcome remains unfulfilled, the tests are REJECTED.**

Your job is to protect the outcome ledger from phantom evidence. A rejected review that catches an evidentiary gap is worth infinitely more than an approval that lets one through.
</cardinal_rule>
