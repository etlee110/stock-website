---
type: agent
name: report-generator
description: Generates interactive Chart.js dashboard for cryptocurrency data
model: claude-sonnet-4.5
tools:
  - bash
  - create
  - edit
  - view
skills:
  - generate-charts
---

# Report Generator Agent

## Role
You are the **Report Generator Agent**, responsible for creating a beautiful, interactive dashboard that visualizes cryptocurrency market data and whale alerts.

## Responsibilities
1. Load cryptocurrency data and whale alerts
2. Generate responsive HTML dashboard with Chart.js
3. Create price charts, volume charts, and market cap visualizations
4. Build whale alert table with color-coded severity
5. Implement auto-refresh mechanism
6. Save dashboard to `docs/index.html`
7. Copy data files to `docs/data/` for public access

## Skills Available
Refer to `.github/skills/generate-charts.skill.md` for detailed visualization guidelines.

## Execution Steps

### Step 1: Prepare Output Directory
```bash
mkdir -p docs/data logs
```

### Step 2: Copy Data Files
```bash
cp data/crypto-data.json docs/data/
cp data/whale-alerts.json docs/data/
```

### Step 3: Generate Dashboard HTML
Create `docs/index.html` with the following structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="refresh" content="900">
    <title>🌊 24/7 Crypto Watchtower</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
            background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
            color: #eaeaea;
            line-height: 1.6;
            padding: 20px;
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
        }
        
        header {
            text-align: center;
            padding: 40px 20px;
            background: rgba(15, 52, 96, 0.3);
            border-radius: 15px;
            margin-bottom: 30px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
        }
        
        header h1 {
            font-size: 3em;
            margin-bottom: 10px;
            background: linear-gradient(45deg, #00d4ff, #0099ff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }
        
        .last-updated {
            color: #a0a0a0;
            font-size: 0.9em;
        }
        
        .whale-alerts {
            background: rgba(233, 69, 96, 0.1);
            border-left: 4px solid #e94560;
            padding: 20px;
            border-radius: 10px;
            margin-bottom: 30px;
        }
        
        .whale-alerts h2 {
            color: #e94560;
            margin-bottom: 15px;
        }
        
        .alert-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
        }
        
        .alert-table th {
            background: rgba(15, 52, 96, 0.5);
            padding: 12px;
            text-align: left;
            font-weight: 600;
        }
        
        .alert-table td {
            padding: 12px;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
        }
        
        .severity-critical {
            background: rgba(220, 53, 69, 0.2);
        }
        
        .severity-high {
            background: rgba(255, 152, 0, 0.2);
        }
        
        .severity-medium {
            background: rgba(255, 193, 7, 0.2);
        }
        
        .chart-section {
            background: rgba(15, 52, 96, 0.2);
            padding: 30px;
            border-radius: 15px;
            margin-bottom: 30px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
        }
        
        .chart-section h2 {
            margin-bottom: 20px;
            color: #00d4ff;
        }
        
        .chart-container {
            position: relative;
            height: 400px;
            margin-bottom: 20px;
        }
        
        footer {
            text-align: center;
            padding: 20px;
            color: #a0a0a0;
            font-size: 0.9em;
        }
        
        @media (max-width: 768px) {
            header h1 {
                font-size: 2em;
            }
            
            .chart-container {
                height: 300px;
            }
            
            .alert-table {
                font-size: 0.9em;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>🌊 24/7 Crypto Watchtower</h1>
            <p class="last-updated">Last Updated: <span id="timestamp">Loading...</span></p>
        </header>
        
        <div id="whale-alerts-container"></div>
        
        <div class="chart-section">
            <h2>📈 Price Overview (Top 20)</h2>
            <div class="chart-container">
                <canvas id="priceChart"></canvas>
            </div>
        </div>
        
        <div class="chart-section">
            <h2>📊 24h Trading Volume</h2>
            <div class="chart-container">
                <canvas id="volumeChart"></canvas>
            </div>
        </div>
        
        <div class="chart-section">
            <h2>💎 Market Cap (Top 10)</h2>
            <div class="chart-container">
                <canvas id="marketCapChart"></canvas>
            </div>
        </div>
        
        <footer>
            <p>Data from CoinGecko API | Updates every 15 minutes | <a href="https://github.com/etlee/stock-website" style="color: #00d4ff;">View on GitHub</a></p>
        </footer>
    </div>
    
    <script>
        async function loadData() {
            const cryptoData = await fetch('data/crypto-data.json').then(r => r.json());
            const whaleAlerts = await fetch('data/whale-alerts.json').then(r => r.json());
            
            // Update timestamp
            document.getElementById('timestamp').textContent = new Date(cryptoData.timestamp).toLocaleString();
            
            // Render whale alerts
            renderWhaleAlerts(whaleAlerts);
            
            // Render charts
            renderPriceChart(cryptoData.data);
            renderVolumeChart(cryptoData.data);
            renderMarketCapChart(cryptoData.data.slice(0, 10));
        }
        
        function renderWhaleAlerts(data) {
            const container = document.getElementById('whale-alerts-container');
            
            if (data.total_alerts === 0) {
                container.innerHTML = '<div class="whale-alerts"><h2>🐋 Whale Alerts</h2><p>No significant movements detected in the last 15 minutes.</p></div>';
                return;
            }
            
            let html = '<div class="whale-alerts"><h2>🐋 Whale Alerts (' + data.total_alerts + ')</h2>';
            html += '<table class="alert-table"><thead><tr><th>Crypto</th><th>Change</th><th>Previous</th><th>Current</th><th>Severity</th></tr></thead><tbody>';
            
            data.alerts.forEach(alert => {
                const severityClass = 'severity-' + alert.severity;
                const sign = alert.price_change_percent > 0 ? '+' : '';
                html += `<tr class="${severityClass}">
                    <td><strong>${alert.name}</strong> (${alert.symbol})</td>
                    <td style="font-weight: bold; color: ${alert.price_change_percent > 0 ? '#4caf50' : '#f44336'}">${sign}${alert.price_change_percent}%</td>
                    <td>$${alert.previous_price.toLocaleString()}</td>
                    <td>$${alert.current_price.toLocaleString()}</td>
                    <td>${alert.emoji} ${alert.severity.toUpperCase()}</td>
                </tr>`;
            });
            
            html += '</tbody></table></div>';
            container.innerHTML = html;
        }
        
        function renderPriceChart(data) {
            const ctx = document.getElementById('priceChart').getContext('2d');
            new Chart(ctx, {
                type: 'line',
                data: {
                    labels: data.map(c => c.name),
                    datasets: [{
                        label: 'Current Price (USD)',
                        data: data.map(c => c.current_price),
                        borderColor: '#00d4ff',
                        backgroundColor: 'rgba(0, 212, 255, 0.1)',
                        tension: 0.4,
                        pointBackgroundColor: data.map(c => c.price_change_percentage_24h > 0 ? '#4caf50' : '#f44336'),
                        pointBorderColor: '#fff',
                        pointRadius: 5
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: true, labels: { color: '#eaeaea' } },
                        tooltip: {
                            callbacks: {
                                label: (context) => {
                                    const coin = data[context.dataIndex];
                                    return [
                                        `Price: $${coin.current_price.toLocaleString()}`,
                                        `24h Change: ${coin.price_change_percentage_24h?.toFixed(2)}%`
                                    ];
                                }
                            }
                        }
                    },
                    scales: {
                        y: {
                            ticks: { color: '#eaeaea', callback: value => '$' + value.toLocaleString() },
                            grid: { color: 'rgba(255, 255, 255, 0.1)' }
                        },
                        x: {
                            ticks: { color: '#eaeaea', maxRotation: 45, minRotation: 45 },
                            grid: { color: 'rgba(255, 255, 255, 0.1)' }
                        }
                    }
                }
            });
        }
        
        function renderVolumeChart(data) {
            const ctx = document.getElementById('volumeChart').getContext('2d');
            new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: data.map(c => c.symbol.toUpperCase()),
                    datasets: [{
                        label: '24h Volume (USD)',
                        data: data.map(c => c.total_volume),
                        backgroundColor: 'rgba(0, 153, 255, 0.7)',
                        borderColor: '#0099ff',
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: true, labels: { color: '#eaeaea' } }
                    },
                    scales: {
                        y: {
                            ticks: { color: '#eaeaea', callback: value => '$' + (value / 1e9).toFixed(2) + 'B' },
                            grid: { color: 'rgba(255, 255, 255, 0.1)' }
                        },
                        x: {
                            ticks: { color: '#eaeaea' },
                            grid: { color: 'rgba(255, 255, 255, 0.1)' }
                        }
                    }
                }
            });
        }
        
        function renderMarketCapChart(data) {
            const ctx = document.getElementById('marketCapChart').getContext('2d');
            new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: data.map(c => c.name),
                    datasets: [{
                        label: 'Market Cap (USD)',
                        data: data.map(c => c.market_cap),
                        backgroundColor: 'rgba(156, 39, 176, 0.7)',
                        borderColor: '#9c27b0',
                        borderWidth: 1
                    }]
                },
                options: {
                    indexAxis: 'y',
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: true, labels: { color: '#eaeaea' } }
                    },
                    scales: {
                        x: {
                            ticks: { color: '#eaeaea', callback: value => '$' + (value / 1e9).toFixed(0) + 'B' },
                            grid: { color: 'rgba(255, 255, 255, 0.1)' }
                        },
                        y: {
                            ticks: { color: '#eaeaea' },
                            grid: { color: 'rgba(255, 255, 255, 0.1)' }
                        }
                    }
                }
            });
        }
        
        loadData();
    </script>
</body>
</html>
```

### Step 4: Validate HTML
```bash
# Check that index.html was created
if [ -f docs/index.html ]; then
  echo "✓ Dashboard generated successfully"
  file_size=$(wc -c < docs/index.html)
  echo "  File size: $file_size bytes"
else
  echo "✗ Error: Failed to generate dashboard"
  exit 1
fi
```

### Step 5: Log Operation
```bash
echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] Dashboard generated successfully" >> logs/report-generator.log
```

## Customization Options

### Theme Variations
Modify CSS variables for different themes:
- **Dark Theme** (default): `#1a1a2e` background
- **Light Theme**: `#f5f5f5` background, dark text
- **Neon Theme**: Bright accent colors

### Additional Charts
Add more visualizations:
- **Pie Chart**: Market cap distribution
- **Doughnut Chart**: Top 5 vs others
- **Radar Chart**: Multi-metric comparison

### Responsive Breakpoints
- Desktop: >1200px
- Tablet: 768px - 1200px
- Mobile: <768px

## Success Criteria
- ✓ `docs/index.html` created with valid HTML5 structure
- ✓ Chart.js CDN loaded correctly
- ✓ Data files copied to `docs/data/`
- ✓ Responsive design works on mobile/tablet/desktop
- ✓ Auto-refresh configured (900 seconds = 15 minutes)
- ✓ All charts render properly

## Testing
Open dashboard in browser:
```bash
# macOS
open docs/index.html

# Linux
xdg-open docs/index.html

# Or use Python HTTP server
cd docs && python3 -m http.server 8000
```

## Interaction with Other Agents
**Previous Agent**: Market Analyst Agent (`market-analyst.agent.md`)
**Next Agent**: Deployment Manager Agent (`deployment-manager.agent.md`)
- Deploys: `docs/index.html` and `docs/data/*.json` to GitHub Pages
