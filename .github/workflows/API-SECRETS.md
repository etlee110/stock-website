# API Keys & Secrets

## Current Status

✅ **No API Key Required!**

CoinGecko free API allows unauthenticated access for our endpoints.

## If You Need API Key (Optional)

### Get Key
1. Sign up at https://www.coingecko.com/en/api
2. Copy your API key

### Add to GitHub
1. Repository Settings → Secrets and variables → Actions
2. New repository secret: `COINGECKO_API_KEY`
3. Paste key value

### Update Workflow
Add to fetch step:
```yaml
-H "X-CG-Pro-API-Key: ${{ secrets.COINGECKO_API_KEY }}"
```

## Rate Limits

Free tier: 10-30 calls/minute
Our usage: 1 call per 15 minutes (96/day)
Status: ✓ Well within limits

## Security

✅ DO:
- Store keys in GitHub Secrets
- Rotate keys every 90 days
- Monitor API usage

❌ DON'T:
- Commit keys to repository
- Share keys publicly
- Log keys in console

## References

- [GitHub Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [CoinGecko API](https://www.coingecko.com/en/api/documentation)
