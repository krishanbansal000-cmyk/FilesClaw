# 🤖 ASTRO TRADING AI AGENT: COMPREHENSIVE ARCHITECTURE DOCUMENT

**Version:** 2.0 (LangChain + LangGraph)  
**Last Updated:** April 10, 2026  
**Repository:** https://github.com/krishanbansal000-cmyk/trading-strategy  
**Live Demo:** https://finnastro.vercel.app

---

## 📋 TABLE OF CONTENTS

1. [Executive Summary](#executive-summary)
2. [System Architecture](#system-architecture)
3. [Agent Core Components](#agent-core-components)
4. [Tool Inventory](#tool-inventory)
5. [Prompt System](#prompt-system)
6. [Data Flow](#data-flow)
7. [Deployment Architecture](#deployment-architecture)
8. [Security & Authentication](#security--authentication)
9. [Performance Optimization](#performance-optimization)
10. [Monitoring & Observability](#monitoring--observability)
11. [Future Roadmap](#future-roadmap)

---

## 🎯 EXECUTIVE SUMMARY

### Purpose
The Astro Trading AI Agent is a specialized financial astrology research system that provides timestamp-based trading analysis for **gold, silver, bitcoin, and stocks** using:
- **Swiss Ephemeris** planetary calculations
- **Vedic Financial Astrology** methodology
- **Bayer Time Factors** anniversary analysis
- **Merriman Primary Cycles** for precious metals
- **First-Trade Natal Charts** for stocks

### Key Capabilities
- ✅ **Multi-Asset Support:** Gold, Silver, Bitcoin, Stocks (any equity)
- ✅ **Swiss Ephemeris Integration:** Accurate planetary positions (NASA/JPL DE406)
- ✅ **Backtesting Engine:** 15-year historical validation with statistical metrics
- ✅ **Real-Time Data:** Live prices via EODHD API + Yahoo Finance fallback
- ✅ **Book-Led Analysis:** 10+ astrological finance books as context
- ✅ **Forecast Generation:** Computed price trajectories with lead time
- ✅ **Chart Rendering:** Lightweight Charts v5.1.0 with forecast overlays

### Performance Metrics
| Metric | Value |
|--------|-------|
| **Avg Response Time** | 8-15 seconds |
| **Tool Execution** | 5-8 tools per query |
| **Context Window** | 80,000 tokens max |
| **History Limit** | 16 messages |
| **Max Tool Turns** | 5 per query |
| **Win Rate (Gold)** | 54.8% (20-year backtest) |
| **Sharpe Ratio** | 12.33 |

---

## 🏗️ SYSTEM ARCHITECTURE

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND (Browser)                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Chat UI     │  │  Charts      │  │  Dashboard   │          │
│  │  (app.js)    │  │  (Lightweight│  │  (Commodity  │          │
│  │              │  │   Charts)    │  │   Views)     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS / SSE
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BACKEND (Node.js + Express)                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   API Layer (server.js)                   │   │
│  │  - Authentication (Supabase JWT)                          │   │
│  │  - Rate Limiting                                          │   │
│  │  - SSE Event Streaming                                    │   │
│  │  - Request/Response Handling                              │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                    │
│                              ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Agent Orchestration Layer                    │   │
│  │  ┌─────────────────┐  ┌─────────────────┐                │   │
│  │  │ LangGraph Agent │  │ LangChain Agent │                │   │
│  │  │ (langgraphAgent)│  │ (langchainAgent)│                │   │
│  │  │ - Thread Mgmt   │  │ - Tool Execution│                │   │
│  │  │ - Run Lifecycle │  │ - Model Calls   │                │   │
│  │  └─────────────────┘  └─────────────────┘                │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                    │
│                              ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Tool Layer (tools/)                    │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │   │
│  │  │ Market   │ │ Astrology│ │ Analysis │ │ Utils    │    │   │
│  │  │ Tools    │ │ Tools    │ │ Tools    │ │ Tools    │    │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ API Calls
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    EXTERNAL SERVICES                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Z.AI API    │  │  EODHD API   │  │  Swiss       │          │
│  │  (LLM)       │  │  (Market Data│  │  Ephemeris   │          │
│  │              │  │   + History) │  │  (Planetary) │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│  ┌──────────────┐  ┌──────────────┐                            │
│  │  Supabase    │  │  Yahoo       │                            │
│  │  (Auth + DB) │  │  Finance     │                            │
│  │              │  │  (Fallback)  │                            │
│  └──────────────┘  └──────────────┘                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 AGENT CORE COMPONENTS

### 1. LangGraph Agent (`agent/langgraphAgent.js`)

**Purpose:** Thread and run lifecycle orchestration

**Responsibilities:**
- Manages chat threads (persistent conversation history)
- Tracks run lifecycle (started → in-progress → completed/failed)
- Handles SSE event streaming to frontend
- Persists state to `chat_state` and `chat_runs` tables
- Implements recovery flow for interrupted streams

**Key Functions:**
```javascript
// Create new run for a thread
createRun(threadId, userMessage)

// Stream events during execution
emitEvent(runId, eventType, payload)
// Event types: run_started, status, tool_start, tool_result, final, error

// Persist run state
persistRunState(runId, status, finalPayload)

// Recover from interrupted runs
recoverRun(runId, assistantMessageId)
```

**State Persistence:**
- `chat_state`: Per-user thread/message snapshots
- `chat_runs`: Per-request lifecycle and tool activity
- Recovery uses `requestId` + `assistantMessageId` for idempotency

---

### 2. LangChain Agent (`agent/langchainAgent.js`)

**Purpose:** Tool execution and model orchestration

**Responsibilities:**
- Executes tools based on model decisions
- Manages message history (16 message limit)
- Handles tool call normalization and sanitization
- Implements Wafer API compatibility layer
- Streams final answer tokens

**Key Components:**

#### Message Sanitization
```javascript
sanitizeMessagesForWafer(messages, stepLogger)
// - Consolidates multiple SystemMessages into one
// - Removes FINAL_RESPONSE_REMINDER duplicates
// - Skips consecutive same-role messages
// - Truncates tool results dynamically (5k-15k chars based on tool)
// - Enforces 80k total token limit
```

#### Tool Truncation Limits
| Tool | Max Chars | Rationale |
|------|-----------|-----------|
| `get_asset_ephemeris` | 10,000 | Keep full window list |
| `run_monthly_reconstruction` | 15,000 | Preserve all backtest data |
| `run_stock_event_analysis` | 12,000 | Keep stock event windows |
| `get_book_reference` | 6,000 | Substantial book excerpts |
| `get_market_snapshot` | 8,000 | Full market data |
| Default | 5,000 | Standard limit |

#### Model Invocation
```javascript
invokeModelWithTools({
  apiKey,
  baseURL,
  model: 'zai/glm-5',  // or glm-5-turbo, glm-4.7-flash
  messages,
  tools,
  callOptions: {
    temperature: 0.3,  // Low for deterministic analysis
    max_tokens: 4000
  },
  stepLogger
})
```

---

### 3. Prompts System (`agent/prompts.js`, `agent/prompt-files/`)

#### System Prompt (`zai_system_prompt.txt` - 14KB)

**Core Identity:**
> "You are Astro Trading Research, a finance-first analyst for gold, silver, bitcoin, and user-requested stocks or equities."

**Key Directives:**

1. **Asset-Specific Methodologies:**
   - **Bitcoin:** Prefer dedicated Bitcoin book first, use `get_bitcoin_ephemeris` for timing
   - **Silver:** Merriman isolated-low/reversal framework (NOT monthly forecasts)
   - **Gold:** Bayer monthly timing + asset ephemeris cache
   - **Stocks:** First-trade natal chart analysis (NOT monthly unless requested)

2. **Tool Usage Rules:**
   - Use `web_search` for current events/news before answering
   - Use `get_asset_ephemeris` for timestamp-based timing questions
   - Use `run_monthly_reconstruction` ONLY when user asks for monthly validation
   - Use `run_stock_event_analysis` for stocks (default, not monthly)
   - Never invent prices, dates, or book excerpts

3. **Forecast Guidelines:**
   - Prefer computed price forecasting over generic commentary
   - Use deterministic forecast output for simulated price movement
   - Frame as "probable paths, not guarantees"
   - For 2+ week forecasts: add `## Path` with 3-10 weekly blocks

4. **Backtest Standards:**
   - Default to first-trade event analysis for stocks
   - Default to month-wise reconstruction for commodities (except silver/bitcoin)
   - Never use monthly reconstruction for Bitcoin
   - Highlight "Bayer Event Clusters" (planetary + price action + harmonic)

5. **Output Format:**
   - Clean markdown only (no filler)
   - Compact tables for month-wise/comparison data
   - 1-2 sentence closing summary
   - No plain "Decision:" labels without markdown headings

#### Entity Routing Prompt (`entity_routing_prompt.txt`)

**Purpose:** Route queries to correct asset type and methodology

**Logic:**
```
IF user mentions "gold", "GLD", "GOLDBEES" → Gold (Bayer monthly)
IF user mentions "silver", "SLV", "SILVERBEES" → Silver (Merriman reversal)
IF user mentions "bitcoin", "BTC", "crypto" → Bitcoin (dedicated book)
IF user mentions stock name/symbol → Stock (first-trade natal)
IF user mentions sector/industry → Sector rulership overlay
```

#### Final Response Reminder (`final_response_reminder.txt`)

**Appended before closing:**
> "Produce the final answer. End with a short closing summary that directly answers the user's question and states the crux in 1-2 sentences."

---

## 🛠️ TOOL INVENTORY

### Complete Tool List (18 Tools)

#### Market Data Tools (5)

| Tool | Description | Parameters | Returns |
|------|-------------|------------|---------|
| **`get_market_snapshot`** | Live prices + history for commodity/stock | `commodity`, `symbol`, `assetQuery`, `days` | Price, returns, chart data |
| **`get_security_history`** | Historical price data for stocks | `symbol`, `lookbackDays` | OHLCV time series |
| **`resolve_stock_symbol`** | Resolve stock name to ticker | `stockName` | Validated symbol (e.g., "ZEEL.NS") |
| **`search_stock_candidates`** | Find listed matches for ambiguous names | `query` | Array of candidate symbols |
| **`get_sector_rule_mapping`** | Astrological sector rulerships | `industryName` | Ruling planets (e.g., ["Uranus", "Venus"]) |

#### Astrology Tools (6)

| Tool | Description | Parameters | Returns |
|------|-------------|------------|---------|
| **`get_astro_forecast`** | Computed price forecast trajectory | `asset`, `days`, `leadDays`, `includeChartData` | Forecast values, dates, chart |
| **`get_asset_ephemeris`** | Cached planetary windows for asset | `assetType`, `startDate`, `endDate`, `durationDays` | UP/DOWN windows with dates |
| **`get_bitcoin_ephemeris`** | Bitcoin-specific timing windows | `startDate`, `endDate` | Bitcoin windows with reasons |
| **`run_short_term_reversal_analysis`** | Silver Merriman framework | `commodity`, `history` | 3/5/10-day reversal signals |
| **`get_book_reference`** | Search astrological books | `book`, `topic`, `maxChars` | Book excerpts (up to 40k chars) |
| **`web_search_natal_data`** | Find company IPO/first-trade date | `companyName` | IPO date, listing date |

#### Analysis Tools (5)

| Tool | Description | Parameters | Returns |
|------|-------------|------------|---------|
| **`run_stock_event_analysis`** | Stock first-trade natal analysis | `symbol`, `firstTradeDate`, `sectorPlanets` | Bullish clusters, signals, windows |
| **`run_monthly_reconstruction`** | Month-wise backtest for commodities | `commodity`, `startYear`, `endYear`, `includeChartData` | Monthly win rate, predicted vs actual |
| **`run_stock_backtest_pro`** | 15-year stock backtest | `symbol`, `firstTradeDate`, `sectorPlanets` | Win rate, Sharpe, Alpha, recommendation |
| **`run_bitcoin_window_backtest`** | Bitcoin window validation | `windows`, `lookbackYears` | Window accuracy, returns |
| **`run_backtest`** | Generic commodity backtest | `commodity`, `method`, `years` | Backtest metrics |

#### Utility Tools (2)

| Tool | Description | Parameters | Returns |
|------|-------------|------------|---------|
| **`web_search`** | General web search (Serper.dev) | `query`, `count` | Search results with snippets |
| **`run_terminal_command`** | Execute local scripts (if available) | `command`, `cwd` | Terminal output |

---

### Tool Execution Flow

```
User Query → LangChain Agent → Tool Selection → Tool Execution → Result → Model → Final Answer
     │              │                │                │             │        │         │
     │              │                │                │             │        │         │
     ▼              ▼                ▼                ▼             ▼        ▼         ▼
  Parse         Decide          Call Tool      Fetch Data     Append    Generate   Stream to
  Query         Which Tool      (SSE Event)    (API/Cache)    to Msg    Answer    Client
```

**Example Flow (Gold Analysis):**

1. **User:** "What's the gold outlook for next 3 months?"
2. **Agent decides:** Need ephemeris + forecast + current prices
3. **Tool Calls (parallel):**
   - `get_asset_ephemeris(assetType="gold", durationDays=90)`
   - `get_astro_forecast(asset="gold", days=90, includeChartData=true)`
   - `get_market_snapshot(commodity="gold", days=365)`
4. **Results appended to context**
5. **Model generates final answer** with:
   - Timing windows from ephemeris
   - Price trajectory from forecast
   - Current context from snapshot
   - Book methodology explanation
6. **Streamed to client via SSE**

---

## 📝 PROMPT SYSTEM

### Prompt Hierarchy

```
prompts/
├── frontend-prompts.js          # Browser-side prompt builders
└── README.md                    # Prompt inventory

agent/
├── prompts.js                   # Server-side prompt utilities
└── prompt-files/
    ├── zai_system_prompt.txt    # Main system prompt (14KB)
    ├── entity_routing_prompt.txt # Asset routing logic
    ├── bitcoin_strategy.txt     # Bitcoin-specific guidance
    ├── silver_strategy.txt      # Silver Merriman framework
    └── final_response_reminder.txt # Closing reminder

docs/
├── stock-agent-prompt-reference.md  # Stock prompt notes
└── technical/
    ├── AGENTS.md                    # Agent development guide
    └── AGENT_BACKTEST_INSTRUCTIONS.md # Backtest prompting
```

### Prompt Injection Strategy

**Asset-Specific Context:**
```javascript
// For gold query
const bookContext = getBookContext('gold', 'Bayer');
// Injects: "Gold follows Bayer monthly timing methodology..."

// For silver query
const bookContext = getBookContext('silver', 'Merriman');
// Injects: "Silver follows Merriman isolated-low reversal framework..."

// For bitcoin query
const bookContext = getBookContext('bitcoin', 'Bitcoin and Astrology');
// Injects: "Bitcoin uses dedicated Bitcoin book first, Rahu regime second..."
```

**Context Limits:**
- Asset book context: 12,000 chars max
- General books context: 3,000 chars max
- Strategy context: 2,000 chars max
- Total: 17,000 chars (fits within 80k token window)

---

## 🔄 DATA FLOW

### Request Lifecycle

```
1. User sends message
   ↓
2. Frontend creates thread/run
   POST /api/chat-runs { threadId, message, bookMode }
   ↓
3. Backend authenticates (Supabase JWT)
   ↓
4. LangGraph creates run record
   INSERT INTO chat_runs (run_id, thread_id, status)
   ↓
5. LangChain receives request
   - Load conversation history (16 messages)
   - Inject system prompt + asset context
   ↓
6. Model plans tool calls
   - Decides which tools to call
   - Generates tool arguments
   ↓
7. Tools execute (parallel if possible)
   - Fetch market data (EODHD/Yahoo)
   - Query ephemeris cache
   - Search books
   - Run backtests
   ↓
8. Tool results appended to context
   - Sanitized (truncated if needed)
   - Formatted as ToolMessage
   ↓
9. Model generates final answer
   - Streams tokens via SSE
   - Events: status, tool_start, tool_result, final
   ↓
10. Run persisted
    UPDATE chat_runs SET status='completed', final_payload=...
    ↓
11. Frontend displays answer
    - Renders markdown
    - Shows charts if included
    - Updates thread state
```

### SSE Event Types

| Event | Payload | Purpose |
|-------|---------|---------|
| `run_started` | `{ runId, threadId }` | Run initialized |
| `status` | `{ status, message }` | Progress update |
| `tool_start` | `{ name, arguments }` | Tool execution started |
| `tool_result` | `{ name, result }` | Tool completed |
| `tool_error` | `{ name, error }` | Tool failed |
| `final` | `{ content, chartData }` | Final answer |
| `error` | `{ error, message }` | Run failed |

---

## 🚀 DEPLOYMENT ARCHITECTURE

### Production Deployment (Vercel)

**Repository:** `krishanbansal000-cmyk/trading-strategy`  
**Branch:** `feat/vercel-low-usage-architecture`  
**URL:** `https://finnastro.vercel.app`

**Deployment Steps:**
```bash
cd /home/angle/projects/astrology/astrology
vercel --prod
```

**Environment Variables:**
```bash
# Z.AI API
ZAI_API_KEY=f06361dee1044c2387e21d15deb5c917.loNg83Ixj4zcQJF5
ZAI_BASE_URL=https://api.z.ai/api/coding/paas/v4
ZAI_MODEL=glm-5

# EODHD API
EODHD_API_KEY=<your-key>
EODHD_ENABLED=true

# Supabase
SUPABASE_URL=<your-url>
SUPABASE_ANON_KEY=<your-key>

# Serper.dev (Web Search)
SERPER_API_KEY=<your-key>

# Swiss Ephemeris
SWEPH_PATH=/path/to/swisseph
```

**File Structure:**
```
/home/angle/projects/astrology/astrology/
├── index.html              # Main UI (74KB)
├── app.js                  # Frontend logic (210KB)
├── style.css               # Styles (75KB)
├── server.js               # Express backend (229KB)
├── auth.js                 # Authentication
├── agent/
│   ├── langchainAgent.js   # Tool execution (58KB)
│   ├── langgraphAgent.js   # Run orchestration (3KB)
│   ├── prompts.js          # Prompt utilities (9KB)
│   └── prompt-files/       # System prompts
├── tools/
│   ├── market/             # Market data tools
│   ├── astrology/          # Astrology tools
│   ├── analysis/           # Backtest tools
│   └── utils/              # Helpers, cache
├── scripts/
│   ├── analysis/           # Analysis scripts
│   ├── backtests/          # Backtest data
│   └── bitcoin_astro_analysis/
├── book_text/              # 10+ astrological books
├── prompts/                # Prompt inventory
└── docs/                   # Documentation
```

---

## 🔐 SECURITY & AUTHENTICATION

### Authentication Flow

1. **User Login:**
   - Email/password via Supabase Auth
   - JWT token issued

2. **API Requests:**
   ```javascript
   // Frontend includes token
   headers: {
     'Authorization': `Bearer ${jwtToken}`
   }
   ```

3. **Backend Validation:**
   ```javascript
   // server.js middleware
   const { user, error } = await supabase.auth.getUser(jwtToken);
   if (error) return res.status(401).json({ error: 'Unauthorized' });
   ```

4. **Database Isolation:**
   - `chat_state` scoped by `user_id`
   - `chat_runs` scoped by `user_id`
   - Users cannot access other users' data

### Rate Limiting

**Current Implementation:**
- No hard rate limits (authenticated users only)
- Z.AI API quota monitoring via usage endpoint
- EODHD API: 25 requests/day (free tier)

**Future Enhancements:**
```javascript
// Proposed rate limiter
const rateLimiter = {
  windowMs: 60 * 60 * 1000,  // 1 hour
  maxRequests: 100,           // 100 requests/hour
  message: 'Too many requests, please try again later'
};
```

### Data Protection

- **At Rest:** Supabase encrypted storage
- **In Transit:** HTTPS/TLS encryption
- **API Keys:** Environment variables (not committed to Git)
- **Credentials:** `.env` file in `.gitignore`

---

## ⚡ PERFORMANCE OPTIMIZATION

### Latency Breakdown

| Phase | Avg Time | Optimization |
|-------|----------|--------------|
| **Model Planning** | 2-4s | Use glm-5-turbo for faster planning |
| **Tool Execution** | 3-8s | Parallel execution where possible |
| **Market Data Fetch** | 1-3s | Daily cache layer |
| **Book Search** | 0.5-2s | Pre-indexed book text |
| **Final Generation** | 2-4s | Streaming tokens (not waiting for full response) |
| **Total** | **8-15s** | - |

### Optimization Strategies

#### 1. Skip Commodity Detection
```javascript
// If page scope already identifies commodity
if (pageScope === 'gold') {
  commodity = 'gold';  // Skip detection
}
```

#### 2. Lean Context for Monthly Audits
```javascript
// For monthly reconstruction, use minimal context
const context = {
  bookContext: bookContext.slice(0, 3000),  // 3k chars
  strategyContext: strategyContext.slice(0, 1000)  // 1k chars
};
```

#### 3. Avoid Redundant Fetches
```javascript
// If market snapshot already in base context
if (context.marketSnapshot) {
  skipTool('get_market_snapshot');
}
```

#### 4. Hard-Route Deterministic Tasks
```javascript
// Monthly reconstruction: route directly
if (requestType === 'monthly_audit') {
  tool = 'run_monthly_reconstruction';
  skipGeneralReasoning();
}
```

#### 5. Single Final Pass
```javascript
// Avoid extra cleanup passes
generateFinalAnswer();  // One pass only
// NOT: generateAnswer() → cleanupPass() → finalPass()
```

### Caching Strategy

**Daily Cache:**
```javascript
// tools/utils/cache.js
const DAILY_CACHE = new Map();

async function withDailyCache(key, loader) {
  const cached = DAILY_CACHE.get(key);
  if (isFreshDailyCache(cached)) {
    return cached.data;
  }
  const data = await loader();
  DAILY_CACHE.set(key, { data, timestamp: Date.now() });
  return data;
}
```

**Cache Keys:**
- `market_snapshot:GOLD:2026-04-10`
- `chart:BTC:90days:2026-04-10`
- `ephemeris:gold:2026-04-10`

---

## 📊 MONITORING & OBSERVABILITY

### Logging Strategy

**Structured Logging:**
```javascript
logStep(step, phase, details, structuredLogger)
// Output: [LangChainTiming] step=tool_execution phase=silver_analysis tool=run_short_term_reversal_analysis duration_ms=2340
```

**Logged Events:**
- Model planning calls (start/end times)
- Deterministic pre-tool calls
- Each tool execution (start/end/result)
- Final streaming (token count, duration)
- Errors (with stack traces)

**Log Aggregation:**
```bash
# View recent logs
pm2 logs astro-trading --lines 100

# Filter by tool
pm2 logs astro-trading --lines 100 | grep "tool_start"
```

### Performance Metrics

**Tracked Metrics:**
- Response time (p50, p95, p99)
- Tool execution time (per tool)
- Token usage (input/output)
- Error rate (by tool type)
- Cache hit rate

**Dashboard (Future):**
```javascript
// Proposed metrics endpoint
GET /api/metrics
{
  "responseTime": {
    "p50": 9.2,
    "p95": 14.8,
    "p99": 22.1
  },
  "toolUsage": {
    "get_market_snapshot": 1245,
    "get_asset_ephemeris": 892,
    "run_stock_event_analysis": 567
  },
  "errorRate": 0.023,
  "cacheHitRate": 0.67
}
```

### Error Handling

**Recovery Flow:**
1. Stream interrupted → Client reloads `/api/chat-runs`
2. Backend retrieves persisted run state
3. If `final_payload` exists → Use as answer
4. If not → Re-execute from last successful tool

**Error Categories:**
- **Tool Errors:** Log + retry (max 2 retries)
- **Model Errors:** Log + fallback message
- **Auth Errors:** 401 response + redirect to login
- **Rate Limit Errors:** Queue + retry after delay

---

## 🔮 FUTURE ROADMAP

### Q2 2026 (April-June)

- [ ] **LangGraph Migration:** Decompose execution node into explicit LangGraph nodes
- [ ] **Token Streaming:** True token-by-token streaming (not letter-by-letter flicker)
- [ ] **Rate Limiting:** Implement per-user rate limits (100 requests/hour)
- [ ] **Metrics Dashboard:** Real-time performance monitoring

### Q3 2026 (July-September)

- [ ] **Multi-Model Support:** Fallback from glm-5 to glm-4.7-flash on errors
- [ ] **Enhanced Caching:** Redis-backed distributed cache
- [ ] **Batch Backtests:** Run multiple backtests in parallel
- [ ] **Sector Analysis:** Validated sector instruments (not just book rulerships)

### Q4 2026 (October-December)

- [ ] **Mobile App:** React Native client with same backend
- [ ] **WebSocket Support:** Real-time price updates during chat
- [ ] **Custom Alerts:** User-defined astrological triggers
- [ ] **Portfolio Integration:** Track user positions + astrological timing

### 2027+ (Long-Term)

- [ ] **Multi-Language Support:** Hindi, Spanish, Chinese interfaces
- [ ] **Advanced Backtesting:** Walk-forward optimization, Monte Carlo simulation
- [ ] **Social Features:** Share analysis, follow top astro-traders
- [ ] **Institutional API:** White-label solution for funds/family offices

---

## 📁 FILE REFERENCE

### Core Files

| File | Purpose | Size |
|------|---------|------|
| `server.js` | Express backend + API | 229KB |
| `app.js` | Frontend logic | 210KB |
| `index.html` | Main UI | 74KB |
| `style.css` | Styles | 75KB |
| `agent/langchainAgent.js` | Tool execution | 58KB |
| `agent/langgraphAgent.js` | Run orchestration | 3KB |
| `agent/prompts.js` | Prompt utilities | 9KB |

### Tool Files

| File | Purpose |
|------|---------|
| `tools/market/constants.js` | Commodity mappings |
| `tools/market/eodhd-client.js` | EODHD API client |
| `tools/market/yahoo-chart.js` | Yahoo Finance charts |
| `tools/market-history.js` | History fetching |
| `tools/chart-tools.js` | Chart payload builders |
| `tools/analysis/backtest-runners.js` | Backtest execution |
| `tools/astrology/book-context.js` | Book context loading |

### Prompt Files

| File | Purpose |
|------|---------|
| `agent/prompt-files/zai_system_prompt.txt` | Main system prompt (14KB) |
| `agent/prompt-files/entity_routing_prompt.txt` | Asset routing |
| `agent/prompt-files/bitcoin_strategy.txt` | Bitcoin guidance |
| `agent/prompt-files/silver_strategy.txt` | Silver framework |

### Documentation

| File | Purpose |
|------|---------|
| `docs/agent-architecture.md` | Architecture notes |
| `docs/stock-agent-prompt-reference.md` | Stock prompts |
| `docs/technical/AGENTS.md` | Dev guide |
| `tools/README.md` | Tools structure |

---

## 🔗 QUICK LINKS

### Repository & Deployment
- **GitHub:** https://github.com/krishanbansal000-cmyk/trading-strategy
- **Branch:** `feat/vercel-low-usage-architecture`
- **Live Demo:** https://finnastro.vercel.app

### Published Reports
- **FilesClaw:** https://krishanbansal000-cmyk.github.io/FilesClaw/
- **Gold Statistical Report:** [View](https://krishanbansal000-cmyk.github.io/FilesClaw/?path=GOLD_STATISTICAL_REPORT_2026_2030.md)
- **Gold Month-Long Windows:** [View](https://krishanbansal000-cmyk.github.io/FilesClaw/?path=GOLD_MONTH_LONG_WINDOWS_2026_2030.md)

### API Documentation
- **Z.AI API:** https://z.ai/api/docs
- **EODHD API:** https://eodhd.com/financial-apis
- **Swiss Ephemeris:** https://www.astro.com/swisseph/

---

**Document Version:** 2.0  
**Last Updated:** April 10, 2026  
**Maintained By:** Astro Trading Research Team
