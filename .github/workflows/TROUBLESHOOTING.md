# Troubleshooting Guide

## Common Issues and Solutions

### 1. GitHub Actions Workflow Failures

#### Error: "Permission denied"
**Symptom**: Workflow fails at git push step

**Solution**:
```
1. Go to Repository Settings
2. Actions → General → Workflow permissions
3. Select "Read and write permissions"
4. Check "Allow GitHub Actions to create and approve pull requests"
5. Click Save
```

#### Error: "Resource not accessible by integration"
**Symptom**: Cannot commit or push

**Solution**:
```yaml
# Ensure workflow has correct permissions
permissions:
  contents: write  # Must be present in workflow file
```

### 2. GitHub Pages Issues

#### 404 Not Found
**Symptoms**: Dashboard shows 404 error

**Checklist**:
- [ ] GitHub Pages is enabled (Settings → Pages)
- [ ] Source is set to: main branch, /docs folder
- [ ] `docs/index.html` exists in repository
- [ ] Wait 2-3 minutes for initial deployment
- [ ] Check Actions tab for deployment status

**Fix**:
```bash
# Verify files exist
ls -la docs/
# Should show: index.html, data/crypto-data.json, data/whale-alerts.json
```

#### Old Content Showing
**Symptoms**: Dashboard not updating with new data

**Solutions**:
1. **Clear Browser Cache**:
   - Chrome/Firefox: `Ctrl + Shift + R`
   - Safari: `Cmd + Shift + R`

2. **Check Timestamp**:
   ```bash
   curl https://etlee.github.io/stock-website/data/crypto-data.json | jq '.timestamp'
   ```

3. **Force Rebuild**:
   - Go to Actions tab
   - Re-run latest workflow
   - Wait 1-2 minutes for Pages to rebuild

#### Jekyll Build Errors
**Symptoms**: Pages won't build, sees Jekyll errors

**Solution**:
```bash
# Add .nojekyll file to bypass Jekyll processing
touch docs/.nojekyll
git add docs/.nojekyll
git commit -m "Disable Jekyll processing"
git push
```

### 3. API Issues

#### Rate Limiting (429 Errors)
**Symptoms**: "Too Many Requests" in logs

**Solutions**:
1. **Reduce Frequency**:
   ```yaml
   # Change from */15 to */30 (every 30 minutes)
   - cron: '*/30 * * * *'
   ```

2. **Add Retry Logic** (already implemented):
   ```bash
   --retry 3 --retry-delay 2
   ```

3. **Get API Key** (optional):
   - Sign up at https://www.coingecko.com/en/api
   - Add key to GitHub Secrets
   - See `.github/workflows/API-SECRETS.md`

#### API Timeout
**Symptoms**: Workflow fails with "timeout exceeded"

**Solutions**:
```bash
# Increase timeout (already set to 10s)
--max-time 15  # Increase to 15 seconds

# Or add retries
--retry 5 --retry-max-time 30
```

### 4. Data Issues

#### No Previous Data
**Symptoms**: Whale detection skipped, "First run" message

**Expected**: Normal on first execution
**Action**: Wait for next run (15 minutes)

#### Invalid JSON
**Symptoms**: Workflow fails at jq parsing step

**Diagnosis**:
```bash
# Validate JSON manually
jq '.' data/crypto-data.json

# Check for syntax errors
python3 -m json.tool data/crypto-data.json
```

**Fix**:
```bash
# Re-fetch data
rm data/crypto-data.json
# Trigger workflow again
```

#### Missing Cryptocurrencies
**Symptoms**: Expected 20 coins, got fewer

**Causes**:
- API returned less data
- Network error during fetch
- Rate limiting

**Fix**:
```bash
# Check actual count
jq '.data | length' data/crypto-data.json

# Verify API endpoint
curl "https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=20" | jq 'length'
```

### 5. Dashboard Display Issues

#### Charts Not Rendering
**Symptoms**: Blank dashboard, charts don't show

**Checklist**:
- [ ] Chart.js CDN loaded (check browser console)
- [ ] Data files exist in `docs/data/`
- [ ] JavaScript not blocked by ad blocker
- [ ] Browser supports ES6 (Chrome 51+, Firefox 54+)

**Debug**:
```javascript
// Open browser console (F12)
// Check for errors
console.log('Data loaded:', await fetch('data/crypto-data.json').then(r => r.json()));
```

#### Mobile Layout Broken
**Symptoms**: Dashboard looks bad on mobile

**Solutions**:
1. **Viewport Tag** (already included):
   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   ```

2. **Test Responsive**:
   - Chrome DevTools: F12 → Toggle device toolbar
   - Test on: iPhone, iPad, Android

3. **Adjust Breakpoints**:
   ```css
   @media (max-width: 768px) {
     /* Mobile styles */
   }
   ```

#### Whale Alerts Not Showing
**Symptoms**: "No alerts" message but prices changed

**Check**:
1. **Threshold**: Are changes > 5%?
   ```bash
   # View detected alerts
   jq '.alerts' data/whale-alerts.json
   ```

2. **Time Interval**: 15 minutes elapsed?
   ```bash
   # Check timestamp difference
   jq '.timestamp' data/crypto-data.json
   jq '.timestamp' data/crypto-data-previous.json
   ```

3. **Calculation**: Verify manually
   ```python
   change = ((current - previous) / previous) * 100
   print(f"Change: {change}%")  # Should be > 5
   ```

### 6. Git Issues

#### Merge Conflicts
**Symptoms**: Push fails with "non-fast-forward"

**Solution**:
```bash
# Workflow should never have conflicts (bot is only committer)
# If it happens, reset to remote:
git fetch origin main
git reset --hard origin/main
git push --force-with-lease origin main
```

#### Large Repository Size
**Symptoms**: Push fails, repository >1 GB

**Solutions**:
1. **Remove Logs** (if committed):
   ```bash
   git rm -r logs/
   echo "logs/" >> .gitignore
   git commit -m "Remove logs from tracking"
   ```

2. **Clear History**:
   ```bash
   # Use git filter-repo to remove large files
   # Or create new repository
   ```

3. **Use LFS** (for large data files):
   ```bash
   git lfs track "data/*.json"
   git add .gitattributes
   ```

### 7. Performance Issues

#### Slow Dashboard Load
**Symptoms**: Dashboard takes >3 seconds to load

**Optimizations**:
1. **Minify HTML/CSS/JS**:
   ```bash
   # Use minifier tools
   html-minifier docs/index.html -o docs/index.html
   ```

2. **Optimize Chart.js**:
   ```html
   <!-- Use minified CDN version -->
   <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
   ```

3. **Enable Caching**:
   ```html
   <!-- Add cache headers (GitHub Pages does this automatically) -->
   ```

#### High Actions Minutes Usage
**Symptoms**: Approaching GitHub Actions limits

**Solutions**:
1. **Reduce Frequency**:
   ```yaml
   - cron: '0 */2 * * *'  # Every 2 hours instead of 15 min
   ```

2. **Conditional Execution**:
   ```yaml
   - name: Check if update needed
     if: github.event_name == 'schedule'  # Only on schedule, not push
   ```

### 8. Debugging Tips

#### Enable Debug Logging
```yaml
# In workflow file
env:
  ACTIONS_STEP_DEBUG: true
  ACTIONS_RUNNER_DEBUG: true
```

#### View Workflow Logs
```bash
# Using GitHub CLI
gh run list
gh run view <run-id> --log

# Or in browser
# Repository → Actions → Select run → View logs
```

#### Test Components Individually
```bash
# Test data fetching
curl "https://api.coingecko.com/api/v3/coins/markets?..." | jq '.[0]'

# Test Python analysis locally
python3 test-analyst.py

# Test dashboard locally
cd docs && python3 -m http.server 8000
```

#### Check Data Integrity
```bash
# Validate all JSON files
for file in data/*.json docs/data/*.json; do
  echo "Checking $file"
  jq '.' "$file" > /dev/null && echo "✓" || echo "✗ Invalid JSON"
done
```

### 9. Quick Fixes

#### Reset Everything
```bash
# Nuclear option: start fresh
rm -rf data/ docs/ logs/
mkdir -p data docs/data logs

# Trigger workflow
gh workflow run crypto-watchtower.yml
```

#### Force Rebuild
```bash
# Commit empty change to trigger workflow
git commit --allow-empty -m "Force rebuild"
git push
```

#### Test Locally
```bash
# Run workflow steps manually
bash << 'EOF'
  # Step 1: Fetch
  curl "https://api.coingecko.com/api/v3/coins/markets?..." -o data/raw.json
  
  # Step 2: Process
  jq '{timestamp:(now|todate),data:[.[]|{id,symbol,name,current_price}]}' data/raw.json > data/crypto-data.json
  
  # Step 3: Analyze
  python3 analyze.py
  
  # Step 4: Generate
  # ... create index.html
EOF
```

## Getting Help

If issues persist:

1. **Check Status**:
   - GitHub Status: https://www.githubstatus.com/
   - CoinGecko Status: https://status.coingecko.com/

2. **Review Logs**:
   - Actions logs: Full execution details
   - Browser console: JavaScript errors
   - Network tab: Failed requests

3. **Community**:
   - GitHub Discussions: Ask questions
   - Stack Overflow: General programming help
   - CoinGecko Support: API issues

4. **File Issue**:
   - https://github.com/etlee/stock-website/issues
   - Include: error message, workflow run ID, steps to reproduce

## Preventive Measures

### Regular Maintenance
- Review logs weekly for warnings
- Update dependencies monthly
- Test dashboard on different devices
- Monitor API usage
- Rotate secrets every 90 days

### Monitoring
```bash
# Set up alerts (optional)
# Monitor workflow failures
# Check data freshness
# Verify deployment success
```

### Backup Strategy
```bash
# Backup data periodically
tar -czf backup-$(date +%Y%m%d).tar.gz data/ docs/

# Or use GitHub Actions
# Store in separate branch or external storage
```
