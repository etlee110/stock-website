# Dashboard Layout Prompt

## Purpose
Defines the HTML structure, styling, and layout guidelines for the cryptocurrency dashboard.

## Design Principles
1. **Dark Theme**: Professional crypto trading aesthetic
2. **Responsive**: Works on mobile, tablet, and desktop
3. **Data-Focused**: Prioritize charts and metrics over decoration
4. **Auto-Updating**: Refresh every 15 minutes automatically
5. **Performance**: Fast load times, minimal dependencies

## HTML Structure

### Page Hierarchy
```
<body>
  └── <div class="container">
      ├── <header>
      │   ├── <h1> - Page title
      │   └── <p> - Last updated timestamp
      ├── <section id="whale-alerts">
      │   ├── <h2> - Section title
      │   └── <table> - Alert data
      ├── <section id="price-chart">
      │   ├── <h2> - Chart title
      │   └── <canvas> - Price visualization
      ├── <section id="volume-chart">
      │   ├── <h2> - Chart title
      │   └── <canvas> - Volume visualization
      ├── <section id="market-cap-chart">
      │   ├── <h2> - Chart title
      │   └── <canvas> - Market cap visualization
      └── <footer>
          └── <p> - Attribution and links
</body>
```

## Color Palette

### Primary Colors
```css
--bg-primary: #1a1a2e;      /* Main background */
--bg-secondary: #16213e;    /* Gradient/sections */
--bg-card: rgba(15, 52, 96, 0.3);  /* Card backgrounds */

--text-primary: #eaeaea;    /* Main text */
--text-secondary: #a0a0a0;  /* Secondary text */

--accent-primary: #00d4ff;  /* Links, highlights */
--accent-secondary: #0099ff; /* Secondary highlights */
--accent-danger: #e94560;   /* Alerts, errors */
```

### Semantic Colors
```css
--color-positive: #4caf50;  /* Price increases, gains */
--color-negative: #f44336;  /* Price decreases, losses */
--color-neutral: #2196f3;   /* Neutral states */

--severity-medium: rgba(255, 193, 7, 0.2);   /* Yellow */
--severity-high: rgba(255, 152, 0, 0.2);     /* Orange */
--severity-critical: rgba(220, 53, 69, 0.2); /* Red */
```

## Typography

### Font Stack
```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
```

### Font Sizes
- **H1**: 3em (desktop), 2em (mobile) - Page title
- **H2**: 1.8em - Section headers
- **Body**: 16px - Base font size
- **Small**: 0.9em - Timestamps, footnotes

### Font Weights
- **Bold**: Cryptocurrency names, important numbers
- **Regular**: Body text
- **Light**: Timestamps, secondary info

## Layout Guidelines

### Container
```css
.container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px;
}
```

### Card Sections
```css
.chart-section {
  background: rgba(15, 52, 96, 0.2);
  padding: 30px;
  border-radius: 15px;
  margin-bottom: 30px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
}
```

### Responsive Breakpoints
```css
/* Desktop: Default styles */
@media (max-width: 1200px) {
  /* Tablet adjustments */
}

@media (max-width: 768px) {
  /* Mobile adjustments */
  header h1 { font-size: 2em; }
  .chart-container { height: 300px; }
}

@media (max-width: 480px) {
  /* Small mobile */
  .container { padding: 10px; }
}
```

## Chart Configuration

### Standard Chart Options
```javascript
{
  responsive: true,
  maintainAspectRatio: false,
  animation: {
    duration: 500,
    easing: 'easeInOutQuad'
  },
  plugins: {
    legend: {
      display: true,
      position: 'top',
      labels: { color: '#eaeaea' }
    },
    tooltip: {
      enabled: true,
      backgroundColor: 'rgba(0, 0, 0, 0.8)',
      titleColor: '#fff',
      bodyColor: '#fff',
      borderColor: '#00d4ff',
      borderWidth: 1
    }
  },
  scales: {
    x: {
      ticks: { color: '#eaeaea' },
      grid: { color: 'rgba(255, 255, 255, 0.1)' }
    },
    y: {
      ticks: { color: '#eaeaea' },
      grid: { color: 'rgba(255, 255, 255, 0.1)' }
    }
  }
}
```

### Chart Heights
```css
.chart-container {
  position: relative;
  height: 400px; /* Desktop */
}

@media (max-width: 768px) {
  .chart-container {
    height: 300px; /* Tablet */
  }
}

@media (max-width: 480px) {
  .chart-container {
    height: 250px; /* Mobile */
  }
}
```

## Component Specifications

### Header
```html
<header>
  <h1>🌊 24/7 Crypto Watchtower</h1>
  <p class="last-updated">
    Last Updated: <span id="timestamp">2026-02-11 01:00:00 UTC</span>
  </p>
</header>
```

**Styling**:
- Centered text alignment
- Gradient background
- Large, eye-catching title with emoji
- Timestamp in lighter color

### Whale Alerts Section
```html
<section class="whale-alerts">
  <h2>🐋 Whale Alerts (3)</h2>
  <table class="alert-table">
    <thead>
      <tr>
        <th>Crypto</th>
        <th>Change</th>
        <th>Previous</th>
        <th>Current</th>
        <th>Severity</th>
      </tr>
    </thead>
    <tbody>
      <!-- Dynamic rows -->
    </tbody>
  </table>
</section>
```

**Table Styling**:
- Full width, collapsed borders
- Row highlighting on hover
- Color-coded severity backgrounds
- Responsive: stack columns on mobile

### Chart Sections
```html
<section class="chart-section">
  <h2>📈 Price Overview (Top 20)</h2>
  <div class="chart-container">
    <canvas id="priceChart"></canvas>
  </div>
</section>
```

### Footer
```html
<footer>
  <p>
    Data from CoinGecko API | Updates every 15 minutes | 
    <a href="https://github.com/etlee/stock-website">View on GitHub</a>
  </p>
</footer>
```

**Styling**:
- Centered text
- Lighter text color
- Links in accent color
- Subtle padding

## Accessibility

### ARIA Labels
```html
<canvas id="priceChart" aria-label="Cryptocurrency price chart showing top 20 coins"></canvas>

<table class="alert-table" aria-label="Whale movement alerts"></table>
```

### Semantic HTML
- Use `<header>`, `<section>`, `<footer>` for structure
- Use `<h1>`, `<h2>` hierarchy properly
- Use `<table>` with `<thead>` and `<tbody>`

### Color Contrast
- Ensure 4.5:1 contrast ratio for text
- Use color + icon/emoji for severity (not color alone)
- Test with color blindness simulators

## Performance Optimizations

### Asset Loading
```html
<!-- CDN with integrity check -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js" 
        integrity="sha384-..." 
        crossorigin="anonymous"></script>

<!-- Preconnect to CDN -->
<link rel="preconnect" href="https://cdn.jsdelivr.net">
```

### Auto-Refresh
```html
<!-- Refresh every 15 minutes (900 seconds) -->
<meta http-equiv="refresh" content="900">
```

### Lazy Loading (Future)
```html
<!-- For images/icons if added -->
<img src="icon.png" loading="lazy" alt="...">
```

## Mobile Optimizations

### Viewport
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0">
```

### Touch Targets
- Minimum 44x44px for interactive elements
- Adequate spacing between clickable items

### Table Responsiveness
```css
@media (max-width: 600px) {
  .alert-table {
    font-size: 0.9em;
  }
  
  .alert-table th:nth-child(3),
  .alert-table td:nth-child(3) {
    display: none; /* Hide "Previous" column on mobile */
  }
}
```

## Browser Compatibility
- **Target**: Modern browsers (Chrome, Firefox, Safari, Edge)
- **Minimum**: ES6 support, CSS Grid
- **Graceful Degradation**: Basic layout works without JavaScript

## Testing Checklist
- [ ] Renders correctly on desktop (1920x1080)
- [ ] Renders correctly on tablet (768x1024)
- [ ] Renders correctly on mobile (375x667)
- [ ] Charts load and display data
- [ ] Auto-refresh works after 15 minutes
- [ ] All links work
- [ ] Color contrast is sufficient
- [ ] No console errors
- [ ] Loads in <3 seconds

## Future Enhancements
- **Dark/Light Theme Toggle**: User preference
- **Interactive Filters**: Filter by price range, volume
- **Historical Charts**: Show 24h price trends
- **Search**: Find specific cryptocurrency
- **Alerts**: Browser notifications for critical whales
