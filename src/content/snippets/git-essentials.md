---
title: "Git Essentials"
description: "Day-to-day Git commands for branching, committing, undoing changes, rebasing, and inspecting history."
category: "Git"
---

## Configuration

```bash
# Identity (per-repo override with --local)
git config --global user.name "Martin Jalaf"
git config --global user.email "you@example.com"

# Default branch name for new repos
git config --global init.defaultBranch main

# Use VS Code as the editor / diff / merge tool
git config --global core.editor "code --wait"

# Sensible pull behavior
git config --global pull.rebase true
git config --global rebase.autoStash true

# Show current config (and where each value comes from)
git config --list --show-origin
```

## Cloning, status, and staging

```bash
# Clone with a shallow history (faster for large repos)
git clone --depth=1 https://github.com/<org>/<repo>.git

# What's going on?
git status -sb              # short + branch info
git diff                    # unstaged changes
git diff --staged           # staged changes

# Stage / unstage
git add -p                  # interactive: stage hunks
git restore --staged <file> # unstage without losing changes
git restore <file>          # discard unstaged changes (DANGEROUS)
```

## Branching

```bash
# Create + switch
git switch -c feature/my-change

# List branches (local + remote)
git branch -a

# Rename current branch
git branch -m new-name

# Delete branches
git branch -d feature/done       # safe (refuses if not merged)
git branch -D feature/abandoned  # force

# Track a remote branch
git switch --track origin/feature/x
```

## Committing & amending

```bash
# Commit with message
git commit -m "feat: add user profile API"

# Add forgotten files to the previous commit (keeps message)
git commit --amend --no-edit

# Rewrite the last commit message
git commit --amend -m "fix: correct null check in profile API"
```

## History & inspection

```bash
# Pretty graph
git log --oneline --graph --decorate --all

# Who changed each line of a file
git blame path/to/file.ts

# Search commits by content
git log -S "functionName" --source --all

# Show what a commit touched
git show <sha> --stat
```

## Rebase, merge & cherry-pick

```bash
# Update feature branch with latest main
git fetch origin
git rebase origin/main

# Interactive rebase (squash/fix/reorder last N commits)
git rebase -i HEAD~5

# Merge without fast-forward (keep merge commit)
git merge --no-ff feature/my-change

# Bring a single commit from another branch
git cherry-pick <sha>
```

## Undoing things

```bash
# Undo last commit, keep changes staged
git reset --soft HEAD~1

# Undo last commit, keep changes in working tree (unstaged)
git reset --mixed HEAD~1

# Throw away last commit AND changes (DANGEROUS — non-recoverable if not pushed)
git reset --hard HEAD~1

# Revert a published commit (creates a new inverse commit — safe to share)
git revert <sha>

# Recover a "lost" commit
git reflog
git switch -c rescue <sha-from-reflog>
```

## Stash

```bash
git stash push -m "WIP: investigating bug"
git stash list
git stash show -p stash@{0}
git stash pop          # apply + drop
git stash apply        # apply, keep the stash
git stash drop stash@{0}
```

## Remotes & tags

```bash
git remote -v
git remote add upstream https://github.com/<org>/<repo>.git
git fetch --all --prune

# Annotated tag + push
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin v1.2.0
```

## Useful one-liners

```bash
# Files changed in current branch vs main
git diff --name-only origin/main...HEAD

# Last commit that touched a file
git log -1 --format="%h %an %s" -- path/to/file

# Remove a file from history (use with caution)
git filter-repo --path secrets.env --invert-paths
```
