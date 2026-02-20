1. Changes before publishing

Must-fix conceptual mismatches

1. “Outcome” is overloaded in a way that product people will challenge
   You open with outcomes (behavior change), then define an “outcome node” as a customer‑facing capability, and your node header example is:

## Outcome

“Users can log in with their email and password.”

That’s an output/capability, not an outcome hypothesis. This will reliably trigger: “You’re using the term outcome incorrectly.”

Two clean options:

- Option A: keep .outcome directories, but make ## Outcome a real outcome hypothesis.
  Require that ## Outcome includes a belief + metric (even if rough), then put the functional behavior under a different heading:
- ## Outcome hypothesis (belief + metric)
- ## Behaviors or ## Contract (assertions)
- Option B: rename the node type and section heading to match what it actually is.
  Use .capability (or .surface / .feature) and ## Capability / ## Intent. Keep “outcome hypotheses” as the higher-level structure driver.

Right now you’re half-way between the two; you should commit to one to avoid getting derailed in comments.

⸻

1. “Spec-as-source” in the title will be read as “code is derived from spec”
   You cite Böckeler’s taxonomy where “spec-as-source” often implies the spec is the source of truth from which code is derived. Your post is not about spec-to-code generation; it’s about specs as durable truth + tests as evidence + locks + deterministic context.

If you keep “spec-as-source,” make it unambiguous:

- Title tweak: “Spec-as-source-of-truth” (adds one phrase that prevents the wrong interpretation), or
- Swap to a more precise tagline:
  “A Git-native spec + lock + context structure for human–agent collaboration.”

This is a positioning risk: you don’t want readers arguing about taxonomy rather than your mechanism.

⸻

1. Your lock file semantics are now coherent for spec/test drift, but ambiguous for “validation is current”
   You correctly tightened the claim:

“A mismatch … does not mean behavior changed — it means the evidence is stale.”

Good. But elsewhere you still say:

“tracking … what has been validated and whether that validation is still current.”

With your current schema, “current” really means:

- spec hasn’t changed since sealing, and
- test files haven’t changed since sealing.

It does not mean “tests still pass on the current implementation” unless you also invalidate on implementation changes (or you define “current” more narrowly).

Before publishing, pick one definition and stick to it in the wording:

- Definition 1: lock = currency of evidence for current spec+tests
  (does not claim anything about current code until tests are run)
- Rename statuses and phrasing accordingly (“sealed” / “stale” / “unsealed”), avoid “valid” implying correctness.
- Make sure the CLI output language matches that meaning.
- Definition 2: lock = seal for spec+tests+relevant implementation
  (stronger, but requires a mechanism)
- Add an “implementation digest” per node (details below).

If you want the strongest version of “validation is current,” you need Definition 2.

⸻

Strong improvements that would materially increase rigor

1. Add a minimal implementation digest or narrow the claims
   Right now, spx verify can say a node is “valid” even if code changes broke the tests, because code changes don’t affect any hashes in the lock. That’s a real practical hole if people start trusting spx verify as a quick gate.

If you want to keep spx verify cheap and still meaningful, you need one of these:

- Implementation digest via declared owned paths
  In the spec, add:

### Owns

- ../../src/auth/login/*
- ../../src/auth/session/*

Then include a tree digest for those paths in the lock:

impl:

- path: ../../src/auth/login/
  tree: 7a9c...

This keeps invalidation scoped.

    - Colocate implementation under the node

Add src/ (or impl/) inside each node directory and hash the node tree excluding spx-lock.yaml. This is cleanest mechanically, but it’s a stronger repo-structure bet.

- If you don’t want either, change the language
  Keep the lock purely about “spec/test evidence currency,” and stop implying it tracks validation “state” of the running product without running tests.

⸻

1. Make the lock schema self-documenting
   You still use:

blob: f4a1b2c

That forces readers to infer it’s the spec blob. Rename it:

- spec_blob instead of blob
- Keep schema (good)
- Consider tests[].blob but call out that these are test file blobs.

This is small but removes friction and nitpicking.

⸻

1. Decide whether you need assertion IDs
   You now link each assertion to its test file. That is already a form of traceability. But you also claim “assertions are first-class, test-addressable objects.”

If you want that to be mechanically true (and future-proof), add IDs:

- LOGIN-001: Valid credentials return a session token ([test](tests/login.unit.test.ts))

Then later parts can evolve toward:

- coverage checks (spx lint can ensure every ID is referenced by at least one test)
- per-assertion status

If you don’t add IDs, tone down “first-class objects” and say “explicitly traceable.”

⸻

1. Clarify indexing semantics precisely, once
   You say:

“decisions at lower indices constrain everything above them”

This will confuse. Replace with a precise rule and include the scope boundary:

- “Within a directory, decision records with lower numeric prefixes constrain all sibling nodes/directories with higher prefixes (and their descendants). Indexing is local to each directory.”

Also clarify retroactivity:

- Does adding a new ADR later retroactively constrain existing nodes? (Your rule implies yes.) Say it plainly.

⸻

Naming and small wording fixes

1. product.product.md looks accidental
   The duplication will distract. If this is a deliberate “type suffix” pattern, pick a cleaner form:
   - product.md with type: product in front matter, or
   - product.hypothesis.md, or
   - 00-product.product/ containing product.md.

The content is strong; don’t make readers trip over the filename.

⸻

1. “Enabler” scope sentence is slightly imprecise
   You wrote:

“a nested enabler serves only its siblings within the same subtree”

“Siblings” and “subtree” don’t go together. If you mean “serves the subtree rooted at the enabler’s parent directory,” say that:

- “A nested enabler serves outcomes within its parent subtree unless explicitly depended on elsewhere.”

⸻

1. The “building the tree” example has minor inconsistencies
   - 22-verify-password.outcome/ lacks spx-lock.yaml in the earlier tree snippet.
   - In the operational loop output, 22-login.outcome/ appears twice; visually it reads like two separate nodes.

Tighten these so a careful reader doesn’t wonder whether the format is inconsistent or the CLI is.

⸻

1. Strongest concerns and counterarguments, and how to preempt them

Concern 1: “You’re calling outputs ‘outcomes’”

Likely critique: outcome-oriented product people will say the .outcome naming and ## Outcome header are incorrect.

Preemptive adjustment: adopt Option A or B above. If you keep .outcome, make ## Outcome hypothesis actually hypothesis + metric, and move behaviors under another heading.

⸻

Concern 2: “Lock files don’t prove tests pass on this code”

Likely critique: “Your spx verify is a hash check; it can say ‘valid’ even if tests fail. This is a false sense of correctness.”

Adjustments:

- Rename statuses: “sealed” (hashes match; evidence currency), “stale,” “unsealed.” Avoid “valid.”
- Add implementation digests (owned paths or colocated implementation) if you want “sealed implies still passing unless tests are flaky.”
- Add one sentence:
  “spx verify checks evidence currency; correctness still requires running tests, which spx lock enforces.”

⸻

Concern 3: “Deterministic context is too restrictive and chicken-and-egg”

Likely critique: “If an agent can’t search, how does it discover a missing dependency to declare?”

Preemptive adjustment: add an explicit escape hatch policy:

- “Search is allowed only in ‘explore mode’; outputs must be turned into explicit ### Depends on entries or decision records.”
- Optionally log/search provenance (even just as a note) to keep humans in the loop.

This preserves determinism as the default without being brittle.

⸻

Concern 4: “Decision scoping by index is under-specified and can become bureaucratic”

Likely critique: “What stops ADR/PDR sprawl? Do decisions apply retroactively? What about conflicting ADRs?”

Preemptive adjustment:

- Add a short “Decision record semantics” subsection:
- scope rule (siblings with higher index + descendants)
- conflict rule (newer ADR supersedes older; must link to superseded record)
- review rule (human approval required for ADR/PDR changes)

⸻

Concern 5: “Cross-cutting assertions in the lowest common ancestor will bloat specs”

Likely critique: “Your ancestor specs will become dumping grounds for integration behavior.”

Preemptive adjustment:

- Add a constraint:
  “Cross-cutting behaviors belong in the lowest common ancestor only when they are true invariants of the composed system. Otherwise create an explicit ‘flow’ node (integration outcome) that depends on both subtrees.”

This keeps the tree modular.

⸻

Concern 6: “This is too much overhead and product won’t live in Git”

Likely critique: “You’re turning discovery and product intent into repo artifacts.”

Preemptive adjustment: add an adoption paragraph:

- “Start with one subtree where drift is costly. Agents maintain lock and traceability; humans review only decision records and outcome hypotheses.”
- “Outcome hypotheses can be authored elsewhere; spx/ stores the operational version agents work against.”

⸻

Concern 7: “Flaky tests and environment differences invalidate the ‘seal of trust’ metaphor”

Likely critique: “Hashes don’t pin the runtime environment; tests can be non-deterministic.”

Preemptive adjustment: one sentence plus a boundary:

- “The seal assumes deterministic tests under a controlled runner; flakiness is treated as a system bug. Part 2 covers hermetic runners/toolchain pinning.”

If you want to be stronger: store a runner/toolchain fingerprint in the lock (optional).

⸻

1. Top 5 writing-style changes to sound more like you and less like generic framework prose
   1. Put the thesis in one blunt sentence near the top
      After the “drift is the default” paragraph, add:

   - “The Spec Tree is a Git-native structure that makes spec/test drift mechanically visible and makes agent context deterministic by default.”

This reduces the “slow reveal” feel.
2. Replace a few rhetorical lines with explicit invariants
You already started doing this (“The invariant: …”). Add 2–3 more, succinctly:

    - “Invariant: a node is sealed iff spx lock ran and the lock hashes match the current spec/tests (and owned implementation, if configured).”
    - “Invariant: injected context is a pure function of (node path, ancestor decisions, declared deps).”

This reads like an engineer, not a manifesto.
3. Avoid “exactly the context it needs” style absolutes
Change to operational defaults:

    - “By default, it receives the context it declared or inherited.”
    - “Search is not the baseline; it becomes an explicit step that must be turned into a dependency.”

    4. Trim metaphors to one per section

The culture/coffee-machine metaphor works. Don’t add more metaphorical framing later (“seal of trust,” “disposable plans,” etc.) unless it adds precision. Pick one metaphor per concept and keep the rest literal.
5. Use one short “failure case” paragraph as an anchor
One concrete incident-style paragraph (5–7 sentences) will make the post feel authored, not assembled:

    - “We changed X in the spec, tests still passed, we shipped Y…”

Then show how the lock would have made it visible.

You already have the operational loop; one real-ish failure story will give it a human voice and credibility.

⸻

If you change only three things before publishing

1. Resolve the “outcome vs capability” naming/heading mismatch.
2. Clarify lock semantics versus implementation changes (either add implementation digests or narrow claims + rename statuses).
3. Tighten decision/index scoping semantics with one explicit rule (local indexing, scope, retroactivity).
