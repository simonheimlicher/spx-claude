# Level 3: E2E Tests

**Speed**: <30s | **Infrastructure**: Chrome + running server | **Framework**: Vitest

Level 3 tests verify complete workflows work end-to-end with real external services.

## What Level 3 Provides

| Component  | What's Real                 | What's Controlled     |
| ---------- | --------------------------- | --------------------- |
| Hugo       | Real Hugo binary            | Sample test site      |
| Caddy      | Real Caddy server           | Ephemeral, local port |
| Lighthouse | Real Lighthouse with Chrome | Controlled URLs       |
| LHCI       | Real LHCI with Chrome       | Test configuration    |
| CLI        | Real CLI execution          | Test arguments        |

## Why Level 3 Exists

Level 2 proves tools work together locally. Level 3 proves:

- Complete user workflows actually work
- Chrome-based tools (Lighthouse, LHCI) produce real results
- CLI commands deliver user value end-to-end
- Reports are generated correctly

---

## The Critical Requirement: Credentials & Test Accounts

> **Before writing ANY Level 3 test, you must know where the credentials are and what test accounts exist.**

### 🚨 THE RULE: No Credentials, No Level 3 Tests

> **If you do not have explicit information about test credentials and accounts, you MUST ask the user before proceeding.**

Do not guess. Do not use production credentials. Ask:

```
I need to write end-to-end tests that use [external service].

To proceed, I need to know:
1. Where are the test credentials stored? (env vars, secrets manager, etc.)
2. What test accounts/environments exist?
3. Are there rate limits or quotas on the test account?
4. How do I reset test data between runs?
5. Is there a staging/sandbox environment, or do tests run against production?

Please provide this information before I proceed with Level 3 tests.
```

### Credential Management Pattern

```typescript
import os from "os";

// Document where credentials come from
const CREDENTIALS_SOURCE = `
Level 3 tests require these environment variables:
- CHROME_PATH: Path to Chrome binary (default: system Chrome)
- TEST_SERVER_URL: URL of test environment (required)
- LHCI_TOKEN: Token for uploading results (optional)

Test credentials location: .env.test (not committed)
`;

beforeAll(() => {
  if (!process.env.TEST_SERVER_URL) {
    throw new Error(`TEST_SERVER_URL not set.\n${CREDENTIALS_SOURCE}`);
  }
});
```

---

## File Location & Naming

```
test/
├── unit/                    # Level 1
├── integration/             # Level 2
└── e2e/                     # Level 3
    ├── cli.e2e.test.ts
    ├── audit.e2e.test.ts
    └── fixtures/
        └── sample-hugo-site/
```

---

## Infrastructure Setup

### Chrome/Lighthouse Availability

```typescript
// test/e2e/conftest.ts
import { execaSync } from "execa";

export function chromeAvailable(): boolean {
  try {
    // Check if Chrome is available via Lighthouse
    execaSync("npx", ["lighthouse", "--version"]);
    return true;
  } catch {
    return false;
  }
}

export function lhciAvailable(): boolean {
  try {
    execaSync("npx", ["lhci", "--version"]);
    return true;
  } catch {
    return false;
  }
}
```

### Skip Markers for CI

```typescript
// test/e2e/cli.e2e.test.ts
import { describe, expect, it } from "vitest";
import { chromeAvailable } from "./conftest";

describe.skipIf(!chromeAvailable())("Lighthouse E2E", () => {
  // Tests that require Chrome...
});
```

---

## Good Patterns

### Pattern: Testing CLI Help Output

```typescript
// test/e2e/cli.e2e.test.ts
import { execa } from "execa";
import { describe, expect, it } from "vitest";

describe("CLI E2E", () => {
  it("GIVEN hugolit installed WHEN running --help THEN shows usage", async () => {
    // When
    const { stdout, exitCode } = await execa("node", [
      "dist/bin/hugolit.js",
      "--help",
    ]);

    // Then
    expect(exitCode).toBe(0);
    expect(stdout).toContain("Usage:");
    expect(stdout).toContain("hugolit");
  });

  it("GIVEN hugolit installed WHEN running --version THEN shows version", async () => {
    // When
    const { stdout, exitCode } = await execa("node", [
      "dist/bin/hugolit.js",
      "--version",
    ]);

    // Then
    expect(exitCode).toBe(0);
    expect(stdout).toMatch(/\d+\.\d+\.\d+/);
  });
});
```

### Pattern: Testing Full Audit Workflow

```typescript
// test/e2e/audit.e2e.test.ts
import { execa, type ExecaChildProcess } from "execa";
import * as fs from "fs";
import getPort from "get-port";
import * as os from "os";
import * as path from "path";
import { afterAll, beforeAll, describe, expect, it } from "vitest";
import { caddyAvailable, chromeAvailable, hugoAvailable } from "./conftest";

const canRunFullE2E = chromeAvailable() && hugoAvailable() && caddyAvailable();

describe.skipIf(!canRunFullE2E)("Full Audit E2E", () => {
  const fixturePath = path.join(__dirname, "fixtures", "sample-hugo-site");
  let tempDir: string;
  let buildDir: string;
  let caddyProcess: ExecaChildProcess | null = null;
  let port: number;

  beforeAll(async () => {
    // Setup: Build Hugo site and start Caddy
    tempDir = await fs.promises.mkdtemp(path.join(os.tmpdir(), "hugolit-e2e-"));
    buildDir = path.join(tempDir, "public");
    port = await getPort();

    // Build Hugo site
    await execa("hugo", ["--destination", buildDir], { cwd: fixturePath });

    // Start Caddy
    const caddyfile = `:${port} {
  root * ${buildDir}
  file_server
}`;
    const caddyfilePath = path.join(tempDir, "Caddyfile");
    await fs.promises.writeFile(caddyfilePath, caddyfile);
    caddyProcess = execa("caddy", ["run", "--config", caddyfilePath]);

    // Wait for server to start
    await new Promise((resolve) => setTimeout(resolve, 1000));
  });

  afterAll(async () => {
    if (caddyProcess) {
      caddyProcess.kill();
    }
    await fs.promises.rm(tempDir, { recursive: true, force: true });
  });

  it("GIVEN running server WHEN running hugolit audit THEN produces scores", async () => {
    // When
    const { stdout, exitCode } = await execa(
      "node",
      ["dist/bin/hugolit.js", "audit", "--url", `http://localhost:${port}/`],
      { timeout: 60000 }, // Lighthouse can be slow
    );

    // Then
    expect(exitCode).toBe(0);
    expect(stdout).toContain("performance");
  });
});
```

### Pattern: Testing Report Generation

```typescript
// test/e2e/reports.e2e.test.ts
import { execa } from "execa";
import * as fs from "fs";
import * as os from "os";
import * as path from "path";
import { afterEach, beforeEach, describe, expect, it } from "vitest";
import { chromeAvailable } from "./conftest";

describe.skipIf(!chromeAvailable())("Report Generation E2E", () => {
  let tempDir: string;

  beforeEach(async () => {
    tempDir = await fs.promises.mkdtemp(path.join(os.tmpdir(), "reports-e2e-"));
  });

  afterEach(async () => {
    await fs.promises.rm(tempDir, { recursive: true, force: true });
  });

  it("GIVEN audit complete WHEN requesting JSON output THEN creates valid JSON file", async () => {
    const outputPath = path.join(tempDir, "report.json");

    // When
    await execa("node", [
      "dist/bin/hugolit.js",
      "audit",
      "--url",
      "http://localhost:1313/",
      "--output",
      outputPath,
      "--format",
      "json",
    ]);

    // Then
    expect(fs.existsSync(outputPath)).toBe(true);
    const content = await fs.promises.readFile(outputPath, "utf-8");
    const report = JSON.parse(content);
    expect(report.scores).toBeDefined();
  });
});
```

### Pattern: Testing Error Handling

```typescript
// test/e2e/errors.e2e.test.ts
import { execa } from "execa";
import { describe, expect, it } from "vitest";

describe("CLI Error Handling E2E", () => {
  it("GIVEN invalid URL WHEN running audit THEN exits with error", async () => {
    // When
    const result = await execa(
      "node",
      ["dist/bin/hugolit.js", "audit", "--url", "not-a-valid-url"],
      { reject: false },
    );

    // Then
    expect(result.exitCode).not.toBe(0);
    expect(result.stderr).toContain("Invalid URL");
  });

  it("GIVEN unreachable server WHEN running audit THEN exits with helpful message", async () => {
    // When
    const result = await execa(
      "node",
      ["dist/bin/hugolit.js", "audit", "--url", "http://localhost:59999/"],
      { reject: false, timeout: 10000 },
    );

    // Then
    expect(result.exitCode).not.toBe(0);
    expect(result.stderr.toLowerCase()).toMatch(/connect|refused|unreachable/);
  });
});
```

---

## Performance Expectations

Level 3 tests are slow by nature. Budget accordingly:

| Test Type         | Target | Max |
| ----------------- | ------ | --- |
| CLI help/version  | <1s    | 5s  |
| Single Lighthouse | <15s   | 30s |
| Full audit flow   | <30s   | 60s |

### Timeout Configuration

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    testTimeout: 60000, // 60 seconds for E2E tests
    hookTimeout: 30000, // 30 seconds for setup/teardown
  },
});
```

---

## CI Configuration

Level 3 tests require Chrome, which may not be available in all CI environments.

### GitHub Actions Example

```yaml
# .github/workflows/test.yml
jobs:
  unit-integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm test -- test/unit/ test/integration/

  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - name: Install Chrome
        run: |
          wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
          sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'
          sudo apt-get update
          sudo apt-get install -y google-chrome-stable
      - run: npm ci
      - run: npm test -- test/e2e/
```

---

## When NOT to Use Level 3

Level 3 is expensive. Only escalate when necessary:

| Behavior                      | Use Level 3? | Why                       |
| ----------------------------- | ------------ | ------------------------- |
| Config parsing logic          | ❌ No        | Level 1 sufficient        |
| Hugo build creates files      | ❌ No        | Level 2 sufficient        |
| CLI argument parsing          | ❌ No        | Level 1 sufficient        |
| Lighthouse actually runs      | ✅ Yes       | Need real Chrome          |
| Full workflow produces output | ✅ Yes       | Need complete environment |
| Report format is correct      | ✅ Yes       | Need real audit data      |

---

## Completion Requirements

Level 3 tests prove **user value is delivered**:

| Work Item      | Level 3 Proves            | Combined With   |
| -------------- | ------------------------- | --------------- |
| Story          | N/A                       | Level 1 only    |
| Feature        | N/A                       | Level 1 + 2     |
| **Capability** | End-to-end value delivery | Level 1 + 2 + 3 |

**Use Level 3 for**: CLI workflows, Lighthouse audits, report generation, complete user journeys.

---

## Anti-Patterns

### Anti-Pattern: Skipping Lower Levels

```typescript
// ❌ Don't jump straight to E2E
it("runs full audit", async () => {
  // If this fails, you won't know if it's:
  // - Your code logic (Level 1)
  // - Tool integration (Level 2)
  // - Chrome/environment (Level 3)
  const result = await execa("hugolit", ["audit"]);
});

// ✅ Build confidence bottom-up
// Level 1: Test command building
// Level 2: Test Hugo/Caddy work together
// Level 3: Test full workflow
```

### Anti-Pattern: Testing Third-Party Tools

```typescript
// ❌ This tests Lighthouse, not your code
it("Lighthouse returns scores", async () => {
  const result = await execa("npx", ["lighthouse", "https://example.com"]);
  expect(result.stdout).toContain("performance");
});

// ✅ Test YOUR code that uses Lighthouse
it("GIVEN Lighthouse completes WHEN parsing result THEN extracts scores", async () => {
  // Your parsing logic, tested with real Lighthouse output
});
```

### Anti-Pattern: Flaky Timing

```typescript
// ❌ Arbitrary sleeps are flaky
await new Promise((r) => setTimeout(r, 5000));
const result = await fetch(`http://localhost:${port}/`);

// ✅ Poll until ready with timeout
const waitForServer = async (url: string, timeout = 10000) => {
  const start = Date.now();
  while (Date.now() - start < timeout) {
    try {
      const res = await fetch(url);
      if (res.ok) return;
    } catch {
      await new Promise((r) => setTimeout(r, 100));
    }
  }
  throw new Error(`Server at ${url} not ready after ${timeout}ms`);
};

await waitForServer(`http://localhost:${port}/`);
```

---

*Level 3 is where you prove user value. If it works here, you've shipped something real.*
