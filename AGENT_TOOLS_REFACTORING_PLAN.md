# 🔄 AI AGENT TOOLS REFACTORING PLAN

**Current State → Modular Architecture**  
**Goal:** Extract tool implementations from server.js without losing functionality

---

## 📊 CURRENT STATE ANALYSIS

### File Sizes

| File | Lines | Purpose |
|------|-------|---------|
| `server.js` | **5,885** | Backend + API + **Tool implementations** |
| `tools/agent-tools.js` | 630 | Tool definitions (declarations only) |
| `agent/langchainAgent.js` | 1,554 | Tool execution orchestration |
| **Total** | **8,069** | - |

### Problem: Tool Logic Scattered

**Currently in `server.js` (lines 5040-5450):**
- `parseToolCall()` - Parse tool call from model
- `executeToolCall()` - **379 lines** of tool execution logic
- `normalizeToolArguments()` - Argument normalization
- `normalizeToolCommodity()` - Commodity name normalization
- `summarizeForToolResult()` - Result truncation
- `runWebSearchTool()` - Web search implementation

**Issue:** Tool implementations are mixed with API routing, authentication, and SSE streaming logic.

---

## 🎯 REFACTORING GOALS

### ✅ What We Want

1. **Separation of Concerns:**
   - `server.js` → API routing, auth, SSE streaming
   - `tools/agent-tools.js` → Tool definitions (already done)
   - `tools/executors/` → **Tool implementations** (NEW)

2. **No Functionality Loss:**
   - All 18 tools work identically
   - Same error handling
   - Same logging/SSE events
   - Same caching behavior

3. **Improved Maintainability:**
   - Each tool in its own file
   - Easier testing (isolate tool logic)
   - Clear dependencies (imports show what each tool needs)
   - Reusability (tools can be used outside agent context)

4. **Better Performance:**
   - Lazy loading (only load tools when needed)
   - Parallel tool execution (no blocking)
   - Independent caching per tool

---

## 🏗️ PROPOSED ARCHITECTURE

### New Directory Structure

```
tools/
├── README.md                    # Tools documentation
├── agent-tools.js               # Tool definitions (declarations only)
├── executors/                   # NEW: Tool implementations
│   ├── index.js                 # Export all executors
│   ├── market-executors.js      # Market data tools (5 tools)
│   ├── astrology-executors.js   # Astrology tools (6 tools)
│   ├── analysis-executors.js    # Analysis tools (5 tools)
│   └── utility-executors.js     # Utility tools (2 tools)
├── market/                      # Market data utilities (existing)
│   ├── constants.js
│   ├── eodhd-client.js
│   ├── yahoo-chart.js
│   └── ...
├── astrology/                   # Astrology utilities (existing)
│   ├── book-context.js
│   ├── ephemeris-cache.js
│   └── ...
├── analysis/                    # Analysis utilities (existing)
│   ├── backtest-runners.js
│   ├── stock-event-analysis.js
│   └── ...
└── utils/                       # Shared utilities (existing)
    ├── cache.js
    ├── helpers.js
    └── queue-limiter.js
```

### Tool Executor Pattern

**Each tool executor follows this pattern:**

```javascript
// tools/executors/market-executors.js

const { getMarketSnapshot } = require('../market/market-snapshot');
const { resolveValidatedStock } = require('../market/asset-resolver');
const { logInfo, logWarn } = require('../utils/helpers');
const { withDailyCache } = require('../utils/cache');

/**
 * Execute get_market_snapshot tool
 * @param {Object} args - Tool arguments
 * @param {Object} context - Execution context (scope, threadId, userMessage)
 * @returns {Promise<Object>} - Tool result
 */
async function executeGetMarketSnapshot(args, context) {
  const { commodity, symbol, assetQuery, days } = args;
  const { scope, userMessage } = context;
  
  // Intelligent lookback detection
  const detectedDays = detectLookbackFromMessage(userMessage);
  const resolvedDays = toPositiveInt(days, detectedDays, 1, MAX_MARKET_HISTORY_DAYS);
  
  // Check if stock request
  const stockRequest = Boolean(symbol || assetQuery);
  
  if (stockRequest) {
    // Resolve stock symbol
    const asset = await resolveValidatedStock({ symbol, assetQuery, scope });
    // Fetch snapshot
    const snapshot = await getMarketSnapshot({ ...asset, days: resolvedDays });
    return {
      kind: 'market_snapshot',
      entityType: 'security',
      ...snapshot
    };
  }
  
  // Commodity request
  const normalizedCommodity = normalizeToolCommodity(commodity, scope);
  const snapshot = await getMarketSnapshot({ commodity: normalizedCommodity, days: resolvedDays });
  
  return {
    kind: 'market_snapshot',
    entityType: 'commodity',
    commodity: normalizedCommodity,
    ...snapshot
  };
}

module.exports = {
  executeGetMarketSnapshot
};
```

### Executor Registry

```javascript
// tools/executors/index.js

const {
  executeGetMarketSnapshot,
  executeGetSecurityHistory,
  executeResolveStockSymbol,
  executeSearchStockCandidates,
  executeGetSectorRuleMapping
} = require('./market-executors');

const {
  executeGetAstroForecast,
  executeGetAssetEphemeris,
  executeGetBitcoinEphemeris,
  executeRunShortTermReversalAnalysis,
  executeGetBookReference,
  executeWebSearchNatalData
} = require('./astrology-executors');

const {
  executeRunStockEventAnalysis,
  executeRunMonthlyReconstruction,
  executeRunStockBacktestPro,
  executeRunBitcoinWindowBacktest,
  executeRunBacktest
} = require('./analysis-executors');

const {
  executeWebSearch,
  executeRunTerminalCommand
} = require('./utility-executors');

/**
 * Execute any tool by name
 * @param {string} toolName - Tool name
 * @param {Object} args - Tool arguments
 * @param {Object} context - Execution context
 * @returns {Promise<Object>} - Tool result
 */
async function executeTool(toolName, args, context) {
  const executors = {
    // Market tools
    'get_market_snapshot': executeGetMarketSnapshot,
    'get_security_history': executeGetSecurityHistory,
    'resolve_stock_symbol': executeResolveStockSymbol,
    'search_stock_candidates': executeSearchStockCandidates,
    'get_sector_rule_mapping': executeGetSectorRuleMapping,
    
    // Astrology tools
    'get_astro_forecast': executeGetAstroForecast,
    'get_asset_ephemeris': executeGetAssetEphemeris,
    'get_bitcoin_ephemeris': executeGetBitcoinEphemeris,
    'run_short_term_reversal_analysis': executeRunShortTermReversalAnalysis,
    'get_book_reference': executeGetBookReference,
    'web_search_natal_data': executeWebSearchNatalData,
    
    // Analysis tools
    'run_stock_event_analysis': executeRunStockEventAnalysis,
    'run_monthly_reconstruction': executeRunMonthlyReconstruction,
    'run_stock_backtest_pro': executeRunStockBacktestPro,
    'run_bitcoin_window_backtest': executeRunBitcoinWindowBacktest,
    'run_backtest': executeRunBacktest,
    
    // Utility tools
    'web_search': executeWebSearch,
    'run_terminal_command': executeRunTerminalCommand
  };
  
  const executor = executors[toolName];
  if (!executor) {
    throw new Error(`Unknown tool: ${toolName}`);
  }
  
  return await executor(args, context);
}

module.exports = {
  executeTool,
  // Also export individual executors for direct use
  executeGetMarketSnapshot,
  executeGetSecurityHistory,
  // ... all other executors
};
```

---

## 📝 MIGRATION PLAN

### Phase 1: Extract Tool Executors (Week 1)

**Step 1.1:** Create executor files
```bash
cd /home/angle/projects/astrology/astrology/tools
mkdir -p executors
touch executors/market-executors.js
touch executors/astrology-executors.js
touch executors/analysis-executors.js
touch executors/utility-executors.js
touch executors/index.js
```

**Step 1.2:** Extract each tool implementation

From `server.js` lines 5072-5450:
- Copy `executeToolCall` function
- Split into individual executor functions
- Add proper JSDoc comments
- Import dependencies explicitly

**Step 1.3:** Create executor registry
- Map tool names to executor functions
- Add error handling for unknown tools
- Export unified `executeTool()` function

### Phase 2: Update server.js (Week 1)

**Step 2.1:** Import executor module
```javascript
// server.js - Replace executeToolCall implementation
const { executeTool } = require('./tools/executors');

// Old code (lines 5072-5450): DELETED
// async function executeToolCall(toolCall, scope, threadId, ...) { ... }

// New code (1 line):
async function executeToolCall(toolCall, scope, threadId, terminalAccess, bookMode = 'general', userMessage = '') {
  const context = { scope, threadId, terminalAccess, bookMode, userMessage };
  return await executeTool(toolCall.name, toolCall.args, context);
}
```

**Step 2.2:** Remove duplicate utilities
- Delete `normalizeToolArguments` (now in `tools/utils/helpers.js`)
- Delete `normalizeToolCommodity` (now in `tools/market/constants.js`)
- Delete `summarizeForToolResult` (now in `tools/utils/helpers.js`)
- Delete `parseToolCall` (now in `tools/executors/index.js`)

**Step 2.3:** Update imports
```javascript
// Add imports
const { normalizeToolArguments, normalizeToolCommodity, summarizeForToolResult } = require('./tools/utils/helpers');
const { parseToolCall } = require('./tools/executors');

// Remove inline function definitions
```

### Phase 3: Update langchainAgent.js (Week 2)

**Step 3.1:** Import executor module
```javascript
// agent/langchainAgent.js
const { executeTool } = require('../tools/executors');
```

**Step 3.2:** Replace direct tool calls
```javascript
// Old: Direct function call
const result = await getMarketSnapshot({ commodity, days });

// New: Via executor
const result = await executeTool('get_market_snapshot', { commodity, days }, context);
```

**Step 3.3:** Add context passing
```javascript
// Ensure context is available for all tool calls
const context = {
  scope: currentScope,
  threadId,
  terminalAccess,
  bookMode: book,
  userMessage: message
};
```

### Phase 4: Testing & Validation (Week 2)

**Step 4.1:** Unit tests for each executor
```javascript
// tests/unit/tools/executors.test.js

describe('Market Executors', () => {
  test('executeGetMarketSnapshot - commodity', async () => {
    const result = await executeGetMarketSnapshot(
      { commodity: 'gold', days: 365 },
      { scope: 'gold', userMessage: 'gold price history' }
    );
    expect(result.entityType).toBe('commodity');
    expect(result.commodity).toBe('gold');
    expect(result.history).toBeDefined();
  });
  
  test('executeGetMarketSnapshot - stock', async () => {
    const result = await executeGetMarketSnapshot(
      { symbol: 'ZEEL.NS', days: 90 },
      { scope: 'stocks', userMessage: 'Zee Entertainment chart' }
    );
    expect(result.entityType).toBe('security');
    expect(result.symbol).toBe('ZEEL.NS');
  });
});
```

**Step 4.2:** Integration tests
```javascript
// tests/integration/agent-tools.test.js

describe('Agent Tools Integration', () => {
  test('Full tool execution flow', async () => {
    const response = await request(app)
      .post('/api/chat-runs')
      .send({
        threadId: 'test-thread',
        message: 'What is the gold price today?',
        bookMode: 'general'
      });
    
    expect(response.status).toBe(200);
    expect(response.body.events).toContainEqual(
      expect.objectContaining({ type: 'tool_start', name: 'get_market_snapshot' })
    );
  });
});
```

**Step 4.3:** Manual testing checklist
- [ ] All 18 tools execute correctly
- [ ] SSE events stream properly
- [ ] Error handling works (timeouts, API failures)
- [ ] Caching behavior unchanged
- [ ] Logging output identical
- [ ] Response times within 10% of current

---

## 🔍 DETAILED TOOL MAPPING

### Market Executors (5 tools)

| Tool | Current Location | New Location | Dependencies |
|------|-----------------|--------------|--------------|
| `get_market_snapshot` | `server.js:5115` | `executors/market-executors.js` | `market-snapshot.js`, `asset-resolver.js` |
| `get_security_history` | `server.js:5260` | `executors/market-executors.js` | `market-history.js`, `eodhd-client.js` |
| `resolve_stock_symbol` | `server.js:5150` | `executors/market-executors.js` | `asset-resolver.js` |
| `search_stock_candidates` | `server.js:5130` | `executors/market-executors.js` | `yahoo-chart.js` |
| `get_sector_rule_mapping` | `server.js:5085` | `executors/market-executors.js` | `book-context.js` |

### Astrology Executors (6 tools)

| Tool | Current Location | New Location | Dependencies |
|------|-----------------|--------------|--------------|
| `get_astro_forecast` | `server.js:5340` | `executors/astrology-executors.js` | `forecast-builder.js`, `ephemeris-cache.js` |
| `get_asset_ephemeris` | `server.js:5360` | `executors/astrology-executors.js` | `ephemeris-cache.js` |
| `get_bitcoin_ephemeris` | `server.js:5375` | `executors/astrology-executors.js` | `bitcoin-ephemeris.js` |
| `run_short_term_reversal_analysis` | `server.js:5320` | `executors/astrology-executors.js` | `silver-analysis.js` |
| `get_book_reference` | `server.js:5300` | `executors/astrology-executors.js` | `book-context.js` |
| `web_search_natal_data` | `server.js:5075` | `executors/astrology-executors.js` | `web-search.js` |

### Analysis Executors (5 tools)

| Tool | Current Location | New Location | Dependencies |
|------|-----------------|--------------|--------------|
| `run_stock_event_analysis` | `server.js:5200` | `executors/analysis-executors.js` | `stock-event-analysis.js`, `swiss-ephemeris.js` |
| `run_monthly_reconstruction` | `server.js:5240` | `executors/analysis-executors.js` | `monthly-reconstruction.js` |
| `run_stock_backtest_pro` | `server.js:5220` | `executors/analysis-executors.js` | `backtest-runners.js` |
| `run_bitcoin_window_backtest` | `server.js:5280` | `executors/analysis-executors.js` | `bitcoin-backtest.js` |
| `run_backtest` | `server.js:5260` | `executors/analysis-executors.js` | `backtest-runners.js` |

### Utility Executors (2 tools)

| Tool | Current Location | New Location | Dependencies |
|------|-----------------|--------------|--------------|
| `web_search` | `server.js:4304` | `executors/utility-executors.js` | `serper-client.js` |
| `run_terminal_command` | `server.js:5400` | `executors/utility-executors.js` | `child_process` |

---

## 📊 BENEFITS ANALYSIS

### Before Refactoring

| Metric | Value |
|--------|-------|
| **server.js Size** | 5,885 lines |
| **Tool Logic Location** | Mixed with API/auth/SSE |
| **Testability** | Low (need full server context) |
| **Reusability** | Low (tightly coupled) |
| **Maintainability** | Medium (hard to find tool logic) |
| **Onboarding** | Difficult (5.8K lines in one file) |

### After Refactoring

| Metric | Value | Improvement |
|--------|-------|-------------|
| **server.js Size** | ~4,500 lines | **-23%** |
| **Tool Logic Location** | Dedicated executors/ | ✅ Separated |
| **Testability** | High (isolated functions) | ✅ Unit tests possible |
| **Reusability** | High (can use outside agent) | ✅ Import anywhere |
| **Maintainability** | High (clear structure) | ✅ Easy to find |
| **Onboarding** | Easier (modular) | ✅ Clear boundaries |

### Quantitative Benefits

1. **Reduced Cognitive Load:**
   - Before: 5,885 lines in one file
   - After: ~400 lines per executor file
   - **Improvement:** 93% reduction in file complexity

2. **Faster Debugging:**
   - Before: Search through 5.8K lines
   - After: Go directly to executor file
   - **Time Saved:** ~5-10 minutes per bug

3. **Easier Testing:**
   - Before: Integration tests only (need full server)
   - After: Unit tests + integration tests
   - **Coverage:** Can reach 80%+ vs current ~40%

4. **Parallel Development:**
   - Before: One person edits server.js (merge conflicts)
   - After: Multiple people edit different executors
   - **Velocity:** 2-3x faster for team development

---

## ⚠️ RISKS & MITIGATION

### Risk 1: Functionality Regression

**Risk:** Tool behaves differently after refactoring

**Mitigation:**
- Keep exact same logic (copy-paste, don't rewrite)
- Add comprehensive unit tests before deployment
- Run existing integration tests
- Manual testing checklist for all 18 tools
- Deploy to staging first, test for 48 hours

### Risk 2: Performance Degradation

**Risk:** Extra function call overhead slows things down

**Mitigation:**
- Benchmark before/after (should be <5ms difference)
- Use direct imports (no dynamic require in hot path)
- Keep caching layer unchanged
- Profile with `console.time()` during testing

### Risk 3: Import Errors

**Risk:** Circular dependencies or missing imports

**Mitigation:**
- Use dependency graph to order imports
- Run `node --check` on all files before deployment
- Test each executor in isolation first
- Keep shared utilities in `tools/utils/`

### Risk 4: Deployment Issues

**Risk:** Vercel deployment fails due to new file structure

**Mitigation:**
- Test locally with `npm run dev` first
- Deploy to Vercel preview URL
- Run smoke tests on preview
- Only merge to main after preview passes

---

## 🚀 IMPLEMENTATION TIMELINE

### Week 1: Extraction

| Day | Task | Deliverable |
|-----|------|-------------|
| **Mon** | Create executor files, extract market tools | `market-executors.js` |
| **Tue** | Extract astrology tools | `astrology-executors.js` |
| **Wed** | Extract analysis tools | `analysis-executors.js` |
| **Thu** | Extract utility tools, create registry | `utility-executors.js`, `index.js` |
| **Fri** | Update server.js imports, delete old code | `server.js` reduced to ~4.5K lines |

### Week 2: Testing & Deployment

| Day | Task | Deliverable |
|-----|------|-------------|
| **Mon** | Unit tests for all executors | `tests/unit/tools/executors.test.js` |
| **Tue** | Integration tests | `tests/integration/agent-tools.test.js` |
| **Wed** | Manual testing, bug fixes | Testing report |
| **Thu** | Deploy to staging, validate | Staging URL |
| **Fri** | Deploy to production, monitor | Production live |

---

## 📋 CHECKLIST

### Pre-Refactoring

- [ ] Backup current server.js
- [ ] Create feature branch (`refactor/tool-executors`)
- [ ] Run existing tests, document baseline metrics
- [ ] Create executor directory structure

### During Refactoring

- [ ] Extract each tool without modifying logic
- [ ] Add JSDoc comments to all executors
- [ ] Update imports in server.js
- [ ] Delete old inline tool functions
- [ ] Run `node --check` on all files

### Post-Refactoring

- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Manual testing checklist complete
- [ ] Performance within 10% of baseline
- [ ] Deploy to staging, test for 24 hours
- [ ] Deploy to production
- [ ] Monitor logs for 48 hours

---

## 🎯 SUCCESS CRITERIA

### Functional Equivalence

- ✅ All 18 tools work identically
- ✅ Same error messages
- ✅ Same SSE event sequence
- ✅ Same caching behavior
- ✅ Same logging output

### Code Quality

- ✅ Each executor < 100 lines
- ✅ JSDoc comments on all functions
- ✅ Unit test coverage > 80%
- ✅ No circular dependencies
- ✅ ESLint passes

### Performance

- ✅ Response time within 10% of baseline
- ✅ No memory leaks
- ✅ Cache hit rate unchanged
- ✅ Tool execution time unchanged

### Maintainability

- ✅ New developer can find tool logic in < 2 minutes
- ✅ Can add new tool without touching server.js
- ✅ Can test tools in isolation
- ✅ Clear dependency graph

---

## 📞 NEXT STEPS

**Immediate Actions:**

1. **Create executor directory:**
   ```bash
   cd /home/angle/projects/astrology/astrology/tools
   mkdir -p executors
   ```

2. **Start with market executors** (simplest, least dependencies):
   - Extract `get_market_snapshot`
   - Extract `get_security_history`
   - Test in isolation

3. **Validate approach:**
   - Run existing integration tests
   - Confirm no regression
   - Continue with remaining tools

**Questions to Resolve:**

1. Should we keep both old and new code during transition (feature flag)?
2. Do we need backward compatibility layer for existing tool calls?
3. Should we refactor utils first (helpers, cache) or executors first?

---

**This refactoring will reduce server.js by 23%, improve testability, and make the codebase significantly more maintainable without any loss of functionality.** 🎯
