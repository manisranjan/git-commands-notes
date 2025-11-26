# How to Move Commits from Wrong Branch to a New Branch

A comprehensive guide to fixing the common Git mistake of committing to the wrong branch.

---

## TL;DR - Quick Fix (Most Common Case)

**You committed to `main` but should have used a feature branch, and you haven't pushed yet:**

```bash
# 1. Create new branch (saves your commits)
git switch -c feature/my-feature

# 2. Go back to main
git switch main

# 3. Remove commits from main
git reset --hard origin/main

# Done! Continue working on feature/my-feature
```

**For older Git versions:** Replace `git switch` with `git checkout` and `git switch -c` with `git checkout -b`.

**Made a mistake?** Use `git reflog` to recover anything - see [Recovery Options](#recovery-options).

---

## Table of Contents
1. [Overview](#overview)
   - [Git Version Compatibility](#git-version-compatibility)
2. [Visual Example](#visual-example)
3. [Quick Decision Guide](#quick-decision-guide)
4. [Method 1: Create New Branch and Reset](#method-1-create-new-branch-and-reset-recommended) ⭐ Most Common
5. [Method 2: Cherry-Pick to New Branch](#method-2-cherry-pick-to-new-branch)
6. [Method 3: Interactive Rebase](#method-3-interactive-rebase)
7. [Common Scenarios](#common-scenarios)
   - [Uncommitted Changes](#scenario-a-uncommitted-changes-on-wrong-branch)
   - [All Recent Commits](#scenario-b-all-recent-commits-to-wrong-branch)
   - [Selective Commits](#scenario-c-want-only-some-commits-in-new-branch)
   - [Already Pushed](#scenario-e-already-pushed-to-remote)
8. [Recovery Options](#recovery-options) - Use reflog!
9. [Best Practices](#best-practices)
10. [Quick Reference](#quick-reference)
11. [Real-World Examples](#real-world-examples)
12. [Platform-Specific: Pull Requests](#platform-specific-creating-pull-requests)
13. [FAQ](#frequently-asked-questions)
14. [Troubleshooting](#troubleshooting-tips)

---

## Overview

### The Problem
You've made commits to the wrong branch and need to move them to a new branch without losing your work.

### Git Version Compatibility

This guide uses modern Git commands (`git switch`) introduced in Git 2.23 (August 2019).

**Check your Git version:**
```bash
git --version
```

**Command compatibility:**
- **Git 2.23+:** Use `git switch` (recommended)
- **Git < 2.23:** Use `git checkout` (legacy, still works in newer versions)

Both are shown throughout this guide.

### Before You Start
```bash
# Check your current status
git status
git log --oneline -10
git branch -a
```

---

## Visual Example

**The Problem:**
```
BEFORE (Oops! Committed to main instead of feature branch)

main branch:
  abc123 (origin/main) ← Synced with remote
  def456 ← Your commit A (wrong place!)
  ghi789 ← Your commit B (wrong place!)
  jkl012 ← Your commit C (wrong place!) ← HEAD
```

**The Solution:**
```
AFTER (Commits moved to correct branch)

main branch:
  abc123 (origin/main) ← HEAD (back to clean state)

feature/new-branch:
  abc123 ← Based on main
  def456 ← Your commit A ✓
  ghi789 ← Your commit B ✓
  jkl012 ← Your commit C ✓ ← HEAD
```

**How we get there:**
```bash
# 1. Create new branch (saves your commits)
git switch -c feature/new-branch

# 2. Go back to main
git switch main

# 3. Reset main to remove wrong commits
git reset --hard origin/main

# Done! Commits are now on feature/new-branch, main is clean
```

---

## Quick Decision Guide

**Choose your method based on your situation:**

```
Did you commit yet?
├─ NO → Use Method 0 (Stash)
└─ YES → Did you push?
    ├─ NO → Do you want all commits?
    │   ├─ YES → Use Method 1 (Create Branch + Reset) ⭐
    │   └─ NO → Use Method 3 (Interactive Rebase)
    └─ YES → Use Method 2 (Cherry-Pick)
```

**Quick recommendations:**
- **Not committed yet?** → [Stash Method (Scenario A)](#scenario-a-uncommitted-changes-on-wrong-branch)
- **Want all recent commits?** → [Method 1](#method-1-create-new-branch-and-reset-recommended) ⭐ Most common
- **Want selective commits?** → [Method 2](#method-2-cherry-pick-to-new-branch) or [Method 3](#method-3-interactive-rebase)
- **Already pushed?** → [Method 2](#method-2-cherry-pick-to-new-branch) + careful cleanup
- **Made a mistake?** → [Use reflog to recover](#recovery-options)

---

## Method 1: Create New Branch and Reset (Recommended)

**Best for:** When you made commits on the wrong branch and **haven't pushed yet**.

### Quick Steps

**1. Create a new branch at your current location:**
```bash
# Modern Git (2.23+)
git switch -c feature/my-correct-branch

# Older Git
git checkout -b feature/my-correct-branch
```

This new branch now contains all your commits.

**2. Switch back to the wrong branch:**
```bash
# Modern Git
git switch wrong-branch-name

# Older Git
git checkout wrong-branch-name
```

**3. Reset the wrong branch to its previous state:**
```bash
# Option A: Reset to match remote (if the branch was synced before)
git reset --hard origin/wrong-branch-name

# Option B: Go back N commits
git reset --hard HEAD~3  # replace 3 with number of wrong commits

# Option C: Reset to specific commit
git reset --hard <commit-hash>
```

**4. Push your new branch when ready:**
```bash
git push -u origin feature/my-correct-branch
```

### Why This Is Safe
- You haven't pushed, so no one else is affected
- All commits are preserved on the new branch
- The wrong branch is simply reset to its previous state

### Detailed Example

```bash
# Current state: accidentally committed to main
$ git log --oneline -3
abc1234 (HEAD -> main) Add feature C
def5678 Add feature B
ghi9012 Add feature A

# Step 1: Create new branch with these commits
$ git switch -c feature/awesome-feature
Switched to a new branch 'feature/awesome-feature'

# Step 2: Go back to wrong branch
$ git switch main
Switched to branch 'main'

# Step 3: Reset main to match remote
$ git reset --hard origin/main
HEAD is now at xyz7890 Previous main commit

# Step 4: Verify main is clean
$ git log --oneline -3
xyz7890 (HEAD -> main, origin/main) Previous main commit
# Your commits are gone from main

# Step 5: Verify new branch has commits
$ git switch feature/awesome-feature
$ git log --oneline -3
abc1234 (HEAD -> feature/awesome-feature) Add feature C
def5678 Add feature B
ghi9012 Add feature A

# Step 6: Push when ready
$ git push -u origin feature/awesome-feature
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

**2. Create and switch to a new branch from the correct base:**
```bash
# Start from the correct base branch
git switch main
# or: git checkout main

git pull origin main

# Create new branch
git switch -c feature-new-branch
# or: git checkout -b feature-new-branch
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
git switch wrong-branch
git reset --hard origin/wrong-branch
```

### Example

```bash
# View commits on wrong branch
$ git switch wrong-branch
$ git log --oneline -5
abc1234 Feature commit 3
def5678 Feature commit 2
ghi9012 Feature commit 1
jkl3456 (origin/wrong-branch) Previous work

# Create new branch from correct base
$ git switch main
$ git switch -c feature-correct-branch

# Cherry-pick the commits (in order from oldest to newest)
$ git cherry-pick ghi9012 def5678 abc1234

# Verify
$ git log --oneline -3
abc1234 (HEAD -> feature-correct-branch) Feature commit 3
def5678 Feature commit 2
ghi9012 Feature commit 1
```

---

## Method 3: Interactive Rebase

**Best for:** Complex scenarios where you need to reorganize, squash, or reorder commits.

### Steps

**1. Create a new branch:**
```bash
# Modern Git
git switch -c feature-new-branch

# Legacy Git
git checkout -b feature-new-branch
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
fixup pqr1234 Commit message 6 (squash without message)
```

**Interactive rebase commands:**
- `pick` = keep this commit as-is
- `drop` = remove this commit entirely
- `reword` = keep commit but edit the message
- `edit` = stop and amend this commit
- `squash` = combine with previous commit, keep both messages
- `fixup` = combine with previous commit, discard this message

**4. Save and close the editor**

**5. Resolve any conflicts if they occur**

### Example

```bash
# Create new branch
$ git switch -c feature-cleaned

# Start interactive rebase
$ git rebase -i HEAD~5

# Editor opens with:
pick 1a2b3c4 Add initial feature
pick 5d6e7f8 Fix typo
pick 9g0h1i2 Add tests
pick 3j4k5l6 Another typo fix
pick 7m8n9o0 Update documentation

# Modify to:
pick 1a2b3c4 Add initial feature
fixup 5d6e7f8 Fix typo
pick 9g0h1i2 Add tests
fixup 3j4k5l6 Another typo fix
pick 7m8n9o0 Update documentation

# Save and close - now you have 3 clean commits instead of 5
```

---

## Common Scenarios

### Scenario A: Uncommitted Changes on Wrong Branch

**Problem:** You're on the wrong branch but **haven't committed yet**.

**Solution: Use stash to move uncommitted changes**

```bash
# Save your uncommitted changes
git stash

# Switch to correct branch (or create it)
git switch feature/my-correct-branch
# or: git switch -c feature/my-correct-branch

# Apply the stashed changes
git stash pop
```

**Example:**
```bash
$ git status
On branch main
Changes not staged for commit:
  modified:   src/feature.js
  modified:   src/tests.js

# Stash changes
$ git stash
Saved working directory and index state WIP on main

# Switch to correct branch
$ git switch -c feature/new-feature

# Apply changes
$ git stash pop
On branch feature/new-feature
Changes not staged for commit:
  modified:   src/feature.js
  modified:   src/tests.js
```

---

### Scenario B: All Recent Commits to Wrong Branch

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
git switch -c feature-branch

# Reset main
git switch main
git reset --hard origin/main

# Continue on new branch
git switch feature-branch
```

---

### Scenario C: Want Only Some Commits in New Branch

**Problem:** You made multiple commits but only want some of them on the new branch.

**Solution: Use interactive rebase**

```bash
# On the wrong branch
git switch wrong-branch-name

# Start interactive rebase for last N commits
git rebase -i HEAD~5  # replace 5 with number of commits

# In the editor, mark each commit:
# - pick: keep this commit
# - drop: remove this commit
# - squash: combine with previous commit
# - reword: change commit message
```

**Then create new branch from specific commit:**
```bash
git switch -c feature/my-correct-branch <commit-sha>
```

**Example:**
```bash
# View commits
$ git log --oneline -5
abc1234 Commit E (want)
def5678 Commit D (don't want)
ghi9012 Commit C (want)
jkl3456 Commit B (don't want)
mno7890 Commit A (want)

# Interactive rebase
$ git rebase -i HEAD~5

# Editor shows:
pick mno7890 Commit A
drop jkl3456 Commit B
pick ghi9012 Commit C
drop def5678 Commit D
pick abc1234 Commit E

# Save and close - now branch only has A, C, E
```

---

### Scenario D: Mixed Commits (Some Wrong, Some Right)

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
git switch -c separate-feature <commit-before-B>
git cherry-pick <hash-of-B> <hash-of-C>

# Remove from original branch
git switch feature-branch
git rebase -i <commit-before-B>
# Mark B and C as 'drop'
```

---

### Scenario E: Already Pushed to Remote

**⚠️ Warning:** Only use force push if you're **absolutely certain** no one else is using these commits.

**Solution:**
```bash
# Create new branch with commits
git switch -c feature-correct-branch

# Reset local branch
git switch wrong-branch
git reset --hard origin/wrong-branch

# Force push if necessary (dangerous!)
git push origin wrong-branch --force

# Push new branch
git switch feature-correct-branch
git push origin feature-correct-branch
```

**Better approach if others are using the branch:**
```bash
# Use revert instead of reset
git switch wrong-branch
git revert <commit-hash-1> <commit-hash-2>
git push origin wrong-branch

# Work continues on new branch
git switch feature-correct-branch
git push origin feature-correct-branch
```

---

### Scenario F: Multiple Branches Affected

**Problem:**
```
develop
  └─ feature-1 (has wrong commits)
       └─ feature-2 (based on feature-1)
```

**Solution:**
```bash
# Fix feature-1 first
git switch feature-1
git branch feature-1-backup
git reset --hard origin/develop

# Rebase feature-2 onto corrected feature-1
git switch feature-2
git rebase --onto feature-1 feature-1-backup feature-2
```

---

## Recovery Options

### If You Accidentally Ran Wrong Commands - Use Reflog

**Git tracks every move you make** - nothing is truly lost for 30-90 days!

The `reflog` is your time machine for Git operations.

**View your complete history:**
```bash
git reflog
```

**Output example:**
```
abc1234 HEAD@{0}: reset: moving to origin/main
def5678 HEAD@{1}: commit: Add feature C
ghi9012 HEAD@{2}: commit: Add feature B
jkl3456 HEAD@{3}: commit: Add feature A
mno7890 HEAD@{4}: checkout: moving from main to feature-branch
```

**Recover to a previous state:**
```bash
# Option 1: Go back to specific reflog entry
git reset --hard HEAD@{2}

# Option 2: Use the commit hash directly
git reset --hard def5678
```

**Create a new branch from any point in history:**
```bash
# Rescue commits from reflog
git switch -c rescued-branch HEAD@{2}

# Or from specific commit
git switch -c rescued-branch def5678
```

**Example Recovery Scenario:**
```bash
# Oops! I reset too far back
$ git reset --hard origin/main
# Realized I lost important commits

# Check reflog
$ git reflog
abc1234 HEAD@{0}: reset: moving to origin/main
def5678 HEAD@{1}: commit: Important feature
ghi9012 HEAD@{2}: commit: Critical fix

# Rescue the commits
$ git switch -c rescued-work HEAD@{1}
Switched to a new branch 'rescued-work'

# Verify commits are back
$ git log --oneline -2
def5678 (HEAD -> rescued-work) Important feature
ghi9012 Critical fix
```

### Emergency Recovery Commands

```bash
# Find all dangling commits
git fsck --lost-found

# View specific commit from reflog
git show HEAD@{5}

# See reflog for specific branch
git reflog show branch-name

# Reflog for all branches
git reflog show --all
```

---

## Best Practices

### 0. Team Hygiene - Work on Feature Branches

**Golden Rule:** Always work on a fresh branch per feature/fix and keep main branches clean.

**Recommended Workflow:**
```bash
# Start from clean main/develop
git switch main
git pull origin main

# Create feature branch
git switch -c feature/descriptive-name

# Work, commit, push
git add .
git commit -m "Implement feature"
git push -u origin feature/descriptive-name

# Open pull request for review
# Merge via PR after approval
```

**Why This Matters:**
- ✅ Main branches stay stable and deployable
- ✅ Easy to isolate and test features
- ✅ Simple to abandon or revise work
- ✅ Clean history for code review
- ✅ No accidental commits to wrong branch

**Common Platforms:**
- **GitHub:** Create Pull Request
- **GitLab:** Create Merge Request  
- **Azure DevOps:** Create Pull Request
- **Bitbucket:** Create Pull Request

---

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

### Create and Switch Branches
```bash
# Modern Git (2.23+) - recommended
git switch -c new-branch           # Create and switch
git switch existing-branch         # Switch to existing branch
git switch -                       # Switch to previous branch

# Legacy Git (still works)
git checkout -b new-branch         # Create and switch
git checkout existing-branch       # Switch to existing branch
git checkout -                     # Switch to previous branch

# Create without switching
git branch new-branch              # Create from current position
git branch new-branch <commit>     # Create from specific commit
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

| Method | Best For | When to Use | Difficulty | Safety |
|--------|----------|-------------|------------|--------|
| Stash | Uncommitted changes | Haven't committed yet | Easy | Very High |
| Create Branch + Reset | All recent commits | Haven't pushed | Easy | High |
| Cherry-Pick | Selective commits | Already pushed | Medium | High |
| Interactive Rebase | Reorganize/clean commits | Complex history | Hard | Medium |

### Essential Commands Quick Reference

```bash
# Most common scenario: committed to wrong branch, not pushed
git switch -c feature/correct-branch    # Save commits to new branch
git switch wrong-branch                  # Go back to wrong branch
git reset --hard origin/wrong-branch    # Remove commits from wrong branch

# Haven't committed yet
git stash                               # Save changes
git switch correct-branch               # Switch branch
git stash pop                           # Apply changes

# Made a mistake
git reflog                              # Find your commits
git switch -c rescue <commit-hash>      # Rescue them
```

**Remember:**
1. Nothing is ever truly lost - use `git reflog` to recover
2. Always work on feature branches, not main/develop directly
3. Create backups before destructive operations (`git branch backup-$(date +%s)`)
4. Verify changes before pushing (`git log --oneline`)
5. Communicate with team if working on shared branches
6. When in doubt, create a backup branch first

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
git switch main
git pull origin main
git switch -c feature/new-feature

# Work and commit
git add .
git commit -m "Implement feature"

# Keep up to date with main
git fetch origin
git rebase origin/main

# Push feature
git push -u origin feature/new-feature

# Create pull request on GitHub/GitLab/Azure DevOps
```

### Hotfix Workflow
```bash
# Create hotfix from production
git switch main
git pull origin main
git switch -c hotfix/critical-fix

# Fix and commit
git add .
git commit -m "Fix critical bug"

# Test thoroughly

# Merge back (via PR or direct)
git switch main
git merge hotfix/critical-fix
git push origin main

# Clean up
git branch -d hotfix/critical-fix
```

### Daily Work Workflow
```bash
# Start your day
git switch main
git pull origin main

# Create daily work branch
git switch -c feature/my-task-$(date +%Y%m%d)

# Work throughout the day
git add .
git commit -m "Progress on feature"

# End of day - push to remote
git push -u origin feature/my-task-$(date +%Y%m%d)

# Next day - continue
git switch feature/my-task-20251126
git pull origin feature/my-task-20251126
# Continue working...
```

---

## Platform-Specific: Creating Pull Requests

After moving commits to the correct branch, you'll typically want to create a pull request.

### GitHub
```bash
# Push your branch
git push -u origin feature/my-branch

# Create PR via CLI (requires GitHub CLI)
gh pr create --title "Add new feature" --body "Description"

# Or visit: https://github.com/your-org/your-repo/pull/new/feature/my-branch
```

### GitLab
```bash
# Push your branch
git push -u origin feature/my-branch

# Create MR via CLI (requires glab)
glab mr create --title "Add new feature" --description "Description"

# Or visit: https://gitlab.com/your-org/your-repo/-/merge_requests/new?merge_request[source_branch]=feature/my-branch
```

### Azure DevOps
```bash
# Push your branch
git push -u origin feature/my-branch

# Create PR via CLI (requires Azure CLI)
az repos pr create --title "Add new feature" --description "Description"

# Or visit: https://dev.azure.com/your-org/your-project/_git/your-repo/pullrequestcreate?sourceRef=feature/my-branch
```

### Bitbucket
```bash
# Push your branch
git push -u origin feature/my-branch

# Visit: https://bitbucket.org/your-org/your-repo/pull-requests/new?source=feature/my-branch
```

---

## Real-World Examples

### Example 1: Junior Developer's First Day

**Situation:** New developer commits directly to `develop` branch.

```bash
# Oops - made commits to develop
$ git branch
* develop

$ git log --oneline -3
abc1234 Add my feature
def5678 Fix bug
ghi9012 Update tests

# Fix it
$ git switch -c feature/my-first-feature
$ git switch develop
$ git reset --hard origin/develop
$ git switch feature/my-first-feature
$ git push -u origin feature/my-first-feature

# Now create PR from feature/my-first-feature → develop
```

---

### Example 2: Emergency Hotfix on Wrong Branch

**Situation:** Critical bug fix committed to `feature` branch instead of `hotfix` branch.

```bash
# Currently on feature/new-dashboard
$ git log --oneline -1
abc1234 Fix critical security vulnerability

# This needs to go to production NOW!
# Create hotfix branch from main
$ git switch main
$ git pull origin main
$ git switch -c hotfix/security-fix

# Cherry-pick the fix
$ git cherry-pick abc1234

# Push and deploy
$ git push -u origin hotfix/security-fix
# Create PR: hotfix/security-fix → main (priority)

# Also need in develop
$ git switch develop
$ git cherry-pick abc1234
$ git push origin develop
```

---

### Example 3: Forgot to Create Branch

**Situation:** Started working directly on `main`, realized after several commits.

```bash
# Realized after 5 commits on main
$ git log --oneline -5
aaa1111 Commit 5
bbb2222 Commit 4  
ccc3333 Commit 3
ddd4444 Commit 2
eee5555 Commit 1

# All these should be on a feature branch
$ git switch -c feature/forgot-to-branch
$ git switch main
$ git reset --hard origin/main

# Verify
$ git switch feature/forgot-to-branch
$ git log --oneline -5
# All 5 commits are here ✓
```

---

### Example 4: Mixed Work on Single Branch

**Situation:** Two unrelated features on one branch, need to split them.

```bash
# Current branch has mixed commits
$ git log --oneline -6
aaa1111 Feature B: part 3
bbb2222 Feature A: part 2
ccc3333 Feature B: part 2
ddd4444 Feature A: part 1
eee5555 Feature B: part 1
fff6666 (origin/main) Base

# Split into two feature branches
# Feature A (commits ddd4444, bbb2222)
$ git switch main
$ git switch -c feature/feature-a
$ git cherry-pick ddd4444 bbb2222

# Feature B (commits eee5555, ccc3333, aaa1111)
$ git switch main
$ git switch -c feature/feature-b
$ git cherry-pick eee5555 ccc3333 aaa1111

# Clean up original branch
$ git switch original-branch
$ git reset --hard origin/main

# Now you have two clean feature branches
```

---

## Frequently Asked Questions

### Q: Will I lose my commits?
**A:** No! Git keeps everything in reflog for 30-90 days. Use `git reflog` to recover.

### Q: What if I already pushed to the wrong branch?
**A:** If no one else has pulled your commits, you can force push after resetting. Otherwise, use cherry-pick to copy commits to the correct branch and leave the wrong branch as-is (or revert).

### Q: Can I move commits between unrelated branches?
**A:** Yes! Use cherry-pick to copy commits from any branch to any other branch.

### Q: What's the difference between `git switch` and `git checkout`?
**A:** `git switch` is newer (Git 2.23+) and clearer - it only switches branches. `git checkout` has multiple purposes which can be confusing. Both work the same for switching branches.

### Q: How do I know which commits to move?
**A:** Use `git log --oneline` to see commit history. Look for where `origin/branch-name` appears - that's where the remote is. Commits after that are local only.

### Q: What if I have merge conflicts?
**A:** During cherry-pick or rebase, resolve conflicts in files, then:
```bash
git add .
git cherry-pick --continue
# or
git rebase --continue
```

### Q: Can I undo a branch deletion?
**A:** Yes! Use `git reflog` to find the commit, then:
```bash
git switch -c recovered-branch <commit-hash>
```

---

## Additional Tools

### Git Aliases for Faster Workflow

Add these to `~/.gitconfig`:

```ini
[alias]
    # Quick branch creation and switching
    sw = switch
    swc = switch -c
    
    # Visual log
    lg = log --oneline --graph --all --decorate
    
    # Show recent branches
    recent = branch --sort=-committerdate
    
    # Undo last commit (keep changes)
    undo = reset --soft HEAD~1
    
    # Save WIP
    save = stash push -m
    
    # List stashes
    stashes = stash list
    
    # Show what's on current branch vs main
    new = log --oneline main..HEAD
```

Usage:
```bash
git sw main              # switch to main
git swc feature/new      # create and switch to new branch
git lg                   # visual history
git recent               # see recent branches
git undo                 # undo last commit
git save "WIP: feature"  # stash with message
```

---

## Troubleshooting Tips

### Branch shows as "up to date" but has wrong commits
```bash
# Force git to check remote
git fetch --all
git status
```

### Can't switch branches - "uncommitted changes"
```bash
# Stash them first
git stash
git switch other-branch
git stash pop
```

### Accidentally deleted local changes
```bash
# Check reflog immediately
git reflog
git switch -c recovery HEAD@{1}
```

### Branch doesn't exist on remote
```bash
# Push with upstream flag
git push -u origin branch-name
```

---

## License

This document is provided as-is for educational purposes.

## Contributing

Feel free to suggest improvements or report issues with this guide.

## Last Updated

November 26, 2025

