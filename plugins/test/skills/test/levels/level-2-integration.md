# Level 2: Integration

## The Question This Level Answers

> **"Does our code correctly interact with real external dependencies?"**

Level 1 proved your logic is correct in isolation. Level 2 proves your code actually works with the real databases, binaries, file systems, and local services it depends on.

---

## 🚨 The Harness Requirement

> **Before writing ANY Level 2 test, you must identify or build the test harness for each external dependency.**

A test harness is the infrastructure that lets you run tests against a real dependency in a controlled, repeatable, isolated way.

### If You Don't Know the Harness, STOP

Do not guess. Do not assume. Ask the user:

```
I need to write integration tests for [dependency].

To proceed, I need to know:
1. What test harness exists or should I build?
2. How do I start/stop/reset it between tests?
3. Where are fixture files or seed data located?
4. What environment variables configure it?
5. Is there an existing docker-compose.test.yml or similar?

Please provide this information or point me to existing test infrastructure.
```

---

## Harness Categories

| Dependency Type | Harness Strategy | Reset Strategy |
|-----------------|------------------|----------------|
| **Database** | Docker container or test DB | Truncate tables / drop schema |
| **CLI Binary** | Installed binary + temp dirs | Delete temp dirs |
| **HTTP Service** | Local server or test container | Restart or clear state |
| **File System** | Temp directories | Delete on teardown |
| **Message Queue** | Docker container | Purge queues |

---

## Pattern: Harness Directory Structure

```
test/
├── harnesses/
│   ├── index.ts              # Re-exports all harnesses
│   ├── postgres.ts           # Postgres container harness
│   ├── hugo.ts               # Hugo binary harness
│   ├── caddy.ts              # Caddy server harness
│   └── temp-site.ts          # Temp Hugo site generator
├── fixtures/
│   ├── sample-site/          # Minimal Hugo site for tests
│   │   ├── config.toml
│   │   └── content/
│   ├── seed-data.sql         # Database seed data
│   └── test-config.yaml      # Test configuration
├── integration/
│   ├── database.test.ts
│   ├── hugo-build.test.ts
│   └── caddy-server.test.ts
└── setup.ts                  # Global test setup
```

---

## Pattern: Hugo Binary Harness

```typescript
// test/harnesses/hugo.ts
import { execa, ExecaReturnValue } from 'execa'
import { mkdtemp, rm, cp } from 'fs/promises'
import { join } from 'path'
import { tmpdir } from 'os'

type HugoHarness = {
  siteDir: string
  outputDir: string
  build: (args?: string[]) => Promise<ExecaReturnValue>
  cleanup: () => Promise<void>
}

async function verifyHugoInstalled(): Promise<void> {
  try {
    await execa('hugo', ['version'])
  } catch {
    throw new Error(
      'Hugo binary not found. Install Hugo or skip integration tests.\n' +
      'Install: https://gohugo.io/installation/'
    )
  }
}

async function createHugoHarness(
  fixturePath?: string
): Promise<HugoHarness> {
  await verifyHugoInstalled()
  
  const siteDir = await mkdtemp(join(tmpdir(), 'hugo-test-site-'))
  const outputDir = join(siteDir, 'public')
  
  // Copy fixture or create minimal site
  if (fixturePath) {
    await cp(fixturePath, siteDir, { recursive: true })
  } else {
    await createMinimalSite(siteDir)
  }
  
  return {
    siteDir,
    outputDir,
    
    async build(args: string[] = []) {
      return execa('hugo', [
        '--source', siteDir,
        '--destination', outputDir,
        ...args
      ])
    },
    
    async cleanup() {
      await rm(siteDir, { recursive: true, force: true })
    }
  }
}

async function createMinimalSite(dir: string): Promise<void> {
  const { writeFile, mkdir } = await import('fs/promises')
  
  await writeFile(join(dir, 'config.toml'), `
title = "Test Site"
baseURL = "http://localhost:1313/"
`)
  
  await mkdir(join(dir, 'content'), { recursive: true })
  await writeFile(join(dir, 'content', '_index.md'), `
---
title: "Home"
---

# Welcome

This is a test site.
`)
  
  await mkdir(join(dir, 'layouts', '_default'), { recursive: true })
  await writeFile(join(dir, 'layouts', '_default', 'baseof.html'), `
<!DOCTYPE html>
<html>
<head><title>{{ .Title }}</title></head>
<body>{{ block "main" . }}{{ end }}</body>
</html>
`)
  await writeFile(join(dir, 'layouts', '_default', 'list.html'), `
{{ define "main" }}{{ .Content }}{{ end }}
`)
  await writeFile(join(dir, 'layouts', '_default', 'single.html'), `
{{ define "main" }}{{ .Content }}{{ end }}
`)
}

export { createHugoHarness, verifyHugoInstalled }
```

### Using the Hugo Harness

```typescript
// test/integration/hugo-build.test.ts
import { createHugoHarness } from '../harnesses/hugo'
import { existsSync, readFileSync } from 'fs'
import { join } from 'path'

describe('Hugo Build Integration', () => {
  test('builds minimal site successfully', async () => {
    const harness = await createHugoHarness()
    
    try {
      const result = await harness.build()
      
      expect(result.exitCode).toBe(0)
      expect(existsSync(join(harness.outputDir, 'index.html'))).toBe(true)
    } finally {
      await harness.cleanup()
    }
  })
  
  test('builds with minify flag', async () => {
    const harness = await createHugoHarness()
    
    try {
      const result = await harness.build(['--minify'])
      
      expect(result.exitCode).toBe(0)
      
      const html = readFileSync(
        join(harness.outputDir, 'index.html'), 
        'utf-8'
      )
      // Minified HTML has no unnecessary whitespace
      expect(html).not.toMatch(/\n\s+\n/)
    } finally {
      await harness.cleanup()
    }
  })
  
  test('fails on invalid config', async () => {
    const harness = await createHugoHarness()
    const { writeFile } = await import('fs/promises')
    
    // Corrupt the config
    await writeFile(
      join(harness.siteDir, 'config.toml'), 
      'invalid toml {{{'
    )
    
    try {
      const result = await harness.build().catch(e => e)
      
      expect(result.exitCode).not.toBe(0)
    } finally {
      await harness.cleanup()
    }
  })
})
```

---

## Pattern: Caddy Server Harness

```typescript
// test/harnesses/caddy.ts
import { execa, ExecaChildProcess } from 'execa'
import { writeFile, mkdtemp, rm } from 'fs/promises'
import { join } from 'path'
import { tmpdir } from 'os'
import getPort from 'get-port'

type CaddyHarness = {
  port: number
  baseUrl: string
  rootDir: string
  start: () => Promise<void>
  stop: () => Promise<void>
  cleanup: () => Promise<void>
}

async function verifyCaddyInstalled(): Promise<void> {
  try {
    await execa('caddy', ['version'])
  } catch {
    throw new Error(
      'Caddy binary not found. Install Caddy or skip integration tests.\n' +
      'Install: https://caddyserver.com/docs/install'
    )
  }
}

async function createCaddyHarness(staticDir: string): Promise<CaddyHarness> {
  await verifyCaddyInstalled()
  
  const port = await getPort()
  const configDir = await mkdtemp(join(tmpdir(), 'caddy-test-'))
  const caddyfilePath = join(configDir, 'Caddyfile')
  
  let process: ExecaChildProcess | null = null
  
  return {
    port,
    baseUrl: `http://localhost:${port}`,
    rootDir: staticDir,
    
    async start() {
      await writeFile(caddyfilePath, `
:${port} {
  root * ${staticDir}
  file_server
  log {
    output discard
  }
}
`)
      
      process = execa('caddy', ['run', '--config', caddyfilePath], {
        reject: false
      })
      
      // Wait for server to be ready
      await waitForServer(`http://localhost:${port}`, 5000)
    },
    
    async stop() {
      if (process) {
        process.kill()
        await process.catch(() => {}) // Ignore kill errors
        process = null
      }
    },
    
    async cleanup() {
      await this.stop()
      await rm(configDir, { recursive: true, force: true })
    }
  }
}

async function waitForServer(url: string, timeoutMs: number): Promise<void> {
  const start = Date.now()
  
  while (Date.now() - start < timeoutMs) {
    try {
      const response = await fetch(url)
      if (response.ok || response.status === 404) {
        return // Server is up
      }
    } catch {
      // Server not ready yet
    }
    await new Promise(r => setTimeout(r, 100))
  }
  
  throw new Error(`Server at ${url} did not start within ${timeoutMs}ms`)
}

export { createCaddyHarness, verifyCaddyInstalled }
```

### Using the Caddy Harness

```typescript
// test/integration/caddy-server.test.ts
import { createHugoHarness } from '../harnesses/hugo'
import { createCaddyHarness } from '../harnesses/caddy'

describe('Caddy Server Integration', () => {
  test('serves Hugo-built site', async () => {
    // Build the site first
    const hugo = await createHugoHarness()
    await hugo.build()
    
    // Serve with Caddy
    const caddy = await createCaddyHarness(hugo.outputDir)
    
    try {
      await caddy.start()
      
      const response = await fetch(`${caddy.baseUrl}/`)
      const html = await response.text()
      
      expect(response.status).toBe(200)
      expect(html).toContain('Welcome')
    } finally {
      await caddy.cleanup()
      await hugo.cleanup()
    }
  })
  
  test('returns 404 for missing pages', async () => {
    const hugo = await createHugoHarness()
    await hugo.build()
    
    const caddy = await createCaddyHarness(hugo.outputDir)
    
    try {
      await caddy.start()
      
      const response = await fetch(`${caddy.baseUrl}/nonexistent-page/`)
      
      expect(response.status).toBe(404)
    } finally {
      await caddy.cleanup()
      await hugo.cleanup()
    }
  })
})
```

---

## Pattern: Database Harness (PostgreSQL)

```typescript
// test/harnesses/postgres.ts
import { Pool } from 'pg'
import { execa } from 'execa'
import { readFile } from 'fs/promises'

type PostgresHarness = {
  connectionString: string
  pool: Pool
  query: <T>(sql: string, params?: unknown[]) => Promise<T[]>
  seed: (sqlFile: string) => Promise<void>
  reset: () => Promise<void>
  cleanup: () => Promise<void>
}

const TEST_DB_CONFIG = {
  host: process.env.TEST_DB_HOST || 'localhost',
  port: parseInt(process.env.TEST_DB_PORT || '5432'),
  database: process.env.TEST_DB_NAME || 'test_db',
  user: process.env.TEST_DB_USER || 'postgres',
  password: process.env.TEST_DB_PASSWORD || 'postgres',
}

async function createPostgresHarness(): Promise<PostgresHarness> {
  // Verify database is available
  const connectionString = 
    `postgresql://${TEST_DB_CONFIG.user}:${TEST_DB_CONFIG.password}@` +
    `${TEST_DB_CONFIG.host}:${TEST_DB_CONFIG.port}/${TEST_DB_CONFIG.database}`
  
  const pool = new Pool(TEST_DB_CONFIG)
  
  try {
    await pool.query('SELECT 1')
  } catch (error) {
    throw new Error(
      `Cannot connect to test database.\n` +
      `Connection string: ${connectionString}\n` +
      `Start database: docker-compose -f docker-compose.test.yml up -d postgres\n` +
      `Error: ${error}`
    )
  }
  
  return {
    connectionString,
    pool,
    
    async query<T>(sql: string, params?: unknown[]): Promise<T[]> {
      const result = await pool.query(sql, params)
      return result.rows as T[]
    },
    
    async seed(sqlFile: string): Promise<void> {
      const sql = await readFile(sqlFile, 'utf-8')
      await pool.query(sql)
    },
    
    async reset(): Promise<void> {
      // Drop and recreate public schema
      await pool.query(`
        DROP SCHEMA public CASCADE;
        CREATE SCHEMA public;
        GRANT ALL ON SCHEMA public TO ${TEST_DB_CONFIG.user};
      `)
    },
    
    async cleanup(): Promise<void> {
      await pool.end()
    }
  }
}

export { createPostgresHarness, TEST_DB_CONFIG }
```

### Using the Postgres Harness

```typescript
// test/integration/database.test.ts
import { createPostgresHarness } from '../harnesses/postgres'

describe('User Repository Integration', () => {
  let db: Awaited<ReturnType<typeof createPostgresHarness>>
  
  beforeAll(async () => {
    db = await createPostgresHarness()
    await db.seed('test/fixtures/schema.sql')
  })
  
  afterAll(async () => {
    await db.cleanup()
  })
  
  beforeEach(async () => {
    // Clear data but keep schema
    await db.query('TRUNCATE users CASCADE')
  })
  
  test('creates and retrieves user', async () => {
    const repo = createUserRepository(db.pool)
    
    const created = await repo.create({
      email: 'test@example.com',
      name: 'Test User'
    })
    
    const retrieved = await repo.findById(created.id)
    
    expect(retrieved).toEqual({
      id: created.id,
      email: 'test@example.com',
      name: 'Test User',
    })
  })
  
  test('returns null for nonexistent user', async () => {
    const repo = createUserRepository(db.pool)
    
    const result = await repo.findById('nonexistent-id')
    
    expect(result).toBeNull()
  })
  
  test('enforces unique email constraint', async () => {
    const repo = createUserRepository(db.pool)
    
    await repo.create({ email: 'dupe@example.com', name: 'First' })
    
    await expect(
      repo.create({ email: 'dupe@example.com', name: 'Second' })
    ).rejects.toThrow(/unique/)
  })
})
```

---

## Pattern: Python Binary Harness

```python
# test/harnesses/hugo.py
import subprocess
import tempfile
import shutil
from pathlib import Path
from dataclasses import dataclass
from typing import Optional

@dataclass
class HugoResult:
    exit_code: int
    stdout: str
    stderr: str

@dataclass
class HugoHarness:
    site_dir: Path
    output_dir: Path
    _temp_dir: Optional[tempfile.TemporaryDirectory] = None
    
    def build(self, args: list[str] = None) -> HugoResult:
        args = args or []
        result = subprocess.run(
            ["hugo", "--source", str(self.site_dir), 
             "--destination", str(self.output_dir)] + args,
            capture_output=True,
            text=True
        )
        return HugoResult(
            exit_code=result.returncode,
            stdout=result.stdout,
            stderr=result.stderr
        )
    
    def cleanup(self):
        if self._temp_dir:
            self._temp_dir.cleanup()

def verify_hugo_installed():
    try:
        subprocess.run(["hugo", "version"], check=True, capture_output=True)
    except (subprocess.CalledProcessError, FileNotFoundError):
        raise RuntimeError(
            "Hugo binary not found. Install Hugo or skip integration tests.\n"
            "Install: https://gohugo.io/installation/"
        )

def create_hugo_harness(fixture_path: Optional[Path] = None) -> HugoHarness:
    verify_hugo_installed()
    
    temp_dir = tempfile.TemporaryDirectory(prefix="hugo-test-")
    site_dir = Path(temp_dir.name)
    output_dir = site_dir / "public"
    
    if fixture_path:
        shutil.copytree(fixture_path, site_dir, dirs_exist_ok=True)
    else:
        _create_minimal_site(site_dir)
    
    harness = HugoHarness(
        site_dir=site_dir,
        output_dir=output_dir,
        _temp_dir=temp_dir
    )
    return harness

def _create_minimal_site(site_dir: Path):
    (site_dir / "config.toml").write_text('''
title = "Test Site"
baseURL = "http://localhost:1313/"
''')
    
    content_dir = site_dir / "content"
    content_dir.mkdir()
    (content_dir / "_index.md").write_text('''
---
title: "Home"
---

# Welcome
''')
    
    layouts_dir = site_dir / "layouts" / "_default"
    layouts_dir.mkdir(parents=True)
    
    (layouts_dir / "baseof.html").write_text('''
<!DOCTYPE html>
<html>
<head><title>{{ .Title }}</title></head>
<body>{{ block "main" . }}{{ end }}</body>
</html>
''')
    (layouts_dir / "list.html").write_text('{{ define "main" }}{{ .Content }}{{ end }}')
    (layouts_dir / "single.html").write_text('{{ define "main" }}{{ .Content }}{{ end }}')
```

### Using the Python Harness

```python
# test/integration/test_hugo_build.py
import pytest
from harnesses.hugo import create_hugo_harness

def test_builds_minimal_site():
    harness = create_hugo_harness()
    
    try:
        result = harness.build()
        
        assert result.exit_code == 0
        assert (harness.output_dir / "index.html").exists()
    finally:
        harness.cleanup()

def test_builds_with_minify():
    harness = create_hugo_harness()
    
    try:
        result = harness.build(["--minify"])
        
        assert result.exit_code == 0
        html = (harness.output_dir / "index.html").read_text()
        # Minified HTML has minimal whitespace
        assert "\n\n" not in html
    finally:
        harness.cleanup()

# Using pytest fixtures for cleaner tests
@pytest.fixture
def hugo():
    harness = create_hugo_harness()
    yield harness
    harness.cleanup()

def test_with_fixture(hugo):
    result = hugo.build()
    assert result.exit_code == 0
```

---

## Pattern: HTTP Client Integration

Test that your HTTP client code works with real HTTP responses.

```typescript
// test/harnesses/http-server.ts
import { createServer, Server, IncomingMessage, ServerResponse } from 'http'
import getPort from 'get-port'

type Route = {
  method: string
  path: string
  status: number
  body: string | object
  headers?: Record<string, string>
}

type HttpServerHarness = {
  port: number
  baseUrl: string
  addRoute: (route: Route) => void
  start: () => Promise<void>
  stop: () => Promise<void>
  getRequests: () => Array<{ method: string; path: string; body: string }>
}

async function createHttpServerHarness(): Promise<HttpServerHarness> {
  const port = await getPort()
  const routes: Route[] = []
  const requests: Array<{ method: string; path: string; body: string }> = []
  let server: Server | null = null
  
  return {
    port,
    baseUrl: `http://localhost:${port}`,
    
    addRoute(route: Route) {
      routes.push(route)
    },
    
    async start() {
      server = createServer(async (req: IncomingMessage, res: ServerResponse) => {
        // Collect request body
        const chunks: Buffer[] = []
        for await (const chunk of req) {
          chunks.push(chunk as Buffer)
        }
        const body = Buffer.concat(chunks).toString()
        
        requests.push({
          method: req.method || 'GET',
          path: req.url || '/',
          body
        })
        
        // Find matching route
        const route = routes.find(
          r => r.method === req.method && r.path === req.url
        )
        
        if (route) {
          res.statusCode = route.status
          if (route.headers) {
            Object.entries(route.headers).forEach(([k, v]) => {
              res.setHeader(k, v)
            })
          }
          const responseBody = typeof route.body === 'string' 
            ? route.body 
            : JSON.stringify(route.body)
          res.end(responseBody)
        } else {
          res.statusCode = 404
          res.end('Not Found')
        }
      })
      
      await new Promise<void>(resolve => {
        server!.listen(port, resolve)
      })
    },
    
    async stop() {
      if (server) {
        await new Promise<void>(resolve => server!.close(() => resolve()))
        server = null
      }
    },
    
    getRequests() {
      return [...requests]
    }
  }
}

export { createHttpServerHarness }
```

### Using the HTTP Harness

```typescript
// test/integration/api-client.test.ts
import { createHttpServerHarness } from '../harnesses/http-server'
import { createApiClient } from '../../src/api/client'

describe('API Client Integration', () => {
  let server: Awaited<ReturnType<typeof createHttpServerHarness>>
  
  beforeEach(async () => {
    server = await createHttpServerHarness()
  })
  
  afterEach(async () => {
    await server.stop()
  })
  
  test('fetches resources', async () => {
    server.addRoute({
      method: 'GET',
      path: '/api/users/123',
      status: 200,
      body: { id: '123', name: 'Test User' },
      headers: { 'Content-Type': 'application/json' }
    })
    await server.start()
    
    const client = createApiClient(server.baseUrl)
    const user = await client.getUser('123')
    
    expect(user).toEqual({ id: '123', name: 'Test User' })
  })
  
  test('handles 404 responses', async () => {
    server.addRoute({
      method: 'GET',
      path: '/api/users/999',
      status: 404,
      body: { error: 'Not found' }
    })
    await server.start()
    
    const client = createApiClient(server.baseUrl)
    const result = await client.getUser('999')
    
    expect(result).toBeNull()
  })
  
  test('sends correct request body', async () => {
    server.addRoute({
      method: 'POST',
      path: '/api/users',
      status: 201,
      body: { id: 'new-id' }
    })
    await server.start()
    
    const client = createApiClient(server.baseUrl)
    await client.createUser({ name: 'New User', email: 'new@test.com' })
    
    const requests = server.getRequests()
    expect(requests[0].body).toBe(
      JSON.stringify({ name: 'New User', email: 'new@test.com' })
    )
  })
})
```

---

## Pattern: Vitest Setup for Integration Tests

```typescript
// test/setup.ts
import { beforeAll, afterAll } from 'vitest'

// Environment check - fail fast if dependencies aren't available
beforeAll(async () => {
  const checks = [
    { name: 'Hugo', cmd: 'hugo version' },
    { name: 'Caddy', cmd: 'caddy version' },
  ]
  
  const { execa } = await import('execa')
  
  for (const check of checks) {
    try {
      await execa('sh', ['-c', check.cmd])
    } catch {
      console.warn(`⚠️  ${check.name} not available - some integration tests will be skipped`)
    }
  }
})

// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    include: ['test/**/*.test.ts'],
    setupFiles: ['test/setup.ts'],
    testTimeout: 30000, // Integration tests can be slow
    hookTimeout: 30000,
    
    // Run integration tests sequentially to avoid port conflicts
    poolOptions: {
      threads: {
        singleThread: true
      }
    }
  }
})
```

---

## Pattern: Docker Compose for Test Infrastructure

```yaml
# docker-compose.test.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: test_db
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI / API
```

```bash
# Start test infrastructure
docker-compose -f docker-compose.test.yml up -d

# Wait for healthy
docker-compose -f docker-compose.test.yml ps

# Run integration tests
npm run test:integration

# Tear down
docker-compose -f docker-compose.test.yml down -v
```

---

## Checklist: Is This a Level 2 Test?

Before writing the test, verify:

- [ ] Harness is documented and reproducible
- [ ] Harness can be started/stopped/reset programmatically
- [ ] Test uses real dependency, not mock
- [ ] No external network calls (that's Level 3)
- [ ] No production credentials (that's Level 3)
- [ ] Cleanup happens even on test failure

If the test requires production credentials or external services, escalate to Level 3.

---

## What Level 2 Proves

✅ PostgreSQL accepts your queries  
✅ Hugo builds your site structure  
✅ Caddy serves your files correctly  
✅ Your HTTP client handles real responses  
✅ Your file operations work on real filesystems  

## What Level 2 Cannot Prove

❌ That production credentials work  
❌ That third-party APIs behave the same in prod  
❌ That the full user workflow succeeds  
❌ That performance is acceptable  

**When you need these guarantees, escalate to Level 3.**
