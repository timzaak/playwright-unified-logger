# playwright-unified-logger

Playwright E2E 测试统一日志库：在一次测试运行中协调捕获网络请求、浏览器控制台、路由变化和测试代码日志，缓冲写入文件，并在测试结束时输出摘要。

## 特性

- 四类日志统一协调：网络（请求/响应/失败）、控制台、路由变化、测试代码 `console`
- 缓冲文件写入，避免高频 I/O 阻塞测试
- 重复日志去重与聚合统计
- Playwright fixture 零配置接入，测试结束自动打印摘要并写日志
- 事件总线（`network:error` 等）支持自定义处理
- 编程配置 + 环境变量双通道

## 安装

```bash
npm install -D playwright-unified-logger
```

peer 依赖 `@playwright/test >= 1.40.0`，请确保项目已安装 Playwright。

## 快速开始（fixture，推荐）

```typescript
import { test, expect } from 'playwright-unified-logger/fixtures'

test('example', async ({ page, unifiedLogger }) => {
  await page.goto('https://example.com')
  // unifiedLogger 由 fixture 自动创建，测试结束自动销毁并输出日志
})
```

自定义配置时用 `createLoggerFixture`：

```typescript
import { createLoggerFixture } from 'playwright-unified-logger/fixtures'

const test = createLoggerFixture({ logLevel: 'verbose' })

test('example', async ({ page, unifiedLogger }) => {
  // ...
})
```

## 直接使用

```typescript
import { UnifiedLogger } from 'playwright-unified-logger'

const logger = new UnifiedLogger(page, testTitle)
// ... 测试逻辑 ...
logger.printSummary('Test Summary')
await logger.finalize()
```

## 单独使用子日志器

```typescript
import { NetworkLogger, ConsoleLogger, RouteLogger, TestCodeLogger } from 'playwright-unified-logger'
```

## 事件总线

```typescript
logger.events.on('network:error', (event) => {
  // 自定义处理，事件类型见导出的 LoggerEventType
})
```

## 配置

编程配置（构造参数）：

```typescript
const logger = new UnifiedLogger(page, testTitle, {
  logLevel: 'verbose',            // mini | normal | verbose | silent，默认 mini
  outputDir: 'test-results/unified-logs',
  deduplication: true,            // 重复日志去重，默认 true
  aggregation: true,              // 聚合统计，默认 true
  compactMode: false,             // 紧凑输出，默认 false
  network: { urlFilter: url => url.includes('/api/') },
  console: { filterPatterns: [/HMR/i, /React DevTools/i] },
})
```

环境变量（兼容旧 `DEMO_LOG_*` 前缀）：

| 变量 | 值 | 默认 |
|---|---|---|
| `UNIFIED_LOG_LEVEL` | mini / normal / verbose / silent | mini |
| `UNIFIED_LOG_DEDUP` | true / false | true |
| `UNIFIED_LOG_AGGREGATE` | true / false | true |
| `UNIFIED_LOG_COMPACT` | true / false | false |
| `UNIFIED_LOG_OUTPUT_DIR` | 目录路径 | `test-results/unified-logs`（相对当前工作目录） |

## 开发与发布

```bash
npm install
npm run build        # tsc 构建到 dist/
npm run type-check
npm run lint
```

发布流程：更新 `package.json` 版本号并提交，打 `v*` tag 推送，GitHub Actions 自动执行 `npm ci -> build -> npm publish --provenance`。

## License

MIT
