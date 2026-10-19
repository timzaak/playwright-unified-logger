# playwright-unified-logger

[简体中文](README.zh.md)

Unified logging for Playwright E2E tests: coordinates capture of network requests, browser console output, route changes, and test code logs within a single test run, writes them to files with buffered I/O, and prints a summary when the test finishes.

## Features

- Four log categories coordinated together: network (requests / responses / failures), console, route changes, and test-code `console`
- Buffered file writes keep high-frequency I/O from stalling tests
- Duplicate log deduplication and aggregated statistics
- Zero-config Playwright fixture that prints the summary and writes log files automatically when each test ends
- Event bus (`network:error` and more) for custom handling
- Dual configuration channels: programmatic options and environment variables

## Installation

```bash
npm install -D playwright-unified-logger
```

Peer dependency: `@playwright/test >= 1.40.0` — make sure Playwright is installed in your project.

## Quick Start (fixture, recommended)

```typescript
import { test, expect } from 'playwright-unified-logger/fixtures'

test('example', async ({ page, unifiedLogger }) => {
  await page.goto('https://example.com')
  // unifiedLogger is created by the fixture and destroyed automatically;
  // logs are written and the summary printed when the test ends
})
```

Use `createLoggerFixture` for custom configuration:

```typescript
import { createLoggerFixture } from 'playwright-unified-logger/fixtures'

const test = createLoggerFixture({ logLevel: 'verbose' })

test('example', async ({ page, unifiedLogger }) => {
  // ...
})
```

## Direct Usage

```typescript
import { UnifiedLogger } from 'playwright-unified-logger'

const logger = new UnifiedLogger(page, testTitle)
// ... test logic ...
logger.printSummary('Test Summary')
await logger.finalize()
```

## Standalone Sub-loggers

```typescript
import { NetworkLogger, ConsoleLogger, RouteLogger, TestCodeLogger } from 'playwright-unified-logger'
```

## Event Bus

```typescript
logger.events.on('network:error', (event) => {
  // custom handling; event types are exported as LoggerEventType
})
```

## Configuration

Programmatic (constructor options):

```typescript
const logger = new UnifiedLogger(page, testTitle, {
  logLevel: 'verbose',            // mini | normal | verbose | silent, default mini
  outputDir: 'test-results/unified-logs',
  deduplication: true,            // dedupe repeated logs, default true
  aggregation: true,              // aggregated statistics, default true
  compactMode: false,             // compact output, default false
  network: { urlFilter: url => url.includes('/api/') },
  console: { filterPatterns: [/HMR/i, /React DevTools/i] },
})
```

Environment variables (the legacy `DEMO_LOG_*` prefix is also supported):

| Variable | Values | Default |
|---|---|---|
| `UNIFIED_LOG_LEVEL` | mini / normal / verbose / silent | mini |
| `UNIFIED_LOG_DEDUP` | true / false | true |
| `UNIFIED_LOG_AGGREGATE` | true / false | true |
| `UNIFIED_LOG_COMPACT` | true / false | false |
| `UNIFIED_LOG_OUTPUT_DIR` | directory path | `test-results/unified-logs` (relative to the current working directory) |

## Development & Publishing

```bash
npm install
npm run build        # tsc build into dist/
npm run type-check
npm run lint
```

Publishing: bump the version in `package.json`, commit, and push a `v*` tag; GitHub Actions then runs `npm ci -> build -> npm publish --provenance`.

## License

MIT
