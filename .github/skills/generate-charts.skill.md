# Generate Charts Skill

## Purpose
Create interactive Chart.js visualizations for cryptocurrency market data and whale alerts.

## Capabilities
- Generate responsive HTML dashboard with Chart.js
- Create price trend charts (line charts)
- Create volume comparison charts (bar charts)
- Build whale alert table with color-coded severity
- Auto-refresh data display
- Mobile-responsive layout

## Chart Types

### 1. Price Trend Chart
- **Type**: Line chart
- **X-axis**: Cryptocurrency names
- **Y-axis**: Current price (USD)
- **Colors**: Green (positive 24h change), Red (negative 24h change)
- **Features**: Tooltips with detailed info, responsive scaling

### 2. Volume Chart
- **Type**: Bar chart
- **X-axis**: Cryptocurrency names
- **Y-axis**: 24h trading volume (USD)
- **Colors**: Blue gradient
- **Features**: Sorted by volume (descending)

### 3. Market Cap Overview
- **Type**: Horizontal bar chart
- **X-axis**: Market cap (USD billions)
- **Y-axis**: Top 10 cryptocurrencies
- **Colors**: Purple gradient

### 4. Whale Alert Table
- **Type**: HTML table
- **Columns**: Crypto, Price Change, Previous/Current Price, Severity, Time
- **Colors**: 
  - 🟡 Medium: Yellow background
  - 🟠 High: Orange background
  - 🔴 Critical: Red background

## Input Files
- `data/crypto-data.json` - Current market data
- `data/whale-alerts.json` - Detected whale movements

## Output
Generate `index.html` with embedded:
- Chart.js CDN (v4.x)
- Custom CSS styling
- JavaScript for chart rendering
- Auto-refresh meta tag (15 minutes)

## HTML Structure
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="refresh" content="900">
  <title>24/7 Crypto Watchtower</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4"></script>
  <style>/* Responsive CSS */</style>
</head>
<body>
  <header>
    <h1>🌊 24/7 Crypto Watchtower</h1>
    <p>Last Updated: <span id="timestamp"></span></p>
  </header>
  
  <section id="whale-alerts">
    <!-- Whale alert table -->
  </section>
  
  <section id="charts">
    <canvas id="priceChart"></canvas>
    <canvas id="volumeChart"></canvas>
    <canvas id="marketCapChart"></canvas>
  </section>
  
  <footer>
    <p>Data from CoinGecko API | Updates every 15 minutes</p>
  </footer>
  
  <script>/* Chart.js initialization */</script>
</body>
</html>
```

## Styling Guidelines
- Dark theme: `#1a1a2e` background, `#eaeaea` text
- Accent colors: `#0f3460` (primary), `#e94560` (alert)
- Font: System fonts (no external font loading)
- Max width: 1400px, centered layout
- Card-based design with shadows
- Responsive breakpoints: 768px (tablet), 480px (mobile)

## Chart Configuration
- Responsive: true
- Animation: 500ms ease-in-out
- Legend position: top
- Grid lines: Semi-transparent
- Tooltips: Enabled with custom formatting
