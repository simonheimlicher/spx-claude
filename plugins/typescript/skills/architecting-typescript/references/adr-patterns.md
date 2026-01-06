# Common ADR Patterns for TypeScript

## Pattern: External Tool Integration

When integrating with external tools (Hugo, Caddy, LHCI):

```markdown
## Decision

Use dependency injection for all external tool invocations.

### Implementation Constraints

1. All functions that call external tools must accept a `deps` parameter
2. The `deps` interface must include all external dependencies
3. Default implementations use real tools; tests inject controlled implementations

### Testing Strategy

| Component        | Level | Justification                          |
| ---------------- | ----- | -------------------------------------- |
| Command building | 1     | Pure function, no external deps        |
| Tool invocation  | 2     | Needs real binary to verify acceptance |
| Full workflow    | 3     | Needs real environment                 |
```

## Pattern: Configuration Loading

When defining configuration approach:

```markdown
## Decision

Use Zod schemas for all configuration validation.

### Implementation Constraints

1. All config files must have corresponding Zod schemas
2. Config loading must validate at load time, not use time
3. Invalid config must fail fast with descriptive errors

### Testing Strategy

| Component      | Level | Justification                   |
| -------------- | ----- | ------------------------------- |
| Schema parsing | 1     | Pure validation logic           |
| File loading   | 1     | Uses DI for fs operations       |
| Config merging | 1     | Pure function with typed inputs |
```

## Pattern: CLI Structure

When defining CLI architecture:

```markdown
## Decision

Use Commander.js with subcommand pattern.

### Implementation Constraints

1. Each command must be a separate module
2. Command modules export a function that registers with Commander
3. Commands must not contain business logic—delegate to runners

### Testing Strategy

| Component        | Level | Justification                     |
| ---------------- | ----- | --------------------------------- |
| Argument parsing | 1     | Can test with Commander's parse() |
| Command routing  | 1     | Pure function mapping             |
| Full CLI         | 3     | Needs real invocation to verify   |
```

## Pattern: Error Handling

When defining error handling approach:

```markdown
## Decision

Use typed error classes with structured error codes.

### Implementation Constraints

1. All errors must extend a base AppError class
2. Error codes must be unique and documented
3. Error messages must be user-facing and actionable

### Testing Strategy

| Component          | Level | Justification            |
| ------------------ | ----- | ------------------------ |
| Error construction | 1     | Pure class instantiation |
| Error formatting   | 1     | Pure string formatting   |
| Error propagation  | 1     | Can verify with DI       |
```

## Pattern: Async Operations

When defining async patterns:

```markdown
## Decision

Use async/await with explicit error handling and timeouts.

### Implementation Constraints

1. All async functions must have explicit return types
2. Timeouts must be configurable via dependency injection
3. Errors must be caught and converted to typed errors

### Testing Strategy

| Component        | Level | Justification                |
| ---------------- | ----- | ---------------------------- |
| Promise handling | 1     | Can test with controlled DI  |
| Timeout behavior | 1     | Inject fake timer            |
| Real async ops   | 2/3   | Depends on external resource |
```
