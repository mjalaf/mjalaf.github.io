---
title: "GitHub CLI Essentials"
description: "Most-used GitHub CLI (gh) commands for repos, pull requests, issues, releases, and Actions."
category: "GitHub"
---

## Setup

```bash
# Install (Linux/macOS via package manager, or see https://cli.github.com)
# Authenticate (browser-based)
gh auth login

# Check status / switch accounts
gh auth status
gh auth switch

# Set default repo for the current directory (avoids -R on every command)
gh repo set-default <owner>/<repo>
```

## Repositories

```bash
# Create a new repo from the current directory
gh repo create <name> --public --source=. --remote=origin --push

# Clone (works with shorthand)
gh repo clone <owner>/<repo>

# Fork + clone
gh repo fork <owner>/<repo> --clone

# View repo metadata in the terminal
gh repo view --web         # open on GitHub
gh repo view <owner>/<repo>
```

## Pull Requests

```bash
# Create a PR from the current branch
gh pr create --fill                              # use commit/branch as title+body
gh pr create --title "feat: X" --body "..." --base main --reviewer @teammate

# List / view / check out PRs
gh pr list --state open --author "@me"
gh pr view 123 --web
gh pr checkout 123

# Review actions
gh pr review 123 --approve -b "LGTM"
gh pr review 123 --request-changes -b "Please fix X"
gh pr review 123 --comment   -b "Question about Y"

# CI status for a PR
gh pr checks 123 --watch

# Merge (squash) and delete branch
gh pr merge 123 --squash --delete-branch
```

## Issues

```bash
gh issue create --title "Bug: ..." --body "..." --label bug --assignee @me
gh issue list --state open --label bug --assignee "@me"
gh issue view 42 --web
gh issue close 42 --comment "Fixed in #50"
gh issue develop 42 --checkout      # creates a branch linked to the issue
```

## Releases

```bash
# Draft a release from the latest tag
gh release create v1.2.0 \
  --title "v1.2.0" \
  --notes-from-tag \
  ./dist/*.zip ./dist/*.tar.gz

# Generate release notes automatically from PRs
gh release create v1.3.0 --generate-notes

# List / download
gh release list
gh release download v1.2.0 --pattern "*.zip"
```

## GitHub Actions

```bash
# Workflows + runs
gh workflow list
gh run list --workflow=deploy.yml --limit 10
gh run view <run-id> --log
gh run watch                       # follow the latest run live
gh run rerun <run-id> --failed     # rerun only failed jobs

# Trigger a manual workflow
gh workflow run deploy.yml -f environment=prod
```

## Gists, secrets, and API

```bash
# Quick gist from a file or stdin
gh gist create notes.md --public
echo "hello" | gh gist create -f hello.txt

# Repo secrets (Actions)
gh secret set MY_TOKEN --body "$VALUE"
gh secret list

# Raw API calls (great for things the CLI doesn't wrap)
gh api repos/<owner>/<repo>/stats/contributors --jq '.[].author.login'
gh api graphql -f query='query { viewer { login } }'
```

## Productivity tips

```bash
# Browse the current branch on GitHub
gh browse

# Open the file at the current line on GitHub
gh browse path/to/file.ts:42

# Aliases
gh alias set co 'pr checkout'
gh alias set prs 'pr list --author "@me"'
```
