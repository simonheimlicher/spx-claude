---
name: typescript-test
description: "TypeScript-specific testing patterns. REQUIRES reading /test first for foundational principles."
allowed-tools: Read, Bash, Glob, Grep, Write, Edit
---

# TypeScript Testing Patterns

> **PREREQUISITE:** Read the foundational `/test` skill first. This skill only provides TypeScript-specific patterns.

This skill provides TypeScript-specific implementations of the three-tier testing methodology defined in `/test`.

---

## TypeScript Tooling

| Level          | Tools                                                  | Speed |
| -------------- | ------------------------------------------------------ | ----- |
| 1: Unit        | Vitest, temp directories, dependency injection         | <50ms |
| 2: Integration | Vitest + real binaries (Hugo, Caddy), test harnesses   | <1s   |
| 3: E2E         | Vitest/Playwright + Chrome, real services, credentials | <30s  |

---

## Level 1: Unit Patterns (TypeScript)

### Dependency Injection

```typescript
// ❌ BAD: Hardcoded dependency requiring mocks
async function buildSite(siteDir: string): Promise<BuildResult> {
  const { exitCode } = await execa("hugo", ["--source", siteDir]);
  return { success: exitCode === 0 };
}

// ✅ GOOD: Dependencies as parameters
type BuildDeps = {
  runCommand: (cmd: string, args: string[]) => Promise<{ exitCode: number }>;
};

async function buildSite(siteDir: string, deps: BuildDeps): Promise<BuildResult> {
  const { exitCode } = await deps.runCommand("hugo", ["--source", siteDir]);
  return { success: exitCode === 0 };
}

// Test with controlled implementation
test("returns success on zero exit code", async () => {
  const deps: BuildDeps = {
    runCommand: async () => ({ exitCode: 0 }),
  };

  const result = await buildSite("/site", deps);

  expect(result.success).toBe(true);
});
```

### Pure Function Testing

```typescript
test("buildHugoCommand includes source flag", () => {
  const cmd = buildHugoCommand({ source: "/site" });

  expect(cmd).toEqual(["hugo", "--source", "/site"]);
});

test("buildHugoCommand adds minify when enabled", () => {
  const cmd = buildHugoCommand({ source: "/site", minify: true });

  expect(cmd).toContain("--minify");
});

test("parseConfig rejects missing required fields", () => {
  const result = parseConfig({ base_url: "http://localhost" });

  expect(result.ok).toBe(false);
  expect(result.errors).toContain("site_dir: Required");
});
```

### Temporary Directories

```typescript
import { mkdtemp, rm, writeFile } from "fs/promises";
import { join } from "path";
import { tmpdir } from "os";

test("loadConfig reads YAML file", async () => {
  const tempDir = await mkdtemp(join(tmpdir(), "config-test-"));

  try {
    const configPath = join(tempDir, "config.yaml");
    await writeFile(
      configPath,
      `
site_dir: ./site
base_url: http://localhost:1313
`
    );

    const config = await loadConfig(configPath);

    expect(config.site_dir).toBe("./site");
  } finally {
    await rm(tempDir, { recursive: true });
  }
});
```

### Data Factories

```typescript
let idCounter = 0;

function createAuditResult(overrides: Partial<AuditResult> = {}): AuditResult {
  return {
    id: `audit-${++idCounter}`,
    url: `https://example.com/page-${idCounter}`,
    scores: {
      performance: 90,
      accessibility: 100,
    },
    ...overrides,
  };
}

function createConfig(overrides: Partial<Config> = {}): Config {
  return {
    site_dir: "./test-site",
    base_url: "http://localhost:1313",
    url_sets: { default: ["/"] },
    ...overrides,
  };
}

// Usage
test("fails on low performance score", async () => {
  const result = createAuditResult({ scores: { performance: 45, accessibility: 100 } });

  const analysis = await analyzeResults([result], deps);

  expect(analysis.passed).toBe(false);
});
```

---

## Level 2: Integration Patterns (TypeScript)

### Test Harness: Real Binary (Hugo)

```typescript
// test/harnesses/hugo.ts
import { execa, ExecaReturnValue } from "execa";
import { mkdtemp, rm, cp, writeFile, mkdir } from "fs/promises";
import { join } from "path";
import { tmpdir } from "os";

type HugoHarness = {
  siteDir: string;
  outputDir: string;
  build: (args?: string[]) => Promise<ExecaReturnValue>;
  cleanup: () => Promise<void>;
};

async function createHugoHarness(): Promise<HugoHarness> {
  // Verify Hugo is installed
  try {
    await execa("hugo", ["version"]);
  } catch {
    throw new Error("Hugo binary not found. Install Hugo or skip integration tests.");
  }

  const siteDir = await mkdtemp(join(tmpdir(), "hugo-test-"));
  const outputDir = join(siteDir, "public");

  await createMinimalSite(siteDir);

  return {
    siteDir,
    outputDir,

    async build(args: string[] = []) {
      return execa("hugo", ["--source", siteDir, "--destination", outputDir, ...args]);
    },

    async cleanup() {
      await rm(siteDir, { recursive: true, force: true });
    },
  };
}
```

### Test Harness: HTTP Server (Caddy)

```typescript
// test/harnesses/caddy.ts
import { execa, ExecaChildProcess } from "execa";
import { writeFile, mkdtemp, rm } from "fs/promises";
import { join } from "path";
import { tmpdir } from "os";
import getPort from "get-port";

type CaddyHarness = {
  port: number;
  baseUrl: string;
  start: () => Promise<void>;
  stop: () => Promise<void>;
  cleanup: () => Promise<void>;
};

async function createCaddyHarness(staticDir: string): Promise<CaddyHarness> {
  const port = await getPort();
  const configDir = await mkdtemp(join(tmpdir(), "caddy-test-"));
  let process: ExecaChildProcess | null = null;

  return {
    port,
    baseUrl: `http://localhost:${port}`,

    async start() {
      const caddyfile = join(configDir, "Caddyfile");
      await writeFile(caddyfile, `:${port} {\n  root * ${staticDir}\n  file_server\n}`);

      process = execa("caddy", ["run", "--config", caddyfile], { reject: false });
      await waitForServer(`http://localhost:${port}`, 5000);
    },

    async stop() {
      process?.kill();
      process = null;
    },

    async cleanup() {
      await this.stop();
      await rm(configDir, { recursive: true, force: true });
    },
  };
}
```

### Using Harnesses with Vitest

```typescript
import { describe, test, expect, beforeAll, afterAll } from "vitest";
import { createHugoHarness } from "../harnesses/hugo";
import { existsSync } from "fs";
import { join } from "path";

describe("Hugo Build Integration", () => {
  test("builds minimal site successfully", async () => {
    const harness = await createHugoHarness();

    try {
      const result = await harness.build();

      expect(result.exitCode).toBe(0);
      expect(existsSync(join(harness.outputDir, "index.html"))).toBe(true);
    } finally {
      await harness.cleanup();
    }
  });

  test("builds with minify flag", async () => {
    const harness = await createHugoHarness();

    try {
      const result = await harness.build(["--minify"]);

      expect(result.exitCode).toBe(0);
    } finally {
      await harness.cleanup();
    }
  });
});
```

---

## Level 3: E2E Patterns (TypeScript)

### Credential Management

```typescript
// test/e2e/credentials.ts

/**
 * Level 3 tests require these environment variables.
 *
 * Required:
 *   LHCI_SERVER_URL    - LHCI server URL
 *   LHCI_TOKEN         - LHCI build token
 *
 * Where to find:
 *   - 1Password: "Engineering/Test Credentials"
 *
 * Setup:
 *   cp .env.test.example .env.test
 */

type Credentials = {
  lhciServerUrl: string;
  lhciToken: string;
  testSiteUrl: string;
};

function loadCredentials(): Credentials | null {
  const lhciServerUrl = process.env.LHCI_SERVER_URL;
  const lhciToken = process.env.LHCI_TOKEN;
  const testSiteUrl = process.env.TEST_SITE_URL || "https://staging.example.com";

  if (!lhciServerUrl || !lhciToken) {
    return null;
  }

  return { lhciServerUrl, lhciToken, testSiteUrl };
}

export { loadCredentials, Credentials };
```

### Skip If No Credentials

```typescript
import { describe, test, expect, beforeAll } from "vitest";
import { loadCredentials } from "./credentials";

describe("LHCI End-to-End", () => {
  const credentials = loadCredentials();

  beforeAll(() => {
    if (!credentials) {
      console.log("⏭️  Skipping LHCI E2E tests - credentials not configured");
    }
  });

  test.skipIf(!credentials)("uploads audit results to LHCI server", async () => {
    // Test implementation using credentials!
  });
});
```

### CLI E2E Testing

```typescript
import { execa } from "execa";

test.skipIf(!credentials)("CLI full workflow succeeds", async () => {
  const result = await execa("node", ["./bin/cli.js", "run", "--url", credentials!.testSiteUrl, "--upload"], {
    env: {
      ...process.env,
      LHCI_SERVER_URL: credentials!.lhciServerUrl,
      LHCI_TOKEN: credentials!.lhciToken,
    },
    reject: false,
  });

  expect(result.exitCode).toBe(0);
  expect(result.stdout).toContain("Audit complete");
});
```

### Playwright E2E (Browser Tests)

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./test/e2e",
  use: {
    baseURL: process.env.TEST_BASE_URL || "http://localhost:3000",
  },
  projects: [{ name: "chromium", use: { ...devices["Desktop Chrome"] } }],
  webServer: process.env.TEST_BASE_URL
    ? undefined
    : {
        command: "npm run dev",
        url: "http://localhost:3000",
        reuseExistingServer: !process.env.CI,
      },
});
```

```typescript
// test/e2e/workflow.spec.ts
import { test, expect } from "@playwright/test";

test("user can run audit from dashboard", async ({ page }) => {
  await page.goto("/dashboard");
  await page.fill('[data-testid="url-input"]', "https://example.com");
  await page.click('[data-testid="run-audit"]');

  await expect(page.locator('[data-testid="results"]')).toBeVisible({ timeout: 30000 });
});
```

---

## Vitest Configuration

```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    include: ["test/**/*.test.ts"],
    testTimeout: 30000,
    hookTimeout: 30000,

    // Run integration tests sequentially to avoid port conflicts
    poolOptions: {
      threads: {
        singleThread: true,
      },
    },
  },
});

// package.json scripts
{
  "scripts": {
    "test": "vitest run",
    "test:unit": "vitest run --exclude '**/integration/**' --exclude '**/e2e/**'",
    "test:integration": "vitest run test/integration",
    "test:e2e": "vitest run test/e2e"
  }
}
```

---

## Quick Reference

| Pattern      | Level 1                  | Level 2            | Level 3              |
| ------------ | ------------------------ | ------------------ | -------------------- |
| Dependencies | Injected types/functions | Real via harness   | Real via credentials |
| Data         | Factories + tmpdir       | Fixtures + harness | Test accounts        |
| Speed        | <50ms                    | <1s                | <30s                 |
| CI           | Every commit             | Every commit       | Nightly/pre-release  |

---

_For foundational principles (no mocking, progress vs regression tests, escalation justification), see `/test`._
