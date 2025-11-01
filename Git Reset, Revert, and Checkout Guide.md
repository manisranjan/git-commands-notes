# Git Reset, Revert, and Checkout Guide

## Table of Contents
- [Introduction](#introduction)
- [Git Reset](#git-reset)
  - [What is Git Reset?](#what-is-git-reset)
  - [Reset Modes](#reset-modes)
  - [Reset Examples](#reset-examples)
  - [Reset Use Cases](#reset-use-cases)
- [Git Revert](#git-revert)
  - [What is Git Revert?](#what-is-git-revert)
  - [Revert vs Reset](#revert-vs-reset)
  - [Revert Examples](#revert-examples)
  - [Revert Use Cases](#revert-use-cases)
- [Git Checkout](#git-checkout)
  - [What is Git Checkout?](#what-is-git-checkout)
  - [Checkout Modes](#checkout-modes)
  - [Checkout Examples](#checkout-examples)
  - [Checkout Use Cases](#checkout-use-cases)
- [Comparison Table](#comparison-table)
- [Real-World Scenarios](#real-world-scenarios)
- [Best Practices](#best-practices)
- [Common Pitfalls](#common-pitfalls)
- [Recovery Techniques](#recovery-techniques)

---

## Introduction

Git provides several commands to undo changes or move between different states of your repository: `reset`, `revert`, and `checkout`. While they may seem similar, each serves distinct purposes and has different implications for your commit history and working directory.

**Quick Overview:**
- **`git reset`** - Moves branch pointers and optionally changes staging area and working directory
- **`git revert`** - Creates new commits that undo previous commits (safe for shared branches)
- **`git checkout`** - Switches branches or restores files from commits

Understanding when to use each command is crucial for maintaining a clean and safe Git workflow.

---

## Git Reset

### What is Git Reset?

`git reset` moves the current branch pointer to a specified commit, effectively "resetting" your branch to that point in history. It can also modify the staging area (index) and working directory depending on the mode used.

**Key Characteristics:**
- Rewrites history by moving branch pointers
- Dangerous on shared/public branches
- Three modes: `--soft`, `--mixed`, `--hard`
- Can operate on specific files or entire commits
- Cannot be undone easily (use reflog for recovery)

**Basic Syntax:**
```bash
git reset [mode] [commit]
git reset [mode] [commit] -- <file>
```

---

### Reset Modes

#### 1. Soft Reset (`--soft`)

Moves the branch pointer but keeps staging area and working directory unchanged.

```bash
git reset --soft HEAD~1
```

**What happens:**
- ✅ Branch pointer moves to previous commit
- ✅ Staging area keeps all changes
- ✅ Working directory unchanged
- ✅ Previous commit's changes appear as staged

**Use when:**
- You want to re-commit with a different message
- You want to combine multiple commits into one
- You made a commit too early and want to add more changes

**Example:**
```bash
# You accidentally committed too early
git add file1.txt
git commit -m "Add feature"

# Oops! Forgot to add file2.txt
git reset --soft HEAD~1

# Now add the missing file
git add file2.txt

# Commit everything together
git commit -m "Add complete feature with all files"
```

---

#### 2. Mixed Reset (`--mixed`, default)

Moves the branch pointer and resets staging area, but keeps working directory unchanged.

```bash
git reset HEAD~1
# OR
git reset --mixed HEAD~1
```

**What happens:**
- ✅ Branch pointer moves to specified commit
- ✅ Staging area is reset
- ✅ Working directory keeps all changes (unstaged)
- ✅ Previous commit's changes appear as unstaged modifications

**Use when:**
- You want to unstage files
- You want to reorganize commits
- You want to split one commit into multiple commits

**Example:**
```bash
# Unstage a specific file
git add file1.txt file2.txt file3.txt
git reset HEAD file3.txt  # Only file3.txt is unstaged

# Unstage all files
git reset HEAD

# Undo last commit but keep changes
git reset HEAD~1
# Now you can stage and commit files separately
git add file1.txt
git commit -m "First logical change"
git add file2.txt
git commit -m "Second logical change"
```

---

#### 3. Hard Reset (`--hard`)

⚠️ **DANGER**: Moves branch pointer, resets staging area, AND discards all changes in working directory.

```bash
git reset --hard HEAD~1
```

**What happens:**
- ❌ Branch pointer moves to specified commit
- ❌ Staging area is completely cleared
- ❌ Working directory is reset to match commit
- ❌ ALL uncommitted changes are permanently lost

**Use when:**
- You want to completely discard all local changes
- You want to match your local branch to a remote branch
- You're absolutely sure you don't need the changes

**Example:**
```bash
# Discard all local changes and uncommitted work
git reset --hard HEAD

# Go back 3 commits and discard everything
git reset --hard HEAD~3

# Match local branch to remote (after fetch)
git fetch origin
git reset --hard origin/main

# Discard merge and all changes
git reset --hard HEAD~1  # if merge commit is the last commit
```

---

### Reset Examples

#### Example 1: Undo Last Commit (Keep Changes)

```bash
# Scenario: You committed but want to modify the commit
git log --oneline
# abc1234 Fix bug (HEAD)
# def5678 Add feature
# 789ghij Previous work

git reset --soft HEAD~1

# Now the changes from abc1234 are staged
git status
# Changes to be committed:
#   modified: bug_fix.php

# Modify files if needed
echo "// Additional fix" >> bug_fix.php

# Commit again with better message
git add bug_fix.php
git commit -m "Fix critical bug in payment module"
```

---

#### Example 2: Unstage Files

```bash
# Scenario: You staged too many files
git add .
git status
# Changes to be committed:
#   modified: important.php
#   modified: debug_temp.php
#   modified: test_file.php

# Unstage specific files
git reset HEAD debug_temp.php test_file.php

git status
# Changes to be committed:
#   modified: important.php
# Changes not staged for commit:
#   modified: debug_temp.php
#   modified: test_file.php
```

---

#### Example 3: Completely Discard Local Work

```bash
# Scenario: Your local changes are a mess, start fresh
git status
# Modified files: 15
# Untracked files: 8

# Nuclear option - discard everything
git reset --hard HEAD
git clean -fd  # Remove untracked files

# Verify clean state
git status
# nothing to commit, working tree clean
```

---

#### Example 4: Move Branch to Different Commit

```bash
# View commit history
git log --oneline
# abc1234 Wrong commit (HEAD)
# def5678 Another wrong commit
# 789ghij Good commit
# 012jklm Base commit

# Move branch back to "Good commit"
git reset --hard 789ghij

git log --oneline
# 789ghij Good commit (HEAD)
# 012jklm Base commit

# The "wrong commits" are gone from history
# (but can be recovered via reflog if needed)
```

---

### Reset Use Cases

#### Use Case 1: Squashing Commits Locally

**Scenario:** You made 5 small commits while developing a feature, but want to combine them into one clean commit before pushing.

```bash
# Current state
git log --oneline
# a1b2c3d Fix typo (HEAD)
# e4f5g6h Add tests
# i7j8k9l Fix bug
# m0n1o2p Add feature
# q3r4s5t Previous work

# Reset to before your 4 commits (soft mode keeps changes)
git reset --soft HEAD~4

# All changes are now staged
git status
# Changes to be committed:
#   new file: feature.php
#   new file: feature_test.php
#   modified: related_file.php

# Create one clean commit
git commit -m "Add new feature with tests and bug fixes"

git log --oneline
# w6x7y8z Add new feature with tests and bug fixes (HEAD)
# q3r4s5t Previous work
```

---

#### Use Case 2: Splitting a Large Commit

**Scenario:** You committed too many unrelated changes in one commit and want to split them.

```bash
# Current state
git log --oneline
# abc123 Huge commit with everything (HEAD)
# def456 Previous work

# Reset to previous commit (mixed mode)
git reset HEAD~1

# Now all changes are unstaged
git status
# Changes not staged for commit:
#   modified: feature_a.php
#   modified: feature_b.php
#   modified: feature_c.php

# Stage and commit logically
git add feature_a.php
git commit -m "Implement feature A"

git add feature_b.php
git commit -m "Implement feature B"

git add feature_c.php
git commit -m "Implement feature C"

git log --oneline
# 111aaa Implement feature C (HEAD)
# 222bbb Implement feature B
# 333ccc Implement feature A
# def456 Previous work
```

---

#### Use Case 3: Syncing with Remote After Force Push

**Scenario:** Someone force-pushed to the remote branch and your local branch is out of sync.

```bash
# Your local branch is ahead but diverged
git status
# Your branch and 'origin/develop' have diverged

# Fetch latest changes
git fetch origin

# See the divergence
git log --oneline --graph --all
# * abc123 Your local commit (HEAD -> develop)
# | * def456 Force-pushed commit (origin/develop)
# |/
# * 789xyz Common ancestor

# Reset to match remote (discard your local commits)
git reset --hard origin/develop

git log --oneline
# def456 Force-pushed commit (HEAD -> develop, origin/develop)
# 789xyz Common ancestor

# Your local commits (abc123) are gone
# (but can be recovered from reflog if needed)
```

---

#### Use Case 4: Undoing a Merge (Local Only)

**Scenario:** You merged a branch but realized it was the wrong branch or had problems.

```bash
# You just merged
git merge feature-branch
# Merge made by the 'recursive' strategy

git log --oneline --graph
# *   abc123 Merge branch 'feature-branch' (HEAD)
# |\
# | * def456 Feature commit
# * | 789xyz Main branch commit
# |/

# Undo the merge
git reset --hard HEAD~1

# OR, if you know the commit before merge
git reset --hard 789xyz

git log --oneline
# 789xyz Main branch commit (HEAD)
# (merge is undone)
```

---

## Git Revert

### What is Git Revert?

`git revert` creates a new commit that undoes the changes introduced by a previous commit. Unlike `reset`, it doesn't rewrite history—it adds to it.

**Key Characteristics:**
- Creates new commits to undo changes
- Safe for shared/public branches
- Preserves complete history
- Can revert multiple commits
- Can be reverted itself (revert the revert)
- Non-destructive operation

**Basic Syntax:**
```bash
git revert [commit]
git revert [commit1]..[commit2]
git revert --no-commit [commit]
```

---

### Revert vs Reset

| Aspect | git reset | git revert |
|--------|-----------|------------|
| **History** | Rewrites/deletes history | Preserves and adds to history |
| **Safety** | Dangerous on shared branches | Safe for shared branches |
| **Method** | Moves branch pointer backward | Creates inverse commit |
| **Visibility** | Changes disappear | Changes visible in history |
| **Use on** | Local/private branches | Any branch, especially public |
| **Undo** | Difficult (needs reflog) | Easy (just revert the revert) |

**When to use which:**

✅ **Use `git reset`** when:
- Working on a local/private branch
- You haven't pushed yet
- You want to clean up messy commit history
- You want to completely remove commits

✅ **Use `git revert`** when:
- Working on a shared/public branch
- You've already pushed commits
- You want to maintain complete history
- Multiple people are working on the branch

---

### Revert Examples

#### Example 1: Revert a Single Commit

```bash
# View history
git log --oneline
# abc123 Bad commit that broke things (HEAD)
# def456 Good commit
# 789xyz Another commit

# Revert the bad commit
git revert abc123

# Git opens editor for commit message
# Default message: "Revert 'Bad commit that broke things'"
# Save and close

git log --oneline
# fed321 Revert "Bad commit that broke things" (HEAD)
# abc123 Bad commit that broke things
# def456 Good commit
# 789xyz Another commit

# The bad commit still exists in history
# but its changes are undone by the revert commit
```

---

#### Example 2: Revert Without Creating Commit

```bash
# Revert but don't commit yet
git revert --no-commit abc123

git status
# Changes to be committed:
#   modified: file.php (reverting changes)

# You can now:
# - Review the changes
# - Make additional modifications
# - Commit manually when ready

git commit -m "Revert bad changes and add fixes"
```

---

#### Example 3: Revert Multiple Commits

```bash
# View history
git log --oneline
# aaa111 Commit 5 (HEAD)
# bbb222 Commit 4 - bad
# ccc333 Commit 3 - bad
# ddd444 Commit 2 - bad
# eee555 Commit 1 - good

# Revert multiple commits (creates separate revert commits)
git revert bbb222 ccc333 ddd444

# OR revert a range (note: oldest..newest)
git revert ddd444..bbb222

# Each commit gets its own revert commit
git log --oneline
# fff666 Revert "Commit 4" (HEAD)
# ggg777 Revert "Commit 3"
# hhh888 Revert "Commit 2"
# aaa111 Commit 5
# bbb222 Commit 4
# ccc333 Commit 3
# ddd444 Commit 2
# eee555 Commit 1
```

---

#### Example 4: Revert a Merge Commit

⚠️ **Special case**: Reverting merge commits requires specifying which parent to keep.

```bash
# View merge commit
git log --oneline --graph
# *   abc123 Merge branch 'feature' (HEAD)
# |\
# | * def456 Feature commit
# * | 789xyz Main commit
# |/

# Revert merge commit (parent 1 is main branch)
git revert -m 1 abc123

# -m 1 means "keep parent 1" (the main branch)
# -m 2 would mean "keep parent 2" (the feature branch)

git log --oneline
# fed321 Revert "Merge branch 'feature'" (HEAD)
# abc123 Merge branch 'feature'
# (feature changes are now undone)
```

---

### Revert Use Cases

#### Use Case 1: Undo Bug in Production

**Scenario:** You deployed a commit to production that has a critical bug. Multiple developers have already pulled the branch.

```bash
# Production branch (main)
git checkout main
git log --oneline
# abc123 Deploy feature X - HAS BUG (HEAD)
# def456 Previous release
# 789xyz Base

# You can't use reset because others have this commit
# Use revert instead
git revert abc123

# Creates new commit that undoes the bug
git log --oneline
# fed321 Revert "Deploy feature X" (HEAD)
# abc123 Deploy feature X - HAS BUG
# def456 Previous release

# Push to production
git push origin main

# Other developers can safely pull
# They get the revert commit and their history stays intact
```

---

#### Use Case 2: Temporarily Remove a Feature

**Scenario:** A feature was merged but needs to be removed temporarily, then re-added later.

```bash
# Feature was merged
git log --oneline
# abc123 Merge feature-login (HEAD)
# def456 Previous work

# Revert the feature
git revert abc123 -m 1

git log --oneline
# fed321 Revert "Merge feature-login" (HEAD)
# abc123 Merge feature-login
# def456 Previous work

# Push the revert
git push origin main

# Later, when you want the feature back
# Revert the revert!
git revert fed321

git log --oneline
# 111aaa Revert "Revert 'Merge feature-login'" (HEAD)
# fed321 Revert "Merge feature-login"
# abc123 Merge feature-login
# (feature is back!)
```

---

#### Use Case 3: Revert Multiple Commits as One

**Scenario:** Several related commits introduced bugs and you want to revert them all at once.

```bash
# View history
git log --oneline
# aaa111 Commit 4 (HEAD)
# bbb222 Commit 3 - problematic
# ccc333 Commit 2 - problematic
# ddd444 Commit 1 - problematic
# eee555 Base commit

# Revert multiple commits without committing each
git revert --no-commit bbb222
git revert --no-commit ccc333
git revert --no-commit ddd444

git status
# Changes to be committed:
#   (all reverts staged together)

# Commit all reverts as one
git commit -m "Revert problematic feature (commits 1-3)"

git log --oneline
# fff666 Revert problematic feature (commits 1-3) (HEAD)
# aaa111 Commit 4
# bbb222 Commit 3
# ccc333 Commit 2
# ddd444 Commit 1
# eee555 Base commit
```

---

#### Use Case 4: Revert with Conflicts

**Scenario:** Reverting a commit causes conflicts with subsequent changes.

```bash
# Try to revert an old commit
git revert abc123
# error: could not revert abc123... Conflict!
# CONFLICT (content): Merge conflict in file.php

git status
# Unmerged paths:
#   both modified: file.php

# Open file.php and resolve conflicts manually
vim file.php

# Look for conflict markers
# <<<<<<< HEAD
# Current version
# =======
# Version being reverted
# >>>>>>> parent of abc123... commit message

# Resolve conflicts, then continue
git add file.php
git revert --continue

# OR abort if you change your mind
git revert --abort
```

---

## Git Checkout

### What is Git Checkout?

`git checkout` serves multiple purposes: switching branches, creating branches, and restoring files from different commits. In Git 2.23+, some functionality was split into `git switch` and `git restore` for clarity.

**Key Characteristics:**
- Switches between branches
- Restores files from commits
- Creates new branches
- Can detach HEAD (view old commits)
- Updates working directory
- Non-destructive to repository history

**Basic Syntax:**
```bash
git checkout [branch]                    # Switch branch
git checkout -b [new-branch]             # Create and switch
git checkout [commit]                    # Detached HEAD
git checkout [commit] -- [file]          # Restore file
```

**Modern Alternatives (Git 2.23+):**
```bash
git switch [branch]                      # Switch branch (clearer)
git switch -c [new-branch]               # Create and switch
git restore [file]                       # Restore file (clearer)
git restore --source=[commit] [file]     # Restore from commit
```

---

### Checkout Modes

#### 1. Switch Branches

```bash
# Switch to existing branch
git checkout develop
# Switched to branch 'develop'

# Modern alternative
git switch develop
```

**What happens:**
- HEAD moves to the specified branch
- Working directory updates to match branch
- Staging area is cleared
- Any uncommitted changes are preserved (if no conflicts)

---

#### 2. Create and Switch to New Branch

```bash
# Create and switch in one command
git checkout -b feature/new-feature

# Modern alternative
git switch -c feature/new-feature

# Create branch from specific commit
git checkout -b hotfix/bug-123 abc1234

# Create branch from remote branch
git checkout -b local-feature origin/feature
```

---

#### 3. Restore Files from Commits

```bash
# Restore file from HEAD (discard local changes)
git checkout -- file.php

# Restore file from specific commit
git checkout abc1234 -- file.php

# Restore multiple files
git checkout HEAD -- file1.php file2.php

# Restore entire directory
git checkout HEAD -- src/

# Modern alternatives
git restore file.php
git restore --source=abc1234 file.php
```

---

#### 4. Detached HEAD State

```bash
# Checkout a specific commit (not a branch)
git checkout abc1234
# Note: switching to 'abc1234'
# You are in 'detached HEAD' state

# View old code, run tests, etc.
# Any commits here won't belong to any branch

# Return to a branch
git checkout main
```

---

### Checkout Examples

#### Example 1: Switch Between Branches

```bash
# Current branch: main
git branch
# * main
#   develop
#   feature/login

# Switch to develop
git checkout develop
# Switched to branch 'develop'

git branch
#   main
# * develop
#   feature/login

# Switch back to main
git checkout main

# Quick switch to previous branch
git checkout -
# Switched to branch 'develop'

git checkout -
# Switched to branch 'main'
```

---

#### Example 2: Create Feature Branch

```bash
# On main branch, create feature branch
git checkout main
git checkout -b feature/user-profile

# This is equivalent to:
# git branch feature/user-profile
# git checkout feature/user-profile

# Verify
git branch
#   main
#   develop
# * feature/user-profile

# Make commits on feature branch
git add profile.php
git commit -m "Add user profile page"
```

---

#### Example 3: Discard Local Changes to a File

```bash
# You modified a file but want to discard changes
echo "bad code" >> important.php

git status
# Changes not staged for commit:
#   modified: important.php

# Discard changes (restore from HEAD)
git checkout -- important.php

git status
# nothing to commit, working tree clean

# Modern alternative
git restore important.php
```

---

#### Example 4: Restore File from Old Commit

```bash
# View history
git log --oneline
# abc123 Latest commit (HEAD)
# def456 Modified important.php
# 789xyz Original important.php implementation
# 012jkl Base commit

# Current file is broken, restore from old commit
git checkout 789xyz -- important.php

git status
# Changes to be committed:
#   modified: important.php

# The file now contains the version from commit 789xyz
# and is staged for commit

# Commit the restoration
git commit -m "Restore important.php from working version"
```

---

#### Example 5: View Old Code (Detached HEAD)

```bash
# Want to see code from 6 months ago
git log --since="6 months ago" --oneline
# abc123 Commit from 6 months ago

# Checkout that commit
git checkout abc123
# Note: switching to 'abc123'.
# You are in 'detached HEAD' state.

# Browse the old code
ls -la
cat old-file.php

# Run old tests
npm test

# To save changes here, create a branch
git checkout -b explore-old-code

# OR just return to main
git checkout main
```

---

#### Example 6: Checkout Remote Branch

```bash
# List remote branches
git branch -r
# origin/main
# origin/develop
# origin/feature/new-ui

# Checkout remote branch (creates local tracking branch)
git checkout feature/new-ui
# Branch 'feature/new-ui' set up to track 'origin/feature/new-ui'
# Switched to a new branch 'feature/new-ui'

# Or explicitly
git checkout -b feature/new-ui origin/feature/new-ui

# Modern alternative
git switch feature/new-ui
```

---

### Checkout Use Cases

#### Use Case 1: Switching Context Between Tasks

**Scenario:** You're working on a feature but need to quickly fix a bug on the main branch.

```bash
# Working on feature branch
git checkout feature/user-dashboard
# ... making changes ...

# Urgent bug report comes in
# Save current work
git add .
git commit -m "WIP: user dashboard progress"

# Switch to main
git checkout main

# Create hotfix branch
git checkout -b hotfix/critical-bug

# Fix the bug
vim bug-file.php
git add bug-file.php
git commit -m "Fix critical bug in payment processing"

# Merge to main
git checkout main
git merge hotfix/critical-bug

# Deploy fix
git push origin main

# Return to feature work
git checkout feature/user-dashboard

# Continue working
```

---

#### Use Case 2: Recover Deleted File

**Scenario:** You accidentally deleted a file and committed the deletion.

```bash
# File was deleted
git log --oneline -- deleted-file.php
# abc123 Delete old file (HEAD)
# def456 Modify file
# 789xyz Add file

# Find last commit where file existed
git log --oneline --all --full-history -- deleted-file.php
# def456 Modify file (last commit with file)

# Restore the file
git checkout def456 -- deleted-file.php

git status
# Changes to be committed:
#   new file: deleted-file.php

# Commit the restoration
git commit -m "Restore accidentally deleted file"
```

---

#### Use Case 3: Cherry-pick Specific File from Another Branch

**Scenario:** You need just one file from another branch without merging.

```bash
# On main branch
git checkout main

# Want specific file from feature branch
git checkout feature/new-login -- src/Auth/Login.php

git status
# Changes to be committed:
#   new file: src/Auth/Login.php

# Review the file
cat src/Auth/Login.php

# Commit if it looks good
git commit -m "Import Login.php from feature branch"
```

---

#### Use Case 4: Experiment Without Committing

**Scenario:** You want to try out an old version of code without creating commits.

```bash
# Save current work first
git stash

# Checkout old commit
git checkout abc1234
# You are in 'detached HEAD' state

# Test old code
npm test
# Tests pass! The bug was introduced later

# Find which commit broke it
git log --oneline abc1234..HEAD
# def456 Commit A
# 789xyz Commit B - likely culprit
# 012jkl Commit C

# Test commit B
git checkout 789xyz
npm test
# Tests fail! Found the problematic commit

# Return to current state
git checkout main
git stash pop
```

---

#### Use Case 5: Partial File Restore

**Scenario:** You changed too many files and want to keep some changes but discard others.

```bash
# Modified multiple files
git status
# Changes not staged for commit:
#   modified: keep-this.php
#   modified: discard-this.php
#   modified: also-discard.php

# Discard specific files
git checkout -- discard-this.php also-discard.php

git status
# Changes not staged for commit:
#   modified: keep-this.php

# Stage and commit the file you want
git add keep-this.php
git commit -m "Update keep-this.php"
```

---

## Comparison Table

### Command Comparison Matrix

| Command | Changes History | Safe for Shared Branches | Working Directory | Staging Area | Use Case |
|---------|----------------|--------------------------|-------------------|--------------|----------|
| **git reset --soft** | ✅ Yes (moves HEAD) | ❌ No | Unchanged | Unchanged | Re-commit, squash commits |
| **git reset --mixed** | ✅ Yes (moves HEAD) | ❌ No | Unchanged | Reset | Unstage files, split commits |
| **git reset --hard** | ✅ Yes (moves HEAD) | ❌ No | ⚠️ Discarded | Reset | Discard everything, sync with remote |
| **git revert** | Adds commits | ✅ Yes | Updated | Updated | Undo public commits safely |
| **git checkout branch** | No | ✅ Yes | Updated | Cleared | Switch branches |
| **git checkout file** | No | ✅ Yes | Restored | Staged (if from commit) | Restore specific files |
| **git checkout commit** | No (detached) | ✅ Yes | Updated | Cleared | View old code |

---

### Decision Flow Chart

```
Need to undo changes?
│
├─ Changes pushed to shared branch?
│  │
│  ├─ Yes → Use git revert
│  │        Creates new commit undoing changes
│  │        Safe for collaboration
│  │
│  └─ No → Continue below
│
├─ Want to keep changes?
│  │
│  ├─ Keep as staged → Use git reset --soft
│  │                    Move HEAD, keep staging area
│  │
│  ├─ Keep as unstaged → Use git reset --mixed (or just git reset)
│  │                      Move HEAD, clear staging, keep working dir
│  │
│  └─ Discard completely → Use git reset --hard
│                          ⚠️ DANGER: Permanently removes changes
│
├─ Want to restore specific file?
│  │
│  └─ Use git checkout [commit] -- file
│     or git restore --source=[commit] file
│
└─ Want to view old code?
   │
   └─ Use git checkout [commit]
      Creates detached HEAD state
```

---

## Real-World Scenarios

### Scenario 1: "I committed to the wrong branch!"

**Problem:** You made commits on `main` that should be on a feature branch.

**Solution:**

```bash
# You're on main with 3 new commits
git log --oneline
# abc123 Commit 3 (HEAD -> main)
# def456 Commit 2
# 789xyz Commit 1
# 012jkl Base commit (origin/main)

# Create feature branch from current position
git checkout -b feature/my-work

# Now move main back to origin/main
git checkout main
git reset --hard origin/main

# Verify
git log --oneline
# 012jkl Base commit (HEAD -> main, origin/main)

git checkout feature/my-work
git log --oneline
# abc123 Commit 3 (HEAD -> feature/my-work)
# def456 Commit 2
# 789xyz Commit 1
# 012jkl Base commit

# Your commits are now on the feature branch!
```

---

### Scenario 2: "I need to undo a commit in production!"

**Problem:** A bad commit was pushed to production and others have pulled it.

**Solution:**

```bash
# On production branch
git checkout production

git log --oneline
# abc123 Bad commit causing errors (HEAD)
# def456 Last good release
# 789xyz Previous work

# Use revert (safe for shared branches)
git revert abc123

# Editor opens with message:
# "Revert 'Bad commit causing errors'"
# Save and close

git log --oneline
# fed321 Revert "Bad commit causing errors" (HEAD)
# abc123 Bad commit causing errors
# def456 Last good release

# Push to production
git push origin production

# Everyone can safely pull the fix
# No history rewriting, no conflicts for other developers
```

---

### Scenario 3: "I want to test code from last week"

**Problem:** There's a bug now that didn't exist last week. Want to test old code.

**Solution:**

```bash
# Find commits from last week
git log --since="1 week ago" --oneline
# abc123 This week's work (HEAD)
# def456 Monday's commit
# ...

git log --until="1 week ago" --oneline -1
# 789xyz Last commit from 1 week ago

# Checkout that commit
git checkout 789xyz
# Note: switching to '789xyz'
# You are in 'detached HEAD' state.

# Run your application
npm start
# Test functionality - works fine!

# Run tests
npm test
# All tests pass

# The bug was introduced this week
# Use git bisect to find exact commit (see below)

# Return to present
git checkout main
```

---

### Scenario 4: "I have mixed changes - some good, some bad"

**Problem:** Your working directory has changes to 5 files, but you only want to keep 2.

**Solution:**

```bash
git status
# Changes not staged for commit:
#   modified: file1.php (keep)
#   modified: file2.php (keep)
#   modified: file3.php (discard)
#   modified: file4.php (discard)
#   modified: file5.php (discard)

# Discard unwanted files
git checkout -- file3.php file4.php file5.php

git status
# Changes not staged for commit:
#   modified: file1.php
#   modified: file2.php

# Stage and commit good changes
git add file1.php file2.php
git commit -m "Update important features"
```

---

### Scenario 5: "Need to pull but have local changes"

**Problem:** You have uncommitted local changes but need to pull latest from remote.

**Solution:**

```bash
# You have local changes
git status
# Changes not staged for commit:
#   modified: local_work.php

# Try to pull
git pull
# error: Your local changes would be overwritten by merge
# Please commit or stash them

# Option 1: Stash changes temporarily
git stash
git pull
git stash pop

# Option 2: If local changes are junk, discard them
git reset --hard HEAD
git pull

# Option 3: Commit local changes first
git add .
git commit -m "WIP: local work"
git pull
# Handle merge if needed
```

---

### Scenario 6: "Accidentally committed secrets/passwords"

**Problem:** You committed a file with API keys or passwords.

**⚠️ Critical:** If you've pushed to remote, consider the secrets compromised. Rotate them immediately!

**Solution for local-only commits:**

```bash
# Check recent commits
git log --oneline
# abc123 Add API integration (HEAD) - contains secrets!
# def456 Previous work

# Remove the commit but keep changes
git reset --soft HEAD~1

git status
# Changes to be committed:
#   modified: config.php (contains secrets)

# Remove secrets from file
vim config.php
# Replace actual secrets with placeholders/environment variables

# Create .env file for secrets (not tracked)
echo "API_KEY=your-secret-key" > .env
echo ".env" >> .gitignore

# Stage corrected files
git add config.php .gitignore
git commit -m "Add API integration (using environment variables)"

# Verify .env is not tracked
git status
# nothing to commit, working tree clean
# (.env should not appear)
```

**If already pushed to remote:**

```bash
# ⚠️ This requires force push - coordinate with team!

# Remove commit locally
git reset --hard HEAD~1

# Force push (use with caution!)
git push --force origin branch-name

# ⚠️ IMPORTANT: Rotate/change all exposed secrets immediately!
# They should be considered compromised even after removal

# Tell team members to reset their branches:
# git fetch origin
# git reset --hard origin/branch-name
```

---

### Scenario 7: "Want to continue work from a specific commit"

**Problem:** You want to start a new feature based on a specific old commit, not the latest.

**Solution:**

```bash
# View history
git log --oneline
# abc123 Latest commit (HEAD -> main)
# def456 Recent commit
# 789xyz Good stable commit - want to branch from here
# 012jkl Older commit

# Create branch from specific commit
git checkout -b feature/new-work 789xyz

git log --oneline
# 789xyz Good stable commit (HEAD -> feature/new-work)
# 012jkl Older commit

# Work on your feature
# This branch won't include commits abc123 and def456
git add new-feature.php
git commit -m "Start new feature from stable base"
```

---

### Scenario 8: "Merge conflict during revert"

**Problem:** Reverting an old commit causes conflicts with newer code.

**Solution:**

```bash
# Try to revert old commit
git log --oneline
# abc123 Latest work (HEAD)
# def456 More changes
# 789xyz Problem commit - want to revert this
# 012jkl Base

git revert 789xyz
# error: could not revert 789xyz...
# CONFLICT (content): Merge conflict in modified_file.php

git status
# Unmerged paths:
#   both modified: modified_file.php

# Open file and resolve conflicts
vim modified_file.php

# File contains:
# <<<<<<< HEAD
# Current code (keep this)
# =======
# Code being reverted (remove this)
# >>>>>>> parent of 789xyz

# Manually decide what to keep
# Remove conflict markers

# After resolving
git add modified_file.php

# Continue the revert
git revert --continue

# OR abort if it's too complicated
git revert --abort

# Alternative: Manual revert
git checkout 789xyz^ -- modified_file.php  # Get version before problem commit
git add modified_file.php
git commit -m "Manually revert changes from 789xyz"
```

---

### Scenario 9: "Clean up messy commit history before pushing"

**Problem:** You have 10 commits with messages like "fix", "oops", "try again" that you want to clean up.

**Solution:**

```bash
# Current messy history (not pushed yet)
git log --oneline
# aaa111 try this (HEAD)
# bbb222 oops typo
# ccc333 fix
# ddd444 actually works now
# eee555 attempt 2
# fff666 attempt 1
# ggg777 start feature
# hhh888 previous work (origin/main)

# Count commits since origin/main
git log --oneline origin/main..HEAD
# 7 commits

# Reset to origin/main (soft keeps all changes)
git reset --soft origin/main

git status
# Changes to be committed:
#   (all your work from those 7 commits)

# Create one clean commit
git commit -m "Implement feature X with comprehensive tests

- Add core functionality
- Implement error handling
- Add unit tests
- Update documentation"

git log --oneline
# iii999 Implement feature X with comprehensive tests (HEAD)
# hhh888 previous work (origin/main)

# Now push clean history
git push origin main
```

---

## Best Practices

### General Guidelines

#### 1. **Know Your Branch Status**

```bash
# Before using reset/revert/checkout, always check:
git status                    # Current state
git log --oneline -10         # Recent history
git log --oneline origin/main..HEAD  # Unpushed commits
git branch -vv                # Branch tracking status

# Is your branch:
# - Pushed to remote? → Use revert
# - Local only? → Reset is safe
# - Shared with others? → Use revert
```

---

#### 2. **Reset Safety Checklist**

Before using `git reset --hard`, ask yourself:

- [ ] Have I pushed these commits to remote?
- [ ] Is anyone else working on this branch?
- [ ] Do I have uncommitted changes I want to keep?
- [ ] Have I created a backup branch?
- [ ] Am I sure I don't need these changes?

If answer is "No" to all, `reset --hard` is safe.

```bash
# Create safety backup before reset
git branch backup-$(date +%Y%m%d-%H%M%S)

# Now safe to reset
git reset --hard HEAD~3

# If you change your mind
git checkout backup-20241101-143022
```

---

#### 3. **Revert for Collaboration**

Use `git revert` on shared branches:

```bash
# ✅ GOOD: Safe for shared branches
git revert abc123
git push origin main

# ❌ BAD: Rewrites history others may have
git reset --hard HEAD~1
git push --force origin main  # Don't do this!
```

---

#### 4. **Checkout Best Practices**

```bash
# Always save work before checkout
git stash  # or commit

# Use meaningful branch names
git checkout -b feature/user-authentication  # ✅ Good
git checkout -b stuff                        # ❌ Bad

# Don't stay in detached HEAD
git checkout abc123    # View old code
# ... look around ...
git checkout main      # Return to branch (not detached)

# Use modern alternatives when available (Git 2.23+)
git switch feature-branch     # Clearer than checkout
git restore file.php          # Clearer than checkout --
```

---

#### 5. **Commit Messages for Reverts**

```bash
# Good revert messages explain WHY
git revert abc123 -m "Revert login feature due to security vulnerability

The feature introduced in abc123 has a critical security flaw
that allows unauthorized access. Reverting until fix is ready.

Issue: SEC-2024-001
See: https://jira.company.com/SEC-2024-001"

# Bad revert messages
git revert abc123 -m "revert"                    # ❌ No context
git revert abc123 -m "oops"                      # ❌ Unprofessional
```

---

### Command-Specific Best Practices

#### Reset Best Practices

```bash
# 1. Use --soft to preserve work
git reset --soft HEAD~1   # Undo commit, keep changes staged

# 2. Use --mixed to reorganize
git reset HEAD            # Unstage everything, keep changes

# 3. Use --hard only when certain
git reset --hard HEAD     # ⚠️ Discards everything

# 4. Reset specific files to unstage
git reset HEAD file.php   # Unstage one file

# 5. Know your targets
git reset HEAD~1          # Back 1 commit
git reset HEAD~3          # Back 3 commits
git reset abc123          # To specific commit
git reset origin/main     # Match remote branch
```

---

#### Revert Best Practices

```bash
# 1. Revert recent commits first (chronological)
git revert HEAD           # Most recent
git revert HEAD~1         # Then previous
# Not: git revert HEAD~1 HEAD (wrong order)

# 2. Use --no-commit for multiple reverts
git revert --no-commit abc123
git revert --no-commit def456
git commit -m "Revert feature X commits"

# 3. Always specify -m for merge commits
git revert -m 1 abc123    # Revert merge, keep first parent

# 4. Test after reverting
git revert abc123
npm test                  # Verify tests still pass
git push                  # Then push

# 5. Document reverts in commit message
git revert abc123 -m "Revert feature due to [reason]

Original commit: abc123
Reason: [detailed explanation]
Ticket: [issue tracker link]"
```

---

#### Checkout Best Practices

```bash
# 1. Use switch for branches (Git 2.23+)
git switch main           # ✅ Clear intent
git checkout main         # ❌ Ambiguous

# 2. Use restore for files (Git 2.23+)
git restore file.php      # ✅ Clear intent
git checkout -- file.php  # ❌ Confusing syntax

# 3. Be explicit with file checkouts
git checkout HEAD -- file.php       # ✅ Clear source
git checkout file.php               # ❌ May fail if branch named "file.php"

# 4. Create branch before making changes in detached HEAD
git checkout abc123                 # Detached HEAD
# If you make changes:
git checkout -b explore-old-code    # Create branch first!

# 5. Use checkout for temporary file recovery
git checkout abc123 -- deleted-file.php   # Restore deleted file
git restore --source=abc123 deleted-file.php  # Modern alternative
```

---

### Workflow Recommendations

#### Development Workflow

```bash
# 1. Start feature
git checkout main
git pull origin main
git checkout -b feature/new-work

# 2. Make commits
git add .
git commit -m "Add feature"

# 3. Before pushing, clean up history
git log --oneline origin/main..HEAD
# If messy, squash with reset
git reset --soft origin/main
git commit -m "Clean feature commit"

# 4. Push
git push origin feature/new-work

# 5. If changes needed after push, use revert not reset
git revert HEAD  # Safe for pushed commits
```

---

#### Bug Fix Workflow

```bash
# 1. Create hotfix from main
git checkout main
git checkout -b hotfix/critical-bug

# 2. Fix and test
vim fix-file.php
npm test
git add fix-file.php
git commit -m "Fix critical bug in payment"

# 3. Merge to main
git checkout main
git merge hotfix/critical-bug

# 4. If bug fix was wrong, revert on main
git revert HEAD
git push origin main

# 5. Fix properly on hotfix branch
git checkout hotfix/critical-bug
# Make correct fix
```

---

#### Experimental Workflow

```bash
# 1. Save current work
git stash

# 2. Checkout old code
git checkout abc123

# 3. Experiment
npm test
# Tests pass in old code

# 4. Find where it broke (use bisect)
git bisect start
git bisect bad HEAD
git bisect good abc123
# Git will checkout commits for you to test
npm test
git bisect good  # or bad
# Repeat until found

# 5. Return to current work
git bisect reset
git checkout main
git stash pop
```

---

## Common Pitfalls

### Pitfall 1: Force Pushing After Reset

**❌ Problem:**

```bash
git reset --hard HEAD~3
git push origin main
# error: failed to push
# Updates were rejected because tip of your branch is behind

# DON'T DO THIS:
git push --force origin main  # ⚠️ DANGER: Rewrites shared history
```

**✅ Solution:**

```bash
# If you haven't pushed yet:
git reset --hard HEAD~3  # Safe

# If you already pushed:
git revert HEAD~2..HEAD  # Safe for shared branches
git push origin main     # No force needed
```

---

### Pitfall 2: Losing Work with Hard Reset

**❌ Problem:**

```bash
# Uncommitted work
git status
# modified: important.php (changes not staged)

# Oops, hard reset!
git reset --hard HEAD
# Work is GONE!
```

**✅ Prevention:**

```bash
# Always check status first
git status

# If you have changes:
git stash              # Save temporarily
# OR
git add .
git commit -m "WIP"    # Commit as work-in-progress

# Then reset if needed
git reset --hard HEAD~1
```

**🔧 Recovery (if it happens):**

```bash
# Check reflog for lost commits
git reflog
# abc123 HEAD@{0}: reset: moving to HEAD~1
# def456 HEAD@{1}: commit: Lost work

# Recover lost commit
git checkout def456
git checkout -b recovered-work

# OR cherry-pick
git checkout main
git cherry-pick def456
```

---

### Pitfall 3: Reverting Without Testing

**❌ Problem:**

```bash
git revert abc123
git push origin main
# Deployed to production
# Breaks everything because of dependencies
```

**✅ Solution:**

```bash
# Always test after revert
git revert abc123

# Test thoroughly
npm test
npm run e2e-tests

# Manual testing
npm start
# Check functionality

# Only then push
git push origin main
```

---

### Pitfall 4: Detached HEAD Commits

**❌ Problem:**

```bash
git checkout abc123
# You are in 'detached HEAD' state

# Make changes and commit
git add file.php
git commit -m "Fix bug"
# Commit created: def456

# Switch back to main
git checkout main
# Commit def456 is now orphaned!
```

**✅ Solution:**

```bash
# Always create a branch if making changes
git checkout abc123
# In detached HEAD

# Create branch before committing
git checkout -b fix-from-old-code

# Now commits are safe
git add file.php
git commit -m "Fix bug"

# Can merge to main later
git checkout main
git merge fix-from-old-code
```

**🔧 Recovery (if it happens):**

```bash
# Find orphaned commit
git reflog
# def456 HEAD@{2}: commit: Fix bug
# (in detached HEAD state)

# Recover by creating branch
git branch recovered-fixes def456
git checkout recovered-fixes
```

---

### Pitfall 5: Checkout Confusion

**❌ Problem:**

```bash
# Want to switch to branch "file.php"
git checkout file.php
# Oops! Restored file.php from HEAD instead
# Local changes to file.php are lost!
```

**✅ Solution:**

```bash
# Be explicit with branches
git checkout -b file.php  # Create branch
git checkout -- file.php  # Restore file (note the --)

# Better: Use modern commands (Git 2.23+)
git switch file.php       # Switch to branch (will error if file)
git restore file.php      # Restore file (clear intent)
```

---

### Pitfall 6: Reverting Merge Commits Incorrectly

**❌ Problem:**

```bash
# Revert merge without -m flag
git revert abc123
# error: commit abc123 is a merge but no -m option was given
```

**✅ Solution:**

```bash
# View merge commit parents
git show abc123
# Merge: def456 789xyz
# Parent 1: def456 (main branch)
# Parent 2: 789xyz (feature branch)

# Revert merge, keep first parent (main)
git revert -m 1 abc123

# If you chose wrong parent
git revert HEAD  # Undo the revert
git revert -m 2 abc123  # Try other parent
```

---

### Pitfall 7: Reset Specific Files (Doesn't Work)

**❌ Problem:**

```bash
# Try to reset file to old commit
git reset --hard abc123 -- file.php
# error: cannot do hard reset with paths
```

**✅ Solution:**

```bash
# Use checkout for specific files
git checkout abc123 -- file.php

# OR modern alternative
git restore --source=abc123 file.php

# Reset doesn't work with file paths in --hard mode
# Only checkout/restore work for specific files
```

---

### Pitfall 8: Reverting Multiple Commits in Wrong Order

**❌ Problem:**

```bash
# Try to revert oldest to newest
git log --oneline
# aaa111 Commit 3 (HEAD)
# bbb222 Commit 2
# ccc333 Commit 1

git revert ccc333 bbb222 aaa111
# Conflicts! Wrong order causes issues
```

**✅ Solution:**

```bash
# Revert in reverse chronological order (newest first)
git revert aaa111 bbb222 ccc333

# OR use range notation
git revert ccc333..aaa111  # Reverts ccc333 to aaa111 (exclusive..inclusive)

# OR use --no-commit for complex cases
git revert --no-commit ccc333
git revert --no-commit bbb222  
git revert --no-commit aaa111
git commit -m "Revert commits 1-3"
```

---

## Recovery Techniques

### Using Reflog to Recover Lost Commits

The reflog records every movement of HEAD, allowing you to recover "lost" commits.

```bash
# View reflog
git reflog
# abc123 HEAD@{0}: reset: moving to HEAD~3
# def456 HEAD@{1}: commit: Lost commit 3
# 789xyz HEAD@{2}: commit: Lost commit 2
# 012jkl HEAD@{3}: commit: Lost commit 1

# Each entry shows:
# - Commit hash
# - HEAD position (HEAD@{n})
# - Action that created entry
# - Description

# Recover specific commit
git checkout def456

# Create branch to save recovered work
git checkout -b recovered-work

# OR cherry-pick specific commits
git checkout main
git cherry-pick def456 789xyz 012jkl
```

---

### Recovering After Hard Reset

**Scenario:** You did `git reset --hard` and lost commits.

```bash
# Oh no!
git reset --hard HEAD~5
# 5 commits gone

# Don't panic! Check reflog
git reflog
# abc123 HEAD@{0}: reset: moving to HEAD~5
# def456 HEAD@{1}: commit: Last commit (THIS ONE!)

# Restore to before reset
git reset --hard def456

# OR
git reset --hard HEAD@{1}

# Your commits are back!
git log --oneline
# def456 Last commit (HEAD)
# (all 5 commits restored)
```

---

### Recovering Deleted Branch

**Scenario:** You deleted a branch that wasn't merged.

```bash
# Delete branch
git branch -D feature/important-work
# Deleted branch feature/important-work (was abc123)

# Oops! Needed that work
# Find it in reflog
git reflog --all
# abc123 refs/heads/feature/important-work@{0}: commit: Important work

# Restore branch
git branch feature/important-work abc123

# OR checkout directly
git checkout -b feature/important-work abc123
```

---

### Recovering Deleted Files

**Scenario:** You deleted a file and committed, but need it back.

```bash
# Find last commit where file existed
git log --all --full-history -- path/to/deleted-file.php
# abc123 Author: Delete old file
# def456 Author: Modify file (LAST TIME IT EXISTED)

# Restore from that commit
git checkout def456 -- path/to/deleted-file.php

git status
# Changes to be committed:
#   new file: path/to/deleted-file.php

# Commit restoration
git commit -m "Restore deleted file from def456"
```

---

### Finding Lost Commits (Orphaned)

**Scenario:** You were in detached HEAD, made commits, and they're now orphaned.

```bash
# Find all orphaned commits
git fsck --lost-found
# dangling commit abc123
# dangling commit def456

# View orphaned commit
git show abc123

# Recover by creating branch
git branch recovered-work abc123

# OR use reflog (easier)
git reflog
# Look for commits in detached HEAD state
```

---

### Undoing a Bad Revert

**Scenario:** You reverted a commit but the revert was wrong.

```bash
# Original history
git log --oneline
# abc123 Feature commit (HEAD)

# Revert it
git revert abc123
git log --oneline
# def456 Revert "Feature commit" (HEAD)
# abc123 Feature commit

# Oops! Revert was wrong, need feature back
# Revert the revert
git revert def456

git log --oneline
# 789xyz Revert "Revert 'Feature commit'" (HEAD)
# def456 Revert "Feature commit"
# abc123 Feature commit

# Feature is back!
# (History shows all the back-and-forth, which is good for audit)
```

---

### Emergency Recovery Procedure

If something goes catastrophically wrong:

```bash
# 1. DON'T PANIC
# As long as you didn't run git gc, commits are recoverable

# 2. Immediately check reflog
git reflog --all

# 3. Find the last known good state
git reflog | grep "commit:"
# Look for your last good commit

# 4. Create backup branch at that point
git branch emergency-backup <commit-hash>

# 5. Restore to that state
git reset --hard <commit-hash>

# 6. If reset deleted files, check git fsck
git fsck --lost-found

# 7. For deleted branch, find in reflog
git reflog show --all | grep "branch-name"

# 8. Last resort: Search all reachable objects
git log --all --full-history --source --remotes --reflog

# 9. Clone from remote as backup
cd ..
git clone <remote-url> project-backup
# Compare to see what's missing
```

---

### Reflog Expiration

Be aware: reflog entries expire after 90 days by default.

```bash
# View reflog expiration settings
git config --get gc.reflogExpire
# default: 90.days

git config --get gc.reflogExpireUnreachable  
# default: 30.days

# Extend expiration (if doing risky operations)
git config gc.reflogExpire never
git config gc.reflogExpireUnreachable never

# Later, reset to default
git config --unset gc.reflogExpire
git config --unset gc.reflogExpireUnreachable

# Force garbage collection (careful!)
git gc --prune=now  # Removes unreachable objects

# Prevent garbage collection temporarily
git config gc.auto 0  # Disable auto GC
```

---

## Summary

### Quick Reference Card

| Scenario | Command | Notes |
|----------|---------|-------|
| **Undo last commit, keep changes** | `git reset --soft HEAD~1` | Changes stay staged |
| **Undo last commit, unstage changes** | `git reset HEAD~1` | Changes stay in working dir |
| **Discard everything** | `git reset --hard HEAD` | ⚠️ Permanent loss |
| **Undo public commit safely** | `git revert <commit>` | Creates new commit |
| **Unstage file** | `git reset HEAD <file>` | Or `git restore --staged <file>` |
| **Discard file changes** | `git checkout -- <file>` | Or `git restore <file>` |
| **Restore deleted file** | `git checkout <commit> -- <file>` | File must exist in commit |
| **View old code** | `git checkout <commit>` | Creates detached HEAD |
| **Switch branches** | `git checkout <branch>` | Or `git switch <branch>` |
| **Create new branch** | `git checkout -b <branch>` | Or `git switch -c <branch>` |
| **Undo merge (local)** | `git reset --hard HEAD~1` | Before pushing only |
| **Undo merge (public)** | `git revert -m 1 <merge>` | Safe for shared branches |
| **Recover lost commits** | `git reflog` → `git reset --hard <hash>` | Use reflog to find hash |
| **Match remote branch** | `git reset --hard origin/<branch>` | After fetch |

---

### When to Use What

**Use `git reset --soft`:**
- Redo commit with better message
- Squash multiple commits
- Add more changes to last commit

**Use `git reset --mixed` (default):**
- Unstage files
- Split commit into multiple
- Reorganize changes

**Use `git reset --hard`:**
- Discard all local changes
- Sync with remote branch
- Start fresh from a commit
- ⚠️ Only on unpushed, unshared work

**Use `git revert`:**
- Undo commits on shared branches
- Undo commits already pushed
- Maintain complete history
- When collaboration is active

**Use `git checkout`:**
- Switch between branches
- View old code (detached HEAD)
- Restore specific files
- Create new branches

---

### Safety First!

**Before doing anything destructive:**

```bash
# 1. Check status
git status

# 2. Check history
git log --oneline -10

# 3. Check if pushed
git log --oneline origin/main..HEAD

# 4. Create backup branch
git branch backup-$(date +%Y%m%d-%H%M)

# 5. Verify reflog exists
git reflog

# NOW safe to proceed with reset/revert/checkout
```

**Golden Rules:**

1. ✅ **Reset** for local changes (not pushed)
2. ✅ **Revert** for public changes (already pushed)
3. ✅ **Checkout** for switching context or restoring files
4. ⚠️ **Never** force push to shared branches without team coordination
5. ⚠️ **Always** backup before destructive operations
6. ✅ **Test** after reverting or resetting
7. ✅ **Document** why you reverted something

---

### Further Reading

**Git Documentation:**
- `git help reset` - Complete reset documentation
- `git help revert` - Complete revert documentation
- `git help checkout` - Complete checkout documentation
- `git help reflog` - Learn about recovery

**Related Commands:**
- `git stash` - Temporarily save work
- `git cherry-pick` - Apply specific commits
- `git bisect` - Find bugs in history
- `git clean` - Remove untracked files

**Modern Alternatives (Git 2.23+):**
- `git switch` - Replaces `checkout` for branches
- `git restore` - Replaces `checkout` for files
- Clearer separation of concerns
- Less confusing syntax

---

## Conclusion

Understanding `reset`, `revert`, and `checkout` is essential for effective Git usage. Each command serves a specific purpose:

- **Reset** rewrites history (use locally)
- **Revert** preserves history (use publicly)  
- **Checkout** switches context (use for navigation)

Always consider whether you're working on shared or local branches before choosing a command. When in doubt, `revert` is the safest option for shared work, while `reset` gives you more control for local history cleanup.

Remember: Git's reflog is your safety net—commits are rarely truly lost. But prevention is better than recovery, so always double-check before running destructive commands!

---

**Document Version:** 1.0  
**Last Updated:** November 1, 2025  
**Author:** Magento Development Team

