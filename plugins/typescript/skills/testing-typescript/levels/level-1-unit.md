# Level 1: Unit Tests

**Speed**: <50ms | **Infrastructure**: Standard developer CLI tools + temp fixtures | **Framework**: Vitest

Level 1 tests verify our code logic is correct using only tools installed by default on modern macOS and Linux developer machines.

## What Counts as "Standard Developer Tools" (Level 1)

These are **NOT** external dependencies—they're installed by default on modern macOS/Linux developer machines:

| Tool                                          | Why It's Level 1                                       |
| --------------------------------------------- | ------------------------------------------------------ |
| `git`, `node`, `npm`, `npx`, `curl`, `python` | Standard CLI tools on every developer machine          |
| `fs`, `path`, `os`, `crypto`                  | Node.js standard library, always available             |
| `os.tmpdir()`                                 | OS-provided temporary directory (MUST use exclusively) |
| `execa` (mocked)                              | Testing OUR command-building logic, not execution      |

**CRITICAL FILESYSTEM RULE:**

All Level 1 tests MUST use `os.tmpdir()` exclusively. Never write outside temporary directories.
Fast execution (<50ms) is possible thanks to SSDs.

The developer has bigger problems if git, node, npm, or curl are not available. These are part of the standard developer environment.

## What IS an External Dependency

| Tool        | Level | Why                                    |
| ----------- | ----- | -------------------------------------- |
| Hugo        | 2     | External binary (not Node.js standard) |
| Caddy       | 2     | External binary (not Node.js standard) |
| Claude Code | 2     | External tool requiring configuration  |
| GitHub API  | 3     | Network dependency                     |
| Lighthouse  | 3     | Requires Chrome (external browser)     |
| LHCI        | 3     | Requires Chrome + network              |

---

## File Location & Naming

```
Source File                      → Test File
src/config/loader.ts             → test/unit/config/loader.test.ts
src/runners/lhci.ts              → test/unit/runners/lhci.test.ts
src/hugo/build.ts                → test/unit/hugo/build.test.ts
```

---

## Dependency Injection Pattern

The core technique for Level 1 testing: design code to accept dependencies as parameters.

### Production Code Design

```typescript
// src/runners/lhci.ts
import { execa } from "execa";
import getPort from "get-port";

export interface LhciDependencies {
  execa: typeof execa;
  getPort: typeof getPort;
  mkdtemp: (prefix: string) => Promise<string>;
  writeFile: (path: string, content: string) => Promise<void>;
}

// Default dependencies for production
export const defaultDeps: LhciDependencies = {
  execa,
  getPort,
  mkdtemp: fs.promises.mkdtemp,
  writeFile: fs.promises.writeFile,
};

export interface LhciOptions {
  url?: string;
  set?: string;
  port?: number;
  runs?: number;
}

export function buildLhciCommand(urls: string[], options: { runs?: number; checksum?: boolean } = {}): string[] {
  // Pure function, no I/O
  const cmd = ["npx", "lhci", "collect"];

  for (const url of urls) {
    cmd.push("--url", url);
  }

  if (options.runs) {
    cmd.push("--numberOfRuns", String(options.runs));
  }

  return cmd;
}

export async function runLhci(options: LhciOptions, config: Config, deps: LhciDependencies = defaultDeps): Promise<LhciResult> {
  const urls = options.url ? [options.url] : config.url_sets[options.set ?? "all"];
  const cmd = buildLhciCommand(urls, { runs: options.runs });

  const port = options.port ?? (await deps.getPort());
  const tempDir = await deps.mkdtemp("/tmp/hugolit-");

  // ... rest of implementation
}
```

### Level 1 Tests

```typescript
// test/unit/runners/lhci.test.ts
import { buildLhciCommand, type LhciDependencies, runLhci } from "@/runners/lhci";
import { describe, expect, it, vi } from "vitest";
import { createTestConfig } from "../../fixtures/factories";

describe("buildLhciCommand", () => {
  /**
   * Level 1: Pure function tests—no dependencies.
   */

  it("GIVEN urls WHEN building command THEN includes lhci collect", () => {
    const cmd = buildLhciCommand(["/", "/about/"]);

    expect(cmd[0]).toBe("npx");
    expect(cmd[1]).toBe("lhci");
    expect(cmd[2]).toBe("collect");
  });

  it("GIVEN multiple urls WHEN building command THEN includes all urls", () => {
    const cmd = buildLhciCommand(["/", "/about/", "/contact/"]);

    expect(cmd.filter((arg) => arg === "--url").length).toBe(3);
    expect(cmd).toContain("/");
    expect(cmd).toContain("/about/");
    expect(cmd).toContain("/contact/");
  });

  it("GIVEN runs option WHEN building command THEN includes numberOfRuns", () => {
    const cmd = buildLhciCommand(["/"], { runs: 5 });

    const runsIndex = cmd.indexOf("--numberOfRuns");
    expect(runsIndex).toBeGreaterThan(-1);
    expect(cmd[runsIndex + 1]).toBe("5");
  });

  it("GIVEN unicode path WHEN building command THEN path is preserved", () => {
    const cmd = buildLhciCommand(["/статьи/"]);

    expect(cmd).toContain("/статьи/");
  });
});

describe("runLhci", () => {
  /**
   * Level 1: Testing logic with injected dependencies.
   */

  const createMockDeps = (): LhciDependencies => ({
    execa: vi.fn().mockResolvedValue({ exitCode: 0, stdout: "" }),
    getPort: vi.fn().mockResolvedValue(4000),
    mkdtemp: vi.fn().mockResolvedValue("/tmp/hugolit-12345"),
    writeFile: vi.fn().mockResolvedValue(undefined),
  });

  it("GIVEN URL set configured WHEN running THEN audits each URL", async () => {
    // Given
    const config = createTestConfig({
      url_sets: { critical: ["/", "/about/"] },
    });
    const mockDeps = createMockDeps();

    // When
    await runLhci({ set: "critical" }, config, mockDeps);

    // Then: execa called with lhci collect
    expect(mockDeps.execa).toHaveBeenCalled();
    const calls = mockDeps.execa.mock.calls;
    const lhciCall = calls.find(([cmd, args]) => cmd === "npx" && args?.[0] === "lhci");
    expect(lhciCall).toBeDefined();
  });

  it("GIVEN port not specified WHEN running THEN auto-picks free port", async () => {
    // Given
    const config = createTestConfig();
    const mockDeps = createMockDeps();

    // When
    await runLhci({}, config, mockDeps);

    // Then: getPort was called
    expect(mockDeps.getPort).toHaveBeenCalled();
  });

  it("GIVEN port specified WHEN running THEN uses specified port", async () => {
    // Given
    const config = createTestConfig();
    const mockDeps = createMockDeps();

    // When
    await runLhci({ port: 8080 }, config, mockDeps);

    // Then: getPort not called
    expect(mockDeps.getPort).not.toHaveBeenCalled();
  });

  it("GIVEN Hugo build fails WHEN running THEN throws descriptive error", async () => {
    // Given
    const config = createTestConfig();
    const mockDeps = createMockDeps();
    mockDeps.execa.mockRejectedValueOnce(new Error("hugo: command not found"));

    // When/Then
    await expect(runLhci({}, config, mockDeps)).rejects.toThrow("Hugo build failed");
  });
});
```

---

## Testing Pure Functions

Pure functions are the easiest to test—no dependencies needed.

```typescript
// src/config/schema.ts
import { z } from "zod";

export const configSchema = z.object({
  url_sets: z.record(z.array(z.string())),
  thresholds: z
    .object({
      performance: z.number().min(0).max(100).optional(),
      accessibility: z.number().min(0).max(100).optional(),
    })
    .optional(),
  lhci: z
    .object({
      runs: z.number().min(1).max(10).default(3),
      preset: z.enum(["mobile", "desktop"]).default("mobile"),
    })
    .optional(),
});

export function validateConfig(config: unknown): Config {
  return configSchema.parse(config);
}
```

```typescript
// test/unit/config/schema.test.ts
import { configSchema, validateConfig } from "@/config/schema";
import { describe, expect, it } from "vitest";

describe("validateConfig", () => {
  it("GIVEN valid config WHEN validating THEN returns parsed config", () => {
    const input = {
      url_sets: { critical: ["/", "/about/"] },
    };

    const result = validateConfig(input);

    expect(result.url_sets.critical).toEqual(["/", "/about/"]);
  });

  it("GIVEN missing url_sets WHEN validating THEN throws ZodError", () => {
    const input = {};

    expect(() => validateConfig(input)).toThrow();
  });

  it("GIVEN invalid threshold WHEN validating THEN throws ZodError", () => {
    const input = {
      url_sets: { all: ["/"] },
      thresholds: { performance: 150 }, // Invalid: > 100
    };

    expect(() => validateConfig(input)).toThrow();
  });

  it("GIVEN missing optional fields WHEN validating THEN uses defaults", () => {
    const input = {
      url_sets: { all: ["/"] },
    };

    const result = validateConfig(input);

    expect(result.lhci?.runs).toBe(3); // Default
    expect(result.lhci?.preset).toBe("mobile"); // Default
  });
});
```

---

## Testing Error Handling

```typescript
// src/errors.ts
export function parseExecaError(error: unknown): AppError {
  if (error instanceof Error) {
    const message = error.message.toLowerCase();

    if (message.includes("command not found")) {
      return new AppError("tool_not_found", "Required tool not installed", {
        fix: "Install the required tool and ensure it is in PATH",
      });
    }

    if (message.includes("permission denied")) {
      return new AppError("permission_denied", "Permission denied", {
        fix: "Check file permissions",
      });
    }
  }

  return new AppError("unknown", "Unknown error occurred");
}
```

```typescript
// test/unit/errors.test.ts
import { AppError, parseExecaError } from "@/errors";
import { describe, expect, it } from "vitest";

describe("parseExecaError", () => {
  it("GIVEN command not found WHEN parsing THEN identifies as tool_not_found", () => {
    const error = new Error("hugo: command not found");

    const result = parseExecaError(error);

    expect(result.code).toBe("tool_not_found");
    expect(result.fix).toContain("Install");
  });

  it("GIVEN permission denied WHEN parsing THEN identifies as permission_denied", () => {
    const error = new Error("EACCES: permission denied");

    const result = parseExecaError(error);

    expect(result.code).toBe("permission_denied");
  });

  it("GIVEN unknown error WHEN parsing THEN returns unknown", () => {
    const error = new Error("Something unexpected");

    const result = parseExecaError(error);

    expect(result.code).toBe("unknown");
  });
});
```

---

## Using Temporary Directories

Temp directories are ephemeral and reentrant—they're Level 1.

```typescript
// test/unit/hugo/build.test.ts
import * as fs from "fs";
import * as os from "os";
import * as path from "path";
import { afterEach, beforeEach, describe, expect, it, vi } from "vitest";

describe("file operations", () => {
  let tempDir: string;

  beforeEach(async () => {
    tempDir = await fs.promises.mkdtemp(path.join(os.tmpdir(), "hugolit-test-"));
  });

  afterEach(async () => {
    await fs.promises.rm(tempDir, { recursive: true, force: true });
  });

  it("GIVEN nested path WHEN preparing directory THEN creates structure", async () => {
    const nested = path.join(tempDir, "a", "b", "c");

    await prepareDirectory(nested);

    expect(fs.existsSync(nested)).toBe(true);
  });

  it("GIVEN source files WHEN copying THEN files appear at destination", async () => {
    const source = path.join(tempDir, "source");
    const dest = path.join(tempDir, "dest");
    await fs.promises.mkdir(source);
    await fs.promises.writeFile(path.join(source, "file.txt"), "content");

    await copyFiles(source, dest);

    expect(fs.existsSync(path.join(dest, "file.txt"))).toBe(true);
    expect(await fs.promises.readFile(path.join(dest, "file.txt"), "utf-8")).toBe("content");
  });
});
```

---

## When to Escalate to Level 2

Escalate when you need to verify behavior that requires the **actual external tool**:

| Behavior                 | Level 1 Sufficient? | Why                                 |
| ------------------------ | ------------------- | ----------------------------------- |
| Command is well-formed   | ✅ Yes              | Pure function, can verify structure |
| Hugo accepts the command | ❌ No               | Need real Hugo to verify            |
| Files actually build     | ❌ No               | Need real Hugo execution            |
| Caddy serves files       | ❌ No               | Need real Caddy                     |
| Config file parsed       | ✅ Yes              | Pure parsing logic                  |
| Lighthouse runs          | ❌ No               | Need real Chrome (Level 3)          |

### Escalation Decision

```typescript
// This test belongs at Level 1—testing our logic
it("GIVEN checksum enabled WHEN building command THEN includes flag", () => {
  const cmd = buildLhciCommand(["/"], { checksum: true });
  expect(cmd).toContain("--checksum");
});

// This test requires Level 2—verifying Hugo behavior
it("GIVEN valid site WHEN building THEN Hugo outputs files", async () => {
  const result = await buildHugo("test/fixtures/sample-site");
  expect(result.exitCode).toBe(0);
  expect(fs.existsSync(result.buildDir)).toBe(true);
});
```

---

## Completion Requirements

Level 1 tests are **required** but **not sufficient** for feature completion:

| Work Item  | Level 1 Proves   | But Cannot Prove  |
| ---------- | ---------------- | ----------------- |
| Story      | Logic is correct | Integration works |
| Feature    | Algorithms work  | Real tools work   |
| Capability | N/A              | Needs Level 2+    |

**Use Level 1 for**: Command building, config parsing, error handling logic, path manipulation, data transformation, validation rules.

---

## Anti-Patterns

### Anti-Pattern: Mocking Instead of DI

```typescript
// ❌ Mocking couples test to implementation
vi.mock("execa", () => ({ execa: vi.fn() }));

it("calls execa", async () => {
  await buildHugo(siteDir);
  expect(execa).toHaveBeenCalled();
});

// ✅ DI tests behavior, not implementation
it("GIVEN valid site WHEN building THEN returns build dir", async () => {
  const deps = { execa: vi.fn().mockResolvedValue({ exitCode: 0 }) };
  const result = await buildHugo(siteDir, deps);
  expect(result.buildDir).toBeDefined();
});
```

### Anti-Pattern: Testing External Tool Behavior

```typescript
// ❌ This tests Hugo, not our code
it("hugo creates public directory", async () => {
  await execa("hugo", ["--minify"]);
  expect(fs.existsSync("public")).toBe(true);
});

// ✅ This tests our code's command building
it("GIVEN minify option WHEN building command THEN includes --minify", () => {
  const cmd = buildHugoCommand({ minify: true });
  expect(cmd).toContain("--minify");
});
```

### Anti-Pattern: Hardcoded Test Data

```typescript
// ❌ Magic strings hide assumptions
it("runs lhci", async () => {
  await runLhci({ url: "http://localhost:1313/about/" });
});

// ✅ Generated data makes assumptions explicit
it("GIVEN url set WHEN running THEN audits each URL", async () => {
  const config = createTestConfig({
    url_sets: { critical: ["/", "/about/"] },
  });
  await runLhci({ set: "critical" }, config, mockDeps);
});
```

---

*Level 1 is the foundation. Get this right, and higher levels become verification, not debugging.*
