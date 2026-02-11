# Whale Alert Format Prompt

## Purpose
Defines the standard format and messaging for whale movement alerts.

## Alert Criteria
**Whale Movement**: Price change > 5% in 15-minute interval

**Severity Classification**:
- 🟡 **Medium** (5-10%): Significant movement worth noting
- 🟠 **High** (10-20%): Major whale activity requiring attention
- 🔴 **Critical** (>20%): Extreme volatility, potential market event

## Message Format

### Standard Alert Message
```
{emoji} {crypto_name} {direction} {percentage}% in 15 minutes
```

**Examples**:
- `🟡 Bitcoin surged 7.5% in 15 minutes`
- `🟠 Ethereum dropped 12.3% in 15 minutes`
- `🔴 Cardano surged 25.8% in 15 minutes`

### Direction Verbs
- **Positive change**: "surged", "spiked", "jumped"
- **Negative change**: "dropped", "plunged", "fell"

### Full Alert Object (JSON)
```json
{
  "crypto_id": "bitcoin",
  "symbol": "BTC",
  "name": "Bitcoin",
  "price_change_percent": 7.5,
  "previous_price": 45000.50,
  "current_price": 48375.54,
  "severity": "high",
  "emoji": "🟠",
  "time_interval": "15min",
  "alert_message": "Bitcoin surged 7.5% in 15 minutes",
  "timestamp": "2026-02-11T01:00:00.000Z"
}
```

## Dashboard Display

### Table Format
| Crypto | Change | Previous | Current | Severity |
|--------|--------|----------|---------|----------|
| Bitcoin (BTC) | +7.5% | $45,000.50 | $48,375.54 | 🟠 HIGH |

### Color Coding
- **Background Color by Severity**:
  - Medium: `rgba(255, 193, 7, 0.2)` - Yellow tint
  - High: `rgba(255, 152, 0, 0.2)` - Orange tint
  - Critical: `rgba(220, 53, 69, 0.2)` - Red tint

- **Text Color by Direction**:
  - Positive: `#4caf50` - Green
  - Negative: `#f44336` - Red

### Notification Banner (Optional Future Feature)
```html
<div class="whale-banner severity-{level}">
  <span class="emoji">{emoji}</span>
  <strong>{crypto_name}</strong> {direction} <strong>{percentage}%</strong> in 15 minutes
  <span class="price">Now at ${current_price}</span>
</div>
```

## Sorting Priority
1. **Severity** (Critical > High > Medium)
2. **Absolute Change** (Higher % first)
3. **Market Cap** (Larger coins prioritized if tied)

## Alert Threshold Configuration
```json
{
  "thresholds": {
    "medium": 5.0,
    "high": 10.0,
    "critical": 20.0
  },
  "time_interval": "15min",
  "max_alerts_display": 10
}
```

## Use Cases

### No Alerts
```
🐋 Whale Alerts
No significant movements detected in the last 15 minutes.
Market conditions are relatively stable.
```

### Single Alert
```
🐋 Whale Alert (1)
🟡 Solana surged 6.2% in 15 minutes
```

### Multiple Alerts
```
🐋 Whale Alerts (3)
🔴 Dogecoin plunged 22.1% in 15 minutes
🟠 Ripple dropped 11.5% in 15 minutes
🟡 Polkadot surged 5.8% in 15 minutes
```

## Context Enrichment (Optional)
For enhanced alerts, include:
- **Volume Change**: "with 3x normal volume"
- **Market Context**: "following Bitcoin's movement"
- **Time Context**: "after market opening"

**Example**:
```
🔴 Ethereum plunged 21.5% in 15 minutes with 5x normal volume
```

## Notification Channels (Future)
- **Dashboard**: Primary display on website
- **Email**: Send alerts to subscribers (future)
- **Discord/Telegram**: Bot notifications (future)
- **SMS**: Critical alerts only (future)

## Alert History
Store last 24 hours of alerts:
```json
{
  "alert_history": [
    {
      "timestamp": "2026-02-11T01:00:00Z",
      "alerts": [...]
    },
    {
      "timestamp": "2026-02-11T00:45:00Z",
      "alerts": [...]
    }
  ]
}
```

## False Positive Handling
Ignore alerts if:
- Data quality issue (price spike to $0 or unreasonably high)
- API error or missing data
- First data point (no comparison available)

## Accessibility
- Use emoji as visual indicator, but include text severity
- Ensure color coding has sufficient contrast
- Provide text alternatives for screen readers

## Mobile Responsive
- Stack table columns on narrow screens
- Use abbreviations: "Prev" instead of "Previous"
- Hide less critical columns on mobile (e.g., crypto ID)

## Prompt for Agent
```
Generate whale alerts using this format:
1. Calculate price change: ((current - previous) / previous) * 100
2. Filter: Only changes > 5% (absolute value)
3. Classify severity: >20% critical, >10% high, >5% medium
4. Create message: "{emoji} {name} {direction} {percent}% in 15 minutes"
5. Sort by: severity desc, then abs(change) desc
6. Output to: data/whale-alerts.json
```
