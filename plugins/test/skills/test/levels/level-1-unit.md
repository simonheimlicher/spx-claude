# Level 1: Unit / Pure Logic

## The Question This Level Answers

> **"Is our logic correct, independent of any external system?"**

When something fails in production, Level 1 tests let you instantly rule out (or identify) bugs in your core logic—the part you fully control.

---

## What Belongs at Level 1

| ✅ Test Here | ❌ Push to Level 2 |
|--------------|-------------------|
| Pure functions (parsers, validators, calculators) | Database queries |
| Command/argument building | HTTP calls to external services |
| Configuration parsing and validation | File operations on real filesystems |
| Business logic with injected dependencies | Binary execution (Hugo, ffmpeg, etc.) |
| Error message formatting | Network operations |
| Data transformation and mapping | Browser interactions |

---

## The DI Principle: Functions, Not Mocks

Dependency injection at Level 1 means **passing dependencies as function parameters**. You're not mocking—you're providing real (but simple) implementations.

```typescript
// ❌ BAD: Hardcoded dependency requiring mocks
async function syncFiles(source: string, dest: string) {
  const files = await fs.readdir(source)  // Hardcoded fs!
  await execa('rsync', [source, dest])    // Hardcoded execa!
}

// ✅ GOOD: Dependencies as parameters
type SyncDeps = {
  listFiles: (dir: string) => Promise<string[]>
  runCommand: (cmd: string, args: string[]) => Promise<{ exitCode: number }>
}

async function syncFiles(
  source: string, 
  dest: string, 
  deps: SyncDeps
): Promise<SyncResult> {
  const files = await deps.listFiles(source)
  if (files.length === 0) {
    return { synced: 0, status: 'empty' }
  }
  const result = await deps.runCommand('rsync', [source, dest])
  return { synced: files.length, status: result.exitCode === 0 ? 'ok' : 'failed' }
}
```

At Level 1, you test with simple implementations:

```typescript
// Level 1 test - no mocking framework needed
test('syncFiles returns empty status when no files', async () => {
  const deps: SyncDeps = {
    listFiles: async () => [],  // Simple implementation, not a mock
    runCommand: async () => ({ exitCode: 0 }),
  }
  
  const result = await syncFiles('/source', '/dest', deps)
  
  expect(result).toEqual({ synced: 0, status: 'empty' })
})

test('syncFiles returns failed status on non-zero exit', async () => {
  const deps: SyncDeps = {
    listFiles: async () => ['file1.txt', 'file2.txt'],
    runCommand: async () => ({ exitCode: 1 }),  // Simulate failure
  }
  
  const result = await syncFiles('/source', '/dest', deps)
  
  expect(result).toEqual({ synced: 2, status: 'failed' })
})
```

**Key insight**: You're not testing that `rsync` works (that's Level 2). You're testing that your logic correctly interprets results and handles edge cases.

---

## Pattern: Pure Function Testing

Pure functions are Level 1's sweet spot. No dependencies, deterministic output.

### CLI Argument Parsing

```typescript
// src/cli/args.ts
type LhciOptions = {
  url?: string
  config?: string
  upload?: boolean
  numberOfRuns?: number
}

function parseArgs(argv: string[]): LhciOptions {
  const args: LhciOptions = {}
  
  for (let i = 0; i < argv.length; i++) {
    switch (argv[i]) {
      case '--url':
        args.url = argv[++i]
        break
      case '--config':
        args.config = argv[++i]
        break
      case '--upload':
        args.upload = true
        break
      case '-n':
      case '--number-of-runs':
        args.numberOfRuns = parseInt(argv[++i], 10)
        break
    }
  }
  
  return args
}

// test/unit/args.test.ts
describe('parseArgs', () => {
  test('parses --url flag', () => {
    const result = parseArgs(['--url', 'http://localhost:3000'])
    expect(result.url).toBe('http://localhost:3000')
  })
  
  test('parses boolean --upload flag', () => {
    const result = parseArgs(['--upload'])
    expect(result.upload).toBe(true)
  })
  
  test('parses short -n flag', () => {
    const result = parseArgs(['-n', '5'])
    expect(result.numberOfRuns).toBe(5)
  })
  
  test('handles multiple flags', () => {
    const result = parseArgs([
      '--url', 'http://localhost:3000',
      '--upload',
      '-n', '3'
    ])
    expect(result).toEqual({
      url: 'http://localhost:3000',
      upload: true,
      numberOfRuns: 3,
    })
  })
})
```

### Command Building

```typescript
// src/commands/hugo.ts
type HugoBuildOptions = {
  source: string
  destination?: string
  minify?: boolean
  baseURL?: string
  environment?: string
}

function buildHugoCommand(options: HugoBuildOptions): string[] {
  const args = ['hugo', '--source', options.source]
  
  if (options.destination) {
    args.push('--destination', options.destination)
  }
  if (options.minify) {
    args.push('--minify')
  }
  if (options.baseURL) {
    args.push('--baseURL', options.baseURL)
  }
  if (options.environment) {
    args.push('--environment', options.environment)
  }
  
  return args
}

// test/unit/hugo-command.test.ts
describe('buildHugoCommand', () => {
  test('includes required source flag', () => {
    const cmd = buildHugoCommand({ source: '/site' })
    expect(cmd).toEqual(['hugo', '--source', '/site'])
  })
  
  test('adds minify flag when enabled', () => {
    const cmd = buildHugoCommand({ source: '/site', minify: true })
    expect(cmd).toContain('--minify')
  })
  
  test('builds complete production command', () => {
    const cmd = buildHugoCommand({
      source: '/site',
      destination: '/dist',
      minify: true,
      baseURL: 'https://example.com',
      environment: 'production',
    })
    
    expect(cmd).toEqual([
      'hugo',
      '--source', '/site',
      '--destination', '/dist',
      '--minify',
      '--baseURL', 'https://example.com',
      '--environment', 'production',
    ])
  })
})
```

---

## Pattern: Configuration Validation

Use schema validation libraries (Zod, etc.) but test the validation logic, not the library.

```typescript
// src/config/schema.ts
import { z } from 'zod'

const urlSetSchema = z.record(z.array(z.string().url()))

const configSchema = z.object({
  site_dir: z.string().min(1),
  base_url: z.string().url(),
  url_sets: urlSetSchema,
  lighthouse: z.object({
    runs: z.number().min(1).max(10).default(3),
    throttling: z.enum(['mobile', 'desktop']).default('mobile'),
  }).default({}),
})

type Config = z.infer<typeof configSchema>

function parseConfig(raw: unknown): { ok: true; config: Config } | { ok: false; errors: string[] } {
  const result = configSchema.safeParse(raw)
  
  if (result.success) {
    return { ok: true, config: result.data }
  }
  
  const errors = result.error.issues.map(issue => 
    `${issue.path.join('.')}: ${issue.message}`
  )
  return { ok: false, errors }
}

// test/unit/config.test.ts
describe('parseConfig', () => {
  test('parses valid minimal config', () => {
    const result = parseConfig({
      site_dir: './site',
      base_url: 'http://localhost:1313',
      url_sets: { critical: ['/', '/about/'] },
    })
    
    expect(result.ok).toBe(true)
    if (result.ok) {
      expect(result.config.lighthouse.runs).toBe(3)  // default
    }
  })
  
  test('rejects missing required fields', () => {
    const result = parseConfig({
      base_url: 'http://localhost:1313',
    })
    
    expect(result.ok).toBe(false)
    if (!result.ok) {
      expect(result.errors).toContain('site_dir: Required')
    }
  })
  
  test('rejects invalid URL in url_sets', () => {
    const result = parseConfig({
      site_dir: './site',
      base_url: 'http://localhost:1313',
      url_sets: { critical: ['not-a-url'] },
    })
    
    expect(result.ok).toBe(false)
  })
  
  test('rejects runs outside valid range', () => {
    const result = parseConfig({
      site_dir: './site',
      base_url: 'http://localhost:1313',
      url_sets: {},
      lighthouse: { runs: 100 },
    })
    
    expect(result.ok).toBe(false)
  })
})
```

---

## Pattern: MCP Tool Definitions

MCP servers define tools as pure data structures. Test the definitions and input validation.

```typescript
// src/mcp/tools.ts
type ToolDefinition = {
  name: string
  description: string
  inputSchema: {
    type: 'object'
    properties: Record<string, { type: string; description: string }>
    required: string[]
  }
}

const runAuditTool: ToolDefinition = {
  name: 'run_audit',
  description: 'Run Lighthouse audit on specified URLs',
  inputSchema: {
    type: 'object',
    properties: {
      urls: { type: 'array', description: 'URLs to audit' },
      preset: { type: 'string', description: 'Audit preset (desktop/mobile)' },
    },
    required: ['urls'],
  },
}

type RunAuditInput = {
  urls: string[]
  preset?: 'desktop' | 'mobile'
}

function validateRunAuditInput(input: unknown): RunAuditInput | null {
  if (typeof input !== 'object' || input === null) return null
  
  const obj = input as Record<string, unknown>
  
  if (!Array.isArray(obj.urls)) return null
  if (!obj.urls.every(u => typeof u === 'string')) return null
  
  if (obj.preset !== undefined) {
    if (obj.preset !== 'desktop' && obj.preset !== 'mobile') return null
  }
  
  return {
    urls: obj.urls,
    preset: obj.preset as 'desktop' | 'mobile' | undefined,
  }
}

// test/unit/mcp-tools.test.ts
describe('validateRunAuditInput', () => {
  test('accepts valid input with urls only', () => {
    const result = validateRunAuditInput({ urls: ['/', '/about/'] })
    expect(result).toEqual({ urls: ['/', '/about/'], preset: undefined })
  })
  
  test('accepts valid input with preset', () => {
    const result = validateRunAuditInput({ 
      urls: ['/'], 
      preset: 'desktop' 
    })
    expect(result).toEqual({ urls: ['/'], preset: 'desktop' })
  })
  
  test('rejects missing urls', () => {
    expect(validateRunAuditInput({})).toBeNull()
    expect(validateRunAuditInput({ preset: 'mobile' })).toBeNull()
  })
  
  test('rejects invalid preset', () => {
    expect(validateRunAuditInput({ 
      urls: ['/'], 
      preset: 'invalid' 
    })).toBeNull()
  })
  
  test('rejects non-string urls', () => {
    expect(validateRunAuditInput({ urls: [1, 2, 3] })).toBeNull()
  })
})
```

---

## Pattern: Business Logic with DI

For logic that needs external data, use dependency injection.

```typescript
// src/audit/analyzer.ts
type AuditResult = {
  url: string
  scores: { performance: number; accessibility: number }
}

type AnalyzerDeps = {
  getThresholds: () => Promise<{ performance: number; accessibility: number }>
}

type AnalysisResult = {
  passed: boolean
  failures: string[]
  summary: string
}

async function analyzeResults(
  results: AuditResult[],
  deps: AnalyzerDeps
): Promise<AnalysisResult> {
  const thresholds = await deps.getThresholds()
  const failures: string[] = []
  
  for (const result of results) {
    if (result.scores.performance < thresholds.performance) {
      failures.push(
        `${result.url}: performance ${result.scores.performance} < ${thresholds.performance}`
      )
    }
    if (result.scores.accessibility < thresholds.accessibility) {
      failures.push(
        `${result.url}: accessibility ${result.scores.accessibility} < ${thresholds.accessibility}`
      )
    }
  }
  
  return {
    passed: failures.length === 0,
    failures,
    summary: failures.length === 0 
      ? `All ${results.length} URLs passed`
      : `${failures.length} failures across ${results.length} URLs`,
  }
}

// test/unit/analyzer.test.ts
describe('analyzeResults', () => {
  const defaultDeps: AnalyzerDeps = {
    getThresholds: async () => ({ performance: 90, accessibility: 100 }),
  }
  
  test('passes when all scores meet thresholds', async () => {
    const results: AuditResult[] = [
      { url: '/', scores: { performance: 95, accessibility: 100 } },
      { url: '/about/', scores: { performance: 92, accessibility: 100 } },
    ]
    
    const analysis = await analyzeResults(results, defaultDeps)
    
    expect(analysis.passed).toBe(true)
    expect(analysis.failures).toHaveLength(0)
  })
  
  test('fails when performance below threshold', async () => {
    const results: AuditResult[] = [
      { url: '/', scores: { performance: 85, accessibility: 100 } },
    ]
    
    const analysis = await analyzeResults(results, defaultDeps)
    
    expect(analysis.passed).toBe(false)
    expect(analysis.failures).toContain('/: performance 85 < 90')
  })
  
  test('reports multiple failures', async () => {
    const results: AuditResult[] = [
      { url: '/', scores: { performance: 85, accessibility: 95 } },
    ]
    
    const analysis = await analyzeResults(results, defaultDeps)
    
    expect(analysis.failures).toHaveLength(2)
  })
  
  test('uses custom thresholds', async () => {
    const lenientDeps: AnalyzerDeps = {
      getThresholds: async () => ({ performance: 50, accessibility: 50 }),
    }
    const results: AuditResult[] = [
      { url: '/', scores: { performance: 60, accessibility: 60 } },
    ]
    
    const analysis = await analyzeResults(results, lenientDeps)
    
    expect(analysis.passed).toBe(true)
  })
})
```

---

## Pattern: Temporary Directories

Temp directories are **not** external dependencies—they're part of the runtime. Use them freely at Level 1.

```typescript
// test/unit/config-file.test.ts
import { mkdtemp, writeFile, rm } from 'fs/promises'
import { join } from 'path'
import { tmpdir } from 'os'

describe('loadConfig', () => {
  let tempDir: string
  
  beforeEach(async () => {
    tempDir = await mkdtemp(join(tmpdir(), 'config-test-'))
  })
  
  afterEach(async () => {
    await rm(tempDir, { recursive: true })
  })
  
  test('loads YAML config file', async () => {
    const configPath = join(tempDir, 'hugolit.yaml')
    await writeFile(configPath, `
site_dir: ./site
base_url: http://localhost:1313
url_sets:
  critical:
    - /
    - /about/
`)
    
    const config = await loadConfig(configPath)
    
    expect(config.site_dir).toBe('./site')
    expect(config.url_sets.critical).toEqual(['/', '/about/'])
  })
  
  test('returns error for missing file', async () => {
    const result = await loadConfig(join(tempDir, 'nonexistent.yaml'))
    
    expect(result.ok).toBe(false)
  })
})
```

### Python Example

```python
# test/unit/test_config.py
import tempfile
from pathlib import Path
import pytest
from hugolit.config import load_config

def test_loads_yaml_config():
    with tempfile.TemporaryDirectory() as tmpdir:
        config_path = Path(tmpdir) / "hugolit.yaml"
        config_path.write_text("""
site_dir: ./site
base_url: http://localhost:1313
url_sets:
  critical:
    - /
    - /about/
""")
        
        config = load_config(config_path)
        
        assert config.site_dir == "./site"
        assert config.url_sets["critical"] == ["/", "/about/"]

def test_returns_error_for_missing_file():
    result = load_config(Path("/nonexistent/config.yaml"))
    
    assert result.ok is False
    assert "not found" in result.error.lower()
```

---

## Pattern: Next.js Server Actions Logic

Server actions often mix logic with framework concerns. Extract the logic for Level 1 testing.

```typescript
// src/app/actions/audit.ts
import { runAudit } from '@/lib/audit'

// This is the server action - hard to test
export async function auditPageAction(formData: FormData) {
  const url = formData.get('url') as string
  const result = await runAudit(url)
  return result
}

// src/lib/audit.ts
// This is the extracted logic - easy to test at Level 1

type AuditDeps = {
  fetchPage: (url: string) => Promise<{ status: number; html: string }>
  runLighthouse: (html: string) => Promise<{ score: number }>
}

type AuditInput = {
  url: string
  options?: { timeout?: number }
}

function validateAuditInput(input: unknown): AuditInput | { error: string } {
  if (typeof input !== 'object' || input === null) {
    return { error: 'Input must be an object' }
  }
  
  const obj = input as Record<string, unknown>
  
  if (typeof obj.url !== 'string') {
    return { error: 'URL must be a string' }
  }
  
  try {
    new URL(obj.url)
  } catch {
    return { error: 'Invalid URL format' }
  }
  
  return { url: obj.url, options: obj.options as AuditInput['options'] }
}

async function executeAudit(
  input: AuditInput,
  deps: AuditDeps
): Promise<{ ok: true; score: number } | { ok: false; error: string }> {
  try {
    const page = await deps.fetchPage(input.url)
    
    if (page.status !== 200) {
      return { ok: false, error: `Page returned status ${page.status}` }
    }
    
    const result = await deps.runLighthouse(page.html)
    return { ok: true, score: result.score }
  } catch (e) {
    return { ok: false, error: `Audit failed: ${e}` }
  }
}

// test/unit/audit.test.ts
describe('validateAuditInput', () => {
  test('accepts valid URL', () => {
    const result = validateAuditInput({ url: 'https://example.com' })
    expect(result).toEqual({ url: 'https://example.com', options: undefined })
  })
  
  test('rejects invalid URL', () => {
    const result = validateAuditInput({ url: 'not-a-url' })
    expect(result).toEqual({ error: 'Invalid URL format' })
  })
})

describe('executeAudit', () => {
  test('returns score on success', async () => {
    const deps: AuditDeps = {
      fetchPage: async () => ({ status: 200, html: '<html></html>' }),
      runLighthouse: async () => ({ score: 95 }),
    }
    
    const result = await executeAudit({ url: 'https://example.com' }, deps)
    
    expect(result).toEqual({ ok: true, score: 95 })
  })
  
  test('returns error on non-200 status', async () => {
    const deps: AuditDeps = {
      fetchPage: async () => ({ status: 404, html: '' }),
      runLighthouse: async () => ({ score: 0 }),
    }
    
    const result = await executeAudit({ url: 'https://example.com' }, deps)
    
    expect(result).toEqual({ ok: false, error: 'Page returned status 404' })
  })
  
  test('returns error when fetch fails', async () => {
    const deps: AuditDeps = {
      fetchPage: async () => { throw new Error('Network error') },
      runLighthouse: async () => ({ score: 0 }),
    }
    
    const result = await executeAudit({ url: 'https://example.com' }, deps)
    
    expect(result.ok).toBe(false)
    expect(result.ok === false && result.error).toContain('Network error')
  })
})
```

---

## Pattern: Error Message Formatting

Error messages are pure functions—perfect for Level 1.

```typescript
// src/errors/format.ts
type ValidationError = {
  field: string
  message: string
  value?: unknown
}

function formatValidationErrors(errors: ValidationError[]): string {
  if (errors.length === 0) return ''
  
  const lines = errors.map(e => {
    const value = e.value !== undefined ? ` (got: ${JSON.stringify(e.value)})` : ''
    return `  • ${e.field}: ${e.message}${value}`
  })
  
  return `Validation failed:\n${lines.join('\n')}`
}

// test/unit/error-format.test.ts
describe('formatValidationErrors', () => {
  test('returns empty string for no errors', () => {
    expect(formatValidationErrors([])).toBe('')
  })
  
  test('formats single error', () => {
    const result = formatValidationErrors([
      { field: 'url', message: 'is required' }
    ])
    
    expect(result).toBe('Validation failed:\n  • url: is required')
  })
  
  test('includes value when provided', () => {
    const result = formatValidationErrors([
      { field: 'port', message: 'must be a number', value: 'abc' }
    ])
    
    expect(result).toContain('(got: "abc")')
  })
  
  test('formats multiple errors', () => {
    const result = formatValidationErrors([
      { field: 'url', message: 'is required' },
      { field: 'runs', message: 'must be positive', value: -1 },
    ])
    
    expect(result).toContain('• url: is required')
    expect(result).toContain('• runs: must be positive')
  })
})
```

---

## Pattern: Data Factories

Generate test data programmatically. Never use arbitrary literals.

```typescript
// test/factories.ts
let idCounter = 0

function createAuditResult(overrides: Partial<AuditResult> = {}): AuditResult {
  return {
    id: `audit-${++idCounter}`,
    url: `https://example.com/page-${idCounter}`,
    timestamp: new Date().toISOString(),
    scores: {
      performance: 90,
      accessibility: 100,
      bestPractices: 95,
      seo: 90,
    },
    ...overrides,
  }
}

function createConfig(overrides: Partial<Config> = {}): Config {
  return {
    site_dir: './test-site',
    base_url: 'http://localhost:1313',
    url_sets: { default: ['/'] },
    lighthouse: { runs: 1, throttling: 'desktop' },
    ...overrides,
  }
}

// Usage in tests
test('handles low performance score', async () => {
  const result = createAuditResult({ 
    scores: { ...createAuditResult().scores, performance: 45 } 
  })
  
  const analysis = await analyzeResults([result], deps)
  
  expect(analysis.passed).toBe(false)
})
```

### Python Example

```python
# test/factories.py
from dataclasses import dataclass, field
from typing import Optional
import itertools

_id_counter = itertools.count(1)

@dataclass
class AuditResultFactory:
    url: str = field(default_factory=lambda: f"https://example.com/page-{next(_id_counter)}")
    performance: int = 90
    accessibility: int = 100
    
    def build(self) -> dict:
        return {
            "url": self.url,
            "scores": {
                "performance": self.performance,
                "accessibility": self.accessibility,
            }
        }

def create_audit_result(**kwargs) -> dict:
    return AuditResultFactory(**kwargs).build()

# Usage
def test_handles_low_performance():
    result = create_audit_result(performance=45)
    
    analysis = analyze_results([result], deps)
    
    assert analysis.passed is False
```

---

## Checklist: Is This a Level 1 Test?

Before writing the test, verify:

- [ ] No real database connections
- [ ] No real HTTP requests
- [ ] No real binary execution
- [ ] No real network operations
- [ ] Dependencies are injected, not imported
- [ ] Test data is generated, not hardcoded
- [ ] Testing behavior (WHAT), not implementation (HOW)

If any check fails, this belongs at Level 2 or Level 3.

---

## What Level 1 Proves

✅ Your parsing logic handles all expected formats  
✅ Your validation catches invalid input  
✅ Your business logic produces correct results  
✅ Your error handling works as designed  
✅ Your command building produces correct arguments  

## What Level 1 Cannot Prove

❌ That the database accepts your queries  
❌ That Hugo accepts your command arguments  
❌ That the external API returns what you expect  
❌ That the file system behaves as expected  
❌ That the full workflow succeeds  

**When you need these guarantees, escalate to Level 2.**
