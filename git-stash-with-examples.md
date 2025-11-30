# Git Stash: Comprehensive Guide with Real-World Examples

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Concepts](#basic-concepts)
3. [Basic Usage](#basic-usage)
4. [Advanced Operations](#advanced-operations)
5. [Real-World Examples](#real-world-examples)
6. [Stash vs Other Git Operations](#stash-vs-other-git-operations)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)
9. [Common Pitfalls](#common-pitfalls)
10. [Tips and Tricks](#tips-and-tricks)

---

## Introduction

### What is Git Stash?

`git stash` is a powerful Git command that temporarily saves your uncommitted changes (both staged and unstaged) and reverts your working directory to a clean state matching the last commit. Think of it as a temporary clipboard for your work-in-progress changes.

### When to Use Stash

**Good Use Cases:**
- Switching branches to work on urgent fixes without committing incomplete work
- Pulling latest changes when you have local modifications
- Testing your code without recent changes
- Cleaning your working directory temporarily
- Moving changes between branches
- Creating a clean state for operations that require it (rebase, merge, etc.)

**When NOT to Use:**
- For long-term storage (use branches instead)
- As a backup solution (use commits or branches)
- When you should actually commit your work
- To avoid dealing with merge conflicts (address them properly)

---

## Basic Concepts

### How Stash Works

When you stash your changes:
1. Git saves your modified tracked files and staged changes
2. Git creates a commit-like object on a special reference (stash stack)
3. Git resets your working directory to match HEAD
4. Your changes are stored in a stack (LIFO - Last In, First Out)
5. You can apply, pop, or drop stashed changes later

### Understanding the Stash Stack

```
Stash Stack (LIFO):
┌─────────────────────────┐
│ stash@{0} ← Most recent │  ← Top of stack
├─────────────────────────┤
│ stash@{1}               │
├─────────────────────────┤
│ stash@{2}               │
├─────────────────────────┤
│ stash@{3} ← Oldest      │  ← Bottom of stack
└─────────────────────────┘
```

### Important Notes

- Stashes are **local** - they don't transfer to remote repositories
- Stash entries are referenced by `stash@{n}` where n is the index
- `stash@{0}` is always the most recent stash
- Stashes can include both staged and unstaged changes
- Untracked and ignored files are **not** stashed by default

---

## Basic Usage

### Stash Your Changes

```bash
# Stash all changes (staged and unstaged tracked files)
git stash

# Stash with a descriptive message
git stash save "WIP: implementing user authentication"

# Modern syntax (Git 2.16+)
git stash push -m "WIP: implementing user authentication"

# Stash including untracked files
git stash -u
git stash --include-untracked

# Stash including ignored files (and untracked)
git stash -a
git stash --all

# Stash only unstaged changes (keep staged changes)
git stash --keep-index
```

### View Stashed Changes

```bash
# List all stashes
git stash list

# Show latest stash details
git stash show

# Show latest stash with full diff
git stash show -p

# Show specific stash
git stash show stash@{1}
git stash show stash@{1} -p

# Show stash statistics
git stash show --stat
```

### Apply Stashed Changes

```bash
# Apply latest stash (keeps it in stash stack)
git stash apply

# Apply specific stash
git stash apply stash@{2}

# Apply latest stash and remove it from stack
git stash pop

# Apply specific stash and remove it
git stash pop stash@{1}
```

### Remove Stashed Changes

```bash
# Remove latest stash
git stash drop

# Remove specific stash
git stash drop stash@{2}

# Remove all stashes
git stash clear

# Apply and drop in one command (equivalent to pop)
git stash apply && git stash drop
```

---

## Advanced Operations

### 1. Stash Specific Files

```bash
# Stash only specific files
git stash push -m "Stashing config files" config/database.yml config/settings.json

# Stash files matching a pattern
git stash push -m "Stashing all test files" -- "**/*test.js"

# Stash specific file types
git stash push -m "Stashing CSS changes" -- "*.css"
```

### 2. Stash with Pathspec

```bash
# Stash everything except specific files
git stash push -- . ':!config/secrets.yml'

# Stash only files in a directory
git stash push -m "Stashing frontend changes" -- src/frontend/

# Stash with complex patterns
git stash push -m "Stashing components" -- "src/components/**/*.tsx"
```

### 3. Create Stash from Diff

```bash
# Create stash but keep changes in working directory
git stash create
# Returns: SHA of stash object (e.g., 1a2b3c4d)

# Store the stash with a custom reference
git stash store -m "Custom stash message" 1a2b3c4d
```

### 4. Stash Interactive Mode

```bash
# Interactively select hunks to stash
git stash -p
git stash --patch

# You'll be prompted for each hunk:
# y - stash this hunk
# n - do not stash this hunk
# q - quit (don't stash remaining hunks)
# a - stash this hunk and all remaining hunks in file
# d - do not stash this hunk or any remaining hunks in file
# s - split the current hunk into smaller hunks
# e - manually edit the current hunk
```

### 5. Create Branch from Stash

```bash
# Create a new branch and apply stash
git stash branch feature-branch

# Create branch from specific stash
git stash branch feature-branch stash@{2}

# This is useful when stash has conflicts with current branch
```

### 6. Stash with Index

```bash
# Stash but preserve staging area
git stash --keep-index

# Stash staged changes only
git stash push --staged  # Git 2.35+

# Stash everything except staged changes
git stash push --keep-index
git restore --staged .  # Unstage everything after stash
```

---

## Real-World Examples

### Example 1: Urgent Hotfix During Feature Development

**Scenario:** You're working on a feature when an urgent bug report comes in.

```bash
# You're on feature-branch with uncommitted changes
git status
# Modified: src/UserService.js, src/utils/helpers.js

# Stash your feature work
git stash save "WIP: user profile feature - validation logic"

# Switch to main branch
git checkout main

# Create hotfix branch
git checkout -b hotfix-login-bug

# Fix the bug
vim src/AuthService.js
git add src/AuthService.js
git commit -m "Fix: Login session timeout issue"

# Deploy the fix
git push origin hotfix-login-bug

# Return to your feature work
git checkout feature-branch
git stash pop

# Continue working on your feature
```

### Example 2: Pull with Local Changes

**Scenario:** You need to pull updates but have uncommitted local changes.

```bash
# You have local changes
git status
# Modified: README.md, src/app.js

# Try to pull
git pull origin main
# error: Your local changes would be overwritten by merge

# Stash your changes
git stash

# Pull the updates
git pull origin main

# Reapply your changes
git stash pop

# If conflicts occur, resolve them
# Conflicts in src/app.js
vim src/app.js  # Resolve conflicts
git add src/app.js
# stash is automatically dropped after successful pop
```

### Example 3: Testing Without Recent Changes

**Scenario:** You want to test if a bug exists without your recent changes.

```bash
# You've made changes that might have introduced a bug
git stash save "Testing: temporarily removing my changes"

# Run tests or reproduce bug
npm test

# Bug still exists - it wasn't your changes
# Apply your changes back
git stash pop

# Or if bug is fixed - your changes caused it
# Review what you changed
git stash show -p
# Keep changes stashed while you investigate
```

### Example 4: Moving Work Between Branches

**Scenario:** You started working on the wrong branch.

```bash
# You're on main branch with uncommitted changes
git status
# Modified: src/NewFeature.js
# Oh no! Should be on feature-branch!

# Stash changes
git stash save "Feature work started on wrong branch"

# Switch to correct branch (or create it)
git checkout -b feature-new-feature

# Apply stashed changes
git stash pop

# Continue working
vim src/NewFeature.js
git add src/NewFeature.js
git commit -m "Add new feature implementation"
```

### Example 5: Partial Stashing

**Scenario:** You want to stash only some of your changes.

```bash
# You have changes in multiple files
git status
# Modified: src/feature.js (working changes)
# Modified: src/debug.js (debugging code to remove)
# Modified: config/test.yml (temporary test config)

# Stash only the files you want
git stash push -m "Stashing temporary debug code" src/debug.js config/test.yml

# Now only src/feature.js remains modified
git status
# Modified: src/feature.js

# Commit your real work
git add src/feature.js
git commit -m "Implement feature X"

# Discard the stashed debug code
git stash drop
```

### Example 6: Interactive Stashing

**Scenario:** You want to stash specific hunks from a file.

```bash
# You have one file with multiple logical changes
# src/UserService.js has:
# - Feature work (lines 10-50)
# - Debug console.logs (lines 100-110)
# - Experimental code (lines 200-250)

# Interactively stash
git stash -p

# For feature work hunks: press 'n' (don't stash)
# For debug logs: press 'y' (stash)
# For experimental code: press 'y' (stash)

# Now only feature work remains in working directory
git add src/UserService.js
git commit -m "Implement user validation"

# Later, review stashed experimental code
git stash show -p
# Decide whether to apply it back or drop it
```

### Example 7: Stash with Untracked Files

**Scenario:** You have new files that aren't committed yet.

```bash
# You created new files
git status
# Modified: src/existing.js
# Untracked: src/newfile.js
# Untracked: temp/debug.log

# Regular stash won't include untracked files
git stash
git status
# Untracked: src/newfile.js  ← Still there!
# Untracked: temp/debug.log

# Pop and try again with untracked files
git stash pop

# Stash including untracked files
git stash -u
git status
# Clean! Everything is stashed

# Apply back
git stash pop
```

### Example 8: Stash Before Rebase

**Scenario:** You want to rebase but have uncommitted changes.

```bash
# You're on feature-branch with local changes
git status
# Modified: src/feature.js

# Can't rebase with dirty working directory
git rebase main
# error: cannot rebase with uncommitted changes

# Stash changes
git stash save "WIP before rebase on main"

# Now rebase
git rebase main

# If rebase successful, apply changes
git stash pop

# If conflicts after pop, resolve them
vim src/feature.js
git add src/feature.js
# stash automatically drops after successful pop
```

### Example 9: Managing Multiple Stashes

**Scenario:** You're juggling multiple work-in-progress tasks.

```bash
# Working on feature A
vim src/featureA.js
git stash save "WIP: Feature A - user authentication"

# Switch to feature B
git checkout feature-b
vim src/featureB.js
git stash save "WIP: Feature B - payment integration"

# Urgent bug fix needed
git checkout main
vim src/bugfix.js
git add src/bugfix.js
git commit -m "Fix critical bug"

# List your stashes
git stash list
# stash@{0}: On feature-b: WIP: Feature B - payment integration
# stash@{1}: On main: WIP: Feature A - user authentication

# Resume feature B
git checkout feature-b
git stash apply stash@{0}

# Later, resume feature A
git checkout main
git stash apply stash@{1}

# Clean up applied stashes
git stash drop stash@{0}
git stash drop stash@{0}  # Index shifts after first drop
```

### Example 10: Stash and Cherry-Pick

**Scenario:** Combine stash with cherry-pick for complex workflows.

```bash
# You're on feature-branch with mixed changes
# Some changes should go to main as hotfix
# Other changes are feature work

# Stash everything
git stash save "Mixed changes - need to separate"

# Create hotfix branch
git checkout main
git checkout -b hotfix-urgent

# Apply stash without committing
git stash apply --index

# Interactively select hotfix changes
git add -p
# Select only hotfix hunks

# Commit hotfix
git commit -m "Hotfix: Critical security patch"
git push origin hotfix-urgent

# Discard remaining changes
git restore .

# Return to feature branch
git checkout feature-branch

# Reapply stash to get all changes back
git stash pop

# Interactively select feature changes
git add -p
git commit -m "Feature: New user dashboard"
```

---

## Stash vs Other Git Operations

### Stash vs Commit

| Stash | Commit |
|-------|--------|
| Temporary storage | Permanent history |
| Not shared with team | Pushed to remote |
| Quick and dirty | Requires meaningful message |
| Local only | Part of project history |
| Easy to discard | Hard to undo once pushed |

**Example:**
```bash
# Commit: For real progress
git add src/feature.js
git commit -m "Implement user authentication"

# Stash: For temporary interruption
git stash save "WIP: half-done authentication"
```

### Stash vs Branch

| Stash | Branch |
|-------|--------|
| Temporary/short-term | Long-term development |
| Anonymous changes | Named development line |
| Stack-based | Tree-based |
| Quick context switch | Formal feature work |

**Example:**
```bash
# Branch: For proper feature development
git checkout -b feature-authentication
# Work, commit, push, create PR

# Stash: For quick interruptions
git stash  # Handle urgent issue
git stash pop  # Resume work
```

### Stash vs WIP Commit

| Stash | WIP Commit |
|-------|------------|
| Not in history | Clutters history |
| Easy to discard | Requires amend/rebase |
| Local only | May be pushed |
| No commit message | Requires message |

**Example:**
```bash
# WIP Commit (less ideal)
git add .
git commit -m "WIP - don't merge"
# Later: git rebase -i to clean up

# Stash (cleaner)
git stash save "WIP: feature work"
# Later: git stash pop and commit properly
```

### Stash vs Worktree

| Stash | Worktree |
|-------|----------|
| Switch in same directory | Separate directories |
| Quick context switch | Multiple simultaneous tasks |
| One context at a time | Multiple contexts active |
| No disk duplication | Uses more disk space |

**Example:**
```bash
# Stash: Quick switch
git stash
git checkout hotfix-branch
# Fix
git checkout main
git stash pop

# Worktree: Parallel work
git worktree add ../hotfix hotfix-branch
# Work in both directories simultaneously
```

---

## Best Practices

### 1. Always Use Descriptive Messages

```bash
# Bad: No context
git stash

# Good: Descriptive message
git stash save "WIP: User authentication - implementing JWT validation"

# Better: Even more context
git stash save "WIP: Sprint 23 - User auth - JWT validation + unit tests in progress"
```

### 2. Review Stash Before Applying

```bash
# Before applying, review what's in the stash
git stash show -p

# Check which files will be affected
git stash show --name-only

# If it looks good, then apply
git stash pop
```

### 3. Keep Stash Stack Small

```bash
# Bad: Accumulating many stashes
git stash list
# stash@{0}: WIP: feature
# stash@{1}: WIP: another feature
# stash@{2}: WIP: old work
# ... 15 more stashes

# Good: Clean up regularly
git stash list
# stash@{0}: WIP: current work

# Clean old stashes
git stash clear  # Remove all
# Or selectively drop
git stash drop stash@{1}
```

### 4. Don't Use Stash as Long-Term Storage

```bash
# Bad: Stashing for days/weeks
git stash save "Feature I'll work on someday"
# ... weeks later ...
# What was this again?

# Good: Create a branch instead
git checkout -b feature-for-later
git add .
git commit -m "WIP: Feature initial work"
git push origin feature-for-later
```

### 5. Test After Applying Stash

```bash
# Apply stash
git stash pop

# Always test
npm test
npm run build

# If tests fail, investigate
git diff stash@{0}  # If you haven't dropped it yet
```

### 6. Use `apply` Instead of `pop` When Unsure

```bash
# Use apply when you might need the stash again
git stash apply

# Test your changes
npm test

# If everything works, then drop
git stash drop

# If something breaks, you still have the stash
git reset --hard HEAD
git stash show -p  # Review what was in stash
```

### 7. Stash Before Risky Operations

```bash
# Before risky operations, stash as backup
git stash save "Backup before risky rebase"

# Perform risky operation
git rebase -i HEAD~10

# If successful, drop the backup stash
git stash drop

# If something goes wrong
git rebase --abort
git stash pop  # Restore your work
```

---

## Troubleshooting

### Problem 1: Stash Apply Conflicts

**Symptoms:**
```bash
$ git stash pop
Auto-merging src/app.js
CONFLICT (content): Merge conflict in src/app.js
```

**Solution:**
```bash
# Stash creates conflicts when code changed since stashing

# Option 1: Resolve conflicts manually
vim src/app.js  # Resolve conflicts
git add src/app.js
# Don't need to commit - stash pop doesn't create commit

# Remove conflict markers
git status  # Should show clean state
git stash drop  # Manually drop since pop failed

# Option 2: Abort and review
git reset --hard HEAD  # Discard conflict
git stash show -p  # Review what was in stash
# Manually apply changes
```

### Problem 2: Lost Stash After Failed Pop

**Symptoms:**
```bash
$ git stash pop
# Conflicts occurred
$ git stash list
# stash@{0} is gone!
```

**Solution:**
```bash
# Don't panic! Stash is not deleted on failed pop

# Check if stash still exists
git stash list
# If not listed, it's been dropped

# Recover from reflog
git fsck --unreachable | grep commit | cut -d ' ' -f3 | xargs git log --merges --no-walk --grep=WIP

# Or find in reflog
git log --graph --oneline --decorate $(git rev-list --walk-reflogs --all)

# Once found (e.g., abc123)
git stash apply abc123
```

### Problem 3: Accidentally Cleared All Stashes

**Solution:**
```bash
# Find stash commits in reflog
git fsck --no-reflog | awk '/dangling commit/ {print $3}'

# Or
git reflog | grep "WIP"

# View potential stashes
git show <commit-hash>

# Recover specific stash
git stash apply <commit-hash>

# Create new stash from recovered commit
git stash store -m "Recovered stash" <commit-hash>
```

### Problem 4: Can't Stash - "No Local Changes"

**Symptoms:**
```bash
$ git stash
No local changes to save
```

**Solution:**
```bash
# Check what's actually changed
git status

# If you see untracked files
git stash -u  # Include untracked files

# If changes are in gitignored files
git stash -a  # Include all files

# If changes are already staged
git stash --include-untracked  # Will include staged changes
```

### Problem 5: Stash Apply Doesn't Restore Staged Files

**Solution:**
```bash
# By default, apply doesn't restore index (staging area)

# Apply with index restoration
git stash apply --index

# Or when popping
git stash pop --index

# Verify staging area
git status
```

### Problem 6: Can't Switch Branches - "Working Directory Not Clean"

**Symptoms:**
```bash
$ git checkout other-branch
error: Your local changes would be overwritten by checkout
```

**Solution:**
```bash
# Stash changes first
git stash save "Changes before switching to other-branch"

# Switch branch
git checkout other-branch

# If you need changes on new branch
git stash pop

# If changes should stay on original branch
# Just leave them stashed and switch back later
git checkout original-branch
git stash pop
```

---

## Common Pitfalls

### Pitfall 1: Forgetting What's in Old Stashes

**Problem:**
```bash
$ git stash list
stash@{0}: WIP on main: 1a2b3c4 Some commit
stash@{1}: WIP on feature: 5d6e7f8 Another commit
stash@{2}: WIP on main: 9g0h1i2 Old commit
# What's in stash@{2}?
```

**Solution:**
```bash
# Always use descriptive messages
git stash save "WIP: User dashboard - adding charts component"

# Review stashes regularly
git stash show -p stash@{2}

# Clean up old stashes
git stash drop stash@{2}
```

### Pitfall 2: Stashing Untracked Files by Accident

**Problem:** Forgot to commit new files before stashing with `-u`.

**Solution:**
```bash
# Be explicit about what you're stashing
git stash list
git stash show --name-only stash@{0}

# If new files are in there
git stash apply stash@{0}
git add new-file.js
git commit -m "Add new file"
git stash drop stash@{0}
```

### Pitfall 3: Using Stash Instead of Proper Commits

**Problem:**
```bash
# Anti-pattern: Using stash as version control
git stash save "Version 1"
# Work...
git stash save "Version 2"
# Work...
git stash save "Version 3"
```

**Solution:**
```bash
# Use commits for versions
git add .
git commit -m "Version 1: Initial implementation"
# Work...
git add .
git commit -m "Version 2: Add validation"
# Work...
git add .
git commit -m "Version 3: Refactor structure"
```

### Pitfall 4: Not Testing After Stash Pop

**Problem:** Code breaks after applying stash but you don't notice.

**Solution:**
```bash
# Always test after applying stash
git stash pop

# Run tests
npm test
npm run lint
npm run build

# If tests fail
git diff  # See what changed
git stash show -p stash@{0}  # Compare with original stash
```

### Pitfall 5: Stashing During Merge/Rebase

**Problem:**
```bash
# During merge with conflicts
git merge feature-branch
# CONFLICT in src/app.js

# Bad: Stashing to "deal with it later"
git stash  # This won't work as expected
```

**Solution:**
```bash
# Complete or abort merge/rebase first

# Option 1: Complete merge
vim src/app.js  # Resolve conflicts
git add src/app.js
git commit

# Option 2: Abort merge
git merge --abort

# Then you can stash
git stash save "Changes before merge"
```

### Pitfall 6: Assuming Stash is Permanent

**Problem:** Treating stash as backup storage.

**Solution:**
```bash
# Bad: Long-term storage
git stash save "Important work"
# ... months later ...
git gc --prune=now  # Might lose unreferenced stashes

# Good: Use branches for important work
git checkout -b backup-important-work
git add .
git commit -m "Important work backup"
git push origin backup-important-work
```

---

## Tips and Tricks

### Tip 1: Alias for Better Stash Workflow

```bash
# Add to ~/.gitconfig
[alias]
    # Stash with message prompt
    st = "!f() { git stash save \"$1\"; }; f"
    
    # List stashes with details
    stl = stash list --pretty=format:\"%C(yellow)%gd%C(reset) - %C(green)(%ar)%C(reset) %s\"
    
    # Show latest stash
    sts = stash show -p
    
    # Apply latest and drop
    stp = stash pop
    
    # Apply without dropping
    sta = stash apply
    
    # Drop stash interactively
    std = "!f() { git stash drop stash@{${1:-0}}; }; f"
    
    # Stash untracked
    stu = stash save -u
    
    # Stash keeping staged
    stk = stash save --keep-index

# Usage
git st "WIP: authentication feature"
git stl  # Pretty list
git stp  # Pop latest
```

### Tip 2: Interactive Stash Selection

```bash
# Create function for interactive stash selection
# Add to ~/.bashrc or ~/.zshrc

git-stash-select() {
    local stash
    stash=$(git stash list | fzf | cut -d: -f1)
    if [ -n "$stash" ]; then
        git stash show -p "$stash"
        echo "\nApply this stash? (y/n)"
        read response
        if [ "$response" = "y" ]; then
            git stash apply "$stash"
        fi
    fi
}

# Usage
git-stash-select
```

### Tip 3: Stash Specific Lines

```bash
# Interactively stash specific hunks
git stash -p

# For each hunk, you can also split it
# Press 's' to split into smaller hunks
# Press 'e' to manually edit the hunk

# This gives you line-level control
```

### Tip 4: Diff Between Stashes

```bash
# Compare two stashes
git diff stash@{0} stash@{1}

# Compare stash with working directory
git diff stash@{0}

# Compare stash with HEAD
git diff HEAD stash@{0}

# Compare specific file in stash
git diff stash@{0} -- src/app.js
```

### Tip 5: Stash Stats

```bash
# Show statistics for each stash
for sha in $(git stash list | cut -d: -f1); do
    echo "=== $sha ==="
    git stash show --stat $sha
    echo ""
done

# Or create an alias
git config --global alias.stash-stats '!for sha in $(git stash list | cut -d: -f1); do echo "=== $sha ==="; git stash show --stat $sha; echo ""; done'
```

### Tip 6: Export and Import Stashes

```bash
# Export stash to patch file
git stash show -p stash@{0} > my-stash.patch

# Share patch file with team or move to another machine

# Import on another machine/repository
git apply my-stash.patch

# Or apply with commit info
git am my-stash.patch
```

### Tip 7: Stash and Switch in One Command

```bash
# Create alias for stash and switch
git config --global alias.stash-switch '!f() { git stash save "Auto-stash from $(git branch --show-current)" && git checkout "$1"; }; f'

# Usage
git stash-switch feature-branch

# Switches to feature-branch with current work stashed
```

### Tip 8: Auto-Stash with Rebase

```bash
# Git can automatically stash before rebase
git rebase --autostash main

# Or configure it globally
git config --global rebase.autoStash true

# Now rebase will auto-stash and auto-apply
git rebase main
```

### Tip 9: Stash Only Specific File Types

```bash
# Stash only JavaScript files
git stash push -m "Stashing JS changes" -- "*.js"

# Stash only files in src directory
git stash push -m "Stashing src" -- "src/*"

# Stash everything except tests
git stash push -m "Non-test changes" -- . ':!**/*test.js'
```

### Tip 10: Recover Deleted Stash

```bash
# Find lost stashes using fsck
git fsck --unreachable | grep commit | cut -d ' ' -f3 | xargs git log --merges --no-walk --grep=WIP

# Or use reflog
git log --graph --oneline --decorate $(git rev-list --walk-reflogs --all) | grep "WIP"

# Once you find the SHA
git stash apply <sha>

# Store it permanently
git stash store -m "Recovered stash" <sha>
```

### Tip 11: Stash Show with Context

```bash
# Show stash with more context lines
git stash show -p --unified=10 stash@{0}

# Show stash with word diff
git stash show -p --word-diff stash@{0}

# Show stash with color-coded changes
git stash show -p --color-words stash@{0}
```

### Tip 12: Conditional Stashing in Scripts

```bash
#!/bin/bash
# auto-update.sh - Update branch with auto-stashing

# Check if working directory is dirty
if ! git diff-index --quiet HEAD --; then
    echo "Working directory dirty, stashing..."
    git stash save "Auto-stash before update $(date)"
    STASHED=true
else
    STASHED=false
fi

# Update branch
git pull origin main

# Run tests
if npm test; then
    echo "Tests passed!"
    
    # Apply stash if we created one
    if [ "$STASHED" = true ]; then
        echo "Applying stash..."
        git stash pop
    fi
else
    echo "Tests failed!"
    if [ "$STASHED" = true ]; then
        echo "Stash preserved: $(git stash list | head -1)"
    fi
    exit 1
fi
```

---

## Advanced Stash Scenarios

### Scenario: Partial Stash with Regex

```bash
# Stash files matching complex pattern
git stash push -m "Stashing all React components" -- "src/components/**/*.tsx" "src/components/**/*.jsx"

# Stash everything except node_modules and build
git stash push -- . ':!node_modules' ':!build' ':!dist'
```

### Scenario: Stash Pipeline

```bash
# Create a workflow with multiple stashes
# Working on feature A
git stash save "Feature A - part 1"

# Quick fix
git stash save "Feature A - part 2"

# Another interruption
git stash save "Feature A - part 3"

# Apply them back in order
git stash apply stash@{2}  # Part 1
git stash apply stash@{1}  # Part 2
git stash apply stash@{0}  # Part 3

# Test combined changes
npm test

# Commit if good
git add .
git commit -m "Complete feature A"

# Clean up stashes
git stash clear
```

### Scenario: Stash as Temporary Backup

```bash
# Before risky operation, create backup
git stash save "BACKUP before cleanup $(date +%Y-%m-%d_%H-%M-%S)"

# Perform risky operation
git reset --hard HEAD~10
git clean -fdx
# ... other risky operations

# If something goes wrong
git stash list
git stash apply stash@{0}  # Restore from backup

# If everything is fine
git stash drop stash@{0}  # Remove backup
```

### Scenario: Stash Grep

```bash
# Search through all stashes
search_stashes() {
    local pattern=$1
    for stash in $(git stash list | cut -d: -f1); do
        if git stash show -p $stash | grep -q "$pattern"; then
            echo "Found in $stash:"
            git stash show $stash
            echo ""
        fi
    done
}

# Usage
search_stashes "UserService"
```

---

## Stash Best Practices Summary

✅ **Do:**
- Use descriptive messages for every stash
- Review stashes before applying
- Keep stash stack small and clean
- Test after applying stashes
- Use stash for short-term storage only
- Include `-u` when you have untracked files
- Use `apply` before `pop` when unsure

❌ **Don't:**
- Use stash as long-term storage (use branches)
- Let stashes accumulate indefinitely
- Forget to test after applying
- Stash without a descriptive message
- Use stash to avoid dealing with merge conflicts
- Treat stash as permanent backup
- Forget that stash is local only

---

## Quick Reference

```bash
# Basic operations
git stash                          # Stash current changes
git stash save "message"           # Stash with message
git stash push -m "msg" file.js    # Stash specific files
git stash -u                       # Stash including untracked
git stash -a                       # Stash including ignored
git stash -p                       # Interactive stashing

# View stashes
git stash list                     # List all stashes
git stash show                     # Show latest stash
git stash show -p                  # Show latest with diff
git stash show stash@{1}           # Show specific stash

# Apply stashes
git stash apply                    # Apply latest stash
git stash apply stash@{1}          # Apply specific stash
git stash apply --index            # Apply with staging info
git stash pop                      # Apply and remove latest
git stash pop stash@{1}            # Pop specific stash

# Remove stashes
git stash drop                     # Remove latest stash
git stash drop stash@{1}           # Remove specific stash
git stash clear                    # Remove all stashes

# Advanced operations
git stash branch new-branch        # Create branch from stash
git stash --keep-index             # Stash only unstaged
git stash create                   # Create stash without applying
git stash store -m "msg" <sha>     # Store created stash
```

---

## Stash Workflow Diagram

```
Working Directory (Modified)
         │
         │ git stash save "message"
         ↓
   Stash Stack
   ┌─────────────────┐
   │ stash@{0}       │ ← Latest
   │ stash@{1}       │
   │ stash@{2}       │
   └─────────────────┘
         │
         │ git stash apply/pop
         ↓
Working Directory (Modified + Stashed)
         │
         │ git add && git commit
         ↓
   Git History (Committed)
```

---

**Last Updated:** November 1, 2025  
**Version:** 1.0


