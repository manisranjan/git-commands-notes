
# Git: Squash All Commits into Single Amended Root Commit
## Overview
This technique allows you to collapse an entire commit history into a single root commit while preserving the final file state. This is useful for cleaning up repository history, starting fresh with a clean slate, or preparing a repository for open-sourcing.
## ⚠️ Important Warnings
- **This rewrites git history** - Use with caution!
- **DO NOT use on shared/public branches** - Only use on branches you own
- **Forces push required** - If already pushed, you'll need `git push --force`
- **Collaborators will need to re-clone** - Their local copies will be out of sync
- **Make a backup first** - Always create a backup branch before proceeding
## Commands
```bash
# Step 1: Create a backup branch (safety net)
git branch beforeReset
# Step 2: Reset to root commit, keeping all changes staged
git reset --soft <root>
# Step 3: Amend the root commit with all changes
git commit --amend
# Step 4: Verify no differences between old and new state
git diff beforeReset
```
## Detailed Explanation
### Step 1: Create Backup Branch
```bash
git branch beforeReset
```
**Purpose:** Creates a safety backup of your current branch state.
**What it does:**
- Creates a new branch named `beforeReset` pointing to your current HEAD
- Does not switch to the new branch (stays on current branch)
- Acts as a restore point if something goes wrong
**Why it's important:**
- Allows you to compare the final result
- Provides a way to recover if the operation fails
- No risk - you can always go back to this state
---
### Step 2: Soft Reset to Root Commit
```bash
git reset --soft <root>
```
**Purpose:** Moves HEAD back to the root (first) commit while keeping all changes staged.
**What it does:**
- Moves the current branch pointer back to `<root>` commit
- **Keeps all file changes in the staging area** (index)
- Does not modify your working directory
- Effectively "undoes" all commits after `<root>`, but keeps the changes
**Parameter `<root>`:**
- Replace `<root>` with the commit hash of your first/root commit
- Find it using: `git rev-list --max-parents=0 HEAD`
- Or use: `git log --reverse` and copy the first commit hash
**Example:**
```bash
# Find root commit
git rev-list --max-parents=0 HEAD
# Output: abc123def456...
# Reset to root
git reset --soft abc123def456
```
**Types of reset (for reference):**
- `--soft`: Keeps changes staged ✓ (what we use)
- `--mixed`: Keeps changes unstaged (default)
- `--hard`: Discards all changes ✗ (dangerous)
---
### Step 3: Amend Root Commit
```bash
git commit --amend
```
**Purpose:** Modify the root commit to include all staged changes.
**What it does:**
- Opens your editor to modify the root commit message
- Combines all staged changes (from step 2) into the root commit
- Replaces the original root commit with a new one containing everything
- Generates a new commit hash (because commit contents changed)
**Interactive options:**
- Edit the commit message in your editor, then save and close
- Or use: `git commit --amend -m "New message"` to skip the editor
- Or use: `git commit --amend --no-edit` to keep the existing message
**Result:**
- Your repository now has a single commit with all your files
- All previous commit history is effectively erased
- The commit tree is now: `root commit (contains everything)`
---
### Step 4: Verify No Differences
```bash
git diff beforeReset
```
**Purpose:** Confirm that the final file state is identical to the original.
**What it does:**
- Compares current working directory with the `beforeReset` branch
- Should output nothing (no differences) if successful
- Proves that only history changed, not file contents
**Expected output:**
```
(no output - means no differences)
```
**If there are differences:**
- Something went wrong in the process
- Review the differences: `git diff beforeReset --stat`
- Consider restoring: `git reset --hard beforeReset`
---
## Complete Examples
### Example 1: Basic Usage (Repository Cleanup)
**Scenario:** You have 50 commits of messy development work and want to start fresh.
```bash
# Check your current history
git log --oneline
# a1b2c3d (HEAD) Fix typo
# d4e5f6g Add feature
# h7i8j9k Initial commit
# Create backup
git branch beforeReset
# Find root commit
ROOT=$(git rev-list --max-parents=0 HEAD)
echo $ROOT
# h7i8j9k
# Reset to root, keeping all changes staged
git reset --soft $ROOT
# Check what's staged
git status
# All your files should be staged
# Amend the root commit with a clean message
git commit --amend -m "Initial commit: Complete project setup"
# Verify no differences
git diff beforeReset
# (no output = success!)
# Check new history
git log --oneline
# k9j8i7h (HEAD) Initial commit: Complete project setup
# Clean up backup branch (optional)
git branch -D beforeReset
```
---
### Example 2: With Specific Root Commit Hash
**Scenario:** You know your root commit hash and want to use it directly.
```bash
# Backup current state
git branch beforeReset
# Reset to specific root commit
git reset --soft abc123def456
# Amend with new message
git commit --amend -m "Initial commit: Magento 2 PHPUnit 12 Migration"
# Verify
git diff beforeReset
# View the single commit
git log --stat
```
---
### Example 3: Preparing for Open Source Release
**Scenario:** Remove internal development history before open-sourcing.
```bash
# Create backup
git branch internal-history-backup
# Find and reset to root
git reset --soft $(git rev-list --max-parents=0 HEAD)
# Create professional initial commit message
git commit --amend -m "Initial public release
This project provides PHPUnit 12 migration utilities for Magento 2.
Features:
- Automated test migration
- Comprehensive documentation
- Quality checks integration
Version: 1.0.0"
# Verify file state unchanged
git diff internal-history-backup
# If satisfied, force push to origin
git push --force-with-lease origin main
```
---
### Example 4: Recovery from Mistakes
**Scenario:** Something went wrong and you need to recover.
```bash
# If you still have beforeReset branch
git reset --hard beforeReset
# Or if you lost the branch but didn't push
git reflog
# Find the commit before you started
git reset --hard HEAD@{5}  # Adjust number as needed
```
---
## Alternative: Using Interactive Rebase
For more control, you can use interactive rebase (preserves root commit):
```bash
# Backup
git branch beforeRebase
# Start interactive rebase from root
git rebase -i --root
# In the editor, mark all commits except the first as 'squash' or 'fixup'
# pick abc123 Initial commit
# squash def456 Second commit
# squash ghi789 Third commit
# ... (mark rest as squash)
# Save and close editor
# Edit the combined commit message if prompted
# Verify
git diff beforeRebase
```
---
## Finding the Root Commit
Several methods to find your repository's root commit:
```bash
# Method 1: Rev-list (most reliable)
git rev-list --max-parents=0 HEAD
# Method 2: Log in reverse order
git log --reverse --oneline | head -1
# Method 3: Using format
git log --pretty=format:"%H" --reverse | head -1
# Method 4: Store in variable for easy use
ROOT=$(git rev-list --max-parents=0 HEAD)
echo $ROOT
```
---
## When to Use This Technique
### ✅ Good Use Cases:
- Cleaning up experimental/development branches
- Preparing a repository for public release
- Starting fresh after major refactoring
- Collapsing migration commits into one
- Creating a clean "base" for future work
- Removing sensitive commits before sharing
### ❌ Bad Use Cases:
- Shared/collaborative branches (breaks others' work)
- Main/master branch with collaborators
- Published releases with tagged versions
- When commit history is valuable
- Required for audit/compliance purposes
- Repositories with signed commits
---
## Troubleshooting
### Problem: "fatal: ambiguous argument 'beforeReset'"
**Solution:** The backup branch doesn't exist. Skip the diff check or create it first.
### Problem: Root commit has wrong content
**Solution:** You likely need `git reset --soft` with a different commit.
```bash
# Reset to beforeReset
git reset --hard beforeReset
# Try again with correct commit
```
### Problem: Can't push after squashing
**Solution:** History was rewritten. You need force push (BE CAREFUL):
```bash
# Safer version (checks remote state)
git push --force-with-lease origin your-branch
# Nuclear option (use with extreme caution)
git push --force origin your-branch
```
### Problem: Collaborator's repository is broken
**Solution:** They need to re-sync:
```bash
# Collaborator's steps:
git fetch origin
git reset --hard origin/main  # or appropriate branch
```
---
## Best Practices
1. **Always create a backup branch first**
   ```bash
   git branch backup-$(date +%Y%m%d-%H%M%S)
   ```
2. **Test on a feature branch first**
   ```bash
   git checkout -b test-squash
   # Run the squash commands here
   # Verify everything works
   ```
3. **Use `--force-with-lease` instead of `--force`**
   ```bash
   git push --force-with-lease origin branch-name
   # Safer - fails if remote has unexpected changes
   ```
4. **Document why you squashed**
   - Add details in the new commit message
   - Update README or changelog
5. **Notify collaborators beforehand**
   - Warn team members of history rewrite
   - Provide re-sync instructions
---
## Related Commands
```bash
# View commit history
git log --oneline --graph --all
# Count total commits
git rev-list --count HEAD
# View root commit details
git show $(git rev-list --max-parents=0 HEAD)
# List all commits
git log --all --pretty=format:"%h %an %ad %s" --date=short
# Check staging area
git status
# View what's staged
git diff --cached
# Unstage everything (if needed)
git reset HEAD
```
---
## Summary
The technique `git reset --soft <root> && git commit --amend` is a powerful way to collapse all commits into a single root commit. It's particularly useful for cleaning up repositories before sharing or when you want a fresh start while preserving all your work.
**Key takeaways:**
- Creates a single commit with all your files
- Preserves final file state perfectly
- Rewrites git history (use carefully)
- Requires force push if already published
- Always create a backup branch first
---
## Quick Reference Card
```bash
# Full workflow in one go
git branch beforeReset && \
git reset --soft $(git rev-list --max-parents=0 HEAD) && \
git commit --amend -m "Your message here" && \
git diff beforeReset
# If everything looks good
git branch -D beforeReset
# If pushed already (CAREFUL!)
git push --force-with-lease origin your-branch
```
---
**Last Updated:** October 30, 2025  
**Git Version Tested:** 2.40+  
**Compatibility:** All modern Git versions supporting `--soft` reset
