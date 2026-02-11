# Fetch Crypto Data Skill

## Purpose
Fetch real-time cryptocurrency market data from CoinGecko API for the top cryptocurrencies by market cap.

## Capabilities
- Query CoinGecko API `/coins/markets` endpoint
- Retrieve top 20 cryptocurrencies by market cap
- Extract: price, 24h change, volume, market cap, last updated timestamp
- Handle API rate limiting with exponential backoff
- Parse and structure data for downstream processing

## Usage
This skill can be invoked by any agent that needs cryptocurrency market data.

## API Endpoint
```
GET https://api.coingecko.com/api/v3/coins/markets
Parameters:
  - vs_currency: usd
  - order: market_cap_desc
  - per_page: 20
  - page: 1
  - sparkline: false
  - price_change_percentage: 1h,24h
```

## Output Format
Save data to `data/crypto-data.json`:
```json
{
  "timestamp": "2026-02-11T01:00:00.000Z",
  "data": [
    {
      "id": "bitcoin",
      "symbol": "btc",
      "name": "Bitcoin",
      "current_price": 45000.50,
      "price_change_percentage_1h": 0.5,
      "price_change_percentage_24h": 2.3,
      "total_volume": 25000000000,
      "market_cap": 850000000000,
      "last_updated": "2026-02-11T01:00:00.000Z"
    }
  ]
}
```

## Error Handling
- Retry on rate limit (429) with exponential backoff: 2s, 4s, 8s
- Log errors to `logs/fetch-errors.log`
- Return cached data if API fails after retries
- Timeout: 10 seconds per request

## Implementation Notes
- No API key required for CoinGecko free tier
- Cache previous response for fallback
- Include User-Agent header for better rate limits
