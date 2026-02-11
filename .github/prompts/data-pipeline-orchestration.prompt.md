# Data Pipeline Orchestration Prompt

## Purpose
This prompt coordinates the execution of all agents in the crypto watchtower data pipeline.

## Usage
Use this prompt with GitHub Copilot CLI to orchestrate the complete data collection, analysis, visualization, and deployment workflow.

## Prompt Template

```
Execute the 24/7 Crypto Watchtower data pipeline:

1. **Data Fetcher Agent** (@.github/agents/data-fetcher.agent.md)
   - Fetch top 20 cryptocurrencies from CoinGecko API
   - Save to data/crypto-data.json
   - Backup previous data

2. **Market Analyst Agent** (@.github/agents/market-analyst.agent.md)
   - Analyze price movements vs previous snapshot
   - Detect whale movements (>5% change in 15 min)
   - Generate whale-alerts.json

3. **Report Generator Agent** (@.github/agents/report-generator.agent.md)
   - Create Chart.js dashboard
   - Generate docs/index.html
   - Copy data files to docs/data/

4. **Deployment Manager Agent** (@.github/agents/deployment-manager.agent.md)
   - Commit changes to Git
   - Push to GitHub
   - Verify deployment at https://etlee.github.io/stock-website

Execute agents in sequence. Report status after each agent completes.
```

## Expected Flow

### Stage 1: Data Collection (2-5 seconds)
```
Running: data-fetcher.agent.md
→ Fetching data from CoinGecko...
→ ✓ Retrieved 20 cryptocurrencies
→ ✓ Saved to data/crypto-data.json
→ ✓ Backed up previous data
```

### Stage 2: Analysis (1-2 seconds)
```
Running: market-analyst.agent.md
→ Comparing current vs previous data...
→ ✓ Analyzed 20 cryptocurrencies
→ ✓ Detected X whale movements
→ ✓ Saved to data/whale-alerts.json
```

### Stage 3: Visualization (2-3 seconds)
```
Running: report-generator.agent.md
→ Generating Chart.js dashboard...
→ ✓ Created price chart
→ ✓ Created volume chart
→ ✓ Created market cap chart
→ ✓ Generated docs/index.html
```

### Stage 4: Deployment (5-10 seconds)
```
Running: deployment-manager.agent.md
→ Validating files...
→ Committing changes...
→ Pushing to GitHub...
→ ✓ Deployed to https://etlee.github.io/stock-website
→ ✓ Verified HTTP 200
```

## Error Handling
If any agent fails:
1. Log error details
2. Check logs in `logs/*.log`
3. Retry failed agent (max 3 attempts)
4. If still failing, alert user and use cached data

## Scheduling
For automated execution:
```bash
# Using cron (Linux/macOS)
*/15 * * * * cd /path/to/Stock-Website && gh copilot run "@.github/prompts/data-pipeline-orchestration.prompt.md"

# Using GitHub Actions (see .github/workflows/crypto-watchtower.yml)
```

## Manual Trigger
```bash
gh copilot run "@.github/prompts/data-pipeline-orchestration.prompt.md"
```

## Success Indicators
- ✓ All 4 agents complete successfully
- ✓ Dashboard updated at etlee.github.io/stock-website
- ✓ Data timestamp is current (within 1 minute)
- ✓ No errors in logs
- ✓ Total execution time < 20 seconds

## Monitoring
Check pipeline health:
```bash
# View latest data timestamp
jq '.timestamp' data/crypto-data.json

# Check whale alerts
jq '.total_alerts' data/whale-alerts.json

# View deployment history
tail logs/deployment-manager.log

# Test live site
curl -I https://etlee.github.io/stock-website
```

## Optimization Tips
- Run during off-peak hours for better API reliability
- Cache previous responses for faster fallback
- Use CDN for Chart.js to minimize loading time
- Monitor CoinGecko API rate limits

## Dependencies
- `curl` - HTTP requests
- `jq` - JSON processing
- `git` - Version control
- `python3` or `node` - Data processing (optional, can use jq)

## Agent Communication
Agents communicate via files:
```
data-fetcher → crypto-data.json → market-analyst
market-analyst → whale-alerts.json → report-generator
report-generator → docs/ → deployment-manager
deployment-manager → GitHub Pages → 🌐 Live!
```
