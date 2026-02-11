# HTML/CSS/JavaScript Standards

## Purpose
Specific standards for front-end code in the Crypto Watchtower dashboard.

## HTML Standards

### Document Structure
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="refresh" content="900">
    <meta name="description" content="Real-time cryptocurrency monitoring dashboard">
    <title>24/7 Crypto Watchtower</title>
    
    <!-- External resources -->
    <link rel="stylesheet" href="styles.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
</head>
<body>
    <!-- Content -->
    <script src="app.js"></script>
</body>
</html>
```

### Semantic HTML
Use appropriate semantic elements:
```html
<!-- Good: Semantic structure -->
<header>
  <h1>24/7 Crypto Watchtower</h1>
  <nav>...</nav>
</header>

<main>
  <section id="whale-alerts">
    <h2>Whale Alerts</h2>
    <article>...</article>
  </section>
</main>

<footer>
  <p>Data from CoinGecko</p>
</footer>

<!-- Bad: Divitis -->
<div class="header">
  <div class="title">24/7 Crypto Watchtower</div>
</div>
```

### Indentation & Formatting
- Use 4 spaces for indentation (not tabs)
- Lowercase for element names and attributes
- Quote attribute values
- Close all tags properly
- One element per line for readability

```html
<!-- Good -->
<div class="container">
    <h1 id="title">Title</h1>
    <p class="description">Description text</p>
</div>

<!-- Bad -->
<DIV class=container><H1 id=title>Title</H1><P class=description>Description text</P></DIV>
```

### Accessibility

#### Alt Text
```html
<!-- All images need descriptive alt text -->
<img src="bitcoin-icon.png" alt="Bitcoin cryptocurrency logo">

<!-- Decorative images use empty alt -->
<img src="decoration.png" alt="">
```

#### ARIA Labels
```html
<!-- For interactive elements without text -->
<button aria-label="Close alert">
    <span aria-hidden="true">&times;</span>
</button>

<!-- For charts and data visualizations -->
<canvas id="priceChart" 
        aria-label="Line chart showing cryptocurrency prices for top 20 coins"
        role="img">
</canvas>
```

#### Form Labels
```html
<!-- Always associate labels with inputs -->
<label for="crypto-search">Search cryptocurrency:</label>
<input type="text" id="crypto-search" name="search">
```

#### Keyboard Navigation
```html
<!-- Ensure logical tab order -->
<nav>
    <a href="#alerts" tabindex="1">Alerts</a>
    <a href="#charts" tabindex="2">Charts</a>
    <a href="#data" tabindex="3">Data</a>
</nav>
```

### Tables
```html
<!-- Use proper table structure -->
<table class="alert-table">
    <caption>Recent Whale Movement Alerts</caption>
    <thead>
        <tr>
            <th scope="col">Cryptocurrency</th>
            <th scope="col">Change %</th>
            <th scope="col">Severity</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Bitcoin</td>
            <td>+7.5%</td>
            <td>High</td>
        </tr>
    </tbody>
</table>
```

## CSS Standards

### Organization
```css
/* 1. Variables */
:root {
    --color-primary: #00d4ff;
    --color-bg: #1a1a2e;
    --spacing-unit: 8px;
}

/* 2. Reset/Base styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* 3. Typography */
body {
    font-family: system-ui, sans-serif;
}

/* 4. Layout */
.container {
    max-width: 1400px;
}

/* 5. Components */
.button {
    padding: 10px 20px;
}

/* 6. Utilities */
.text-center {
    text-align: center;
}

/* 7. Media queries */
@media (max-width: 768px) {
    .container {
        padding: 10px;
    }
}
```

### Naming Conventions (BEM-inspired)
```css
/* Block */
.card {}

/* Element */
.card__header {}
.card__body {}
.card__footer {}

/* Modifier */
.card--highlighted {}
.card--large {}

/* State */
.card.is-loading {}
.card.is-active {}
```

### CSS Variables
```css
/* Define variables for consistency */
:root {
    /* Colors */
    --bg-primary: #1a1a2e;
    --bg-secondary: #16213e;
    --text-primary: #eaeaea;
    --text-secondary: #a0a0a0;
    --accent-primary: #00d4ff;
    --accent-danger: #e94560;
    
    /* Spacing */
    --space-xs: 4px;
    --space-sm: 8px;
    --space-md: 16px;
    --space-lg: 24px;
    --space-xl: 32px;
    
    /* Typography */
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.25rem;
    --font-size-xl: 2rem;
    
    /* Border radius */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 16px;
    
    /* Shadows */
    --shadow-sm: 0 2px 4px rgba(0, 0, 0, 0.1);
    --shadow-md: 0 4px 8px rgba(0, 0, 0, 0.2);
    --shadow-lg: 0 8px 32px rgba(0, 0, 0, 0.3);
}

/* Usage */
.card {
    background: var(--bg-secondary);
    padding: var(--space-md);
    border-radius: var(--radius-md);
    box-shadow: var(--shadow-md);
}
```

### Responsive Design
```css
/* Mobile-first approach */
.container {
    padding: 10px;
}

/* Tablet */
@media (min-width: 768px) {
    .container {
        padding: 20px;
    }
}

/* Desktop */
@media (min-width: 1024px) {
    .container {
        padding: 40px;
    }
}

/* Large desktop */
@media (min-width: 1440px) {
    .container {
        max-width: 1400px;
        margin: 0 auto;
    }
}
```

### Performance
```css
/* Use efficient selectors */
/* Good - class selector */
.button { }

/* Bad - overly specific */
body div.container div.row div.col div.button { }

/* Use transform for animations (GPU accelerated) */
.card {
    transition: transform 0.3s ease;
}
.card:hover {
    transform: translateY(-5px);
}

/* Avoid expensive properties for animations */
/* Bad - causes reflow */
.element {
    animation: expand 0.3s;
}
@keyframes expand {
    to { height: 200px; }
}

/* Good - uses transform */
.element {
    animation: expand 0.3s;
}
@keyframes expand {
    to { transform: scaleY(2); }
}
```

### Color Contrast
```css
/* Ensure WCAG AA compliance (4.5:1 for normal text) */
.text-on-dark {
    color: #eaeaea;  /* Contrast ratio: 13:1 on #1a1a2e */
}

.text-on-light {
    color: #2c2c2c;  /* Contrast ratio: 12:1 on #ffffff */
}

/* Use tools like WebAIM Contrast Checker */
```

## JavaScript Standards

### Modern ES6+ Syntax
```javascript
// Use const/let, never var
const API_URL = 'https://api.coingecko.com/api/v3';
let retryCount = 0;

// Arrow functions
const calculateChange = (prev, curr) => ((curr - prev) / prev) * 100;

// Template literals
const message = `Price changed by ${change}%`;

// Destructuring
const { current_price, symbol, name } = cryptoData;

// Spread operator
const updatedData = { ...oldData, ...newData };

// Async/await
async function fetchData() {
    const response = await fetch(API_URL);
    return await response.json();
}
```

### Error Handling
```javascript
// Use try-catch for async operations
async function loadCryptoData() {
    try {
        const response = await fetch('data/crypto-data.json');
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Failed to load crypto data:', error);
        // Show user-friendly error message
        showErrorMessage('Unable to load cryptocurrency data. Please refresh the page.');
        return null;
    }
}

// Promise rejection handling
fetch(url)
    .then(response => response.json())
    .catch(error => {
        console.error('Fetch failed:', error);
        handleError(error);
    });
```

### Function Documentation
```javascript
/**
 * Calculate percentage change between two prices
 * @param {number} previousPrice - The previous price value
 * @param {number} currentPrice - The current price value
 * @returns {number} Percentage change (positive or negative)
 * @throws {Error} If prices are not valid numbers
 */
function calculatePriceChange(previousPrice, currentPrice) {
    if (typeof previousPrice !== 'number' || typeof currentPrice !== 'number') {
        throw new Error('Prices must be numbers');
    }
    
    if (previousPrice === 0) {
        throw new Error('Previous price cannot be zero');
    }
    
    return ((currentPrice - previousPrice) / previousPrice) * 100;
}
```

### DOM Manipulation
```javascript
// Cache DOM queries
const priceChart = document.getElementById('priceChart');
const alertContainer = document.getElementById('whale-alerts-container');

// Use event delegation
document.addEventListener('click', (event) => {
    if (event.target.matches('.alert-close')) {
        event.target.closest('.alert').remove();
    }
});

// Batch DOM updates
const fragment = document.createDocumentFragment();
data.forEach(item => {
    const row = document.createElement('tr');
    row.innerHTML = `<td>${item.name}</td><td>${item.price}</td>`;
    fragment.appendChild(row);
});
tableBody.appendChild(fragment);
```

### Chart.js Configuration
```javascript
// Reusable chart configuration
const defaultChartOptions = {
    responsive: true,
    maintainAspectRatio: false,
    animation: {
        duration: 500,
        easing: 'easeInOutQuad'
    },
    plugins: {
        legend: {
            display: true,
            labels: {
                color: '#eaeaea',
                font: { size: 12 }
            }
        },
        tooltip: {
            enabled: true,
            backgroundColor: 'rgba(0, 0, 0, 0.8)',
            titleColor: '#fff',
            bodyColor: '#fff'
        }
    },
    scales: {
        x: {
            ticks: { color: '#eaeaea' },
            grid: { color: 'rgba(255, 255, 255, 0.1)' }
        },
        y: {
            ticks: { 
                color: '#eaeaea',
                callback: (value) => '$' + value.toLocaleString()
            },
            grid: { color: 'rgba(255, 255, 255, 0.1)' }
        }
    }
};

// Create chart with custom config
function createPriceChart(data) {
    const ctx = document.getElementById('priceChart').getContext('2d');
    return new Chart(ctx, {
        type: 'line',
        data: {
            labels: data.map(c => c.name),
            datasets: [{
                label: 'Price (USD)',
                data: data.map(c => c.current_price),
                borderColor: '#00d4ff',
                tension: 0.4
            }]
        },
        options: defaultChartOptions
    });
}
```

### Data Formatting
```javascript
// Utility functions for formatting
const formatters = {
    // Currency formatting
    currency: (value) => {
        return new Intl.NumberFormat('en-US', {
            style: 'currency',
            currency: 'USD',
            minimumFractionDigits: 2
        }).format(value);
    },
    
    // Percentage formatting
    percentage: (value) => {
        const sign = value > 0 ? '+' : '';
        return `${sign}${value.toFixed(2)}%`;
    },
    
    // Large number formatting
    largeNumber: (value) => {
        if (value >= 1e9) return `$${(value / 1e9).toFixed(2)}B`;
        if (value >= 1e6) return `$${(value / 1e6).toFixed(2)}M`;
        if (value >= 1e3) return `$${(value / 1e3).toFixed(2)}K`;
        return `$${value.toFixed(2)}`;
    },
    
    // Date/time formatting
    datetime: (dateString) => {
        return new Date(dateString).toLocaleString('en-US', {
            year: 'numeric',
            month: 'short',
            day: 'numeric',
            hour: '2-digit',
            minute: '2-digit',
            timeZoneName: 'short'
        });
    }
};

// Usage
const priceText = formatters.currency(45000.50);  // "$45,000.50"
const changeText = formatters.percentage(7.5);     // "+7.50%"
```

## Testing

### Manual Testing Checklist
- [ ] Page loads without console errors
- [ ] All charts render correctly
- [ ] Data displays accurately
- [ ] Responsive on mobile/tablet/desktop
- [ ] Colors have sufficient contrast
- [ ] Keyboard navigation works
- [ ] Auto-refresh works after 15 minutes
- [ ] Links and buttons are clickable
- [ ] Error states display properly

### Browser Testing
Test on:
- Chrome (latest)
- Firefox (latest)
- Safari (latest)
- Edge (latest)
- Mobile Safari (iOS)
- Chrome Mobile (Android)

### Performance Testing
```javascript
// Measure page load time
window.addEventListener('load', () => {
    const perfData = performance.timing;
    const pageLoadTime = perfData.loadEventEnd - perfData.navigationStart;
    console.log(`Page load time: ${pageLoadTime}ms`);
});

// Measure chart render time
const startTime = performance.now();
renderPriceChart(data);
const endTime = performance.now();
console.log(`Chart rendered in ${(endTime - startTime).toFixed(2)}ms`);
```

## Security

### XSS Prevention
```javascript
// Never use innerHTML with user input
// Bad
element.innerHTML = userInput;

// Good - use textContent
element.textContent = userInput;

// Or sanitize if HTML is needed
element.innerHTML = DOMPurify.sanitize(userInput);
```

### Content Security Policy
```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' https://cdn.jsdelivr.net; 
               style-src 'self' 'unsafe-inline'; 
               img-src 'self' data:;">
```

## Validation

### HTML Validation
```bash
# Use W3C validator
curl -H "Content-Type: text/html; charset=utf-8" \
     --data-binary @docs/index.html \
     https://validator.w3.org/nu/ > validation.txt
```

### CSS Validation
```bash
# Use CSS validator
curl -H "Content-Type: text/css; charset=utf-8" \
     --data-binary @styles.css \
     https://jigsaw.w3.org/css-validator/validator > validation.txt
```

### JavaScript Linting
```bash
# Use ESLint (if installed)
eslint app.js
```

## Resources
- [MDN Web Docs](https://developer.mozilla.org/)
- [Chart.js Documentation](https://www.chartjs.org/docs/)
- [WCAG Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [Can I Use](https://caniuse.com/) - Browser compatibility
