---
title: "Git Advanced"
description: "Power-user Git: interactive rebase, bisect, worktrees, partial commits, submodules, history rewriting, and recovery."
category: "Git"
---

## Interactive rebase recipes

```bash
# Squash the last N commits into one (keeps the first message)
git rebase -i HEAD~N
#   pick   abc123 first commit
#   squash def456 second commit   (or 'fixup' to drop the message)
#   squash 789abc third commit

# Reorder, drop, or reword commits — change the verb in front of each line
#   pick    -> keep as-is
#   reword  -> keep the change, edit the message
#   edit    -> stop so you can amend the working tree
#   squash  -> combine into previous (keeps messages)
#   fixup   -> combine into previous (discards this message)
#   drop    -> remove the commit entirely

# Autosquash workflow: mark commits as fixups inline, then a single rebase
git commit --fixup=<sha>
git commit --squash=<sha>
git rebase -i --autosquash <base>
```

## Bisect (find the offending commit)

```bash
git bisect start
git bisect bad                       # current HEAD is broken
git bisect good v1.4.0               # known-good ref
# Git checks out the midpoint — test, then mark:
git bisect good   # or:  git bisect bad
# Repeat until git prints the first bad commit. Then:
git bisect reset

# Fully automated: let a test script decide
git bisect run npm test
```

## Worktrees (multiple branches checked out at once)

```bash
# Add a parallel working copy of another branch
git worktree add ../repo-hotfix hotfix/urgent-bug

# List + remove
git worktree list
git worktree remove ../repo-hotfix
git worktree prune
```

Great for: reviewing a PR while keeping your feature branch dirty; running long builds on one branch while coding on another.

## Reflog: undo almost anything

```bash
# See every HEAD movement (commits, resets, rebases, checkouts)
git reflog

# Restore the branch to where it was 2 moves ago
git reset --hard HEAD@{2}

# Recover a deleted branch
git branch rescue <sha-from-reflog>

# Reflog for a specific ref
git reflog show feature/x
```

## Partial commits & selective staging

```bash
# Interactively stage hunks of a file
git add -p path/to/file

# Stage a specific hunk and skip the rest by lines
git add -e path/to/file              # opens the diff in $EDITOR

# Commit only some staged files
git commit path/to/specific/file -m "fix: only this file"

# Restore one path from another branch/commit
git restore --source=origin/main --staged --worktree path/to/file
```

## Rebase onto another base

```bash
# Move commits from feature/A onto main, dropping the old base feature/B
#   git rebase --onto <newbase> <oldbase> <branch>
git rebase --onto main feature/B feature/A

# After upstream rebases their branch, rebase your work cleanly
git pull --rebase=interactive
```

## Submodules

```bash
# Add and initialize
git submodule add https://github.com/<org>/<lib>.git vendor/lib
git submodule update --init --recursive

# Pull updates for all submodules
git submodule update --remote --merge

# Clone a repo + all submodules in one shot
git clone --recurse-submodules <url>

# Run a command in every submodule
git submodule foreach 'git checkout main && git pull'
```

## History surgery

```bash
# Modern replacement for filter-branch (install separately: git-filter-repo)
#   Remove a path from ALL history
git filter-repo --path secrets/ --invert-paths

#   Rewrite committer/author emails across history
git filter-repo --email-callback '
  return email.replace(b"old@corp.com", b"new@corp.com")
'

# Show what diverged between two branches with a 3-dot diff
git log --left-right --graph --oneline main...feature/x
```

> WARNING: rewriting published history forces every collaborator to re-clone or hard-reset. Coordinate before pushing with `--force-with-lease`.

## Safer force-push

```bash
# Refuses to overwrite remote work you haven't seen — always prefer this over --force
git push --force-with-lease

# Even safer: also require the remote tip to match what you expect
git push --force-with-lease=origin/feature/x:<expected-sha>
```

## Blame & archeology

```bash
# Ignore mass-formatting commits in blame output
git config blame.ignoreRevsFile .git-blame-ignore-revs

# Follow renames when blaming
git log --follow -p path/to/file

# Find when a string was introduced
git log -S "EXACT_STRING" --source --all -p

# Find when a regex first/last matched
git log -G "regex.*pattern" --all
```

## Hooks & local automation

```bash
# Use a tracked hooks directory (share hooks across the team)
git config core.hooksPath .githooks

# Run hooks for an in-progress merge or rebase only
ls .git/hooks/

# Skip hooks for one command (use sparingly)
git commit --no-verify
```

## Sparse checkout (huge monorepos)

```bash
git clone --filter=blob:none --no-checkout <url> repo
cd repo
git sparse-checkout init --cone
git sparse-checkout set apps/web libs/shared
git checkout main
```

## Cherry-pick ranges and resolve cleanly

```bash
# Pick a range (exclusive..inclusive)
git cherry-pick A..B

# Pick a merge commit (specify which parent's diff to apply)
git cherry-pick -m 1 <merge-sha>

# Continue after resolving conflicts
git cherry-pick --continue
git cherry-pick --abort
```

## Diff tricks

```bash
# Word-level diff (great for prose)
git diff --word-diff

# Diff with rename detection turned up
git diff -M90% --stat

# Diff between two arbitrary refs ignoring whitespace
git diff -w main...feature/x

# Generate a patch series to share by email or apply elsewhere
git format-patch origin/main..HEAD -o patches/
git am patches/*.patch
```

## Performance & maintenance

```bash
# Aggressive repack + prune (run periodically on long-lived repos)
git gc --aggressive --prune=now

# Verify integrity
git fsck --full

# Enable the background maintenance daemon (Git 2.30+)
git maintenance start
```

## Useful aliases

```bash
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.st "status -sb"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.unstage "restore --staged"
git config --global alias.wip "!git add -A && git commit -m 'WIP' --no-verify"
```
