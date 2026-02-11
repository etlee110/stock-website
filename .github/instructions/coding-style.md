# Coding Style Guide

## Purpose
Establish consistent coding standards for all agents and automation scripts in the Crypto Watchtower project.

## General Principles
1. **Simplicity**: Prefer simple, readable code over clever optimizations
2. **Clarity**: Code should be self-documenting with minimal comments
3. **Consistency**: Follow established patterns throughout the project
4. **Maintainability**: Write code that others can easily understand and modify
5. **Error Handling**: Always handle errors gracefully with helpful messages

## File Organization

### Directory Structure
```
Stock-Website/
├── .github/
│   ├── agents/          # Custom agent definitions
│   ├── skills/          # Reusable skill modules
│   ├── prompts/         # Prompt templates
│   ├── instructions/    # This file and related docs
│   └── workflows/       # GitHub Actions workflows
├── data/                # Generated data files (JSON)
├── docs/                # Published to GitHub Pages
├── logs/                # Execution logs
└── scripts/             # Helper scripts (if needed)
```

### File Naming Conventions
- **Agents**: `{name}.agent.md` (e.g., `data-fetcher.agent.md`)
- **Skills**: `{name}.skill.md` (e.g., `fetch-crypto-data.skill.md`)
- **Prompts**: `{name}.prompt.md` (e.g., `whale-alert-format.prompt.md`)
- **Data**: `{name}.json` (lowercase, hyphenated)
- **Logs**: `{agent-name}.log`

## Bash/Shell Scripts

### Shebang and Settings
```bash
#!/usr/bin/env bash
set -euo pipefail  # Exit on error, undefined vars, pipe failures
```

### Variable Naming
```bash
# Constants: UPPERCASE_WITH_UNDERSCORES
readonly API_URL="https://api.coingecko.com"
readonly MAX_RETRIES=3

# Variables: lowercase_with_underscores
local retry_count=0
local timestamp=$(date -u +%Y-%m-%dT%H:%M:%SZ)
```

### Function Definition
```bash
# Function names: lowercase_with_underscores
fetch_crypto_data() {
  local currency="${1:-usd}"
  local per_page="${2:-20}"
  
  # Function body
}
```

### Error Handling
```bash
# Check command success
if curl -s "$API_URL" > data.json; then
  echo "✓ Data fetched successfully"
else
  echo "✗ Error: Failed to fetch data" >&2
  exit 1
fi

# Or use trap for cleanup
trap 'echo "Error on line $LINENO"; cleanup' ERR
```

### Logging
```bash
# Function for consistent logging
log() {
  echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] $*" | tee -a logs/script.log
}

log "INFO: Starting data fetch"
log "ERROR: API request failed"
```

## Python Scripts

### Style: PEP 8
```python
#!/usr/bin/env python3
"""
Brief module description.
"""

import json
import sys
from datetime import datetime
from typing import List, Dict, Optional

# Constants
API_URL = "https://api.coingecko.com/api/v3"
MAX_RETRIES = 3

# Functions
def fetch_data(coin_id: str) -> Optional[Dict]:
    """
    Fetch cryptocurrency data for a specific coin.
    
    Args:
        coin_id: CoinGecko coin identifier
        
    Returns:
        Dict with coin data, or None if request fails
    """
    # Implementation
    pass

# Main execution
if __name__ == "__main__":
    main()
```

### Variable Naming
```python
# Constants: SCREAMING_SNAKE_CASE
MAX_COINS = 20

# Variables/functions: snake_case
crypto_data = fetch_crypto_data()
alert_count = len(whale_alerts)

# Classes: PascalCase
class CryptoAnalyzer:
    pass
```

### Error Handling
```python
# Use try-except with specific exceptions
try:
    with open('data.json', 'r') as f:
        data = json.load(f)
except FileNotFoundError:
    print("Error: Data file not found", file=sys.stderr)
    sys.exit(1)
except json.JSONDecodeError:
    print("Error: Invalid JSON format", file=sys.stderr)
    sys.exit(1)
```

## JavaScript

### Style: Modern ES6+
```javascript
// Use const by default, let when needed, never var
const API_URL = 'https://api.coingecko.com/api/v3';
let retryCount = 0;

// Arrow functions for callbacks
cryptoData.map(coin => coin.current_price);

// Async/await for promises
async function fetchData() {
  try {
    const response = await fetch(API_URL);
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch failed:', error);
    throw error;
  }
}
```

### Variable Naming
```javascript
// Constants: SCREAMING_SNAKE_CASE or camelCase
const MAX_COINS = 20;
const apiUrl = 'https://...';  // OK for const that doesn't change

// Variables/functions: camelCase
const cryptoData = [];
function calculateChange(prev, curr) { }

// Classes: PascalCase
class CryptoAnalyzer { }
```

## JSON Files

### Formatting
```json
{
  "timestamp": "2026-02-11T01:00:00.000Z",
  "data": [
    {
      "id": "bitcoin",
      "symbol": "btc",
      "name": "Bitcoin",
      "current_price": 45000.50
    }
  ]
}
```

### Best Practices
- Use 2-space indentation
- Include timestamps (ISO 8601 format with timezone)
- Use snake_case for keys
- Keep numbers as numbers, not strings
- Validate with `jq` before committing

## Markdown (Agents, Skills, Prompts)

### YAML Front Matter (for .agent.md files)
```yaml
---
type: agent
name: data-fetcher
description: Brief one-line description
model: claude-sonnet-4.5
tools:
  - bash
  - create
  - edit
skills:
  - fetch-crypto-data
---
```

### Document Structure
```markdown
# Title (H1)

## Purpose (H2)
Brief description of what this does.

## Responsibilities (H2)
Bulleted list of what this component handles.

## Execution Steps (H2)
### Step 1: Action Name (H3)
Details and code examples.

## Example (H2)
Code blocks with syntax highlighting.
```

### Code Blocks
````markdown
```bash
# Bash examples
echo "Hello"
```

```python
# Python examples
print("Hello")
```

```json
{"key": "value"}
```
````

## Comments

### When to Comment
- **Do**: Explain WHY, not WHAT
- **Do**: Document complex algorithms or non-obvious logic
- **Do**: Add TODO/FIXME/NOTE markers for future work
- **Don't**: State the obvious
- **Don't**: Comment out large blocks of code (use git instead)

### Examples
```bash
# Good: Explains why
# Retry with exponential backoff to handle rate limiting
sleep $((2 ** retry_count))

# Bad: States the obvious
# Set variable to 0
counter=0
```

```python
# Good
# Calculate price change as percentage to detect whale movements
# Threshold of 5% aligns with CoinGecko's volatility classification
change_percent = ((current - previous) / previous) * 100

# Bad
# Get the change percent
change_percent = ((current - previous) / previous) * 100
```

## Testing & Validation

### Before Committing
```bash
# Validate JSON files
jq '.' data/crypto-data.json > /dev/null

# Check bash syntax
bash -n script.sh

# Python linting (optional)
pylint script.py

# Markdown linting (optional)
markdownlint README.md
```

### Test Scripts
```bash
# Create test script for critical functions
test_fetch_data() {
  local result=$(fetch_crypto_data)
  if [ -z "$result" ]; then
    echo "FAIL: fetch_crypto_data returned empty"
    return 1
  fi
  echo "PASS: fetch_crypto_data"
}
```

## Security Best Practices

### Never Commit Secrets
```bash
# Use environment variables
API_KEY="${COINGECKO_API_KEY:-}"

# Or read from secure storage
API_KEY=$(cat ~/.secrets/coingecko-api-key)
```

### Input Validation
```bash
# Validate inputs before using
if [[ ! "$coin_id" =~ ^[a-z0-9-]+$ ]]; then
  echo "Invalid coin ID format" >&2
  exit 1
fi
```

### Use HTTPS Only
```bash
# Always use secure connections
curl "https://api.coingecko.com/..."  # Good
curl "http://api.coingecko.com/..."   # Bad
```

## Performance Guidelines

### Minimize API Calls
```bash
# Cache responses
if [ -f cache/data.json ] && [ $(($(date +%s) - $(stat -f %m cache/data.json))) -lt 900 ]; then
  echo "Using cached data"
  cp cache/data.json data/crypto-data.json
else
  # Fetch fresh data
fi
```

### Use Efficient Tools
```bash
# Prefer jq over Python for JSON if it's simple manipulation
jq '.data[] | select(.current_price > 1000)' data.json

# Use parallel processing when possible
curl url1 & curl url2 & wait
```

## Version Control

### Commit Messages
```
<type>: <subject>

<body>

<footer>
```

**Types**: feat, fix, docs, style, refactor, test, chore

**Example**:
```
feat: Add whale movement detection

- Implement price change calculation
- Add severity classification (medium/high/critical)
- Create whale-alerts.json output format

Closes #123
```

### .gitignore
```
# Logs
logs/*.log

# Data (if sensitive)
data/crypto-data-previous.json

# System files
.DS_Store
*.swp

# Don't ignore
!.github/
!docs/
```

## Documentation Standards

### README Files
Every major directory should have a README:
- Purpose of the directory
- Files it contains
- How to use/run
- Examples

### Inline Documentation
- Agent files: Include purpose, inputs, outputs, examples
- Skills: Document parameters, return values, error conditions
- Scripts: Add usage instructions at the top

## Maintenance

### Regular Reviews
- Review logs weekly for errors
- Update dependencies monthly
- Audit API usage for rate limit compliance

### Deprecation Process
1. Mark as deprecated in documentation
2. Add warning message when used
3. Provide migration path
4. Remove after 2 versions

## Questions?
Refer to official documentation:
- [Bash Style Guide](https://google.github.io/styleguide/shellguide.html)
- [PEP 8 Python Style](https://pep8.org/)
- [Airbnb JavaScript Style](https://github.com/airbnb/javascript)
