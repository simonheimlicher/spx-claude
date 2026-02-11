# Outcome Types

Four structured outcome types with distinct quantifiers, notations, and test strategies.

## The Four Types

| Type            | Quantifier                    | Notation                        | Test Strategy               |
| --------------- | ----------------------------- | ------------------------------- | --------------------------- |
| **Scenario**    | ∃ (this case works)           | Gherkin                         | Example-based               |
| **Mapping**     | ∀ over finite, enumerable set | Table                           | Parameterized               |
| **Conformance** | External oracle               | Statement + reference           | Tool validation             |
| **Property**    | ∀ over type/value space       | Universal statement + predicate | Property-based (Hypothesis) |

---

## Scenario

**Quantifier:** ∃ — "this specific case works."

**When to use:** Verifying that a specific interaction or workflow produces the expected result. Most outcomes are Scenarios.

**Notation:**

```gherkin
GIVEN [precondition]
WHEN [action]
THEN [expected result]
AND [additional assertion]
```

**Test strategy:** Example-based tests. Each scenario maps to one or more test cases with concrete inputs and expected outputs.

**Connection to governance:** Scenarios implement specific behaviors that ADRs/PDRs constrain. The ADR says "do it this way"; the Scenario proves one case works that way.

**Example:**

```gherkin
GIVEN an SPI master configured for mode 0 (CPOL=0, CPHA=0)
WHEN data 0xAB is transmitted
THEN SCLK idles low before transmission
AND data is sampled on rising edge
AND MOSI outputs 0xAB MSB-first
```

---

## Mapping

**Quantifier:** ∀ over a finite, enumerable set — "every member of this set maps correctly."

**When to use:** When a finite, known set of inputs must each produce a specific output. The set is small enough to enumerate completely.

**Notation:**

| Input      | Expected Output |
| ---------- | --------------- |
| [member 1] | [result 1]      |
| [member 2] | [result 2]      |
| ...        | ...             |

**Test strategy:** Parameterized tests. Generate one test case per table row. All rows must pass.

**Connection to governance:** PDRs often state invariants over finite sets ("every IR expression type must produce valid Verilog"). A Mapping outcome enumerates the set and verifies each member.

**The critical distinction from Property:** The domain is finite and known. You can list every member. No sampling needed.

**Example:**

> Every IR expression type maps to valid Verilog.

| IR Expression | Expected Verilog |
| ------------- | ---------------- |
| `Add(a, b)`   | `a + b`          |
| `Sub(a, b)`   | `a - b`          |
| `Mul(a, b)`   | `a * b`          |
| `Neg(a)`      | `-a`             |
| `Shl(a, n)`   | `a << n`         |

---

## Conformance

**Quantifier:** External oracle — "output satisfies an external standard or tool."

**When to use:** When correctness is defined by an external reference (a standard, a tool, a validator) rather than by enumerable input/output pairs.

**Notation:**

**Conforms to:** [standard or tool reference]

**Predicate:** [what the oracle checks]

**Test strategy:** Tool validation. Run the external tool or validator against the output and assert it passes.

**Connection to governance:** ADRs may mandate compliance with external standards. A Conformance outcome proves the output passes the external oracle.

**Example:**

> Generated HDL passes Verilator lint.

**Conforms to:** Verilator 5.x `--lint-only`

**Predicate:** Zero warnings, zero errors

---

## Property

**Quantifier:** ∀ over a type or value space — "for all values in this domain, the predicate holds."

**When to use:** When the domain is infinite or too large to enumerate. The invariant must hold for any valid input, not just specific examples.

**Notation:**

**Property:** For all [variable] ∈ [domain], [predicate holds].

**Domain:** [description of the value space]

**Test strategy:** Property-based testing (Hypothesis in Python, fast-check in TypeScript). The framework generates random inputs from the domain and verifies the predicate. Falsification disproves the property; surviving many samples builds confidence.

**Connection to governance:** This is how PDR invariants become executable. A PDR states "arithmetic operations always return int regardless of signal type." A Property outcome expresses that universal claim. Hypothesis tests it by sampling.

**The PDR→executable loop:**

```
PDR states invariant → Story outcome expresses as Property → Hypothesis test proves by sampling
```

**The critical distinction from Mapping:** The domain is infinite (or impractically large). You cannot list every member. You sample and rely on the testing framework to find counterexamples.

**Example:**

> For all signal types and widths, arithmetic operations return int.

**Property:** For all `(signal_type, width, op)` ∈ `SignalType × [1..64] × ArithOp`, `op(signal_type(width), signal_type(width)).type == int`.

**Domain:** `SignalType × Width × ArithOp`

**Test:** `@given(signal_types(), widths(), arith_ops())`

---

## Mapping vs Property: When to Use Which

| Factor       | Mapping                                 | Property                                          |
| ------------ | --------------------------------------- | ------------------------------------------------- |
| Domain size  | Finite, enumerable                      | Infinite or impractically large                   |
| Completeness | Exhaustive (every member checked)       | Statistical (samples checked)                     |
| Test type    | Parameterized                           | Property-based (Hypothesis/fast-check)            |
| Example      | "5 IR expression types → valid Verilog" | "All signal type × width × op combinations → int" |
| Failure mode | Missing row in table                    | Counterexample found by framework                 |

**Rule of thumb:** If you can write every row in a table and the table has a clear, finite boundary — use Mapping. If the domain is parameterized by types, ranges, or combinations that make exhaustive listing impractical — use Property.

---

## Choosing the Right Type

```
Q: Is the outcome about a specific interaction or workflow?
YES → Scenario

Q: Is correctness defined by an external tool or standard?
YES → Conformance

Q: Can I enumerate every input that must work?
YES → Mapping

Q: Must the invariant hold for all values in a large/infinite domain?
YES → Property
```

---

## Using Types in Specs

Outcome type appears in the outcome heading:

```markdown
### 1. SPI master transmits mode 0 data (Scenario)

### 2. All IR expression types produce valid Verilog (Mapping)

### 3. Generated HDL passes lint (Conformance)

### 4. Arithmetic always returns int (Property)
```

The type annotation tells agents which test strategy to apply and which quantifier the outcome claims.
