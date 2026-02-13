# Fractional Indexing Strategy

This approach solves the "Infinite Insertion" problem while leveraging standard filesystem sorting rules (ASCII) to keep entries ordered correctly without renaming.

## 1. The Syntax

```text
{index}[.{subindex}...]-{slug}.{type}
```

- **{index}**: Two digits (10–99)
- **`.`**: Fractional level separator
- **`-`**: Index–slug boundary. **Critical:** Must be a hyphen, not an underscore
- **{slug}**: Human-readable name
- **{type}**: `.capability`, `.feature`, `.story`, or `.adr`

## 2. Sort Order

Filesystems sort characters by ASCII code. Hyphen (45) sorts before dot (46), so integer entries appear before their fractional insertions:

| Character      | ASCII | Role                |
| -------------- | ----- | ------------------- |
| **- (Hyphen)** | 45    | Index–slug boundary |
| **. (Dot)**    | 46    | Fractional level    |
| **0-9**        | 48-57 | Digits              |

**Resulting sort order:**

```text
20-auth.capability/       ← Integer index (- sorts first)
20.54-audit.capability/   ← Fractional insert (. sorts after -)
21-billing.capability/    ← Next integer (2 > .)
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

### C. Fractional Insert (no integer space)

**Goal:** Insert between `20-auth` and `21-logging` (no room).

**Strategy:** Descend into 20's fractional space (10–99).

```text
20.floor((10 + 99) / 2) = 20.54 → 20.54-audit.capability/
```

### D. Deep Fraction

**Goal:** Insert between `20.54` and `20.55`.

```text
20.54.50-detailed-trace.story/
```

## 4. File Tree Example

A complex, heavily edited project. Note how the hierarchy remains logical despite insertions.

```text
spx/
├── product.prd.md
│
├── 10-bootstrap.adr.md                # Start here
│
├── 20-auth.capability/                # Integer index
│   ├── auth.capability.md
│   │
│   ├── 10-login.feature/
│   ├── 20-signup.feature/
│   │
│   ├── 20.54-recovery.feature/        ← FRACTIONAL INSERT (between 20 & 21)
│   │   ├── recovery.feature.md
│   │   ├── 10-forgot-pass.story/
│   │   └── 20-reset-link.story/
│   │
│   └── 21-logout.feature/
│
├── 20.50-security.adr.md              ← ADR inserted after auth capability
├── 20.90-legacy-migration.adr.md      ← ADR inserted near end of auth block
│
├── 21-billing.capability/
│   ├── billing.capability.md
│   └── ...
```

## 5. Rules

1. **Hyphens only**: Always `-` between index and slug, never `_`
2. **No renaming**: `20-auth` stays `20-auth` forever; use fractional insertion instead
3. **Rebalance at depth 3**: If you reach `20.50.50.…`, consider rebalancing
4. **Works for files and directories**: `20-auth.capability/` and `20-auth.adr.md` follow the same pattern
