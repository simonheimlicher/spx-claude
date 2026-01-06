# Level 3: System / End-to-End

## The Question This Level Answers

> **"Does the complete system work the way users will actually use it?"**

Level 2 proved your integrations work locally with controlled dependencies. Level 3 proves the **entire system** works with **real credentials** against **real services**—exactly as a user would experience it.

---

## 🚨 The Credentials Requirement

> **Before writing ANY Level 3 test, you must know where the credentials are and what test accounts exist.**

### If You Don't Know the Credentials, STOP

Do not guess. Do not use production credentials. Do not hardcode anything. Ask the user:

```
I need to write end-to-end tests that use [external service].

To proceed, I need to know:
1. Where are the test credentials stored? (env vars, secrets manager, .env.test, etc.)
2. What test accounts/environments exist? (staging, sandbox, test tenant, etc.)
3. Are there rate limits or quotas on the test account?
4. How do I reset test data between runs?
5. Are there any test accounts I should NOT use or modify?

Please provide this information before I proceed with Level 3 tests.
```

---

## Credential Management Patterns

### Pattern: Environment Variables with Documentation

```typescript
// test/e2e/credentials.ts

/**
 * Level 3 tests require these environment variables.
 *
 * Required:
 *   LHCI_SERVER_URL    - LHCI server URL (staging)
 *   LHCI_TOKEN         - LHCI build token
 *
 * Optional:
 *   TEST_SITE_URL      - URL to test (default: staging.example.com)
 *
 * Where to find these:
 *   - Team 1Password vault: "Engineering/Test Credentials"
 *   - Or ask in #engineering Slack channel
 *
 * Local setup:
 *   cp .env.test.example .env.test
 *   # Fill in values from 1Password
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

function requireCredentials(): Credentials {
  const creds = loadCredentials();

  if (!creds) {
    const missing = [!process.env.LHCI_SERVER_URL && "LHCI_SERVER_URL", !process.env.LHCI_TOKEN && "LHCI_TOKEN"].filter(Boolean);

    throw new Error(`Missing required credentials: ${missing.join(", ")}\n\n` + `See test/e2e/credentials.ts for setup instructions.`);
  }

  return creds;
}

export { loadCredentials, requireCredentials };
```

### Pattern: Skip If Credentials Missing

```typescript
// test/e2e/lhci.test.ts
import { describe, test, beforeAll } from "vitest";
import { loadCredentials } from "./credentials";

describe("LHCI End-to-End", () => {
  const credentials = loadCredentials();

  beforeAll(() => {
    if (!credentials) {
      console.log("⏭️  Skipping LHCI E2E tests - credentials not configured");
    }
  });

  test.skipIf(!credentials)("uploads audit results to LHCI server", async () => {
    // Test implementation
  });

  test.skipIf(!credentials)("retrieves historical data from server", async () => {
    // Test implementation
  });
});
```

### Pattern: .env.test.example Template

```bash
# .env.test.example
# Copy to .env.test and fill in values from 1Password

# LHCI Configuration
LHCI_SERVER_URL=https://lhci-staging.example.com
LHCI_TOKEN=           # Get from 1Password: "LHCI Staging Token"

# Test Site
TEST_SITE_URL=https://staging.example.com

# Stripe (Test Mode)
STRIPE_TEST_API_KEY=  # Get from 1Password: "Stripe Test Keys"

# Database (Staging)
DATABASE_URL=         # Get from 1Password: "Staging DB Connection"
```

---

## Pattern: CLI End-to-End Testing

Test the CLI exactly as a user would invoke it.

```typescript
// test/e2e/cli.test.ts
import { execa } from "execa";
import { requireCredentials } from "./credentials";

describe("hugolit CLI E2E", () => {
  const credentials = requireCredentials();

  test("full audit workflow succeeds", async () => {
    const result = await execa("node", ["./bin/hugolit.js", "run", "--url", credentials.testSiteUrl, "--upload"], {
      env: {
        ...process.env,
        LHCI_SERVER_URL: credentials.lhciServerUrl,
        LHCI_TOKEN: credentials.lhciToken,
        CI: "true",
      },
      reject: false, // Don't throw on non-zero exit
    });

    expect(result.exitCode).toBe(0);
    expect(result.stdout).toContain("Audit complete");
    expect(result.stdout).toContain("Results uploaded");
  });

  test("handles invalid URL gracefully", async () => {
    const result = await execa("node", ["./bin/hugolit.js", "run", "--url", "https://nonexistent.invalid/"], {
      env: { ...process.env, CI: "true" },
      reject: false,
    });

    expect(result.exitCode).not.toBe(0);
    expect(result.stderr).toContain("Failed to reach");
  });

  test("respects --dry-run flag", async () => {
    const result = await execa("node", ["./bin/hugolit.js", "run", "--url", credentials.testSiteUrl, "--upload", "--dry-run"], {
      env: {
        ...process.env,
        LHCI_SERVER_URL: credentials.lhciServerUrl,
        LHCI_TOKEN: credentials.lhciToken,
      },
      reject: false,
    });

    expect(result.exitCode).toBe(0);
    expect(result.stdout).toContain("Dry run");
    expect(result.stdout).not.toContain("Results uploaded");
  });
});
```

### Python CLI E2E

```python
# test/e2e/test_cli.py
import subprocess
import os
import pytest

def get_credentials():
    """Load credentials from environment."""
    lhci_url = os.environ.get("LHCI_SERVER_URL")
    lhci_token = os.environ.get("LHCI_TOKEN")
    test_site = os.environ.get("TEST_SITE_URL", "https://staging.example.com")

    if not lhci_url or not lhci_token:
        return None

    return {
        "lhci_url": lhci_url,
        "lhci_token": lhci_token,
        "test_site": test_site,
    }

credentials = get_credentials()
skip_no_creds = pytest.mark.skipif(
    credentials is None,
    reason="LHCI credentials not configured"
)

@skip_no_creds
def test_full_audit_workflow():
    result = subprocess.run(
        ["python", "-m", "hugolit", "run",
         "--url", credentials["test_site"],
         "--upload"],
        capture_output=True,
        text=True,
        env={
            **os.environ,
            "LHCI_SERVER_URL": credentials["lhci_url"],
            "LHCI_TOKEN": credentials["lhci_token"],
            "CI": "true",
        }
    )

    assert result.returncode == 0
    assert "Audit complete" in result.stdout
    assert "Results uploaded" in result.stdout

@skip_no_creds
def test_handles_invalid_url():
    result = subprocess.run(
        ["python", "-m", "hugolit", "run",
         "--url", "https://nonexistent.invalid/"],
        capture_output=True,
        text=True,
        env={**os.environ, "CI": "true"}
    )

    assert result.returncode != 0
    assert "Failed to reach" in result.stderr
```

---

## Pattern: Next.js E2E with Playwright

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./test/e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,

  reporter: "html",

  use: {
    baseURL: process.env.TEST_BASE_URL || "http://localhost:3000",
    trace: "on-first-retry",
  },

  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
  ],

  // Start Next.js dev server for local testing
  webServer: process.env.TEST_BASE_URL
    ? undefined
    : {
        command: "npm run dev",
        url: "http://localhost:3000",
        reuseExistingServer: !process.env.CI,
        timeout: 120000,
      },
});
```

### Playwright Page Object Pattern

```typescript
// test/e2e/pages/login-page.ts
import { Page, Locator, expect } from "@playwright/test";

type LoginPageDeps = {
  page: Page;
};

function createLoginPage({ page }: LoginPageDeps) {
  const emailInput = page.locator('[data-testid="email-input"]');
  const passwordInput = page.locator('[data-testid="password-input"]');
  const submitButton = page.locator('[data-testid="login-submit"]');
  const errorMessage = page.locator('[data-testid="login-error"]');

  return {
    async goto() {
      await page.goto("/login");
    },

    async login(email: string, password: string) {
      await emailInput.fill(email);
      await passwordInput.fill(password);
      await submitButton.click();
    },

    async expectSuccess() {
      await expect(page).toHaveURL("/dashboard");
    },

    async expectError(message: string) {
      await expect(errorMessage).toContainText(message);
    },
  };
}

export { createLoginPage };
```

### Playwright Auth Fixture

```typescript
// test/e2e/fixtures.ts
import { test as base, expect } from "@playwright/test";
import { createLoginPage } from "./pages/login-page";

/**
 * Test credentials for E2E tests.
 *
 * Where to find:
 *   - 1Password: "Engineering/E2E Test Accounts"
 *
 * The test user must exist in the staging environment.
 */
const TEST_USER = {
  email: process.env.TEST_USER_EMAIL || "",
  password: process.env.TEST_USER_PASSWORD || "",
};

function hasTestCredentials(): boolean {
  return Boolean(TEST_USER.email && TEST_USER.password);
}

// Extend Playwright test with authenticated fixture
const test = base.extend<{
  authenticatedPage: Awaited<ReturnType<(typeof base)["page"]>>;
}>({
  authenticatedPage: async ({ page }, use) => {
    if (!hasTestCredentials()) {
      throw new Error("E2E test credentials not configured.\n" + "Set TEST_USER_EMAIL and TEST_USER_PASSWORD environment variables.\n" + "See test/e2e/fixtures.ts for details.");
    }

    const loginPage = createLoginPage({ page });
    await loginPage.goto();
    await loginPage.login(TEST_USER.email, TEST_USER.password);
    await loginPage.expectSuccess();

    await use(page);
  },
});

export { test, expect, hasTestCredentials };
```

### Full Workflow Tests

```typescript
// test/e2e/audit-workflow.test.ts
import { test, expect, hasTestCredentials } from "./fixtures";
import { createDashboardPage } from "./pages/dashboard-page";
import { createAuditPage } from "./pages/audit-page";

test.describe("Audit Workflow", () => {
  test.skip(!hasTestCredentials(), "Requires test credentials");

  test("user can create and view audit", async ({ authenticatedPage }) => {
    const dashboard = createDashboardPage({ page: authenticatedPage });
    const auditPage = createAuditPage({ page: authenticatedPage });

    // Navigate to create audit
    await dashboard.goto();
    await dashboard.clickNewAudit();

    // Fill audit form
    await auditPage.fillUrl("https://staging.example.com/");
    await auditPage.selectPreset("desktop");
    await auditPage.submit();

    // Wait for audit to complete
    await auditPage.waitForComplete({ timeout: 60000 });

    // Verify results displayed
    await expect(auditPage.performanceScore).toBeVisible();
    await expect(auditPage.accessibilityScore).toBeVisible();
  });

  test("user can view historical audits", async ({ authenticatedPage }) => {
    const dashboard = createDashboardPage({ page: authenticatedPage });

    await dashboard.goto();

    // Verify audit history is displayed
    const auditCount = await dashboard.getAuditCount();
    expect(auditCount).toBeGreaterThan(0);

    // Click first audit
    await dashboard.clickAudit(0);

    // Verify detail page loads
    await expect(authenticatedPage).toHaveURL(/\/audits\//);
  });
});
```

---

## Pattern: MCP Server E2E Testing

Test MCP servers by acting as a client.

```typescript
// test/e2e/mcp-server.test.ts
import { spawn, ChildProcess } from "child_process";
import { requireCredentials } from "./credentials";

type MCPMessage = {
  jsonrpc: "2.0";
  id?: number;
  method?: string;
  params?: unknown;
  result?: unknown;
  error?: { code: number; message: string };
};

async function createMCPClient(command: string, args: string[]) {
  let process: ChildProcess | null = null;
  let messageId = 0;
  const pendingRequests = new Map<
    number,
    {
      resolve: (result: unknown) => void;
      reject: (error: Error) => void;
    }
  >();

  return {
    async start() {
      process = spawn(command, args, {
        stdio: ["pipe", "pipe", "pipe"],
        env: { ...process.env },
      });

      let buffer = "";

      process.stdout!.on("data", (data: Buffer) => {
        buffer += data.toString();

        // Parse JSON-RPC messages (newline-delimited)
        const lines = buffer.split("\n");
        buffer = lines.pop() || "";

        for (const line of lines) {
          if (!line.trim()) continue;
          try {
            const msg: MCPMessage = JSON.parse(line);
            if (msg.id !== undefined && pendingRequests.has(msg.id)) {
              const { resolve, reject } = pendingRequests.get(msg.id)!;
              pendingRequests.delete(msg.id);

              if (msg.error) {
                reject(new Error(msg.error.message));
              } else {
                resolve(msg.result);
              }
            }
          } catch (e) {
            console.error("Failed to parse MCP message:", line);
          }
        }
      });

      // Wait for server to be ready
      await new Promise((r) => setTimeout(r, 1000));
    },

    async request(method: string, params?: unknown): Promise<unknown> {
      if (!process) throw new Error("MCP client not started");

      const id = ++messageId;
      const message: MCPMessage = {
        jsonrpc: "2.0",
        id,
        method,
        params,
      };

      return new Promise((resolve, reject) => {
        pendingRequests.set(id, { resolve, reject });
        process!.stdin!.write(JSON.stringify(message) + "\n");

        // Timeout after 30 seconds
        setTimeout(() => {
          if (pendingRequests.has(id)) {
            pendingRequests.delete(id);
            reject(new Error(`Request ${method} timed out`));
          }
        }, 30000);
      });
    },

    async stop() {
      if (process) {
        process.kill();
        process = null;
      }
    },
  };
}

describe("MCP Server E2E", () => {
  const credentials = requireCredentials();
  let client: Awaited<ReturnType<typeof createMCPClient>>;

  beforeAll(async () => {
    client = await createMCPClient("node", ["./bin/mcp-server.js"]);
    await client.start();
  });

  afterAll(async () => {
    await client.stop();
  });

  test("lists available tools", async () => {
    const result = (await client.request("tools/list")) as { tools: Array<{ name: string }> };

    expect(result.tools).toContainEqual(expect.objectContaining({ name: "run_audit" }));
  });

  test("run_audit tool returns scores", async () => {
    const result = (await client.request("tools/call", {
      name: "run_audit",
      arguments: {
        urls: [credentials.testSiteUrl],
        preset: "desktop",
      },
    })) as { content: Array<{ text: string }> };

    // Parse the response
    const text = result.content[0].text;
    const data = JSON.parse(text);

    expect(data.results).toHaveLength(1);
    expect(data.results[0].scores.performance).toBeGreaterThan(0);
  });
});
```

---

## Pattern: Test Data Isolation

Ensure E2E tests don't affect other tests or pollute shared environments.

```typescript
// test/e2e/isolation.ts
import { randomBytes } from "crypto";

/**
 * Generate a unique test run ID.
 * Use this to prefix test data so it can be identified and cleaned up.
 */
function createTestRunId(): string {
  const timestamp = Date.now().toString(36);
  const random = randomBytes(4).toString("hex");
  return `test-${timestamp}-${random}`;
}

/**
 * Create test data with identifiable prefix.
 */
function createTestUser(testRunId: string, overrides = {}) {
  return {
    email: `${testRunId}@test.example.com`,
    name: `Test User ${testRunId}`,
    ...overrides,
  };
}

// Usage in tests
test("creates user successfully", async ({ page }) => {
  const testRunId = createTestRunId();
  const testUser = createTestUser(testRunId);

  // ... create user in UI

  // Cleanup: delete test user after test
  // This is handled by the afterEach hook
});
```

### Cleanup Pattern

```typescript
// test/e2e/setup.ts
import { FullConfig } from "@playwright/test";

async function globalTeardown(config: FullConfig) {
  // Clean up test data from staging environment
  const testDataPrefix = process.env.TEST_RUN_ID;

  if (testDataPrefix && process.env.CLEANUP_TEST_DATA === "true") {
    console.log(`Cleaning up test data with prefix: ${testDataPrefix}`);

    // Call your API to delete test data
    await fetch(`${process.env.TEST_API_URL}/admin/cleanup`, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${process.env.ADMIN_API_KEY}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ prefix: testDataPrefix }),
    });
  }
}

export default globalTeardown;
```

---

## Pattern: CI Configuration

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    # Run nightly to catch third-party API changes
    - cron: "0 6 * * *"

jobs:
  e2e:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium

      - name: Run E2E tests
        run: npm run test:e2e
        env:
          # Credentials from GitHub Secrets
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          LHCI_SERVER_URL: ${{ secrets.LHCI_SERVER_URL }}
          LHCI_TOKEN: ${{ secrets.LHCI_TOKEN }}
          TEST_BASE_URL: https://staging.example.com
          TEST_RUN_ID: ${{ github.run_id }}

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7
```

---

## When Level 3 Tests Fail

Level 3 failures are the most serious. Debug in this order:

### 1. Check Credentials

```typescript
// Add to your test setup
beforeAll(async () => {
  const creds = loadCredentials();

  if (creds) {
    // Verify credentials are valid
    try {
      const response = await fetch(`${creds.lhciServerUrl}/healthcheck`, {
        headers: { Authorization: `Bearer ${creds.lhciToken}` },
      });
      if (!response.ok) {
        console.error("⚠️  Credentials may be expired or invalid");
      }
    } catch (e) {
      console.error("⚠️  Cannot reach LHCI server:", e);
    }
  }
});
```

### 2. Check Third-Party Status

```typescript
// test/e2e/health-checks.test.ts
test.describe("Health Checks", () => {
  test("LHCI server is reachable", async () => {
    const response = await fetch(process.env.LHCI_SERVER_URL + "/healthcheck");
    expect(response.ok).toBe(true);
  });

  test("staging site is reachable", async () => {
    const response = await fetch(process.env.TEST_SITE_URL);
    expect(response.ok).toBe(true);
  });
});
```

### 3. Check Test Data

Did someone manually modify the test account? Did seed data get corrupted?

### 4. Then Look at Your Code

Only after ruling out external factors, investigate your code changes.

---

## Checklist: Is This a Level 3 Test?

Before writing the test, verify:

- [ ] Credentials are documented (not hardcoded)
- [ ] Test accounts exist and are identified
- [ ] Test can skip gracefully if credentials missing
- [ ] Test data is isolated (won't affect other tests/users)
- [ ] Cleanup strategy is defined
- [ ] Timeout is appropriate for real services

---

## What Level 3 Proves

✅ Real credentials work and haven't expired  
✅ Third-party APIs behave as expected  
✅ The full user workflow succeeds  
✅ All integrations work together in production-like environment  
✅ The system works as users will experience it

## When Level 3 Fails

A Level 3 failure means **users are likely experiencing failures**. This is the highest priority to fix.

---

## Level 3 Anti-Patterns

### Anti-Pattern: Hardcoded Credentials

```typescript
// ❌ NEVER DO THIS
const stripe = new Stripe("sk_test_abc123xyz"); // Hardcoded!

// ✅ ALWAYS load from environment
const stripe = new Stripe(process.env.STRIPE_TEST_API_KEY!);
```

### Anti-Pattern: Using Production Credentials

```typescript
// ❌ NEVER DO THIS
// "I'll just use my real account for testing"
const api = createClient({
  apiKey: process.env.PRODUCTION_API_KEY, // NO!
});

// ✅ Use dedicated test accounts/environments
const api = createClient({
  apiKey: process.env.TEST_API_KEY,
  baseUrl: process.env.TEST_API_URL, // Staging, not production
});
```

### Anti-Pattern: Tests That Modify Shared State

```typescript
// ❌ Tests that step on each other
test("creates user", async () => {
  await api.createUser({ email: "test@example.com" }); // Same email every time!
});

// ✅ Isolated test data
test("creates user", async () => {
  const testId = createTestRunId();
  await api.createUser({ email: `${testId}@test.example.com` });
});
```

### Anti-Pattern: No Cleanup

```typescript
// ❌ Leaves test data behind
test("creates resource", async () => {
  await api.createResource({ name: "test" });
  // Resource lives forever, cluttering the test environment
});

// ✅ Clean up after yourself
test("creates resource", async () => {
  const resource = await api.createResource({ name: `test-${testRunId}` });

  // ... assertions

  // Cleanup
  await api.deleteResource(resource.id);
});
```
