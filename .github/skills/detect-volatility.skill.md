# Detect Volatility Skill

## Purpose
Analyze cryptocurrency price movements to identify "whale" activity (high volatility events).

## Capabilities
- Calculate percentage price changes over time intervals
- Detect significant price movements (>5% in 15 minutes)
- Compare current vs previous data snapshots
- Flag whale movements with severity levels
- Generate alerts with context

## Detection Criteria
**Whale Movement Threshold**: Price change > 5% in 15 minutes

Severity Levels:
- 🟡 **Medium** (5-10% change): Significant movement
- 🟠 **High** (10-20% change): Major whale activity
- 🔴 **Critical** (>20% change): Extreme volatility

## Input
Reads two data files:
- `data/crypto-data.json` (current)
- `data/crypto-data-previous.json` (15 minutes ago)

## Output Format
Save alerts to `data/whale-alerts.json`:
```json
{
  "timestamp": "2026-02-11T01:00:00.000Z",
  "alerts": [
    {
      "crypto_id": "bitcoin",
      "symbol": "btc",
      "name": "Bitcoin",
      "price_change_percent": 7.5,
      "previous_price": 45000.50,
      "current_price": 48375.54,
      "severity": "high",
      "time_interval": "15min",
      "alert_message": "Bitcoin surged 7.5% in 15 minutes"
    }
  ],
  "total_alerts": 1
}
```

## Algorithm
1. Load current and previous data snapshots
2. For each cryptocurrency:
   - Calculate: `((current_price - previous_price) / previous_price) * 100`
   - If abs(change) > 5%, create alert
3. Sort alerts by severity (critical > high > medium)
4. Save to whale-alerts.json

## Edge Cases
- First run (no previous data): Skip detection, create baseline
- Missing cryptocurrency in previous data: Log warning, skip
- Invalid price data: Skip that cryptocurrency

## Performance
- Process 20 cryptocurrencies in <100ms
- Keep last 10 snapshots for trend analysis (optional future feature)
