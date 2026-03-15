---
name: understanding
description: |
  Foundation skill — loaded automatically before any other spec-tree skill.
  When the user asks about methodology, node types, or how specs work, invoke this first.
allowed-tools: Read, Glob, Grep
---

<objective>

Load the Spec Tree methodology into the conversation so all subsequent skills operate from a shared foundation. This is a foundation skill — it loads once and emits a marker that other skills check before starting work.

</objective>

<quick_start>

Invoke `/understanding` before any spec-tree work. The skill reads 6 reference files and emits a `<SPEC_TREE_FOUNDATION>` marker. If the marker is already present in the conversation, skip.

</quick_start>

<principles>

1. **FOUNDATION, NOT CONTEXT** — This skill loads methodology; it does not load project-specific artifacts. Use `/contextualizing` for target-specific context injection.
2. **LOAD ONCE** — Check for `<SPEC_TREE_FOUNDATION>` marker before loading. If present, skip.
3. **SPECS ARE PERMANENT** — The Spec Tree is a durable map. Nothing moves, nothing closes. Read `references/durable-map.md`.
4. **TWO NODE TYPES** — Enablers (infrastructure) and outcomes (hypothesis + assertions). No other node types exist. Read `references/node-types.md`.
5. **ASSERTIONS SPECIFY OUTPUT** — Assertions specify what the software does, locally verifiable by automated tests or agent review. The lock file binds spec to evidence.
6. **DETERMINISTIC CONTEXT** — The tree structure defines what context an agent receives. No keyword search, no heuristics. This is handled by `/contextualizing`.
7. **ATEMPORAL VOICE** — Specs state product truth. Never narrate history. Flag temporal language as a quality issue.

</principles>

<workflow>

1. Check conversation for `<SPEC_TREE_FOUNDATION>` marker. If present, skip — already loaded.
2. Read all reference files:
   - `references/durable-map.md` — specs as permanent truth, atemporal voice
   - `references/node-types.md` — enabler vs outcome, spec format, lock files
   - `references/assertion-types.md` — scenario, mapping, conformance, property, compliance
   - `references/decomposition-semantics.md` — when to nest, depth heuristics
   - `references/ordering-rules.md` — sparse integer ordering, dependency encoding
   - `references/what-goes-where.md` — ADR/PDR/spec/test content taxonomy
3. Note template locations (do not read content unless authoring):
   - `templates/product/product-name.product.md`
   - `templates/decisions/decision-name.adr.md`
   - `templates/decisions/decision-name.pdr.md`
   - `templates/nodes/enabler-name.md`
   - `templates/nodes/outcome-name.md`
4. Emit the `<SPEC_TREE_FOUNDATION>` marker:

```text
<SPEC_TREE_FOUNDATION>
Loaded: durable-map, node-types, assertion-types, decomposition-semantics, ordering-rules, what-goes-where
Templates available: product, adr, pdr, enabler, outcome
</SPEC_TREE_FOUNDATION>
```

</workflow>

<success_criteria>

- [ ] All six reference files read and understood
- [ ] Template locations known (not content — read templates only when authoring)
- [ ] `<SPEC_TREE_FOUNDATION>` marker emitted
- [ ] Can explain: What is an enabler? What is an outcome? When to use each?
- [ ] Can explain: What does atemporal voice mean? Why does it matter?
- [ ] Can explain: How does sparse integer ordering encode dependencies?
- [ ] Can explain: What does a lock file prove? What doesn't it prove?

</success_criteria>
