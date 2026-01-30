# Level 2: Integration Tests

**Speed**: <1s | **Infrastructure**: Real binaries (Hugo, Caddy) | **Framework**: Vitest

Level 2 tests verify that real tools work together in a controlled local environment.

## What Level 2 Provides

| Component  | What's Real            | What's Controlled     |
| ---------- | ---------------------- | --------------------- |
| Hugo       | Real Hugo binary       | Sample test site      |
| Caddy      | Real Caddy server      | Ephemeral, local port |
| Filesystem | Real file operations   | Temp directories      |
| Subprocess | Real command execution | Our code invokes it   |

## Why Level 2 Exists

Level 1 proves our logic is correct. Level 2 proves:

- Hugo actually accepts our commands
- Hugo actually creates the output files
- Caddy actually serves files
- File operations work as expected

---

## The Critical Requirement: Test Harnesses

> **Before writing ANY Level 2 test, you must identify or build the test harness for each external dependency.**

### 🚨 THE RULE: If You Don't Know the Harness, STOP

> **If you cannot describe the test harness for a dependency, you MUST ask the user before proceeding.**

Do not guess. Do not assume. Ask:

```
I need to write integration tests for [dependency].

To proceed, I need to know:
1. What test harness exists or should I build?
2. How do I start/stop/reset it?
3. Where are fixture files or seed data?
4. What environment variables configure it?

Please provide this information or point me to existing test infrastructure.
```

### What Is a Test Harness?

A test harness is the infrastructure that lets you run tests against a real dependency in a controlled, repeatable way.

| Dependency Type | Harness Examples                                              |
| --------------- | ------------------------------------------------------------- |
| Hugo binary     | Installed binary with known version, fixture Hugo sites       |
| Caddy server    | Installed binary, ephemeral Caddyfile, auto-allocated ports   |
| Docker service  | docker-compose.test.yml with service definitions              |
| TypeScript      | Project's tsconfig.json + known fixture files                 |
| Database        | Docker container with test schema, or test database + cleanup |

---

## File Location & Naming (CODE Framework)

Tests are co-located with specs in `spx/`. Level is indicated by suffix naming:

```
spx/{capability}/{feature}/
├── {feature}.md                      # Feature spec
└── tests/
    ├── lhci.unit.test.ts            # Level 1
    ├── hugo.integration.test.ts      # Level 2
    ├── caddy.integration.test.ts     # Level 2
    └── lhci.e2e.test.ts              # Level 3

tests/fixtures/                       # Shared fixtures (project root)
└── sample-hugo-site/
```

---

## Infrastructure Setup

### Sample Hugo Site Fixture

Create a minimal Hugo site for testing:

```
test/fixtures/sample-hugo-site/
├── hugo.toml
├── content/
│   ├── _index.md
│   └── about.md
├── layouts/
│   └── _default/
│       ├── baseof.html
│       └── single.html
└── static/
    └── .gitkeep
```

```toml
# test/fixtures/sample-hugo-site/hugo.toml
baseURL = 'http://localhost:1313/'
languageCode = 'en-us'
title = 'Test Site'
```

### Skip Markers for CI

```typescript
// spx/.../tests/conftest.ts
import { execaSync } from "execa";

export function hugoAvailable(): boolean {
  try {
    execaSync("hugo", ["version"]);
    return true;
  } catch {
    return false;
  }
}

export function caddyAvailable(): boolean {
  try {
    execaSync("caddy", ["version"]);
    return true;
  } catch {
    return false;
  }
}
```

```typescript
// spx/.../tests/hugo.integration.test.ts
import { beforeAll, describe, expect, it } from "vitest";
import { hugoAvailable } from "./conftest";

describe.skipIf(!hugoAvailable())("Hugo Integration", () => {
  // Tests here...
});
```

---

## Good Patterns

### Pattern: Testing Real Hugo Builds

```typescript
// spx/.../tests/hugo.integration.test.ts
import { execa } from "execa";
import * as fs from "fs";
import * as os from "os";
import * as path from "path";
import { afterEach, beforeEach, describe, expect, it } from "vitest";
import { hugoAvailable } from "./conftest";

describe.skipIf(!hugoAvailable())("Hugo Build Integration", () => {
  const fixturePath = path.join(__dirname, "fixtures", "sample-hugo-site");
  let tempDir: string;

  beforeEach(async () => {
    tempDir = await fs.promises.mkdtemp(path.join(os.tmpdir(), "hugolit-int-"));
  });

  afterEach(async () => {
    await fs.promises.rm(tempDir, { recursive: true, force: true });
  });

  it("GIVEN valid site WHEN building THEN creates index.html", async () => {
    // When
    const result = await execa("hugo", ["--destination", tempDir], {
      cwd: fixturePath,
    });

    // Then
    expect(result.exitCode).toBe(0);
    expect(fs.existsSync(path.join(tempDir, "index.html"))).toBe(true);
  });

  it("GIVEN minify flag WHEN building THEN output is minified", async () => {
    // When
    await execa("hugo", ["--destination", tempDir, "--minify"], {
      cwd: fixturePath,
    });

    // Then
    const html = await fs.promises.readFile(
      path.join(tempDir, "index.html"),
      "utf-8",
    );
    // Minified HTML has fewer newlines
    expect(html.split("\n").length).toBeLessThan(10);
  });

  it("GIVEN about page WHEN building THEN creates about/index.html", async () => {
    // When
    await execa("hugo", ["--destination", tempDir], { cwd: fixturePath });

    // Then
    expect(fs.existsSync(path.join(tempDir, "about", "index.html"))).toBe(true);
  });
});
```

### Pattern: Testing Real Caddy Server

```typescript
// spx/.../tests/caddy.integration.test.ts
import { execa, type ExecaChildProcess } from "execa";
import * as fs from "fs";
import getPort from "get-port";
import * as os from "os";
import * as path from "path";
import { afterEach, beforeEach, describe, expect, it } from "vitest";
import { caddyAvailable } from "./conftest";

describe.skipIf(!caddyAvailable())("Caddy Server Integration", () => {
  let tempDir: string;
  let caddyProcess: ExecaChildProcess | null = null;
  let port: number;

  beforeEach(async () => {
    tempDir = await fs.promises.mkdtemp(path.join(os.tmpdir(), "caddy-test-"));
    port = await getPort();

    // Create test file
    await fs.promises.writeFile(
      path.join(tempDir, "index.html"),
      "<html><body>Test</body></html>",
    );

    // Create Caddyfile
    const caddyfile = `:${port} {
  root * ${tempDir}
  file_server
}`;
    await fs.promises.writeFile(path.join(tempDir, "Caddyfile"), caddyfile);
  });

  afterEach(async () => {
    if (caddyProcess) {
      caddyProcess.kill();
      caddyProcess = null;
    }
    await fs.promises.rm(tempDir, { recursive: true, force: true });
  });

  it("GIVEN static files WHEN starting Caddy THEN serves index.html", async () => {
    // Start Caddy
    caddyProcess = execa("caddy", [
      "run",
      "--config",
      path.join(tempDir, "Caddyfile"),
    ]);

    // Wait for server to start
    await new Promise((resolve) => setTimeout(resolve, 500));

    // Fetch
    const response = await fetch(`http://localhost:${port}/`);
    const body = await response.text();

    expect(response.ok).toBe(true);
    expect(body).toContain("Test");
  });

  it("GIVEN nested file WHEN requesting THEN serves correct file", async () => {
    // Create nested structure
    await fs.promises.mkdir(path.join(tempDir, "about"));
    await fs.promises.writeFile(
      path.join(tempDir, "about", "index.html"),
      "<html><body>About</body></html>",
    );

    // Start Caddy
    caddyProcess = execa("caddy", [
      "run",
      "--config",
      path.join(tempDir, "Caddyfile"),
    ]);
    await new Promise((resolve) => setTimeout(resolve, 500));

    // Fetch
    const response = await fetch(`http://localhost:${port}/about/`);
    const body = await response.text();

    expect(body).toContain("About");
  });
});
```

### Pattern: Testing Error Conditions

```typescript
// spx/.../tests/hugo-errors.integration.test.ts
import { execa } from "execa";
import * as path from "path";
import { describe, expect, it } from "vitest";
import { hugoAvailable } from "./conftest";

describe.skipIf(!hugoAvailable())("Hugo Error Handling", () => {
  it("GIVEN non-existent site WHEN building THEN returns non-zero exit", async () => {
    // When
    const result = await execa("hugo", ["--source", "/non/existent/path"], {
      reject: false,
    });

    // Then
    expect(result.exitCode).not.toBe(0);
  });

  it("GIVEN invalid config WHEN building THEN error contains helpful message", async () => {
    const invalidSite = path.join(__dirname, "fixtures", "invalid-hugo-site");

    // When
    const result = await execa("hugo", ["--source", invalidSite], {
      reject: false,
    });

    // Then
    expect(result.exitCode).not.toBe(0);
    expect(result.stderr.toLowerCase()).toMatch(/config|toml|yaml/);
  });
});
```

### Pattern: Testing Unicode and Special Paths

```typescript
// spx/.../tests/unicode.integration.test.ts
import * as fs from "fs";
import * as os from "os";
import * as path from "path";
import { afterEach, beforeEach, describe, expect, it } from "vitest";

describe("Unicode Path Handling", () => {
  let tempDir: string;

  beforeEach(async () => {
    tempDir = await fs.promises.mkdtemp(path.join(os.tmpdir(), "unicode-"));
  });

  afterEach(async () => {
    await fs.promises.rm(tempDir, { recursive: true, force: true });
  });

  const unicodePaths = [
    "статьи", // Russian
    "文章", // Chinese
    "מאמרים", // Hebrew
    "file with spaces",
    "file'with'quotes",
  ];

  it.each(unicodePaths)(
    "GIVEN path \"%s\" WHEN creating THEN file exists",
    async (filename) => {
      const filePath = path.join(tempDir, filename);

      await fs.promises.writeFile(filePath, "content");

      expect(fs.existsSync(filePath)).toBe(true);
      expect(await fs.promises.readFile(filePath, "utf-8")).toBe("content");
    },
  );
});
```

---

## When to Escalate to Level 3

Escalate when you need to verify behavior that requires **real network services or Chrome**:

| Behavior               | Level 2 Sufficient? | Why                            |
| ---------------------- | ------------------- | ------------------------------ |
| Hugo builds site       | ✅ Yes              | Local execution, no network    |
| Caddy serves files     | ✅ Yes              | Local server, no external deps |
| Lighthouse runs audits | ❌ No               | Requires Chrome                |
| LHCI collects metrics  | ❌ No               | Requires Chrome + network      |
| Report generation      | ❌ No               | Needs real audit data          |

### Escalation Decision

```typescript
// Level 2 is sufficient—testing Hugo behavior
it("GIVEN valid site WHEN building THEN creates output", async () => {
  const result = await execa("hugo", ["--destination", tempDir], {
    cwd: fixturePath,
  });
  expect(result.exitCode).toBe(0);
});

// Level 3 required—testing Lighthouse behavior
it("GIVEN built site WHEN running Lighthouse THEN returns scores", async () => {
  // Requires Chrome, so this is Level 3
  const result = await runLighthouse({ url: `http://localhost:${port}/` });
  expect(result.scores.performance).toBeGreaterThan(0);
});
```

---

## Performance Expectations

Level 2 tests should complete in **<1 second** each:

```typescript
it("GIVEN small site WHEN building THEN completes quickly", async () => {
  const start = performance.now();

  await execa("hugo", ["--destination", tempDir], { cwd: fixturePath });

  const duration = performance.now() - start;
  expect(duration).toBeLessThan(1000); // < 1 second
});
```

---

## Completion Requirements

Level 2 tests prove **real tools work together**:

| Work Item  | Level 2 Proves             | But Cannot Prove        |
| ---------- | -------------------------- | ----------------------- |
| Story      | Real tools accept commands | Network services work   |
| Feature    | Local integration works    | Chrome-based tools work |
| Capability | N/A                        | Needs Level 3+          |

**Use Level 2 for**: Hugo builds, Caddy serving, file permissions, error handling with real binaries.

---

## Anti-Patterns

### Anti-Pattern: Skipping Integration Tests

```typescript
// ❌ Don't skip integration tests because they're "slow"
it.skip("hugo builds site", async () => {
  // "Too slow, will test manually"
});

// ✅ Level 2 tests should be fast (<1s)
it("GIVEN valid site WHEN building THEN succeeds", async () => {
  const result = await execa("hugo", ["--destination", tempDir], {
    cwd: fixturePath,
  });
  expect(result.exitCode).toBe(0);
});
```

### Anti-Pattern: Using Level 2 When Level 1 Suffices

```typescript
// ❌ This doesn't need real Hugo
it("GIVEN minify flag WHEN building command THEN includes --minify", async () => {
  const result = await execa("hugo", ["--version"]); // Unnecessary!
  const cmd = buildHugoCommand({ minify: true });
  expect(cmd).toContain("--minify");
});

// ✅ Level 1 is sufficient
it("GIVEN minify flag WHEN building command THEN includes --minify", () => {
  const cmd = buildHugoCommand({ minify: true });
  expect(cmd).toContain("--minify");
});
```

### Anti-Pattern: Non-Ephemeral Test Data

```typescript
// ❌ Leaves test data behind
it("creates output", async () => {
  await execa("hugo", ["--destination", "test-output"]);
  // No cleanup!
});

// ✅ Use temp directories
it("creates output", async () => {
  const tempDir = await fs.promises.mkdtemp("/tmp/hugo-test-");
  try {
    await execa("hugo", ["--destination", tempDir]);
  } finally {
    await fs.promises.rm(tempDir, { recursive: true });
  }
});
```

---

*Level 2 is where theory meets reality. If it works here, you have real confidence.*
