# Deploy GitHub Pages Skill

## Purpose
Automate deployment of the crypto dashboard to GitHub Pages.

## Capabilities
- Commit generated files to repository
- Push to GitHub (main branch or gh-pages branch)
- Configure GitHub Pages settings
- Verify deployment success
- Handle deployment errors gracefully

## Deployment Strategy
**Option 1: `/docs` folder approach** (Recommended)
- Generate all files in `docs/` directory
- Commit to main branch
- GitHub Pages serves from `/docs` folder

**Option 2: `gh-pages` branch approach**
- Generate files in project root
- Push to separate `gh-pages` branch
- GitHub Pages serves from branch root

## Files to Deploy
```
docs/
├── index.html          (Dashboard)
├── data/
│   ├── crypto-data.json
│   └── whale-alerts.json
└── assets/
    └── favicon.ico (optional)
```

## Git Workflow
```bash
# Stage files
git add docs/index.html docs/data/*.json

# Commit with timestamp
git commit -m "Update dashboard - $(date -u +%Y-%m-%dT%H:%M:%SZ)"

# Push to GitHub
git push origin main
```

## GitHub Pages Configuration
Repository: `etlee/stock-website`
URL: `https://etlee.github.io/stock-website`
Source: `main` branch, `/docs` folder

## Verification Steps
1. Check git push success (exit code 0)
2. Wait 30 seconds for GitHub Pages build
3. Curl `https://etlee.github.io/stock-website` to verify live
4. Check HTTP status code (should be 200)
5. Validate HTML structure in response

## Error Handling
- **Git conflicts**: Pull, merge, retry push
- **Network errors**: Retry with exponential backoff (3 attempts)
- **GitHub Pages build failure**: Check repository settings, retry
- **Authentication errors**: Verify GitHub token/credentials

## Security
- Use GitHub Actions bot account for commits
- Token stored in GitHub Secrets: `GITHUB_TOKEN`
- Never commit API keys or sensitive data
- Use `.gitignore` for logs and cache files

## Commit Message Format
```
Update dashboard - 2026-02-11T01:00:00Z

- Updated crypto data for top 20 coins
- Detected X whale movements
- Generated charts and alerts
```

## Performance
- Deployment time: <10 seconds (git operations)
- GitHub Pages build time: ~30 seconds
- Total time to live: ~40 seconds
