# Agent Interaction Flows

## Overview

This document visualizes how agents communicate and coordinate in the Crypto Watchtower pipeline.

## Pipeline Execution Flow

```
START
  │
  ├──> [GitHub Actions Scheduler]
  │    - Trigger: Every 15 minutes (cron)
  │    - Or: Manual workflow_dispatch
  │    - Or: Push to .github/agents/**
  │
  ▼
┌────────────────────────────────┐
│  Data Fetcher Agent            │
│  ────────────────────────────  │
│  Skills: fetch-crypto-data     │
│  Model: claude-sonnet-4.5      │
│  Tools: bash, curl, jq         │
└────────────────────────────────┘
  │
  ├─ Action: Backup current data
  │  - IF exists: cp crypto-data.json → crypto-data-previous.json
  │  - ELSE: Skip (first run)
  │
  ├─ Action: Fetch from CoinGecko API
  │  - GET /api/v3/coins/markets
  │  - Parameters: top 20, USD, with 1h/24h changes
  │  - Retry: 3 attempts with exponential backoff
  │
  ├─ Action: Process response
  │  - Parse JSON with jq
  │  - Extract required fields
  │  - Add timestamp
  │
  ├─ Output: crypto-data.json
  │  {
  │    "timestamp": "2026-02-11T01:00:00Z",
  │    "data": [{id, symbol, name, current_price, ...}]
  │  }
  │
  └─ Status: ✓ Success OR ✗ Error (use cached)
  │
  ▼
┌────────────────────────────────┐
│  Market Analyst Agent          │
│  ────────────────────────────  │
│  Skills: detect-volatility     │
│  Model: claude-sonnet-4.5      │
│  Tools: bash, python3          │
└────────────────────────────────┘
  │
  ├─ Input Check: Does previous data exist?
  │  - NO: First run → Create empty alerts, EXIT
  │  - YES: Proceed to analysis
  │
  ├─ Action: Load data
  │  - Read: crypto-data.json (current)
  │  - Read: crypto-data-previous.json (15 min ago)
  │
  ├─ Action: Calculate price changes
  │  FOR each cryptocurrency:
  │    change% = ((current - previous) / previous) × 100
  │    IF |change%| > 5:
  │      - Classify severity (medium/high/critical)
  │      - Create alert object
  │      - Add to alerts array
  │
  ├─ Action: Sort alerts
  │  - Primary: By severity (critical > high > medium)
  │  - Secondary: By |change%| descending
  │
  ├─ Output: whale-alerts.json
  │  {
  │    "timestamp": "2026-02-11T01:00:00Z",
  │    "alerts": [{crypto_id, change%, severity, ...}],
  │    "total_alerts": N
  │  }
  │
  └─ Status: ✓ Success (0-20 alerts detected)
  │
  ▼
┌────────────────────────────────┐
│  Report Generator Agent        │
│  ────────────────────────────  │
│  Skills: generate-charts       │
│  Model: claude-sonnet-4.5      │
│  Tools: bash, create, edit     │
└────────────────────────────────┘
  │
  ├─ Input: Read crypto-data.json
  │  - Extract: prices, volumes, market caps
  │  - Prepare: Chart.js datasets
  │
  ├─ Input: Read whale-alerts.json
  │  - Extract: alerts array
  │  - Format: HTML table rows
  │
  ├─ Action: Generate dashboard HTML
  │  - Template: Embedded in workflow
  │  - Style: Dark theme CSS
  │  - Scripts: Chart.js rendering functions
  │  - Meta: Auto-refresh (900s)
  │
  ├─ Action: Create visualizations
  │  Chart 1: Line chart (prices)
  │  Chart 2: Bar chart (volumes)
  │  Chart 3: Horizontal bar (market caps)
  │  Table: Whale alerts (if any)
  │
  ├─ Output: docs/index.html
  │  - Responsive HTML5
  │  - Inline CSS & JavaScript
  │  - Chart.js CDN
  │
  ├─ Output: docs/data/crypto-data.json (copy)
  ├─ Output: docs/data/whale-alerts.json (copy)
  │
  └─ Status: ✓ Dashboard generated
  │
  ▼
┌────────────────────────────────┐
│  Deployment Manager Agent      │
│  ────────────────────────────  │
│  Skills: deploy-github-pages   │
│  Model: claude-sonnet-4.5      │
│  Tools: bash, git              │
└────────────────────────────────┘
  │
  ├─ Action: Validate files
  │  - Check: docs/index.html exists
  │  - Check: docs/data/*.json exist
  │  - Check: HTML structure valid
  │
  ├─ Action: Check for changes
  │  - git diff --quiet docs/
  │  - IF no changes: Skip deployment, EXIT
  │  - ELSE: Proceed to commit
  │
  ├─ Action: Stage changes
  │  - git add docs/index.html
  │  - git add docs/data/*.json
  │
  ├─ Action: Create commit
  │  Message format:
  │  "Update dashboard - 2026-02-11T01:00:00Z
  │   
  │   - 20 coins updated
  │   - 3 whale movements detected"
  │
  ├─ Action: Push to GitHub
  │  - git push origin main
  │  - Trigger: GitHub Pages rebuild
  │  - Wait: 30 seconds for build
  │
  ├─ Action: Verify deployment
  │  - curl -I https://etlee.github.io/stock-website
  │  - Check: HTTP 200 status
  │
  └─ Status: ✓ Deployed OR ⚠ Building
  │
  ▼
 END
  │
  └──> Wait 15 minutes, REPEAT
```

## Agent Communication Matrix

| From Agent | To Agent | Communication Method | Data Format |
|------------|----------|---------------------|-------------|
| Data Fetcher | Market Analyst | File: `crypto-data.json` | JSON |
| Data Fetcher | Market Analyst | File: `crypto-data-previous.json` | JSON |
| Market Analyst | Report Generator | File: `whale-alerts.json` | JSON |
| Data Fetcher | Report Generator | File: `crypto-data.json` | JSON |
| Report Generator | Deployment Manager | File: `docs/index.html` | HTML |
| Report Generator | Deployment Manager | Files: `docs/data/*.json` | JSON |
| Deployment Manager | GitHub Pages | Git push | Commit |

## State Transitions

### First Run (Cold Start)
```
State: No previous data exists

Flow:
1. Data Fetcher: Creates crypto-data.json
   - No previous file exists yet
   - Skip backup step

2. Market Analyst: Detects first run
   - crypto-data-previous.json missing
   - Creates empty whale-alerts.json
   - Exits early (no comparison possible)

3. Report Generator: Generates dashboard
   - Shows "No alerts" message
   - Displays current prices

4. Deployment Manager: Deploys initial dashboard

Result: Baseline established for next run
```

### Second Run (Warm Start)
```
State: Previous data exists

Flow:
1. Data Fetcher: Has previous data
   - Backups crypto-data.json → crypto-data-previous.json
   - Fetches new data → crypto-data.json

2. Market Analyst: Can compare
   - Compares current vs previous
   - Detects 0-N whale movements

3. Report Generator: Full dashboard
   - Includes whale alerts (if any)
   - Shows updated prices

4. Deployment Manager: Updates live site

Result: Continuous monitoring active
```

### Steady State (Normal Operation)
```
Repeats every 15 minutes:

1. Backup → Fetch → Update
2. Compare → Detect → Alert
3. Visualize → Generate → Publish
4. Commit → Push → Deploy

Loop continues indefinitely
```

## Error Recovery Flows

### API Failure
```
Scenario: CoinGecko API returns 429 (rate limit)

Data Fetcher:
  ├─ Attempt 1: Retry after 2 seconds
  ├─ Attempt 2: Retry after 4 seconds
  ├─ Attempt 3: Retry after 8 seconds
  └─ All failed:
      ├─ Check if crypto-data-previous.json exists
      ├─ YES: cp previous → current (use stale data)
      └─ NO: Exit with error, skip pipeline

Market Analyst: Uses stale data, proceeds normally
Report Generator: Shows timestamp (user sees data is old)
Deployment Manager: Updates with stale data

Result: Degraded but operational
```

### Git Push Failure
```
Scenario: git push fails (network error)

Deployment Manager:
  ├─ Attempt 1: Retry push
  ├─ Attempt 2: git pull --rebase, retry push
  ├─ Attempt 3: Wait 10s, retry push
  └─ All failed:
      └─ Log error, continue
          - Dashboard not updated
          - Old version remains live
          - Next run will succeed

Result: Transient failure, self-healing
```

### Data Corruption
```
Scenario: Invalid JSON in data file

Market Analyst:
  ├─ Try to parse with jq
  ├─ Parse fails:
      └─ Create empty whale-alerts.json
          - {"alerts": [], "total_alerts": 0}
          - Log error

Report Generator: Proceeds with partial data
Deployment Manager: Updates (shows error state)

Result: Graceful degradation
```

## Timing Diagram

```
Time:     0s    5s    10s   15s   20s   25s   30s   35s   40s
          │     │     │     │     │     │     │     │     │
Data      ├─────┤
Fetcher   │ Fetch & Process
          │
Market    │     ├───┤
Analyst   │     │ Analyze
          │     │
Report    │          ├─────┤
Generator │          │ Generate HTML
          │          │
Deploy    │               ├──────────────┤
Manager   │               │ Commit & Push & Verify
          │               │
GitHub    │                              ├─────────┤
Pages     │                              │ Build & Deploy
          │                              │
LIVE!     │                                        ✓

Total: ~40 seconds from trigger to live update
```

## Data Flow Diagram

```
┌──────────────────┐
│  CoinGecko API   │
│  (External)      │
└────────┬─────────┘
         │ HTTPS GET
         ▼
    ┌─────────┐
    │  JSON   │
    │ Response│
    └────┬────┘
         │
         ▼
┌──────────────────────────┐
│  crypto-data-raw.json    │  (Temporary)
└────────┬─────────────────┘
         │ jq transform
         ▼
┌──────────────────────────┐
│  crypto-data.json        │◄─────┐
└────────┬─────────────────┘      │
         │                         │ Backup
         │ Copy                    │
         ▼                         │
┌──────────────────────────┐      │
│crypto-data-previous.json │──────┘
└────────┬─────────────────┘
         │
         │ Both read by
         ▼
┌──────────────────────────┐
│  Market Analyst (Python) │
└────────┬─────────────────┘
         │ Calculation
         ▼
┌──────────────────────────┐
│  whale-alerts.json       │
└────────┬─────────────────┘
         │
         │ Read by
         ▼
┌──────────────────────────┐
│  Report Generator        │◄─── crypto-data.json
└────────┬─────────────────┘
         │ HTML generation
         ▼
┌──────────────────────────┐
│  docs/index.html         │
│  docs/data/*.json        │  (Copies)
└────────┬─────────────────┘
         │ Git commit
         ▼
┌──────────────────────────┐
│  GitHub Repository       │
│  (main branch)           │
└────────┬─────────────────┘
         │ GitHub Pages
         ▼
┌──────────────────────────┐
│  etlee.github.io         │
│  /stock-website          │  🌐 LIVE
└──────────────────────────┘
```

## Concurrency & Locking

### Sequential Execution (Current)
```
Agents run one at a time:
[Data Fetcher] → [Market Analyst] → [Report Gen] → [Deploy]

Pros:
✓ Simple to reason about
✓ No race conditions
✓ Clear dependencies
✓ Easy debugging

Cons:
✗ Slower (40s total)
✗ One agent blocks others
```

### Parallel Execution (Future Enhancement)
```
Independent agents could run in parallel:

[Data Fetcher] ──┐
                 ├─→ [Report Generator] → [Deploy]
[Market Analyst]─┘

Requirements:
- File locking mechanism
- Dependency graph
- Error coordination
```

## Monitoring Points

### Checkpoints
```
1. ✓ API Response: CoinGecko returns 200
2. ✓ Data Valid: JSON parses successfully
3. ✓ Comparison: Previous data available
4. ✓ Alerts: Whale movements detected (0-20)
5. ✓ HTML: Dashboard generated
6. ✓ Commit: Changes staged and committed
7. ✓ Push: Sent to GitHub
8. ✓ Deploy: GitHub Pages returns 200

If any checkpoint fails:
- Log error
- Attempt recovery
- Continue or exit based on criticality
```

### Metrics Collected
```
- API response time (ms)
- Number of coins fetched (should be 20)
- Whale alerts count (0-20)
- HTML file size (bytes)
- Deployment time (seconds)
- Success/failure status
```

## Agent Isolation

Each agent is **stateless** and **idempotent**:

```
Stateless:
- No memory between runs
- Reads state from files
- No global variables

Idempotent:
- Running twice = same result
- Safe to retry
- No side effects beyond expected outputs

Example:
  Run 1: Fetch 20 coins → crypto-data.json
  Run 2: Fetch 20 coins → crypto-data.json (overwrites)
  Result: Same (latest data)
```

## Summary

**Communication**: File-based (JSON)
**Coordination**: Sequential pipeline
**Error Handling**: Retry + fallback + logging
**State Management**: Previous snapshot comparison
**Deployment**: Git-based continuous delivery
**Monitoring**: Logs + timestamps + verification
**Recovery**: Automatic (stale data, retries, graceful degradation)

This architecture provides:
- ✓ Reliability (error recovery)
- ✓ Observability (logs, metrics)
- ✓ Maintainability (clear flows)
- ✓ Extensibility (add new agents easily)
