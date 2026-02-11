# 24/7 Crypto Watchtower - Implementation Plan

## Problem Statement
Build an autonomous system that monitors cryptocurrency markets 24/7, detects whale movements (high volatility), generates visual market reports, and deploys a live dashboard to GitHub Pages—all orchestrated by GitHub Copilot CLI agents without manual code.

## Approach
Create a multi-agent system using GitHub Copilot Custom Agents that work together:
- **Data Agent**: Fetches crypto data from CoinGecko API
- **Analysis Agent**: Detects whale movements based on volatility
- **Visualization Agent**: Generates Chart.js-based market reports
- **Deployment Agent**: Publishes dashboard to GitHub Pages

## Project Specifications
- **Deployment Target**: etlee/stock-website on GitHub Pages
- **Monitoring Scope**: Top 20 cryptocurrencies by market cap
- **Update Frequency**: Every 15 minutes
- **Whale Detection**: Price change > 5% in 15 minutes
- **Visualization**: Chart.js interactive HTML/CSS charts
- **File Format**: Follow GitHub Copilot CLI format (`.agent.md` with YAML front matter)

## Workplan

### Phase 1: Project Structure Setup
- [ ] Create `.github/agents/` directory for custom agents
- [ ] Create `.github/skills/` directory for reusable skills
- [ ] Create `.github/prompts/` directory for prompt templates

### Phase 2: Skills Development
- [ ] Create `fetch-crypto-data.skill.md` - CoinGecko API integration skill
- [ ] Create `detect-volatility.skill.md` - Whale movement detection algorithms
- [ ] Create `generate-charts.skill.md` - Chart.js visualization generation
- [ ] Create `deploy-github-pages.skill.md` - GitHub Pages deployment automation

### Phase 3: Custom Agents Development
- [ ] Create `data-fetcher.agent.md` - Orchestrates crypto data fetching
- [ ] Create `market-analyst.agent.md` - Analyzes volatility and whale movements
- [ ] Create `report-generator.agent.md` - Creates visual market reports
- [ ] Create `deployment-manager.agent.md` - Handles GitHub Pages deployment

### Phase 4: Prompt Templates
- [ ] Create `data-pipeline-orchestration.prompt.md` - Main pipeline coordination
- [ ] Create `whale-alert-format.prompt.md` - Whale movement notification format
- [ ] Create `dashboard-layout.prompt.md` - Dashboard HTML structure guidelines

### Phase 5: Instructions & Standards
- [ ] Create `.github/instructions/coding-style.md` - General coding standards
- [ ] Create `.github/instructions/html-standards.md` - HTML/CSS/JS specific guidelines
- [ ] Create `.github/instructions/agent-interaction.md` - Agent communication protocols

### Phase 6: Automation Configuration
- [ ] Create GitHub Actions workflow for scheduled data fetching (every 15 minutes)
- [ ] Configure API keys and secrets management
- [ ] Set up GitHub Pages deployment configuration

### Phase 7: Testing & Documentation
- [ ] Test complete data pipeline end-to-end
- [ ] Create README.md with system architecture diagram
- [ ] Document agent interaction flows
- [ ] Add troubleshooting guide

## Notes & Considerations

### Agent Design Principles
- Each agent should have a single, well-defined responsibility
- Agents communicate through file-based state (JSON data files)
- Use GitHub Copilot CLI format: `.agent.md` files with YAML front matter
- Specify model (e.g., `claude-sonnet-4.5`), tools, and capabilities in front matter

### Skills vs Agents
- **Skills**: Reusable capabilities that can be invoked by any agent (e.g., API calls, parsing)
- **Agents**: Orchestrators that combine skills to accomplish complex tasks

### Data Flow Architecture
1. **Data Fetcher Agent** → Calls CoinGecko API → Saves `crypto-data.json`
2. **Market Analyst Agent** → Reads `crypto-data.json` → Detects whales → Saves `whale-alerts.json`
3. **Report Generator Agent** → Reads both JSON files → Generates `index.html` with Chart.js
4. **Deployment Manager Agent** → Pushes to GitHub Pages → Verifies deployment

### API Considerations
- CoinGecko free tier: 10-30 calls/minute
- Cache data to minimize API calls
- Implement exponential backoff for rate limiting

### Security
- Store CoinGecko API key in GitHub Secrets (if required)
- Use GitHub token for automated commits/deployments
- No sensitive data in committed files

### Visualization Requirements
- Use Chart.js CDN (no npm dependencies)
- Create responsive HTML dashboard
- Include: price charts, volume charts, whale alert table
- Color coding: green (up), red (down), yellow (whale alert)

### Deployment Strategy
- Use `gh-pages` branch or `/docs` folder approach
- Automated deployment via GitHub Actions
- No manual build steps required

### File Format Example
```markdown
---
type: agent
name: data-fetcher
description: Fetches cryptocurrency data from CoinGecko API
model: claude-sonnet-4.5
tools:
  - bash
  - create
  - edit
skills:
  - fetch-crypto-data
  - error-handling
---

# Data Fetcher Agent

[Agent instructions here...]
```

## Success Criteria
- [ ] System automatically fetches crypto data every 15 minutes
- [ ] Whale movements (>5% change) are detected and highlighted
- [ ] Live dashboard deployed to https://etlee.github.io/stock-website
- [ ] Dashboard updates automatically with new data
- [ ] All functionality works without manual intervention
- [ ] Code follows specified format and standards
