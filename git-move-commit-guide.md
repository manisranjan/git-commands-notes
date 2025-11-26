# How to Move Commits from Wrong Branch to a New Branch

A comprehensive guide to fixing the common Git mistake of committing to the wrong branch.

## Table of Contents
- [Overview](#overview)
- [Method 1: Create New Branch and Reset](#method-1-create-new-branch-and-reset-recommended)
- [Method 2: Cherry-Pick to New Branch](#method-2-cherry-pick-to-new-branch)
- [Method 3: Interactive Rebase](#method-3-interactive-rebase)
- [Common Scenarios](#common-scenarios)
- [Recovery Options](#recovery-options)
- [Best Practices](#best-practices)
- [Quick Reference](#quick-reference)

---

## Overview

### The Problem
You've made commits to the wrong branch and need to move them to a new branch without losing your work.

### Before You Start
```bash
# Check your current status
git status
git log --oneline -10
git branch -a
```

---

## Method 1: Create New Branch and Reset (Recommended)

**Best for:** When you haven't pushed yet and want to move all recent commits.

### Steps

**1. View your current commits:**
```bash
git log --oneline -10
```

**2. Create a new branch from your current position:**
```bash
# This preserves all your commits on the new branch
git branch feature-new-branch
```

**3. Verify the new branch was created:**
```bash
git branch
```

**4. Reset the current branch to remove unwanted commits:**
```bash
# Option A: Reset to a specific commit
git reset --hard <commit-hash>

# Option B: Reset to match remote
git reset --hard origin/main

# Option C: Reset by count (go back 3 commits)
git reset --hard HEAD~3
```

**5. Switch to your new branch:**
```bash
git checkout feature-new-branch
```

**6. Verify everything is correct:**
```bash
git log --oneline
git status
```

### Example

```bash
# Current state: accidentally committed to main
$ git log --oneline -3
abc1234 (HEAD -> main) Add feature C
def5678 Add feature B
ghi9012 Add feature A

# Create new branch with these commits
$ git branch feature-awesome

# Reset main to before your commits
$ git reset --hard origin/main

# Switch to new branch
$ git checkout feature-awesome

# Verify commits are there
$ git log --oneline -3
abc1234 (HEAD -> feature-awesome) Add feature C
def5678 Add feature B
ghi9012 Add feature A
```

---

## Method 2: Cherry-Pick to New Branch

**Best for:** When you need selective commits or already pushed to the wrong branch.

### Steps

**1. Identify the commits you want to move:**
```bash
git log --oneline
# Note the commit hashes you need
```

**2. Create and checkout a new branch from the correct base:**
```bash
# Start from the correct base branch
git checkout main
git pull origin main

# Create new branch
git checkout -b feature-new-branch
```

**3. Cherry-pick the commits:**
```bash
# Single commit
git cherry-pick abc1234

# Multiple specific commits
git cherry-pick abc1234 def5678 ghi9012

# Range of commits (exclusive start, inclusive end)
git cherry-pick abc1234^..ghi9012
```

**4. Clean up the original branch (if needed):**
```bash
git checkout wrong-branch
git reset --hard origin/wrong-branch
```

### Example

```bash
# View commits on wrong branch
$ git checkout wrong-branch
$ git log --oneline -5
abc1234 Feature commit 3
def5678 Feature commit 2
ghi9012 Feature commit 1
jkl3456 (origin/wrong-branch) Previous work

# Create new branch from correct base
$ git checkout main
$ git checkout -b feature-correct-branch

# Cherry-pick the commits
$ git cherry-pick ghi9012 def5678 abc1234

# Verify
$ git log --oneline -3
abc1234 (HEAD -> feature-correct-branch) Feature commit 3
def5678 Feature commit 2
ghi9012 Feature commit 1
```

---

## Method 3: Interactive Rebase

**Best for:** Complex scenarios where you need to reorganize commits.

### Steps

**1. Create a new branch:**
```bash
git branch feature-new-branch
git checkout feature-new-branch
```

**2. Use interactive rebase:**
```bash
# Rebase last 5 commits
git rebase -i HEAD~5
```

**3. In the editor, mark commits:**
```bash
pick abc1234 Commit message 1
drop def5678 Commit message 2 (remove this)
pick ghi9012 Commit message 3
squash jkl3456 Commit message 4 (combine with previous)
reword mno7890 Commit message 5 (edit message)
```

**4. Save and close the editor**

**5. Resolve any conflicts if they occur**

---

## Common Scenarios

### Scenario A: All Recent Commits to Wrong Branch

**Problem:**
```
main (should be clean)
  ├─ Commit A (wrong)
  ├─ Commit B (wrong)
  └─ Commit C (wrong)
```

**Solution:**
```bash
# Create branch with commits
git branch feature-branch

# Reset main
git checkout main
git reset --hard origin/main

# Continue on new branch
git checkout feature-branch
```

---

### Scenario B: Mixed Commits (Some Wrong, Some Right)

**Problem:**
```
feature-branch
  ├─ Commit A (correct)
  ├─ Commit B (belongs elsewhere)
  ├─ Commit C (belongs elsewhere)
  └─ Commit D (correct)
```

**Solution:**
```bash
# Create new branch for misplaced commits
git checkout -b separate-feature <commit-before-B>
git cherry-pick <hash-of-B> <hash-of-C>

# Remove from original branch
git checkout feature-branch
git rebase -i <commit-before-B>
# Mark B and C as 'drop'
```

---

### Scenario C: Already Pushed to Remote

**⚠️ Warning:** Only use force push if you're sure no one else is using these commits.

**Solution:**
```bash
# Create new branch with commits
git branch feature-correct-branch

# Reset local branch
git checkout wrong-branch
git reset --hard origin/wrong-branch

# Force push if necessary (dangerous!)
git push origin wrong-branch --force

# Push new branch
git checkout feature-correct-branch
git push origin feature-correct-branch
```

**Better approach if others are using the branch:**
```bash
# Use revert instead of reset
git checkout wrong-branch
git revert <commit-hash-1> <commit-hash-2>
git push origin wrong-branch

# Work continues on new branch
git checkout feature-correct-branch
git push origin feature-correct-branch
```

---

### Scenario D: Multiple Branches Affected

**Problem:**
```
develop
  └─ feature-1 (has wrong commits)
       └─ feature-2 (based on feature-1)
```

**Solution:**
```bash
# Fix feature-1 first
git checkout feature-1
git branch feature-1-backup
git reset --hard origin/develop

# Rebase feature-2 onto corrected feature-1
git checkout feature-2
git rebase --onto feature-1 feature-1-backup feature-2
```

---

## Recovery Options

### If You Made a Mistake

Git keeps a safety net of all operations for 30-90 days.

**Use reflog to view all operations:**
```bash
git reflog
```

**Output example:**
```
abc1234 HEAD@{0}: reset: moving to origin/main
def5678 HEAD@{1}: commit: Add feature
ghi9012 HEAD@{2}: commit: Add another feature
jkl3456 HEAD@{3}: checkout: moving from main to feature-branch
```

**Recover to a previous state:**
```bash
# Go back to specific reflog entry
git reset --hard HEAD@{2}

# Or use the commit hash
git reset --hard def5678
```

**Create a branch from a reflog entry:**
```bash
git branch recovery-branch HEAD@{2}
```

---

## Best Practices

### 1. Create Backups Before Destructive Operations

**Create a backup branch:**
```bash
git branch backup-$(date +%Y%m%d-%H%M%S)
```

**Create a patch file:**
```bash
# For uncommitted changes
git diff > backup.patch

# For committed changes
git diff origin/main HEAD > backup.patch

# For specific commits
git format-patch -1 <commit-hash>
```

**Apply patch later:**
```bash
git apply backup.patch
```

---

### 2. Verify Before Pushing

```bash
# Check current branch
git branch

# Review commits
git log --oneline origin/main..HEAD

# Review changes
git diff origin/main..HEAD

# Dry-run push
git push --dry-run origin feature-branch
```

---

### 3. Use Descriptive Branch Names

**Good:**
- `feature/user-authentication`
- `bugfix/login-error`
- `refactor/database-queries`
- `hotfix/security-patch`

**Bad:**
- `temp`
- `test`
- `new-branch`
- `fix`

---

### 4. Commit Frequently with Clear Messages

```bash
# Good commit message format
git commit -m "Add user authentication

- Implement login endpoint
- Add password hashing
- Create session management
- Add input validation"
```

---

### 5. Keep Branches Up to Date

```bash
# Update your base branch regularly
git checkout main
git pull origin main

# Rebase your feature branch
git checkout feature-branch
git rebase main
```

---

## Quick Reference

### View History
```bash
git log --oneline -10              # Last 10 commits
git log --oneline --graph --all    # Visual branch history
git reflog                         # All operations history
git show <commit-hash>             # Details of specific commit
```

### Create Branches
```bash
git branch new-branch              # Create from current position
git branch new-branch <commit>     # Create from specific commit
git checkout -b new-branch         # Create and switch
```

### Move/Copy Commits
```bash
git cherry-pick <commit-hash>      # Copy single commit
git cherry-pick A^..C              # Copy range of commits
git reset --hard <commit-hash>     # Move branch pointer
git rebase -i HEAD~5               # Reorganize last 5 commits
```

### Undo Operations
```bash
git reset --soft HEAD~1            # Undo commit, keep changes staged
git reset --mixed HEAD~1           # Undo commit, keep changes unstaged
git reset --hard HEAD~1            # Undo commit, discard changes
git revert <commit-hash>           # Create new commit undoing changes
```

### Remote Operations
```bash
git push origin branch-name        # Push branch
git push origin :branch-name       # Delete remote branch
git push origin --force            # Force push (dangerous!)
git push origin --force-with-lease # Safer force push
```

### Recovery
```bash
git reflog                         # View all operations
git reset --hard HEAD@{2}          # Restore to reflog entry
git branch recovery HEAD@{2}       # Create branch from reflog
git fsck --lost-found              # Find dangling commits
```

---

## Verification Checklist

After moving commits, verify:

- [ ] New branch has the correct commits
  ```bash
  git checkout new-branch
  git log --oneline
  ```

- [ ] Original branch is clean
  ```bash
  git checkout original-branch
  git log --oneline
  git diff origin/original-branch
  ```

- [ ] No uncommitted changes
  ```bash
  git status
  ```

- [ ] Remote state is correct
  ```bash
  git fetch origin
  git log --oneline origin/branch-name
  ```

- [ ] Builds/tests pass
  ```bash
  # Run your project's test suite
  ```

---

## Additional Resources

### Visualizing Branches
```bash
# Install gitk (comes with Git)
gitk --all

# Or use command line visualization
git log --oneline --graph --all --decorate
```

### Aliases for Common Operations
```bash
# Add to ~/.gitconfig
[alias]
    lg = log --oneline --graph --all --decorate
    undo = reset --soft HEAD~1
    unstage = reset HEAD
    branches = branch -a
    remotes = remote -v
```

### Safety Settings
```bash
# Prevent force push to main/master
git config branch.main.pushRemote no_push

# Always show diff when committing
git config --global commit.verbose true

# Set default pull behavior
git config --global pull.rebase true
```

---

## Summary

| Method | Best For | Difficulty | Safety |
|--------|----------|------------|--------|
| Create Branch + Reset | Moving all recent commits | Easy | High |
| Cherry-Pick | Selective commits | Medium | High |
| Interactive Rebase | Complex reorganization | Hard | Medium |

**Remember:**
1. Always create backups before destructive operations
2. Use `git reflog` if you make a mistake
3. Verify changes before pushing
4. Communicate with team if working on shared branches
5. When in doubt, create a backup branch first

---

## Troubleshooting

### "Cannot create branch - already exists"
```bash
# Delete existing branch first
git branch -d old-branch-name

# Or force delete
git branch -D old-branch-name
```

### "Detached HEAD state"
```bash
# Create a branch from current position
git branch new-branch
git checkout new-branch
```

### "Merge conflicts during cherry-pick"
```bash
# Resolve conflicts in files, then:
git add .
git cherry-pick --continue

# Or abort
git cherry-pick --abort
```

### "Lost commits after reset"
```bash
# Use reflog to find them
git reflog
git branch recovery-branch <commit-hash>
```

### "Cannot reset - uncommitted changes"
```bash
# Stash changes first
git stash
git reset --hard <commit-hash>
git stash pop
```

---

## Common Git Workflows

### Feature Branch Workflow
```bash
# Start new feature
git checkout main
git pull origin main
git checkout -b feature/new-feature

# Work and commit
git add .
git commit -m "Implement feature"

# Keep up to date with main
git fetch origin
git rebase origin/main

# Push feature
git push origin feature/new-feature
```

### Hotfix Workflow
```bash
# Create hotfix from production
git checkout main
git pull origin main
git checkout -b hotfix/critical-fix

# Fix and commit
git add .
git commit -m "Fix critical bug"

# Merge back
git checkout main
git merge hotfix/critical-fix
git push origin main

# Clean up
git branch -d hotfix/critical-fix
```

---

## License

This document is provided as-is for educational purposes.

## Contributing

Feel free to suggest improvements or report issues with this guide.

## Last Updated

November 26, 2025

