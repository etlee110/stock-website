# GitHub Pages Setup Guide

## Quick Setup

1. Go to https://github.com/etlee/stock-website
2. Settings → Pages
3. Source: **main** branch, **/ docs** folder
4. Click Save
5. Dashboard live at: https://etlee.github.io/stock-website

## Verify

- Check Actions tab for workflow runs
- Dashboard updates every 15 minutes automatically
- Clear cache with Ctrl+Shift+R if needed

## Troubleshooting

- **404 Error**: Ensure /docs folder exists with index.html
- **Not Updating**: Check workflow permissions (Settings → Actions → Write permissions)
- **Build Failed**: Add .nojekyll file to docs/ to bypass Jekyll

## References

- [GitHub Pages Docs](https://docs.github.com/en/pages)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
