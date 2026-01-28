# Recursive Decimal BSP Strategy

This approach solves the "Infinite Insertion" problem while leveraging standard filesystem sorting rules (ASCII) to keep parents and children ordered correctly without renaming files.

## 1. The Syntax

```text
{BSP}[@{SubBSP}...]-{slug}.{type}
```

- **{BSP}**: Two digits (10-99)
- **@**: The recursion delimiter (read as "at")
- **-**: The slug separator. **Critical:** Must be a hyphen, not an underscore
- **{slug}**: Human-readable name
- **{type}**: `.capability`, `.feature`, `.story`, or `.adr`

## 2. The ASCII Physics

Filesystems sort characters by ASCII code. To ensure the **Parent** (20) always appears before the **Child** (20@50), we rely on the fact that the **Hyphen (-)** has a lower ASCII value than the **At Symbol (@)**.

| Character      | ASCII | Sort Position |
| -------------- | ----- | ------------- |
| **- (Hyphen)** | 45    | 1 (Parent)    |
| **0-9**        | 48-57 | 2 (Siblings)  |
| **@ (At)**     | 64    | 3 (Children)  |
| _ (Underscore) | 95    | Avoid         |

**Resulting sort order:**

```text
20-auth.capability/      ← Parent (- sorts first)
20@50-recovery.feature/  ← Child (@ sorts after -)
21-billing.capability/   ← Sibling (2 > 0)
```

## 3. The Algorithms

### A. Append (new sibling)

**Goal:** Add item after `20-auth`.

```text
floor((20 + 99) / 2) = 59 → 59-billing.capability/
```

### B. Insert (between siblings)

**Goal:** Insert between `20-auth` and `30-billing`.

```text
floor((20 + 30) / 2) = 25 → 25-subscriptions.capability/
```

### C. Recursive Insert (no integer space)

**Goal:** Insert between `20-auth` and `21-logging` (no room).

**Strategy:** "Zoom in" on 20. Treat it as a new 10-99 space.

```text
20 + @ + floor((10 + 99) / 2) = 20@54 → 20@54-audit.capability/
```

### D. Deep Recursion

**Goal:** Insert between `20@54` and `20@55`.

```text
20@54@50-detailed-trace.story/
```

## 4. File Tree Example

A complex, heavily edited project. Note how the hierarchy remains logical despite insertions.

```text
spx/
├── product.prd.md
│
├── 10-bootstrap.adr.md                # Start here
│
├── 20-auth.capability/                # Parent capability
│   ├── auth.capability.md             # Spec file (slug.type.md)
│   │
│   ├── 10-login.feature/              # Feature 1
│   ├── 20-signup.feature/             # Feature 2
│   │
│   ├── 20@50-recovery.feature/        # ← INSERTED (between 20 & 21)
│   │   ├── recovery.feature.md
│   │   ├── 10-forgot-pass.story/
│   │   └── 20-reset-link.story/
│   │
│   └── 21-logout.feature/             # Feature 3
│
├── 20@50-security.adr.md              # ← ADR inserted after Auth capability
├── 20@90-legacy-migration.adr.md      # ← ADR inserted near end of Auth block
│
├── 21-billing.capability/             # Next major sibling
│   ├── billing.capability.md
│   └── ...
```

## 5. Rules

1. **Hyphens only**: Always `-` between BSP and slug, never `_`
2. **No renaming**: `20-auth` stays `20-auth` forever; use `@` to insert
3. **Rebalance at depth 3**: If you reach `20@50@50@...`, consider rebalancing
4. **Works for files and directories**: `20-auth.capability/` and `20-auth.adr.md` follow the same pattern

This is the most robust, collision-free, and tool-friendly strategy.
