# Agent Interaction Protocols

## Purpose
Define how agents communicate, coordinate, and share data in the Crypto Watchtower pipeline.

## Communication Model

### File-Based State Management
Agents communicate through **JSON files** rather than direct API calls or shared memory.

**Rationale**:
- ✓ Simple and debuggable
- ✓ Persistent state survives agent crashes
- ✓ Easy to inspect and validate
- ✓ Works with GitHub Actions
- ✓ Supports manual intervention if needed

### Data Flow Diagram
```
┌─────────────────┐
│ Data Fetcher    │
│ Agent           │
└────────┬────────┘
         │ Writes: crypto-data.json
         │ Writes: crypto-data-previous.json
         ▼
┌─────────────────┐
│ Market Analyst  │
│ Agent           │
└────────┬────────┘
         │ Reads:  crypto-data.json
         │ Reads:  crypto-data-previous.json
         │ Writes: whale-alerts.json
         ▼
┌─────────────────┐
│ Report Gen      │
│ Agent           │
└────────┬────────┘
         │ Reads:  crypto-data.json
         │ Reads:  whale-alerts.json
         │ Writes: docs/index.html
         │ Writes: docs/data/*.json
         ▼
┌─────────────────┐
│ Deployment Mgr  │
│ Agent           │
└────────┬────────┘
         │ Reads:  docs/
         │ Pushes: GitHub Pages
         ▼
      🌐 Live!
```

## Agent Execution Order

### Sequential Pipeline
Agents **MUST** execute in this order:
1. **Data Fetcher** → Fetches fresh data
2. **Market Analyst** → Analyzes changes
3. **Report Generator** → Creates dashboard
4. **Deployment Manager** → Publishes to web

**Why Sequential?**
- Each agent depends on outputs from previous agents
- Prevents race conditions and data corruption
- Simplifies error handling

### Parallel Execution (Future)
Some agents could run in parallel:
- ✓ Data Fetcher + Historical Data Fetcher (different APIs)
- ✗ Market Analyst before Data Fetcher (breaks dependency)

## File Contracts

### crypto-data.json
**Owner**: Data Fetcher Agent  
**Readers**: Market Analyst, Report Generator  
**Update Frequency**: Every 15 minutes

**Schema**:
```json
{
  "timestamp": "2026-02-11T01:00:00.000Z",  // ISO 8601 UTC
  "data": [
    {
      "id": "bitcoin",                       // CoinGecko ID
      "symbol": "btc",                       // Lowercase
      "name": "Bitcoin",                     // Display name
      "current_price": 45000.50,             // USD
      "price_change_percentage_1h": 0.5,     // Percentage
      "price_change_percentage_24h": 2.3,    // Percentage
      "total_volume": 25000000000,           // USD
      "market_cap": 850000000000,            // USD
      "last_updated": "2026-02-11T01:00:00.000Z"
    }
  ]
}
```

**Validation**:
- Must contain exactly 20 cryptocurrencies
- All numeric fields must be numbers (not strings)
- Timestamp must be recent (within last 5 minutes)
- No null or undefined values

### crypto-data-previous.json
**Owner**: Data Fetcher Agent  
**Readers**: Market Analyst  
**Update Frequency**: Every 15 minutes (copy of previous crypto-data.json)

**Schema**: Same as `crypto-data.json`

**Purpose**: Enables comparison for whale detection

### whale-alerts.json
**Owner**: Market Analyst Agent  
**Readers**: Report Generator  
**Update Frequency**: Every 15 minutes

**Schema**:
```json
{
  "timestamp": "2026-02-11T01:00:00.000Z",
  "alerts": [
    {
      "crypto_id": "bitcoin",
      "symbol": "BTC",                       // Uppercase
      "name": "Bitcoin",
      "price_change_percent": 7.5,           // Positive or negative
      "previous_price": 45000.50,
      "current_price": 48375.54,
      "severity": "high",                    // medium|high|critical
      "emoji": "🟠",                          // 🟡|🟠|🔴
      "time_interval": "15min",
      "alert_message": "Bitcoin surged 7.5% in 15 minutes"
    }
  ],
  "total_alerts": 1
}
```

**Validation**:
- `alerts` array may be empty (0 whale movements)
- `total_alerts` must equal `alerts.length`
- Severity must be one of: medium, high, critical
- price_change_percent must match calculation

### docs/index.html
**Owner**: Report Generator Agent  
**Readers**: Deployment Manager, Web Browsers  
**Update Frequency**: Every 15 minutes

**Requirements**:
- Valid HTML5 document
- Includes Chart.js CDN
- Responsive design (mobile/tablet/desktop)
- Auto-refresh meta tag (900 seconds)
- Links to `data/crypto-data.json` and `data/whale-alerts.json`

### Logs
Each agent writes to its own log file:
- `logs/data-fetcher.log`
- `logs/market-analyst.log`
- `logs/report-generator.log`
- `logs/deployment-manager.log`

**Format**:
```
[2026-02-11T01:00:00.000Z] INFO: Operation description
[2026-02-11T01:00:05.000Z] ERROR: Error details
```

## Error Handling Protocol

### Agent Failure Behavior

#### Data Fetcher Fails
**Symptoms**: API timeout, rate limit, invalid response

**Actions**:
1. Retry with exponential backoff (3 attempts)
2. If all retries fail:
   - Check if `crypto-data-previous.json` exists
   - If yes: Copy to `crypto-data.json` (use stale data)
   - If no: Exit with error, alert user
3. Log error details
4. **Do NOT proceed** to next agent if data is too old (>1 hour)

#### Market Analyst Fails
**Symptoms**: Calculation error, missing data

**Actions**:
1. Log error
2. Create empty whale-alerts.json:
   ```json
   {
     "timestamp": "2026-02-11T01:00:00Z",
     "alerts": [],
     "total_alerts": 0,
     "error": "Analysis failed, no alerts generated"
   }
   ```
3. **Proceed** to Report Generator (dashboard still valuable without alerts)

#### Report Generator Fails
**Symptoms**: HTML generation error, Chart.js issue

**Actions**:
1. Log error
2. Check if previous `docs/index.html` exists
3. If yes: Keep old dashboard (better than broken page)
4. If no: Create minimal error page
5. **Proceed** to Deployment Manager (publish error page)

#### Deployment Manager Fails
**Symptoms**: Git push failure, GitHub Pages error

**Actions**:
1. Log error
2. Retry push (3 attempts)
3. If push succeeds but GitHub Pages fails:
   - Check repository settings
   - Wait and verify again (Pages can be slow)
4. If all fails: Alert user via log, do not block next run

### Error Notification
```bash
# Function to log errors
log_error() {
  local agent_name="$1"
  local error_msg="$2"
  echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] ERROR [$agent_name]: $error_msg" | tee -a logs/errors.log
}

# Usage
log_error "data-fetcher" "API request failed after 3 retries"
```

## State Management

### First Run (No Previous Data)
**Scenario**: First time running the pipeline

**Expected Behavior**:
1. Data Fetcher: Creates `crypto-data.json`, no previous file
2. Market Analyst: Detects first run, creates empty `whale-alerts.json`
3. Report Generator: Generates dashboard, shows "No alerts yet"
4. Deployment Manager: Deploys initial dashboard

### Subsequent Runs
**Scenario**: Normal operation (every 15 minutes)

**Expected Behavior**:
1. Data Fetcher: Backs up current → previous, fetches new
2. Market Analyst: Compares current vs previous, detects whales
3. Report Generator: Updates dashboard with new data and alerts
4. Deployment Manager: Commits and pushes changes

### Recovery from Crash
**Scenario**: Pipeline crashed mid-execution

**Expected Behavior**:
- Each agent checks for required input files
- If missing, logs warning and attempts recovery
- Uses cached/previous data if available
- Continues pipeline if safe to do so

## Synchronization

### File Locking (Optional)
For concurrent access prevention:
```bash
# Acquire lock before writing
lockfile="data/.lock"
exec 200>"$lockfile"
flock -n 200 || { echo "Another process is running"; exit 1; }

# Write data
echo "$data" > data/crypto-data.json

# Release lock
flock -u 200
```

### Atomic Writes
Prevent partial file reads:
```bash
# Write to temporary file first
jq '.' data-new.json > data/crypto-data.json.tmp

# Atomic move (on same filesystem)
mv data/crypto-data.json.tmp data/crypto-data.json
```

## Monitoring & Observability

### Health Checks
Each agent should provide status:
```bash
# Check if data is fresh
data_age=$(( $(date +%s) - $(stat -f %m data/crypto-data.json) ))
if [ $data_age -gt 1800 ]; then
  echo "⚠️  Warning: Data is $data_age seconds old"
fi

# Check alert count
alert_count=$(jq '.total_alerts' data/whale-alerts.json)
echo "ℹ️  Current alerts: $alert_count"

# Check deployment status
if curl -sf https://etlee.github.io/stock-website > /dev/null; then
  echo "✓ Dashboard is live"
else
  echo "✗ Dashboard is down"
fi
```

### Metrics to Track
- **Data Fetcher**: API response time, retry count, data freshness
- **Market Analyst**: Processing time, alert count, severity distribution
- **Report Generator**: HTML size, generation time, chart count
- **Deployment Manager**: Push time, Pages build time, verification status

### Logging Best Practices
```bash
# Structured logging format
log() {
  local level="$1"  # INFO, WARN, ERROR
  local agent="$2"
  local message="$3"
  echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] $level [$agent]: $message" | tee -a logs/$agent.log
}

# Usage
log "INFO" "data-fetcher" "Starting data fetch from CoinGecko"
log "WARN" "market-analyst" "No previous data found, skipping comparison"
log "ERROR" "deployment-manager" "Git push failed: authentication error"
```

## Agent Coordination Patterns

### Sequential Execution (Current)
```bash
# Run agents one by one
./agents/data-fetcher.sh && \
./agents/market-analyst.sh && \
./agents/report-generator.sh && \
./agents/deployment-manager.sh
```

### Event-Driven (Future)
```bash
# Agents trigger next agent on success
on_success() {
  local next_agent="$1"
  echo "✓ Success, triggering $next_agent"
  ./$next_agent
}

# In data-fetcher.sh
if fetch_data_success; then
  on_success "agents/market-analyst.sh"
fi
```

### Message Queue (Advanced Future)
```
Data Fetcher → Queue → Market Analyst
                ↓
          Report Generator
                ↓
         Deployment Manager
```

## Testing Agent Interactions

### Unit Test (Individual Agent)
```bash
# Test data-fetcher.agent.md in isolation
echo "Testing data-fetcher agent..."
rm -f data/crypto-data.json  # Clean state
./agents/data-fetcher.sh
if [ -f data/crypto-data.json ]; then
  echo "✓ Data fetcher test passed"
else
  echo "✗ Data fetcher test failed"
fi
```

### Integration Test (Full Pipeline)
```bash
# Test complete pipeline
echo "Testing full pipeline..."
./run-pipeline.sh
if curl -sf https://etlee.github.io/stock-website | grep -q "Crypto Watchtower"; then
  echo "✓ Pipeline test passed"
else
  echo "✗ Pipeline test failed"
fi
```

### Mock Data Testing
```bash
# Test with mock data
cp test/fixtures/crypto-data-mock.json data/crypto-data.json
cp test/fixtures/crypto-data-previous-mock.json data/crypto-data-previous.json
./agents/market-analyst.sh
# Verify whale-alerts.json matches expected output
```

## Troubleshooting Guide

### Pipeline Stuck
**Symptoms**: Agents not completing

**Diagnosis**:
```bash
# Check for lock files
ls -la data/.lock

# Check agent processes
ps aux | grep -E "data-fetcher|market-analyst"

# Check logs for errors
tail -50 logs/*.log
```

### Data Inconsistency
**Symptoms**: Dashboard shows wrong data

**Diagnosis**:
```bash
# Validate JSON files
jq '.' data/crypto-data.json
jq '.' data/whale-alerts.json

# Check timestamps
jq '.timestamp' data/crypto-data.json
jq '.timestamp' data/whale-alerts.json

# Compare data freshness
stat data/crypto-data.json
```

### Deployment Not Updating
**Symptoms**: GitHub Pages shows old data

**Diagnosis**:
```bash
# Check git status
git status
git log -1

# Verify GitHub Pages build
curl -I https://etlee.github.io/stock-website

# Check timestamp in live data
curl https://etlee.github.io/stock-website/data/crypto-data.json | jq '.timestamp'
```

## Future Enhancements

### Agent Registry
Track all agents and their status:
```json
{
  "agents": {
    "data-fetcher": {
      "status": "running",
      "last_run": "2026-02-11T01:00:00Z",
      "next_run": "2026-02-11T01:15:00Z"
    }
  }
}
```

### Agent Dependencies
Declare dependencies explicitly:
```yaml
# In agent front matter
dependencies:
  - data-fetcher  # Must run before this agent
outputs:
  - whale-alerts.json
```

### Webhooks
Notify external systems on events:
```bash
# Send webhook on critical alert
if [ "$severity" = "critical" ]; then
  curl -X POST "$WEBHOOK_URL" \
    -d '{"alert":"Critical whale movement detected"}'
fi
```
