# GitHub setup for this wiki

This wiki is already a local git repository.

## Current local repo
- Path: `/Users/tinycoder/wiki`
- Default branch: `main`

## After GitHub auth is ready
Example commands:

```bash
cd /Users/tinycoder/wiki
# create remote repo (choose one)
# gh repo create <repo-name> --private --source . --remote origin --push
# or manually:
# git remote add origin https://github.com/<owner>/<repo>.git
# git push -u origin main
```

## Recommended repo settings
- private repository
- enable branch protection later if needed
- use README in repo root (already present locally)

## Auth options
1. Install and login `gh`
2. Or provide `GITHUB_TOKEN` in `~/.hermes/.env`
