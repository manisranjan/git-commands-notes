# Git Cherry-Pick: Advanced Guide with Real-World Examples

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Concepts](#basic-concepts)
3. [Basic Usage](#basic-usage)
4. [Advanced Scenarios](#advanced-scenarios)
5. [Real-World Examples](#real-world-examples)
6. [Cherry-Pick vs Other Git Operations](#cherry-pick-vs-other-git-operations)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)
9. [Common Pitfalls](#common-pitfalls)
10. [Tips and Tricks](#tips-and-tricks)

---

## Introduction

### What is Cherry-Pick?

`git cherry-pick` is a powerful Git command that allows you to apply specific commits from one branch to another. Unlike merge or rebase, which apply entire branches, cherry-pick gives you surgical precision to select individual commits.

### When to Use Cherry-Pick

**Good Use Cases:**
- Applying a critical hotfix from `main` to a release branch
- Backporting bug fixes to older versions
- Selectively applying features from a development branch
- Recovering specific commits from a deleted branch
- Moving commits that were accidentally made on the wrong branch

**When NOT to Use:**
- When you need the entire branch history (use merge instead)
- When you want to maintain branch relationships (use merge instead)
- For regular integration workflows (cherry-pick creates duplicate commits)

---

## Basic Concepts

### How Cherry-Pick Works

When you cherry-pick a commit:
1. Git identifies the changes introduced by that commit (the diff)
2. Git applies those changes to your current branch
3. Git creates a **new commit** with a new SHA (not the same as the original)
4. The new commit has the same message and changes, but different metadata

### Important Notes

- Cherry-picked commits are **duplicates**, not the original commits
- The new commit has a different SHA hash
- Parent commit references are different
- Author remains the same, but committer and timestamp are updated

---

## Basic Usage

### Cherry-Pick a Single Commit

```bash
# Switch to the target branch
git checkout feature-branch

# Cherry-pick a specific commit
git cherry-pick abc123
```

### Cherry-Pick Multiple Commits

```bash
# Cherry-pick multiple specific commits
git cherry-pick abc123 def456 ghi789

# Cherry-pick a range of commits (exclusive of first, inclusive of last)
git cherry-pick abc123..ghi789

# Cherry-pick a range of commits (inclusive of both)
git cherry-pick abc123^..ghi789
```

### Cherry-Pick with Options

```bash
# Cherry-pick without committing (stage changes only)
git cherry-pick -n abc123
git cherry-pick --no-commit abc123

# Cherry-pick and edit the commit message
git cherry-pick -e abc123
git cherry-pick --edit abc123

# Cherry-pick and append "cherry picked from..." message
git cherry-pick -x abc123

# Cherry-pick without recording the commit (useful for backports)
git cherry-pick --ff abc123
```

---

## Advanced Scenarios

### 1. Cherry-Pick with Conflicts

When conflicts occur during cherry-pick:

```bash
# Start the cherry-pick
git cherry-pick abc123

# If conflicts occur:
# 1. Resolve conflicts in your editor
# 2. Stage the resolved files
git add <resolved-files>

# 3. Continue the cherry-pick
git cherry-pick --continue

# Alternative: Skip this commit
git cherry-pick --skip

# Alternative: Abort the entire cherry-pick
git cherry-pick --abort
```

### 2. Cherry-Pick with Mainline Parent

For merge commits, you must specify which parent to use:

```bash
# Cherry-pick a merge commit using the first parent
git cherry-pick -m 1 abc123

# Cherry-pick a merge commit using the second parent
git cherry-pick -m 2 abc123
```

**Understanding Mainline:**
- `-m 1`: Uses the branch you merged INTO (usually main/master)
- `-m 2`: Uses the branch you merged FROM (usually the feature branch)

### 3. Cherry-Pick from Another Repository

```bash
# Add the other repository as a remote
git remote add other-repo https://github.com/user/repo.git

# Fetch the commits
git fetch other-repo

# Cherry-pick from the other repo
git cherry-pick other-repo/main~3
```

### 4. Cherry-Pick with Strategy Options

```bash
# Cherry-pick favoring "ours" during conflicts
git cherry-pick -X ours abc123

# Cherry-pick favoring "theirs" during conflicts
git cherry-pick -X theirs abc123

# Cherry-pick ignoring whitespace changes
git cherry-pick -X ignore-space-change abc123
```

### 5. Cherry-Pick and Sign Off

```bash
# Cherry-pick and add your sign-off
git cherry-pick -s abc123
git cherry-pick --signoff abc123
```

---

## Real-World Examples

### Example 1: Critical Hotfix to Multiple Branches

**Scenario:** You fixed a security bug on `main` and need to apply it to release branches.

```bash
# Your hotfix commit on main
# commit: a1b2c3d "Fix: Security vulnerability in authentication"

# Apply to release-2.0 branch
git checkout release-2.0
git cherry-pick -x a1b2c3d
git push origin release-2.0

# Apply to release-1.9 branch
git checkout release-1.9
git cherry-pick -x a1b2c3d
git push origin release-1.9

# The -x flag adds: "(cherry picked from commit a1b2c3d)"
```

### Example 2: Moving Commits to the Correct Branch

**Scenario:** You accidentally made 3 commits on `main` that should be on a feature branch.

```bash
# Current state: main has commits abc, def, ghi that shouldn't be there

# Create/checkout feature branch from before the commits
git checkout -b feature-fix main~3

# Cherry-pick the commits
git cherry-pick abc def ghi

# Verify the commits are on the feature branch
git log --oneline -3

# Reset main to remove the commits
git checkout main
git reset --hard origin/main

# Push the feature branch
git push origin feature-fix
```

### Example 3: Selective Feature Backporting

**Scenario:** A feature branch has 10 commits, but you only want 2 specific improvements.

```bash
# Feature branch has commits: a1, a2, a3, a4, a5, a6, a7, a8, a9, a10
# You want only a3 (performance fix) and a7 (UI improvement)

git checkout release-branch
git cherry-pick a3
git cherry-pick a7

# Add note about partial backport
git commit --amend -m "$(git log -1 --pretty=%B)

Note: Partial backport from feature-branch. Only includes:
- a3: Performance optimization
- a7: UI improvement"
```

### Example 4: Interactive Cherry-Pick with Conflicts

**Scenario:** Cherry-picking multiple commits with conflicts.

```bash
# Cherry-pick a range of commits
git cherry-pick abc123^..def456

# Conflict occurs on commit bbb222
# Terminal shows:
# error: could not apply bbb222... Update user service
# hint: after resolving conflicts, mark the corrected paths
# hint: with 'git add <paths>' or 'git rm <paths>'
# hint: and commit the result with 'git cherry-pick --continue'

# Check what's in conflict
git status

# Resolve conflicts in your editor
vim src/UserService.php

# Stage resolved files
git add src/UserService.php

# Continue with remaining commits
git cherry-pick --continue

# If another conflict occurs, repeat the process
# If you want to stop, use:
# git cherry-pick --abort
```

### Example 5: Cherry-Pick from a Deleted Branch

**Scenario:** You deleted a branch but need one commit from it.

```bash
# Find the commit SHA from reflog
git reflog | grep "deleted-branch"

# Output might show:
# a1b2c3d HEAD@{10}: commit: Important feature implementation

# Cherry-pick the commit
git cherry-pick a1b2c3d

# Alternative: If you remember approximate time
git reflog --since="2 days ago" --until="1 day ago"
```

### Example 6: Cherry-Pick with File-Specific Changes

**Scenario:** A commit changed 5 files, but you only want changes to 2 files.

```bash
# Cherry-pick without committing
git cherry-pick -n abc123

# Unstage files you don't want
git restore --staged file3.js file4.js file5.js

# Also remove changes from working directory
git restore file3.js file4.js file5.js

# Commit only the files you want
git commit -m "Cherry-pick partial changes from abc123

Only includes changes to:
- file1.js
- file2.js"
```

### Example 7: Merge Commit Cherry-Pick

**Scenario:** Cherry-pick a merge commit to backport an entire feature.

```bash
# Find the merge commit
git log --merges --oneline

# Output shows:
# m1n2o3p Merge pull request #123 from feature/new-dashboard

# Cherry-pick the merge commit (using first parent)
git checkout release-branch
git cherry-pick -m 1 m1n2o3p

# If conflicts occur
git status
# Resolve conflicts
git add <resolved-files>
git cherry-pick --continue
```

### Example 8: Cherry-Pick Between Forks

**Scenario:** Apply commits from a contributor's fork.

```bash
# Add contributor's fork as remote
git remote add contributor https://github.com/contributor/repo.git

# Fetch their branches
git fetch contributor

# View their commits
git log contributor/feature-branch --oneline

# Cherry-pick specific commits
git checkout main
git cherry-pick abc123 def456

# Thank the contributor in commit message
git commit --amend -m "$(git log -1 --pretty=%B)

Co-authored-by: Contributor Name <contributor@email.com>
Cherry-picked from: contributor/feature-branch"
```

### Example 9: Automated Cherry-Pick Script

**Scenario:** Regularly backport commits from main to multiple release branches.

```bash
#!/bin/bash
# cherry-pick-to-releases.sh

COMMIT_SHA=$1
RELEASE_BRANCHES=("release-2.0" "release-2.1" "release-2.2")

if [ -z "$COMMIT_SHA" ]; then
    echo "Usage: ./cherry-pick-to-releases.sh <commit-sha>"
    exit 1
fi

for branch in "${RELEASE_BRANCHES[@]}"; do
    echo "Cherry-picking $COMMIT_SHA to $branch..."
    
    git checkout "$branch" || continue
    git pull origin "$branch" || continue
    
    if git cherry-pick -x "$COMMIT_SHA"; then
        echo "✓ Successfully cherry-picked to $branch"
        git push origin "$branch"
    else
        echo "✗ Conflicts on $branch - manual resolution needed"
        git cherry-pick --abort
    fi
done

git checkout main
echo "Done!"
```

Usage:
```bash
chmod +x cherry-pick-to-releases.sh
./cherry-pick-to-releases.sh a1b2c3d
```

### Example 10: Cherry-Pick with Reflog Recovery

**Scenario:** You aborted a cherry-pick but want to recover the resolution.

```bash
# Start cherry-pick
git cherry-pick abc123

# Conflicts occur, you resolve them
git add resolved-file.js

# Accidentally abort
git cherry-pick --abort

# Oh no! You lost your resolution
# Recovery using reflog

# Find the state before abort
git reflog
# Shows: HEAD@{1}: cherry-pick: Fast-forward

# Create a temporary branch at that state
git branch temp-recovery HEAD@{1}

# View the changes
git diff main temp-recovery

# Apply the changes
git checkout main
git cherry-pick temp-recovery

# Clean up
git branch -D temp-recovery
```

---

## Cherry-Pick vs Other Git Operations

### Cherry-Pick vs Merge

| Cherry-Pick | Merge |
|------------|-------|
| Applies specific commits | Applies entire branch |
| Creates duplicate commits | Preserves commit history |
| Changes SHA hashes | Maintains original SHAs |
| No branch relationship | Creates merge commit |
| Good for selective changes | Good for integration |

**Example:**
```bash
# Merge: Brings all commits from feature to main
git checkout main
git merge feature-branch

# Cherry-pick: Brings only specific commits
git checkout main
git cherry-pick abc123 def456
```

### Cherry-Pick vs Rebase

| Cherry-Pick | Rebase |
|------------|--------|
| Manual commit selection | Automatic commit replay |
| Can skip commits | Replays all commits |
| Explicit operation | Changes branch base |
| Good for cross-branch | Good for linear history |

**Example:**
```bash
# Rebase: Moves entire branch
git checkout feature
git rebase main

# Cherry-pick: Selectively applies commits
git checkout main
git cherry-pick feature~3  # Only one commit from feature
```

### Cherry-Pick vs Patch

| Cherry-Pick | Patch |
|------------|-------|
| Works with commits | Works with files |
| Preserves metadata | Manual patch application |
| Git-native operation | Unix-style patch |
| Easier for most cases | More portable |

**Example:**
```bash
# Cherry-pick: Direct commit application
git cherry-pick abc123

# Patch: Create and apply patch file
git format-patch -1 abc123
git apply 0001-commit-message.patch
```

---

## Best Practices

### 1. Always Use `-x` for Traceability

```bash
# Good: Records original commit
git cherry-pick -x abc123

# Result includes: "(cherry picked from commit abc123)"
```

### 2. Keep Cherry-Picks Minimal

```bash
# Bad: Cherry-picking many commits
git cherry-pick abc..xyz  # 20 commits

# Good: Consider merge instead for many commits
git merge feature-branch
```

### 3. Document Why You Cherry-Picked

```bash
# Cherry-pick with explanation
git cherry-pick -x abc123
git commit --amend -m "$(git log -1 --pretty=%B)

Cherry-picked to release-2.0 for critical security fix.
Original commit: abc123 on main branch.
Tested on staging environment before backport."
```

### 4. Test After Cherry-Pick

```bash
# After cherry-picking
git cherry-pick -x abc123

# Always test
npm test
./run-integration-tests.sh

# Only then push
git push origin release-branch
```

### 5. Use `--no-commit` for Multiple Related Commits

```bash
# Bad: Creates many small commits
git cherry-pick abc123
git cherry-pick def456
git cherry-pick ghi789

# Good: Combine related changes
git cherry-pick -n abc123
git cherry-pick -n def456
git cherry-pick -n ghi789
git commit -m "Backport feature X (commits abc, def, ghi)"
```

### 6. Verify What Will Be Cherry-Picked

```bash
# Before cherry-picking, review the changes
git show abc123

# Or show just the files
git show --name-only abc123

# Or show the diff
git show abc123 --stat
```

### 7. Keep Track of Cherry-Picked Commits

```bash
# Create a reference file for backports
echo "Release 2.0 Backports" > BACKPORTS.md
echo "- abc123: Security fix" >> BACKPORTS.md
echo "- def456: Performance improvement" >> BACKPORTS.md
git add BACKPORTS.md
git commit -m "docs: Track backported commits"
```

---

## Troubleshooting

### Problem 1: Cherry-Pick Conflicts

**Symptoms:**
```bash
$ git cherry-pick abc123
error: could not apply abc123... Update feature
hint: after resolving the conflicts, mark the corrected paths
```

**Solution:**
```bash
# Check what's in conflict
git status

# Option 1: Resolve manually
vim conflicted-file.js
git add conflicted-file.js
git cherry-pick --continue

# Option 2: Use theirs/ours strategy
git checkout --ours conflicted-file.js  # Keep current branch version
git checkout --theirs conflicted-file.js  # Use cherry-picked version
git add conflicted-file.js
git cherry-pick --continue

# Option 3: Skip this commit
git cherry-pick --skip

# Option 4: Abort completely
git cherry-pick --abort
```

### Problem 2: Cherry-Pick Wrong Commit

**Solution:**
```bash
# If not pushed yet
git reset --hard HEAD~1

# If already pushed
git revert HEAD
git push origin branch-name

# Alternative: Force push (use with caution)
git reset --hard HEAD~1
git push --force-with-lease origin branch-name
```

### Problem 3: Cherry-Pick Created Empty Commit

**Symptoms:**
```bash
$ git cherry-pick abc123
The previous cherry-pick is now empty, possibly due to conflict resolution.
```

**Solution:**
```bash
# Allow empty commit if intentional
git cherry-pick --allow-empty abc123

# Skip if not needed
git cherry-pick --skip

# Abort if unsure
git cherry-pick --abort
```

### Problem 4: Cherry-Pick Fails with Dirty Working Directory

**Symptoms:**
```bash
$ git cherry-pick abc123
error: Your local changes would be overwritten by cherry-pick.
```

**Solution:**
```bash
# Option 1: Stash your changes
git stash
git cherry-pick abc123
git stash pop

# Option 2: Commit your changes first
git add .
git commit -m "WIP: Save current work"
git cherry-pick abc123

# Option 3: Discard local changes (careful!)
git restore .
git cherry-pick abc123
```

### Problem 5: Can't Cherry-Pick Merge Commit

**Symptoms:**
```bash
$ git cherry-pick abc123
error: commit abc123 is a merge but no -m option was given.
```

**Solution:**
```bash
# Specify which parent to use
git cherry-pick -m 1 abc123  # Usually the main branch
git cherry-pick -m 2 abc123  # Usually the feature branch

# Check parents to decide
git show abc123
# Look for: Merge: a1b2c3d e4f5g6h
# -m 1 = a1b2c3d, -m 2 = e4f5g6h
```

### Problem 6: Cherry-Pick in Progress Blocking Other Operations

**Solution:**
```bash
# Check if cherry-pick is in progress
ls .git/ | grep CHERRY_PICK

# See what's happening
git status

# Complete it
git cherry-pick --continue

# Or abort it
git cherry-pick --abort

# Nuclear option: Remove cherry-pick state manually (last resort)
rm .git/CHERRY_PICK_HEAD
rm .git/sequencer -rf
```

---

## Common Pitfalls

### Pitfall 1: Cherry-Picking Too Many Commits

**Problem:**
```bash
# This creates duplicate history
git cherry-pick main~20..main
```

**Better Approach:**
```bash
# Use merge for many commits
git merge main
```

### Pitfall 2: Cherry-Picking Instead of Merging Regularly

**Problem:** Creates divergent histories and duplicate commits.

**Example:**
```bash
# Bad: Regular integration via cherry-pick
git cherry-pick feature-branch~5
git cherry-pick feature-branch~4
# ... this creates duplicates

# Good: Regular integration via merge
git merge feature-branch
```

### Pitfall 3: Not Testing After Cherry-Pick

**Problem:** Cherry-picked code might work differently in target context.

**Solution:**
```bash
git cherry-pick abc123

# Always test
npm test
npm run build
./run-manual-tests.sh

# Only push when tests pass
git push origin release-branch
```

### Pitfall 4: Forgetting `-x` Flag

**Problem:** No traceability to original commit.

**Solution:**
```bash
# Always use -x for backports
git cherry-pick -x abc123

# Result shows origin in commit message
```

### Pitfall 5: Cherry-Picking in Wrong Direction

**Problem:** Cherry-picking from old releases to main.

**Example:**
```bash
# Bad: Cherry-picking old fixes forward
git checkout main
git cherry-pick release-1.0

# Good: Merge old releases or fix on main first
# Fix on main, then cherry-pick to releases
git checkout main
git commit -m "Fix bug"
git cherry-pick -x HEAD --to-branch release-2.0
```

### Pitfall 6: Not Communicating Cherry-Picks to Team

**Problem:** Team confusion about commit duplicates.

**Solution:**
```bash
# Document in commit message
git cherry-pick -x abc123
git commit --amend -m "$(git log -1 --pretty=%B)

Backported from main for hotfix release.
QA: Tested on staging.
Approved-by: Tech Lead"

# Communicate in PR description
# Create pull request with clear explanation
```

---

## Tips and Tricks

### Tip 1: Find Commits to Cherry-Pick

```bash
# Show commits not in current branch
git log main..feature-branch --oneline

# Show commits in date range
git log --since="2024-01-01" --until="2024-01-31" --oneline

# Show commits by author
git log --author="John Doe" --oneline

# Show commits with specific message
git log --grep="hotfix" --oneline

# Show commits touching specific file
git log --oneline -- path/to/file.js
```

### Tip 2: Preview Cherry-Pick

```bash
# See what changes will be applied
git show abc123

# See which files will be changed
git show abc123 --name-only

# See statistics
git show abc123 --stat

# Apply changes without committing (dry run)
git cherry-pick -n abc123
git diff --cached
git reset HEAD .  # Undo the staging
```

### Tip 3: Cherry-Pick and Edit

```bash
# Cherry-pick with edited message
git cherry-pick -e abc123

# Cherry-pick and amend
git cherry-pick -n abc123
# Make changes
git add modified-files
git commit -m "Custom message for cherry-picked changes"
```

### Tip 4: Bulk Cherry-Pick with Loop

```bash
# Cherry-pick multiple commits with validation
for commit in abc123 def456 ghi789; do
    echo "Cherry-picking $commit"
    if git cherry-pick -x "$commit"; then
        echo "✓ Success"
    else
        echo "✗ Failed on $commit"
        git cherry-pick --abort
        break
    fi
done
```

### Tip 5: Cherry-Pick to Multiple Branches

```bash
# Function to cherry-pick to multiple branches
cherry_pick_to_branches() {
    local commit=$1
    shift
    local branches=("$@")
    
    local current_branch=$(git branch --show-current)
    
    for branch in "${branches[@]}"; do
        echo "Cherry-picking to $branch..."
        git checkout "$branch"
        git pull origin "$branch"
        
        if git cherry-pick -x "$commit"; then
            git push origin "$branch"
            echo "✓ Pushed to $branch"
        else
            echo "✗ Conflict on $branch"
            git cherry-pick --abort
        fi
    done
    
    git checkout "$current_branch"
}

# Usage
cherry_pick_to_branches abc123 release-2.0 release-2.1 release-2.2
```

### Tip 6: View Cherry-Pick History

```bash
# Find all cherry-picked commits
git log --all --grep="cherry picked from"

# Find cherry-picks on specific branch
git log release-2.0 --grep="cherry picked from"

# Show original and cherry-picked commit side by side
git log --all --grep="cherry picked from commit abc123"
```

### Tip 7: Cherry-Pick with GPG Signing

```bash
# Cherry-pick and sign
git cherry-pick -S abc123

# Configure to always sign
git config commit.gpgsign true
git cherry-pick abc123  # Automatically signed
```

### Tip 8: Cherry-Pick from Specific File State

```bash
# Cherry-pick only specific file from commit
git show abc123:path/to/file.js > temp-file.js
# Review temp-file.js
mv temp-file.js path/to/file.js
git add path/to/file.js
git commit -m "Backport specific file from abc123"
```

### Tip 9: Undo Cherry-Pick After Push

```bash
# Create revert commit
git revert HEAD
git push origin branch-name

# Or interactive rebase if other commits after
git rebase -i HEAD~5
# Mark cherry-picked commit as 'drop'
git push --force-with-lease origin branch-name
```

### Tip 10: Cherry-Pick with Custom Commit Message Template

```bash
# Create template
cat > ~/.git-cherry-pick-template << 'EOF'
Cherry-pick: 

Original commit: ${COMMIT_SHA}
Original branch: ${ORIGINAL_BRANCH}
Cherry-picked by: ${USER}
Date: ${DATE}
Reason: ${REASON}

Changes:
${CHANGES}

Testing:
${TESTING}
EOF

# Use in cherry-pick
git cherry-pick -n abc123
git commit -t ~/.git-cherry-pick-template
```

---

## Advanced Cherry-Pick Scenarios

### Scenario: Interactive Cherry-Pick with Selection

```bash
# View commits to select from
git log feature-branch --oneline

# Interactively select commits
git rebase -i main
# Change 'pick' to 'edit' for commits you want
# Then cherry-pick manually during rebase

# Alternative: Using git log with menu
git log --oneline feature-branch | fzf | cut -d' ' -f1 | xargs git cherry-pick
```

### Scenario: Cherry-Pick Across Repositories

```bash
# Create patch from source repo
cd /path/to/source-repo
git format-patch -1 abc123

# Apply to target repo
cd /path/to/target-repo
git am < /path/to/source-repo/0001-commit-message.patch
```

### Scenario: Automated Conflict Resolution

```bash
#!/bin/bash
# auto-resolve-conflicts.sh

git cherry-pick "$1"

if [ $? -ne 0 ]; then
    # Conflicts occurred
    conflicts=$(git diff --name-only --diff-filter=U)
    
    for file in $conflicts; do
        # Attempt automatic resolution
        if [[ $file == *.json ]]; then
            # Take theirs for JSON files
            git checkout --theirs "$file"
            git add "$file"
        elif [[ $file == *.lock ]]; then
            # Regenerate lock files
            npm install
            git add "$file"
        fi
    done
    
    # Continue if all resolved
    git cherry-pick --continue
fi
```

---

## Conclusion

Cherry-pick is a powerful tool for selective commit application. Use it wisely:

✅ **Do:**
- Use for hotfixes and backports
- Always use `-x` flag for traceability
- Test after cherry-picking
- Document your cherry-picks
- Resolve conflicts carefully

❌ **Don't:**
- Use for regular branch integration (use merge)
- Cherry-pick too many commits at once
- Forget to test after cherry-picking
- Create duplicate commits unnecessarily
- Skip conflict resolution

Remember: Cherry-pick creates duplicate commits. Use merge for regular integration workflows, and reserve cherry-pick for special cases where you need surgical precision.

---

## Quick Reference

```bash
# Basic cherry-pick
git cherry-pick <commit>

# Cherry-pick multiple commits
git cherry-pick <commit1> <commit2> <commit3>

# Cherry-pick range
git cherry-pick <commit1>..<commit2>

# Cherry-pick without committing
git cherry-pick -n <commit>

# Cherry-pick with edit
git cherry-pick -e <commit>

# Cherry-pick with traceability
git cherry-pick -x <commit>

# Cherry-pick merge commit
git cherry-pick -m 1 <merge-commit>

# Continue after resolving conflicts
git cherry-pick --continue

# Skip current commit
git cherry-pick --skip

# Abort cherry-pick
git cherry-pick --abort

# Cherry-pick with strategy
git cherry-pick -X theirs <commit>
```
