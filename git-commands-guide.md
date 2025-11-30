# Comprehensive Git Commands Guide

## Table of Contents
1. [Getting Started](#getting-started)
2. [Basic Commands](#basic-commands)
3. [Branching and Merging](#branching-and-merging)
4. [Remote Operations](#remote-operations)
5. [Inspecting History](#inspecting-history)
6. [Undoing Changes](#undoing-changes)
7. [Advanced Operations](#advanced-operations)
8. [Collaboration Workflows](#collaboration-workflows)
9. [Best Practices](#best-practices)

---

## Getting Started

### git init
**Purpose:** Initialize a new Git repository

**Syntax:**
```bash
git init [directory]
```

**Real-World Example:**
```bash
# Start a new project
mkdir my-magento-extension
cd my-magento-extension
git init

# Initialize with a specific branch name
git init --initial-branch=main
```

**Use Cases:**
- Starting a new project from scratch
- Converting an existing project to use Git
- Creating a local repository before connecting to remote

---

### git clone
**Purpose:** Create a copy of an existing repository

**Syntax:**
```bash
git clone <repository-url> [directory]
```

**Real-World Examples:**
```bash
# Clone a repository
git clone https://github.com/magento/magento2.git

# Clone into a specific directory
git clone https://github.com/magento/magento2.git my-magento

# Clone a specific branch
git clone -b develop https://github.com/magento/magento2.git

# Shallow clone (last 10 commits only - faster for large repos)
git clone --depth 10 https://github.com/magento/magento2.git

# Clone with submodules
git clone --recurse-submodules https://github.com/magento/magento2.git
```

**Use Cases:**
- Getting started with an existing project
- Creating a local development environment
- Downloading open-source projects
- Setting up CI/CD pipelines

---

## Basic Commands

### git status
**Purpose:** Show the working tree status

**Syntax:**
```bash
git status [options]
```

**Real-World Examples:**
```bash
# Standard status check
git status

# Short format (more concise)
git status -s

# Show branch and tracking info
git status -sb

# Show untracked files
git status -u
```

**Use Cases:**
- Before committing: check what files will be included
- After pulling: see if there are conflicts
- Daily workflow: understand current state
- Before switching branches: ensure clean state

**Output Interpretation:**
```
On branch AC-15635                    # Current branch
Your branch is up to date             # Sync status with remote
Changes to be committed:              # Staged changes (will be in next commit)
Changes not staged for commit:        # Modified but not staged
Untracked files:                      # New files not tracked by Git
```

---

### git add
**Purpose:** Add file contents to the staging area

**Syntax:**
```bash
git add <file>...
```

**Real-World Examples:**
```bash
# Add a single file
git add app/code/Magento/Catalog/Model/Product.php

# Add multiple specific files
git add file1.php file2.php file3.php

# Add all PHP files in current directory
git add *.php

# Add all files in a directory
git add app/code/Magento/Catalog/

# Add all modified and new files
git add .

# Add all files including deletions
git add -A

# Add only modified files (not new files)
git add -u

# Interactive staging (choose hunks)
git add -p ProductTest.php

# Add parts of a file interactively
git add --patch file.php
```

**Interactive Staging Example:**
```bash
git add -p app/code/Magento/Catalog/Model/Product.php
# You'll see each change and can choose:
# y - stage this hunk
# n - do not stage this hunk
# s - split into smaller hunks
# e - manually edit the hunk
# q - quit (don't stage remaining hunks)
```

**Use Cases:**
- Preparing files for commit
- Staging specific changes while leaving others unstaged
- Creating atomic commits (one logical change per commit)
- Selectively staging parts of a file

---

### git commit
**Purpose:** Record changes to the repository

**Syntax:**
```bash
git commit [options]
```

**Real-World Examples:**
```bash
# Basic commit with message
git commit -m "Fix product price calculation bug"

# Commit with detailed message
git commit -m "Fix product price calculation" -m "- Updated tier pricing logic
- Fixed currency conversion issue
- Added validation for negative prices"

# Commit all modified files (skip git add)
git commit -am "Update configuration files"

# Commit with editor (for longer messages)
git commit

# Amend last commit (change message or add files)
git commit --amend

# Amend without changing message
git commit --amend --no-edit

# Empty commit (for triggering CI)
git commit --allow-empty -m "Trigger CI rebuild"

# Commit with specific author
git commit --author="John Doe <john@example.com>" -m "Fix bug"

# Commit with date
git commit --date="2024-10-15 10:30:00" -m "Backdated commit"
```

**Commit Message Best Practices:**
```bash
# Good commit message structure:
git commit -m "Add customer validation to checkout process

- Implement email validation in checkout form
- Add phone number format verification
- Update customer data model with validation rules
- Add unit tests for validation logic

Fixes #1234"
```

**Common Commit Patterns:**
```bash
# Bug fix
git commit -m "Fix: Resolve null pointer in product listing"

# New feature
git commit -m "Feature: Add gift card support to checkout"

# Refactoring
git commit -m "Refactor: Extract payment processing to service class"

# Documentation
git commit -m "Docs: Update API documentation for customer endpoint"

# Performance
git commit -m "Perf: Optimize product collection query"

# Tests
git commit -m "Test: Add unit tests for order calculation"
```

**Use Cases:**
- Creating a checkpoint in development
- Recording a logical unit of work
- Contributing to a project
- Creating a revertible point in history

---

### git diff
**Purpose:** Show changes between commits, working tree, etc.

**Syntax:**
```bash
git diff [options] [<commit>] [<path>...]
```

**Real-World Examples:**
```bash
# Show unstaged changes
git diff

# Show staged changes (what will be committed)
git diff --staged
git diff --cached  # Same as --staged

# Show changes in a specific file
git diff app/code/Magento/Catalog/Model/Product.php

# Compare two branches
git diff main..feature-branch

# Compare current branch with main
git diff main

# Show changes between two commits
git diff abc123..def456

# Show changes in last commit
git diff HEAD~1 HEAD

# Show only file names that changed
git diff --name-only

# Show file names with status (Added, Modified, Deleted)
git diff --name-status

# Show statistics (how many lines changed)
git diff --stat

# Word-level diff (better for prose)
git diff --word-diff

# Ignore whitespace changes
git diff -w

# Show diff with more context lines
git diff -U10

# Compare with remote branch
git diff origin/main

# Show changes in a commit
git diff abc123^!
```

**Use Cases:**
- Reviewing changes before committing
- Understanding what changed between versions
- Code review preparation
- Debugging: finding what changed
- Creating patches

---

### git log
**Purpose:** Show commit history

**Syntax:**
```bash
git log [options] [<path>...]
```

**Real-World Examples:**
```bash
# Basic log
git log

# One line per commit
git log --oneline

# Show last 5 commits
git log -5

# Show commits with full diff
git log -p

# Show commits that modified a specific file
git log -- app/code/Magento/Catalog/Model/Product.php

# Show commits by author
git log --author="John Doe"

# Show commits since a date
git log --since="2024-10-01"
git log --since="2 weeks ago"

# Show commits in date range
git log --since="2024-10-01" --until="2024-10-31"

# Show commits with search in message
git log --grep="bug fix"

# Show commits that changed specific code
git log -S "calculatePrice" --source --all

# Graphical representation
git log --graph --oneline --all

# Pretty format
git log --pretty=format:"%h - %an, %ar : %s"

# Show commits with files changed
git log --stat

# Show commits with file names only
git log --name-only

# Show commits that touched specific lines
git log -L 100,150:app/code/Magento/Catalog/Model/Product.php

# Show merge commits only
git log --merges

# Exclude merge commits
git log --no-merges

# Show commits on current branch not in main
git log main..HEAD

# Show commits in main not in current branch
git log HEAD..main

# Beautiful graph view
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```

**Custom Git Log Alias:**
```bash
# Add to ~/.gitconfig
[alias]
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
    
# Usage
git lg
```

**Use Cases:**
- Understanding project history
- Finding when a bug was introduced
- Tracking who made specific changes
- Generating release notes
- Code archaeology

---

## Branching and Merging

### git branch
**Purpose:** List, create, or delete branches

**Syntax:**
```bash
git branch [options] [<branch>]
```

**Real-World Examples:**
```bash
# List all local branches
git branch

# List all branches (local and remote)
git branch -a

# List remote branches only
git branch -r

# Create a new branch
git branch feature/AC-15635-add-product-validation

# Create branch from specific commit
git branch bugfix/fix-checkout abc123

# Delete a branch (safe - prevents deleting unmerged)
git branch -d feature/old-feature

# Force delete a branch
git branch -D feature/abandoned-feature

# Rename current branch
git branch -m new-name

# Rename a different branch
git branch -m old-name new-name

# Show branches with last commit
git branch -v

# Show merged branches
git branch --merged

# Show unmerged branches
git branch --no-merged

# Show branches that contain a commit
git branch --contains abc123

# Set upstream for current branch
git branch --set-upstream-to=origin/main

# Copy a branch
git branch new-branch existing-branch
```

**Use Cases:**
- Organizing feature development
- Isolating bug fixes
- Managing multiple versions
- Experimenting without affecting main code

---

### git checkout
**Purpose:** Switch branches or restore working tree files

**Syntax:**
```bash
git checkout [options] <branch|commit|file>
```

**Real-World Examples:**
```bash
# Switch to existing branch
git checkout feature/AC-15635

# Create and switch to new branch
git checkout -b feature/new-payment-method

# Create branch from specific commit
git checkout -b hotfix/critical-bug abc123

# Switch to previous branch
git checkout -

# Checkout a specific commit (detached HEAD)
git checkout abc123

# Checkout a file from another branch
git checkout main -- app/code/Magento/Catalog/Model/Product.php

# Discard changes in a file
git checkout -- file.php

# Checkout all files from specific commit
git checkout abc123 -- .

# Create branch tracking remote
git checkout -b feature/new-feature origin/feature/new-feature

# Checkout and create branch from tag
git checkout -b release-2.4.6 v2.4.6

# Checkout specific file from specific commit
git checkout abc123 -- app/code/Magento/Catalog/Model/Product.php
```

**Use Cases:**
- Switching between features
- Reviewing old code versions
- Recovering deleted files
- Discarding unwanted changes
- Testing different versions

**⚠️ Note:** For more detailed information about checkout, reset, and revert, see [git-reset-revert-checkout-guide.md](git-reset-revert-checkout-guide.md)

---

### git switch (Git 2.23+)
**Purpose:** Switch branches (cleaner alternative to checkout)

**Syntax:**
```bash
git switch [options] <branch>
```

**Real-World Examples:**
```bash
# Switch to existing branch
git switch feature/AC-15635

# Create and switch to new branch
git switch -c feature/new-feature

# Switch to previous branch
git switch -

# Create branch from specific start point
git switch -c hotfix/bug origin/main

# Force switch (discard local changes)
git switch -f main

# Switch and merge changes from current branch
git switch -m feature/other-feature
```

**Use Cases:**
- Modern replacement for `git checkout` for branch operations
- Clearer semantics (switch = branches, restore = files)

---

### git restore (Git 2.23+)
**Purpose:** Restore working tree files (cleaner alternative to checkout)

**Syntax:**
```bash
git restore [options] <file>...
```

**Real-World Examples:**
```bash
# Discard changes in file
git restore file.php

# Restore multiple files
git restore file1.php file2.php

# Restore all files
git restore .

# Unstage a file (remove from staging)
git restore --staged file.php

# Restore file from specific commit
git restore --source=abc123 file.php

# Restore file from another branch
git restore --source=main -- app/code/Magento/Catalog/Model/Product.php

# Restore and unstage
git restore --staged --worktree file.php
```

**Use Cases:**
- Discarding local changes
- Unstaging files
- Recovering files from other commits

---

### git merge
**Purpose:** Join two or more development histories together

**Syntax:**
```bash
git merge [options] <branch>...
```

**Real-World Examples:**
```bash
# Merge feature branch into current branch
git checkout main
git merge feature/AC-15635

# Merge with commit message
git merge feature/AC-15635 -m "Merge feature: Add product validation"

# Merge without fast-forward (always create merge commit)
git merge --no-ff feature/AC-15635

# Fast-forward only (fail if not possible)
git merge --ff-only feature/AC-15635

# Squash merge (combine all commits into one)
git merge --squash feature/AC-15635
git commit -m "Add product validation feature"

# Abort merge in case of conflicts
git merge --abort

# Continue merge after resolving conflicts
git merge --continue

# Use strategy for merge
git merge -X theirs feature/AC-15635  # Prefer their changes
git merge -X ours feature/AC-15635    # Prefer our changes

# Merge multiple branches
git merge feature1 feature2 feature3

# Merge with specific strategy
git merge --strategy=recursive -X patience feature/AC-15635
```

**Handling Merge Conflicts:**
```bash
# 1. Attempt merge
git merge feature/AC-15635

# 2. If conflicts, check status
git status

# 3. Open conflicted files and resolve
# Look for conflict markers:
<<<<<<< HEAD
Current branch code
=======
Incoming branch code
>>>>>>> feature/AC-15635

# 4. After resolving, stage files
git add resolved-file.php

# 5. Complete merge
git commit
```

**Use Cases:**
- Integrating feature branches
- Updating feature branches with main
- Combining parallel work streams
- Release management

**📚 See Also:** [git-merge-guide.md](git-merge-guide.md) for comprehensive merge scenarios

---

### git rebase
**Purpose:** Reapply commits on top of another base tip

**Syntax:**
```bash
git rebase [options] <upstream> [<branch>]
```

**Real-World Examples:**
```bash
# Rebase current branch onto main
git rebase main

# Rebase feature branch onto main
git rebase main feature/AC-15635

# Interactive rebase (last 5 commits)
git rebase -i HEAD~5

# Continue rebase after resolving conflicts
git rebase --continue

# Skip current commit
git rebase --skip

# Abort rebase
git rebase --abort

# Rebase with autosquash
git rebase -i --autosquash main

# Preserve merge commits
git rebase --preserve-merges main

# Rebase without changing commit dates
git rebase --committer-date-is-author-date main
```

**Interactive Rebase Example:**
```bash
git rebase -i HEAD~3

# Editor opens with:
pick abc123 First commit
pick def456 Second commit
pick ghi789 Third commit

# Change to:
pick abc123 First commit
squash def456 Second commit
fixup ghi789 Third commit

# Commands:
# p, pick = use commit
# r, reword = use commit, but edit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous
# f, fixup = like squash, but discard message
# d, drop = remove commit
```

**Use Cases:**
- Cleaning up commit history before merging
- Updating feature branch with latest main
- Squashing related commits
- Reordering commits
- Removing or editing old commits

**⚠️ Warning:** Never rebase commits that have been pushed to public branches!

---

## Remote Operations

### git remote
**Purpose:** Manage set of tracked repositories

**Syntax:**
```bash
git remote [options]
```

**Real-World Examples:**
```bash
# List remotes
git remote

# List remotes with URLs
git remote -v

# Add new remote
git remote add origin https://github.com/user/repo.git

# Add upstream remote (for forks)
git remote add upstream https://github.com/magento/magento2.git

# Remove remote
git remote remove origin

# Rename remote
git remote rename origin github

# Change remote URL
git remote set-url origin https://github.com/user/new-repo.git

# Show detailed info about remote
git remote show origin

# Prune deleted remote branches
git remote prune origin

# Update remote branches
git remote update
```

**Typical Fork Workflow Setup:**
```bash
# Clone your fork
git clone https://github.com/yourusername/magento2.git

# Add upstream remote
cd magento2
git remote add upstream https://github.com/magento/magento2.git

# Verify
git remote -v
# Output:
# origin    https://github.com/yourusername/magento2.git (fetch)
# origin    https://github.com/yourusername/magento2.git (push)
# upstream  https://github.com/magento/magento2.git (fetch)
# upstream  https://github.com/magento/magento2.git (push)
```

**Use Cases:**
- Connecting to remote repositories
- Managing multiple remotes
- Fork workflows
- Backup repositories

---

### git fetch
**Purpose:** Download objects and refs from another repository

**Syntax:**
```bash
git fetch [options] [<repository>] [<refspec>...]
```

**Real-World Examples:**
```bash
# Fetch from default remote (origin)
git fetch

# Fetch from specific remote
git fetch upstream

# Fetch specific branch
git fetch origin main

# Fetch all remotes
git fetch --all

# Fetch and prune deleted branches
git fetch --prune

# Fetch tags
git fetch --tags

# Fetch without tags
git fetch --no-tags

# Fetch with depth (shallow fetch)
git fetch --depth=1

# Dry run (see what would be fetched)
git fetch --dry-run
```

**Use Cases:**
- Checking for remote updates
- Syncing fork with upstream
- Downloading specific branches
- Updating remote branch information

**Fetch vs Pull:**
- `git fetch`: Downloads changes but doesn't merge
- `git pull`: Downloads and automatically merges

---

### git pull
**Purpose:** Fetch from and integrate with another repository

**Syntax:**
```bash
git pull [options] [<repository> [<refspec>...]]
```

**Real-World Examples:**
```bash
# Pull from default remote and branch
git pull

# Pull from specific remote and branch
git pull origin main

# Pull with rebase instead of merge
git pull --rebase

# Pull with fast-forward only
git pull --ff-only

# Pull all remotes
git pull --all

# Pull with specific strategy
git pull -X theirs  # Prefer remote changes on conflict

# Pull and prune deleted branches
git pull --prune

# Pull specific branch
git pull origin feature/AC-15635

# Pull and allow unrelated histories
git pull --allow-unrelated-histories
```

**Daily Workflow:**
```bash
# Start of day: update local main with remote
git checkout main
git pull origin main

# Update feature branch with latest main
git checkout feature/AC-15635
git pull origin main
```

**Use Cases:**
- Syncing local branch with remote
- Getting latest team changes
- Updating feature branches
- Starting daily work

---

### git push
**Purpose:** Update remote refs along with associated objects

**Syntax:**
```bash
git push [options] [<repository> [<refspec>...]]
```

**Real-World Examples:**
```bash
# Push current branch to remote
git push

# Push to specific remote and branch
git push origin main

# Push and set upstream
git push -u origin feature/AC-15635

# Push all branches
git push --all

# Push tags
git push --tags

# Push specific tag
git push origin v2.4.6

# Delete remote branch
git push origin --delete feature/old-feature
git push origin :feature/old-feature  # Alternative syntax

# Force push (dangerous!)
git push --force

# Force push with lease (safer)
git push --force-with-lease

# Push with dry run
git push --dry-run

# Push to multiple remotes
git push origin main
git push github main
```

**Safe Force Push:**
```bash
# Instead of --force, use --force-with-lease
# This fails if remote has commits you don't have
git push --force-with-lease origin feature/AC-15635
```

**Use Cases:**
- Sharing work with team
- Backing up local work
- Triggering CI/CD pipelines
- Publishing releases

**⚠️ Warning:** Never force push to shared branches like main or develop!

---

## Inspecting History

### git show
**Purpose:** Show various types of objects

**Syntax:**
```bash
git show [options] <object>...
```

**Real-World Examples:**
```bash
# Show last commit
git show

# Show specific commit
git show abc123

# Show commit with stats
git show --stat abc123

# Show specific file in commit
git show abc123:app/code/Magento/Catalog/Model/Product.php

# Show file from specific branch
git show main:composer.json

# Show tag info
git show v2.4.6

# Show commit without diff
git show --no-patch abc123

# Show commit in one line
git show --oneline abc123

# Show commit from 3 commits ago
git show HEAD~3

# Show file as it was 2 commits ago
git show HEAD~2:path/to/file.php
```

**Use Cases:**
- Reviewing a specific commit
- Viewing file history
- Inspecting tags
- Code review

---

### git blame
**Purpose:** Show what revision and author last modified each line

**Syntax:**
```bash
git blame [options] <file>
```

**Real-World Examples:**
```bash
# Basic blame
git blame app/code/Magento/Catalog/Model/Product.php

# Show email addresses
git blame -e Product.php

# Show line numbers
git blame -n Product.php

# Ignore whitespace changes
git blame -w Product.php

# Show specific line range
git blame -L 100,150 Product.php

# Show from specific commit
git blame abc123 Product.php

# Show content and date
git blame -c Product.php

# Show in porcelain format (machine readable)
git blame --porcelain Product.php
```

**Use Cases:**
- Finding who introduced a bug
- Understanding code ownership
- Code archaeology
- Accountability

---

### git bisect
**Purpose:** Use binary search to find the commit that introduced a bug

**Syntax:**
```bash
git bisect <subcommand> [options]
```

**Real-World Examples:**
```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark last known good commit
git bisect good abc123

# Git checks out middle commit, test it
# If bad:
git bisect bad
# If good:
git bisect good

# Continue until bug is found

# End bisect
git bisect reset

# Automated bisect with script
git bisect start HEAD abc123
git bisect run ./test-script.sh
```

**Complete Bisect Session:**
```bash
# Bug exists now, worked 10 commits ago
git bisect start
git bisect bad                    # Current commit is bad
git bisect good HEAD~10           # 10 commits ago was good

# Git checks out middle commit
# Test the code...
git bisect bad                    # Still buggy

# Git checks out another commit
# Test the code...
git bisect good                   # This one works

# Git continues narrowing...
# Eventually:
# abc123 is the first bad commit
# [Shows commit details]

# Finish
git bisect reset
```

**Use Cases:**
- Finding when a bug was introduced
- Regression testing
- Performance degradation tracking
- Large commit history investigation

---

### git grep
**Purpose:** Print lines matching a pattern in tracked files

**Syntax:**
```bash
git grep [options] <pattern> [<path>...]
```

**Real-World Examples:**
```bash
# Search for text in tracked files
git grep "calculatePrice"

# Search with line numbers
git grep -n "calculatePrice"

# Search with count
git grep -c "calculatePrice"

# Case insensitive search
git grep -i "calculateprice"

# Search in specific directory
git grep "calculatePrice" -- app/code/Magento/Catalog/

# Search for whole word
git grep -w "price"

# Search with context (3 lines before and after)
git grep -C 3 "calculatePrice"

# Search in specific commit
git grep "calculatePrice" abc123

# Search and show function name
git grep -p "calculatePrice"

# Search with regex
git grep -E "calculate(Price|Total)"

# Search for files containing all terms
git grep --all-match -e "price" -e "tax"
```

**Use Cases:**
- Finding specific code
- Refactoring
- Understanding codebase
- Finding API usage

---

## Undoing Changes

### git reset
**Purpose:** Reset current HEAD to specified state

**Syntax:**
```bash
git reset [options] [<commit>]
```

**Real-World Examples:**
```bash
# Unstage all files
git reset

# Unstage specific file
git reset app/code/Magento/Catalog/Model/Product.php

# Soft reset (keep changes, uncommit)
git reset --soft HEAD~1

# Mixed reset (default - keep changes in working directory)
git reset --mixed HEAD~1
git reset HEAD~1  # Same as above

# Hard reset (discard all changes - dangerous!)
git reset --hard HEAD~1

# Reset to specific commit
git reset --hard abc123

# Reset to remote branch
git reset --hard origin/main

# Reset single file to specific commit
git reset abc123 -- file.php
```

**Common Scenarios:**
```bash
# Undo last commit but keep changes
git reset --soft HEAD~1
# Now you can modify and recommit

# Undo last 3 commits and discard changes
git reset --hard HEAD~3

# Unstage accidentally added files
git reset HEAD large-file.zip

# Reset branch to match remote
git reset --hard origin/main
```

**Use Cases:**
- Undoing commits
- Unstaging files
- Moving branch pointer
- Cleaning up local history

**📚 See Also:** [git-reset-revert-checkout-guide.md](git-reset-revert-checkout-guide.md)

---

### git revert
**Purpose:** Create new commit that undoes changes from previous commit

**Syntax:**
```bash
git revert [options] <commit>...
```

**Real-World Examples:**
```bash
# Revert last commit
git revert HEAD

# Revert specific commit
git revert abc123

# Revert without committing
git revert -n abc123
git revert --no-commit abc123

# Revert multiple commits
git revert HEAD~3..HEAD

# Revert merge commit
git revert -m 1 abc123

# Continue revert after conflicts
git revert --continue

# Abort revert
git revert --abort
```

**Reset vs Revert:**
- `git reset`: Moves branch pointer (rewrites history)
- `git revert`: Creates new commit (safe for public branches)

**Use Cases:**
- Undoing changes in public branches
- Safe rollback of features
- Maintaining history
- Fixing production issues

**📚 See Also:** [git-reset-revert-checkout-guide.md](git-reset-revert-checkout-guide.md)

---

### git clean
**Purpose:** Remove untracked files from working tree

**Syntax:**
```bash
git clean [options]
```

**Real-World Examples:**
```bash
# Dry run (see what would be deleted)
git clean -n

# Remove untracked files
git clean -f

# Remove untracked files and directories
git clean -fd

# Remove ignored files too
git clean -fX

# Remove all untracked and ignored files
git clean -fdx

# Interactive mode
git clean -i

# Clean specific directory
git clean -fd app/code/Magento/Catalog/Test/
```

**Use Cases:**
- Cleaning build artifacts
- Removing generated files
- Fresh start after testing
- Preparing clean environment

**⚠️ Warning:** `git clean` permanently deletes files. Always use `-n` first!

---

## Advanced Operations

### git stash
**Purpose:** Stash changes in a dirty working directory

**Syntax:**
```bash
git stash [options]
```

**Real-World Examples:**
```bash
# Stash changes
git stash

# Stash with message
git stash save "Work in progress on login feature"
git stash push -m "Work in progress on login feature"

# Stash including untracked files
git stash -u

# Stash including ignored files
git stash -a

# List stashes
git stash list

# Show stash contents
git stash show
git stash show -p  # With diff

# Apply stash (keep in stash list)
git stash apply
git stash apply stash@{2}

# Pop stash (apply and remove)
git stash pop
git stash pop stash@{1}

# Drop stash
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch feature/from-stash stash@{0}

# Stash specific files
git stash push -m "Stash only Product.php" app/code/Magento/Catalog/Model/Product.php
```

**Use Cases:**
- Switching branches with uncommitted changes
- Pulling updates without committing
- Testing different approaches
- Temporary code storage

**📚 See Also:** [git-stash-guide.md](git-stash-guide.md)

---

### git cherry-pick
**Purpose:** Apply changes from specific commits

**Syntax:**
```bash
git cherry-pick [options] <commit>...
```

**Real-World Examples:**
```bash
# Cherry-pick single commit
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456 ghi789

# Cherry-pick range
git cherry-pick abc123..def456

# Cherry-pick without committing
git cherry-pick -n abc123

# Cherry-pick and edit message
git cherry-pick -e abc123

# Cherry-pick merge commit
git cherry-pick -m 1 abc123

# Continue after conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort

# Skip current commit
git cherry-pick --skip
```

**Real-World Scenario:**
```bash
# Hotfix needed in production (main) from develop
git checkout main
git cherry-pick abc123  # The fix commit from develop
git push origin main

# Feature commits needed in another branch
git checkout feature/new-branch
git cherry-pick abc123 def456  # Specific commits from another feature
```

**Use Cases:**
- Applying hotfixes to multiple branches
- Porting features between versions
- Selective merging
- Bug fix backporting

**📚 See Also:** [git-cherry-pick-guide.md](git-cherry-pick-guide.md)

---

### git tag
**Purpose:** Create, list, delete tags

**Syntax:**
```bash
git tag [options] [<tagname>]
```

**Real-World Examples:**
```bash
# List tags
git tag

# List tags matching pattern
git tag -l "v2.4.*"

# Create lightweight tag
git tag v2.4.6

# Create annotated tag
git tag -a v2.4.6 -m "Release version 2.4.6"

# Tag specific commit
git tag -a v2.4.5 abc123 -m "Release 2.4.5"

# Show tag info
git show v2.4.6

# Delete local tag
git tag -d v2.4.6

# Delete remote tag
git push origin --delete v2.4.6
git push origin :refs/tags/v2.4.6  # Alternative

# Push tag to remote
git push origin v2.4.6

# Push all tags
git push origin --tags

# Checkout tag
git checkout v2.4.6

# Create branch from tag
git checkout -b hotfix/from-release v2.4.6
```

**Semantic Versioning Tags:**
```bash
# Major release
git tag -a v3.0.0 -m "Major release: Breaking changes"

# Minor release
git tag -a v2.5.0 -m "Minor release: New features"

# Patch release
git tag -a v2.4.7 -m "Patch release: Bug fixes"

# Pre-release
git tag -a v2.5.0-beta.1 -m "Beta release"
git tag -a v2.5.0-rc.1 -m "Release candidate"
```

**Use Cases:**
- Marking releases
- Creating snapshots
- Milestone markers
- Version management

---

### git reflog
**Purpose:** Show reference logs (history of HEAD movements)

**Syntax:**
```bash
git reflog [options]
```

**Real-World Examples:**
```bash
# Show reflog
git reflog

# Show reflog with dates
git reflog --relative-date

# Show reflog for specific branch
git reflog show main

# Recover lost commit
git reflog
# Find the commit hash
git checkout -b recovery-branch abc123

# Undo a reset
git reflog
# Find commit before reset
git reset --hard abc123
```

**Recovery Scenario:**
```bash
# Accidentally did hard reset
git reset --hard HEAD~5

# Recover
git reflog
# Output shows:
# abc123 HEAD@{0}: reset: moving to HEAD~5
# def456 HEAD@{1}: commit: Important feature

# Restore
git reset --hard def456
```

**Use Cases:**
- Recovering lost commits
- Undoing mistakes
- Finding old branch states
- Debugging

---

### git worktree
**Purpose:** Manage multiple working trees

**Syntax:**
```bash
git worktree [options]
```

**Real-World Examples:**
```bash
# List worktrees
git worktree list

# Add new worktree
git worktree add ../magento-hotfix hotfix/critical-bug

# Add worktree for new branch
git worktree add -b feature/new-feature ../magento-feature

# Remove worktree
git worktree remove ../magento-hotfix

# Prune stale worktrees
git worktree prune
```

**Use Case Scenario:**
```bash
# Working on feature, urgent hotfix needed
cd ~/projects/magento2

# Create worktree for hotfix
git worktree add ../magento-hotfix main

# Work on hotfix in separate directory
cd ../magento-hotfix
# Make fixes, commit, push

# Continue feature work
cd ~/projects/magento2
# Feature work still intact

# Clean up after hotfix is deployed
git worktree remove ../magento-hotfix
```

**Use Cases:**
- Working on multiple branches simultaneously
- Urgent fixes while feature development continues
- Testing without switching branches
- Parallel code review

---

### git submodule
**Purpose:** Manage Git repositories within repositories

**Syntax:**
```bash
git submodule [options]
```

**Real-World Examples:**
```bash
# Add submodule
git submodule add https://github.com/vendor/library.git lib/library

# Initialize submodules after clone
git submodule init
git submodule update

# Clone with submodules
git clone --recurse-submodules https://github.com/user/repo.git

# Update submodules
git submodule update --remote

# Update specific submodule
git submodule update --remote lib/library

# Status of submodules
git submodule status

# Execute command in all submodules
git submodule foreach 'git pull origin main'

# Remove submodule
git submodule deinit lib/library
git rm lib/library
rm -rf .git/modules/lib/library
```

**Use Cases:**
- Managing dependencies
- Shared libraries
- Plugin systems
- Vendor code management

---

### git subtree
**Purpose:** Alternative to submodules for managing nested repositories

**Syntax:**
```bash
git subtree [options]
```

**Real-World Examples:**
```bash
# Add subtree
git subtree add --prefix=lib/library https://github.com/vendor/library.git main --squash

# Pull updates from subtree
git subtree pull --prefix=lib/library https://github.com/vendor/library.git main --squash

# Push changes to subtree
git subtree push --prefix=lib/library https://github.com/vendor/library.git main

# Split subtree into separate repo
git subtree split --prefix=lib/library -b split-library
```

**Use Cases:**
- Including external projects
- Extracting subprojects
- Managing vendor code
- Alternative to submodules

---

### git archive
**Purpose:** Create archive of files from repository

**Syntax:**
```bash
git archive [options] <tree-ish> [<path>...]
```

**Real-World Examples:**
```bash
# Create zip of current branch
git archive --format=zip --output=project.zip HEAD

# Create tar.gz of specific tag
git archive --format=tar.gz --output=release-2.4.6.tar.gz v2.4.6

# Archive specific directory
git archive --format=zip --output=catalog.zip HEAD:app/code/Magento/Catalog

# Archive with prefix
git archive --format=tar --prefix=magento-2.4.6/ --output=magento.tar v2.4.6

# Archive and extract via pipe
git archive HEAD | tar -x -C /tmp/export/
```

**Use Cases:**
- Creating releases
- Distributing code
- Deployment packages
- Backup snapshots

---

## Collaboration Workflows

### Feature Branch Workflow
```bash
# 1. Create feature branch from main
git checkout main
git pull origin main
git checkout -b feature/AC-15635-product-validation

# 2. Work on feature
# Edit files...
git add .
git commit -m "Add product validation logic"

# 3. Keep feature updated with main
git checkout main
git pull origin main
git checkout feature/AC-15635-product-validation
git rebase main

# 4. Push feature branch
git push -u origin feature/AC-15635-product-validation

# 5. Create Pull Request (on GitHub/GitLab)

# 6. After approval, merge via PR or:
git checkout main
git merge --no-ff feature/AC-15635-product-validation
git push origin main

# 7. Clean up
git branch -d feature/AC-15635-product-validation
git push origin --delete feature/AC-15635-product-validation
```

---

### Fork Workflow
```bash
# 1. Fork repo on GitHub

# 2. Clone your fork
git clone https://github.com/yourusername/magento2.git
cd magento2

# 3. Add upstream
git remote add upstream https://github.com/magento/magento2.git

# 4. Create feature branch
git checkout -b feature/new-feature

# 5. Work and commit
git add .
git commit -m "Add new feature"

# 6. Keep fork updated
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# 7. Update feature branch
git checkout feature/new-feature
git rebase main

# 8. Push to your fork
git push origin feature/new-feature

# 9. Create Pull Request to upstream
```

---

### Gitflow Workflow
```bash
# Main branches: main (production), develop (integration)

# Start feature
git checkout develop
git checkout -b feature/new-payment-method

# Work on feature
# ... commits ...

# Finish feature
git checkout develop
git merge --no-ff feature/new-payment-method
git branch -d feature/new-payment-method
git push origin develop

# Start release
git checkout develop
git checkout -b release/2.4.7

# Prepare release (version bumps, changelog)
git commit -am "Bump version to 2.4.7"

# Finish release
git checkout main
git merge --no-ff release/2.4.7
git tag -a v2.4.7 -m "Release 2.4.7"
git checkout develop
git merge --no-ff release/2.4.7
git branch -d release/2.4.7

# Hotfix
git checkout main
git checkout -b hotfix/critical-security-fix

# Fix and commit
git commit -am "Fix security vulnerability"

# Finish hotfix
git checkout main
git merge --no-ff hotfix/critical-security-fix
git tag -a v2.4.7-hotfix.1 -m "Security hotfix"
git checkout develop
git merge --no-ff hotfix/critical-security-fix
git branch -d hotfix/critical-security-fix
```

---

### Release Process
```bash
# 1. Prepare release branch
git checkout develop
git pull origin develop
git checkout -b release/2.4.7

# 2. Version bump and changelog
# Edit version files
git add .
git commit -m "Bump version to 2.4.7"

# 3. Testing and bug fixes
# Fix bugs, commit as needed

# 4. Finalize release
git checkout main
git merge --no-ff release/2.4.7
git tag -a v2.4.7 -m "Release version 2.4.7"

# 5. Merge back to develop
git checkout develop
git merge --no-ff release/2.4.7

# 6. Push everything
git push origin main
git push origin develop
git push origin v2.4.7

# 7. Clean up
git branch -d release/2.4.7

# 8. Create GitHub release
# Upload archives, write release notes
```

---

## Best Practices

### Commit Messages

**Good Commit Messages:**
```bash
# Structure:
# <type>: <subject> (max 50 chars)
# 
# <body> (wrap at 72 chars)
#
# <footer> (references, breaking changes)

# Examples:
git commit -m "Fix: Resolve null pointer in checkout process

The checkout process was failing when customer had no default address.
Added null check before accessing address properties.

Fixes #1234"

git commit -m "Feature: Add gift card support

- Implement gift card model
- Add gift card validation
- Create checkout integration
- Add admin management interface

Related to #5678"

git commit -m "Refactor: Extract payment processing logic

Moved payment processing from controller to service class
for better testability and reusability.

Breaking change: PaymentController API changed"
```

**Types:**
- `Feat:` New feature
- `Fix:` Bug fix
- `Docs:` Documentation
- `Style:` Formatting, missing semi-colons, etc
- `Refactor:` Code restructuring
- `Test:` Adding tests
- `Chore:` Maintenance

---

### Branch Naming

**Good Branch Names:**
```bash
# Feature branches
feature/AC-15635-add-product-validation
feature/customer-loyalty-program
feature/payment-gateway-integration

# Bug fixes
bugfix/AC-16789-fix-checkout-total
fix/resolve-null-pointer-in-shipping

# Hotfixes
hotfix/critical-security-vulnerability
hotfix/production-crash-fix

# Release branches
release/2.4.7
release/v3.0.0-rc.1

# Experimental
experiment/new-caching-strategy
spike/graphql-performance
```

---

### Daily Workflow

**Starting Work:**
```bash
# 1. Update main branch
git checkout main
git pull origin main

# 2. Create/update feature branch
git checkout -b feature/AC-15635
# OR
git checkout feature/AC-15635
git rebase main

# 3. Work...
```

**During Work:**
```bash
# Commit often with meaningful messages
git add specific-files
git commit -m "Clear message"

# Push regularly
git push origin feature/AC-15635
```

**End of Day:**
```bash
# Ensure work is backed up
git push origin feature/AC-15635

# OR stash if not ready to commit
git stash save "WIP: working on validation logic"
```

---

### Code Review Workflow

**Preparing for Review:**
```bash
# 1. Update with latest main
git checkout main
git pull origin main
git checkout feature/AC-15635
git rebase main

# 2. Clean up commits
git rebase -i main
# Squash fixup commits, reword messages

# 3. Force push (your branch only!)
git push --force-with-lease origin feature/AC-15635

# 4. Create Pull Request
```

**Addressing Review Comments:**
```bash
# Make changes
git add .
git commit -m "Address review comments: improve error handling"

# Push
git push origin feature/AC-15635

# If asked to squash, before final merge:
git rebase -i main
# Squash review commits
git push --force-with-lease origin feature/AC-15635
```

---

### Useful Aliases

**Add to ~/.gitconfig:**
```ini
[alias]
    # Short forms
    st = status
    co = checkout
    br = branch
    ci = commit
    cp = cherry-pick
    
    # Logging
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    lol = log --oneline --graph --all --decorate
    last = log -1 HEAD --stat
    
    # Shortcuts
    unstage = reset HEAD --
    discard = checkout --
    amend = commit --amend --no-edit
    undo = reset --soft HEAD~1
    
    # Information
    aliases = config --get-regexp alias
    contributors = shortlog -sn
    
    # Branch management
    cleanup = !git branch --merged | grep -v '\\*\\|main\\|develop' | xargs -n 1 git branch -d
    branches = branch -a
    remotes = remote -v
    
    # Diff
    diffc = diff --cached
    diffstat = diff --stat
    
    # Stash
    save = stash save
    pop = stash pop
    
    # History
    hist = log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short
    
    # Sync
    sync = !git fetch --all && git pull --rebase
```

**Usage:**
```bash
git st                          # Instead of git status
git lg                          # Beautiful log
git cleanup                     # Remove merged branches
git sync                        # Fetch and pull
```

---

### Tips and Tricks

**1. Global .gitignore**
```bash
# Create global ignore file
git config --global core.excludesfile ~/.gitignore_global

# Add common patterns
echo ".DS_Store" >> ~/.gitignore_global
echo "*.swp" >> ~/.gitignore_global
echo "*~" >> ~/.gitignore_global
echo ".idea/" >> ~/.gitignore_global
```

**2. Auto-correct Commands**
```bash
git config --global help.autocorrect 1
# Now "git comit" will auto-correct to "git commit"
```

**3. Better Diff Output**
```bash
git config --global diff.algorithm patience
git config --global diff.compactionHeuristic true
```

**4. Reuse Recorded Resolution (rerere)**
```bash
git config --global rerere.enabled true
# Git remembers how you resolved conflicts
```

**5. Default Branch Name**
```bash
git config --global init.defaultBranch main
```

**6. Pull with Rebase**
```bash
git config --global pull.rebase true
```

**7. Push Current Branch Only**
```bash
git config --global push.default current
```

**8. Color Output**
```bash
git config --global color.ui auto
```

**9. Editor Configuration**
```bash
git config --global core.editor "vim"
# Or: code --wait, nano, emacs, etc.
```

**10. Line Ending Configuration**
```bash
# Windows
git config --global core.autocrlf true

# Mac/Linux
git config --global core.autocrlf input
```

---

## Quick Reference

### Common Commands
```bash
# Initialize
git init
git clone <url>

# Basic workflow
git status
git add <file>
git commit -m "message"
git push

# Branching
git branch <name>
git checkout <branch>
git merge <branch>

# Remote
git remote add origin <url>
git fetch
git pull
git push

# Undo
git reset HEAD <file>
git checkout -- <file>
git revert <commit>

# History
git log
git diff
git show <commit>

# Stash
git stash
git stash pop

# Tags
git tag <name>
git push --tags
```

---

## Common Scenarios

### Scenario 1: Made commits on wrong branch
```bash
# Currently on main, should be on feature branch
git branch feature/new-feature
git reset --hard HEAD~3  # Assuming 3 commits to move
git checkout feature/new-feature
```

### Scenario 2: Need to undo last commit
```bash
# Keep changes
git reset --soft HEAD~1

# Discard changes
git reset --hard HEAD~1
```

### Scenario 3: Accidentally committed to main
```bash
git reset --soft HEAD~1
git stash
git checkout feature/correct-branch
git stash pop
git add .
git commit -m "Correct message"
```

### Scenario 4: Need hotfix while working on feature
```bash
git stash
git checkout main
git checkout -b hotfix/critical-bug
# Fix, commit, push
git checkout feature/my-feature
git stash pop
```

### Scenario 5: Merge conflict resolution
```bash
git merge feature/branch
# Conflicts!
git status  # See conflicted files
# Edit files, resolve conflicts
git add .
git commit
```

### Scenario 6: Update fork with upstream
```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### Scenario 7: Squash commits before merge
```bash
git rebase -i HEAD~5
# Change 'pick' to 'squash' for commits to squash
git push --force-with-lease
```

### Scenario 8: Recover deleted branch
```bash
git reflog
# Find commit hash where branch was
git checkout -b recovered-branch <hash>
```

---

## Troubleshooting

### Problem: Merge conflicts
```bash
# Abort merge
git merge --abort

# Or resolve manually
# Edit conflicted files
git add resolved-files
git commit
```

### Problem: Pushed wrong commit
```bash
# If no one pulled yet
git reset --hard HEAD~1
git push --force-with-lease

# If others already pulled
git revert HEAD
git push
```

### Problem: Lost commits
```bash
git reflog
git checkout -b recovery <hash>
```

### Problem: Need to change old commit message
```bash
git rebase -i HEAD~5
# Change 'pick' to 'reword' for commit
# Edit message in editor
```

### Problem: Accidentally deleted file
```bash
# If not committed
git checkout HEAD -- file.php

# If committed
git checkout <commit-hash> -- file.php
```

### Problem: Large file in history
```bash
# Remove from history (dangerous!)
git filter-branch --tree-filter 'rm -f path/to/large/file' HEAD
git push --force
```

---

## Resources

### Documentation
- [Official Git Documentation](https://git-scm.com/doc)
- [Pro Git Book](https://git-scm.com/book)
- [Git Reference](https://git-scm.com/docs)

### Interactive Learning
- [Learn Git Branching](https://learngitbranching.js.org/)
- [Git Immersion](http://gitimmersion.com/)
- [GitHub Git Handbook](https://guides.github.com/introduction/git-handbook/)

### Cheat Sheets
- [GitHub Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
- [Atlassian Git Cheat Sheet](https://www.atlassian.com/git/tutorials/atlassian-git-cheatsheet)

---

## Related Documentation in This Repository

- [Git Reset, Revert, and Checkout Guide](git-reset-revert-checkout-guide.md) - Detailed guide on undoing changes
- [Git Merge Guide](git-merge-guide.md) - Comprehensive merge strategies and conflict resolution
- [Git Stash Guide](git-stash-guide.md) - Complete guide to stashing changes
- [Git Cherry-pick Guide](git-cherry-pick-guide.md) - Guide to cherry-picking commits
- [Git Squash to Root Commit](git-squash-to-root-commit.md) - Squashing commit history

---

## Conclusion

This guide covers the most commonly used Git commands with real-world examples. Git is a powerful tool with many features - this guide provides a foundation for effective version control.

**Key Takeaways:**
1. Commit often with clear messages
2. Use branches for features and fixes
3. Keep main branch stable
4. Pull before you push
5. Never force push to shared branches
6. Use meaningful branch names
7. Review changes before committing

**Practice Makes Perfect:**
The best way to learn Git is through regular use. Start with basic commands and gradually explore advanced features as needed.

---

*Last Updated: November 1, 2025*

