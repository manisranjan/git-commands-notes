# Git Merge: Comprehensive Guide with Real-World Examples

## Table of Contents
- [Introduction](#introduction)
- [What is Git Merge?](#what-is-git-merge)
- [Merge Strategies](#merge-strategies)
- [Basic Merge Workflow](#basic-merge-workflow)
- [Real-World Examples](#real-world-examples)
- [Common Use Cases](#common-use-cases)
- [Merge vs Rebase](#merge-vs-rebase)
- [Handling Merge Conflicts](#handling-merge-conflicts)
- [Best Practices](#best-practices)
- [Advanced Scenarios](#advanced-scenarios)

---

## Introduction

Git merge is one of the fundamental operations in Git that allows you to integrate changes from different branches. Understanding how to properly merge branches is essential for effective collaboration in software development teams.

---

## What is Git Merge?

Git merge combines multiple sequences of commits into one unified history. It takes the contents of a source branch and integrates them with a target branch, creating a new "merge commit" that ties together the histories of both branches.

**Key Characteristics:**
- Preserves complete history of both branches
- Creates a merge commit (in most cases)
- Non-destructive operation
- Maintains branch context

---

## Merge Strategies

### 1. Fast-Forward Merge

Occurs when there's a linear path from the current branch tip to the target branch.

```bash
# Scenario: No new commits on main since feature branch was created
git checkout main
git merge feature-branch
# Result: main pointer moves forward, no merge commit
```

**When it happens:**
- No divergent changes between branches
- Target branch is directly ahead of current branch

### 2. Three-Way Merge (Recursive)

Creates a new merge commit when branches have diverged.

```bash
git checkout main
git merge feature-branch
# Result: New merge commit combining both histories
```

**When it happens:**
- Both branches have new commits since they diverged
- Changes exist in parallel

### 3. Squash Merge

Combines all commits from the feature branch into a single commit.

```bash
git checkout main
git merge --squash feature-branch
git commit -m "Add feature X"
```

**When to use:**
- Clean up messy feature branch history
- Consolidate work-in-progress commits

### 4. No-Fast-Forward Merge

Forces creation of a merge commit even when fast-forward is possible.

```bash
git merge --no-ff feature-branch
```

**When to use:**
- Maintain explicit branch history
- Track when features were integrated

---

## Basic Merge Workflow

### Standard Merge Process

```bash
# 1. Ensure your working directory is clean
git status

# 2. Switch to the branch you want to merge INTO
git checkout main

# 3. Update your branch to latest
git pull origin main

# 4. Merge the feature branch
git merge feature-branch

# 5. Push the merged changes
git push origin main
```

---

## Real-World Examples

### Example 1: Feature Development

**Scenario:** You're working on an e-commerce platform and need to add a payment integration feature.

```bash
# Create feature branch
git checkout -b feature/stripe-integration

# Make commits as you work
git add payment/StripeService.php
git commit -m "Add Stripe API integration"

git add tests/StripeServiceTest.php
git commit -m "Add tests for Stripe service"

git add config/payment.php
git commit -m "Configure Stripe payment settings"

# When feature is complete, merge back to main
git checkout main
git pull origin main  # Get latest changes
git merge feature/stripe-integration

# Push to remote
git push origin main

# Delete feature branch (optional)
git branch -d feature/stripe-integration
```

### Example 2: Hotfix Merge

**Scenario:** Critical bug in production needs immediate fix.

```bash
# Create hotfix from main
git checkout main
git checkout -b hotfix/payment-validation-bug

# Fix the bug
git add src/Validator.php
git commit -m "Fix payment validation edge case"

# Merge to main
git checkout main
git merge hotfix/payment-validation-bug
git push origin main

# Also merge to develop branch
git checkout develop
git merge hotfix/payment-validation-bug
git push origin develop

# Clean up
git branch -d hotfix/payment-validation-bug
```

### Example 3: Release Branch Merge

**Scenario:** Preparing a release and merging back to both main and develop.

```bash
# Create release branch from develop
git checkout develop
git checkout -b release/v2.3.0

# Make release preparations
git commit -m "Bump version to 2.3.0"
git commit -m "Update CHANGELOG.md"

# Merge to main (production)
git checkout main
git merge --no-ff release/v2.3.0
git tag -a v2.3.0 -m "Release version 2.3.0"
git push origin main --tags

# Merge back to develop
git checkout develop
git merge --no-ff release/v2.3.0
git push origin develop

# Delete release branch
git branch -d release/v2.3.0
```

### Example 4: Collaborative Development

**Scenario:** Multiple developers working on different features that need to be integrated.

```bash
# Developer A working on user authentication
git checkout -b feature/oauth-login
# ... makes commits ...
git push origin feature/oauth-login

# Developer B working on profile page
git checkout -b feature/user-profile
# ... makes commits ...
git push origin feature/user-profile

# Tech lead integrates both features
git checkout main
git pull origin main

# Merge first feature
git merge feature/oauth-login
git push origin main

# Merge second feature
git merge feature/user-profile
# If conflicts, resolve them
git push origin main
```

---

## Common Use Cases

### Use Case 1: GitFlow Workflow

```bash
# Main branches
main          # Production-ready code
develop       # Integration branch

# Supporting branches
feature/*     # New features (merge into develop)
hotfix/*      # Production fixes (merge into main and develop)
release/*     # Release preparation (merge into main and develop)
```

**Example workflow:**
```bash
# Start new feature
git checkout develop
git checkout -b feature/new-dashboard

# Complete feature
git checkout develop
git merge feature/new-dashboard

# Create release
git checkout -b release/v1.5.0

# Finalize release
git checkout main
git merge release/v1.5.0
git checkout develop
git merge release/v1.5.0
```

### Use Case 2: GitHub Flow

```bash
# Simple workflow: main + feature branches
# Everything merges to main via pull requests

# Create feature
git checkout -b feature/add-search

# Push and create PR
git push origin feature/add-search
# Create pull request on GitHub

# After PR approval, merge via GitHub interface or CLI
git checkout main
git pull origin main  # Gets the merged changes
```

### Use Case 3: Integration Testing

```bash
# Create integration branch for testing multiple features together
git checkout -b integration/sprint-23

# Merge features for testing
git merge feature/user-notifications
git merge feature/email-templates
git merge feature/admin-dashboard

# Test thoroughly
# If all good, merge features individually to main
```

### Use Case 4: Vendor/Upstream Syncing

```bash
# Scenario: Keeping fork up to date with upstream repository
git remote add upstream https://github.com/original/repo.git
git fetch upstream

# Merge upstream changes
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

---

## Merge vs Rebase

### When to Use Merge

✅ **Use merge when:**
- Working on shared/public branches
- Want to preserve complete history
- Collaborating with multiple developers
- Need to maintain branch context
- Integrating long-lived branches

### When to Use Rebase

✅ **Use rebase when:**
- Cleaning up local commits before pushing
- Updating feature branch with latest main
- Want linear history
- Working on private branches

### Comparison Example

**Merge approach:**
```bash
git checkout feature-branch
git merge main
# Creates merge commit, preserves both histories
```

**Rebase approach:**
```bash
git checkout feature-branch
git rebase main
# Replays feature commits on top of main, linear history
```

---

## Handling Merge Conflicts

### Understanding Conflicts

Conflicts occur when Git can't automatically reconcile differences between branches.

**Common conflict scenarios:**
- Same line modified in both branches
- File deleted in one branch, modified in another
- File renamed in one branch, modified in another

### Resolving Conflicts

#### Step-by-Step Process

```bash
# Attempt merge
git checkout main
git merge feature-branch

# If conflicts occur
# Output: CONFLICT (content): Merge conflict in file.php
# Output: Automatic merge failed; fix conflicts and then commit the result.

# Check status
git status
# Shows: both modified: file.php

# Open file and look for conflict markers
```

**Conflict markers in file:**
```php
<?php
class PaymentProcessor {
<<<<<<< HEAD
    public function process($amount, $currency = 'USD') {
        // Current branch version
        return $this->stripe->charge($amount, $currency);
=======
    public function process($amount, $method = 'card') {
        // Incoming branch version
        return $this->gateway->charge($amount, $method);
>>>>>>> feature-branch
    }
}
```

**Resolution:**
```php
<?php
class PaymentProcessor {
    // Manually resolve - keep what you need
    public function process($amount, $currency = 'USD', $method = 'card') {
        return $this->gateway->charge($amount, $currency, $method);
    }
}
```

**Complete the merge:**
```bash
# After resolving all conflicts
git add file.php
git status  # Verify all conflicts resolved

# Complete merge
git commit -m "Merge feature-branch: resolved conflicts in PaymentProcessor"

# Push
git push origin main
```

### Aborting a Merge

```bash
# If you want to cancel the merge
git merge --abort

# This returns to state before merge started
```

### Tools for Conflict Resolution

```bash
# Use visual merge tool
git mergetool

# Configure merge tool (one-time setup)
git config --global merge.tool vimdiff
# or
git config --global merge.tool meld
# or
git config --global merge.tool kdiff3
```

---

## Best Practices

### 1. Keep Branches Up to Date

```bash
# Regularly sync feature branch with main
git checkout feature-branch
git fetch origin
git merge origin/main  # or rebase

# This minimizes conflicts during final merge
```

### 2. Merge Frequently

- Small, frequent merges are easier than large, infrequent ones
- Reduces complexity of conflict resolution
- Keeps team in sync

### 3. Use Pull Requests

```bash
# Push feature branch
git push origin feature-branch

# Create PR for review before merging
# Allows team review, CI/CD checks, discussions
```

### 4. Clean Commit History

```bash
# Before merging, clean up commits if needed
git checkout feature-branch
git rebase -i main

# Squash work-in-progress commits
# Write clear commit messages
```

### 5. Test Before Merging

```bash
# Run tests before merging
git checkout feature-branch
npm test  # or your test command

# Only merge if tests pass
git checkout main
git merge feature-branch
```

### 6. Use Descriptive Merge Messages

```bash
# Default merge message is often generic
git merge feature-branch

# Provide context
git merge feature-branch -m "Merge stripe integration feature

- Add Stripe API service
- Implement payment processing
- Add tests and documentation
- Resolves #123"
```

### 7. Tag Important Merges

```bash
# After merging release
git tag -a v2.0.0 -m "Release 2.0.0 - Major feature update"
git push origin v2.0.0
```

---

## Advanced Scenarios

### Scenario 1: Merge Specific Files Only

```bash
# Cherry-pick specific changes
git checkout main
git checkout feature-branch -- path/to/specific/file.php
git commit -m "Import specific file from feature branch"
```

### Scenario 2: Undo a Merge

```bash
# If merge hasn't been pushed
git reset --hard HEAD~1

# If merge has been pushed (creates revert commit)
git revert -m 1 <merge-commit-hash>
```

### Scenario 3: Merge with Strategy Options

```bash
# Use specific merge strategy
git merge -s recursive -X theirs feature-branch  # Favor their changes
git merge -s recursive -X ours feature-branch    # Favor our changes

# Ignore whitespace changes
git merge -Xignore-space-change feature-branch
```

### Scenario 4: Octopus Merge (Multiple Branches)

```bash
# Merge multiple branches at once (not common)
git merge feature-1 feature-2 feature-3

# Git will attempt to merge all three
# Only works if no conflicts between them
```

### Scenario 5: Subtree Merge

```bash
# Merge another repository as subdirectory
git remote add external-lib https://github.com/vendor/library.git
git fetch external-lib
git merge -s subtree external-lib/main --allow-unrelated-histories
```

### Scenario 6: Verify Merge Before Committing

```bash
# Merge without committing
git merge --no-commit --no-ff feature-branch

# Inspect changes
git diff --cached

# If satisfied
git commit -m "Merge feature-branch after verification"

# If not satisfied
git merge --abort
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: "Not a valid object name"

```bash
# Error: feature-branch is not a valid object name
# Solution: Branch doesn't exist locally
git fetch origin
git checkout -b feature-branch origin/feature-branch
```

#### Issue 2: "You have not concluded your merge"

```bash
# Previous merge not completed
# Solution: Complete or abort previous merge
git status  # Check what's pending
git merge --abort  # Or resolve conflicts and commit
```

#### Issue 3: "Cannot merge binary files"

```bash
# Binary files conflict
# Solution: Choose one version
git checkout --ours path/to/binary/file    # Keep current branch
git checkout --theirs path/to/binary/file  # Keep incoming branch
git add path/to/binary/file
```

#### Issue 4: Merge Created Duplicate Commits

```bash
# Solution: Use rebase instead next time for cleaner history
# Or use squash merge
git merge --squash feature-branch
```

---

## Quick Reference

### Common Commands

```bash
# Basic merge
git merge <branch-name>

# Merge with options
git merge --no-ff <branch>          # Force merge commit
git merge --squash <branch>         # Squash all commits
git merge --no-commit <branch>      # Merge but don't commit

# During conflict resolution
git status                          # Check conflict status
git merge --abort                   # Cancel merge
git add <file>                      # Mark conflict as resolved
git commit                          # Complete merge

# After merge
git log --graph --oneline           # View merge history
git show <merge-commit>             # View merge details

# Undo merge
git reset --hard HEAD~1             # Not pushed yet
git revert -m 1 <merge-commit>      # Already pushed
```

---

## Summary

Git merge is a powerful tool for integrating work from different branches. Key takeaways:

1. **Choose the right strategy:** Fast-forward for simple cases, three-way for divergent branches
2. **Keep branches updated:** Regular syncing reduces conflicts
3. **Test before merging:** Ensure code quality
4. **Communicate:** Use clear commit messages and pull requests
5. **Practice conflict resolution:** It's a critical skill for team development

By understanding and properly using Git merge, you can maintain a clean, organized codebase while collaborating effectively with your team.

---

## Additional Resources

- [Official Git Documentation - Merge](https://git-scm.com/docs/git-merge)
- [Atlassian Git Merge Tutorial](https://www.atlassian.com/git/tutorials/using-branches/git-merge)
- [Git Branching Strategies](https://git-scm.com/book/en/v2/Git-Branching-Branching-Workflows)

---

**Last Updated:** November 2025

