# Git Stash: Advanced Guide with Real-World Examples

## What is Git Stash?

`git stash` is a powerful Git command that temporarily saves (stashes) uncommitted changes in your working directory, allowing you to switch contexts without committing incomplete work. It's like putting your current work in a drawer so you can work on something else and retrieve it later.

When you stash changes, Git saves both:
- **Staged changes** (files you've added with `git add`)
- **Unstaged changes** (modified tracked files)

After stashing, your working directory becomes clean, matching the HEAD commit.

### Under the Hood

Git stash creates special commit objects:
- A commit for your working directory changes
- A commit for your index (staged changes)
- A special "stash" ref that points to these commits

```bash
# View the commit graph of a stash
git log --graph --oneline stash@{0}

# View the actual stash refs
git show-ref stash
```

---

## Why Use Git Stash?

### Common Use Cases

1. **Context Switching**: Need to switch branches urgently but have uncommitted changes
2. **Emergency Bug Fixes**: A critical bug needs immediate attention while you're mid-feature
3. **Clean Working Directory**: Need a clean state to test something or run commands
4. **Experiment Safely**: Try different approaches without losing your current work
5. **Pull/Rebase**: Need to pull changes but have local modifications that would conflict
6. **Code Review**: Switch to review a colleague's branch without committing incomplete work

---

## Basic Commands

### 1. Stash Your Changes

```bash
# Stash tracked files (modified and staged)
git stash

# Stash with a descriptive message
git stash push -m "WIP: implementing user authentication"

# Stash including untracked files
git stash -u
# or
git stash --include-untracked

# Stash everything including ignored files
git stash -a
# or
git stash --all
```

### 2. List Stashes

```bash
# Show all stashes
git stash list

# Output format:
# stash@{0}: WIP on main: 1234abc Last commit message
# stash@{1}: On feature-branch: 5678def Another message
```

### 3. View Stash Contents

```bash
# Show the most recent stash
git stash show

# Show detailed diff of the most recent stash
git stash show -p

# Show a specific stash
git stash show stash@{1}

# Detailed diff of a specific stash
git stash show -p stash@{2}
```

### 4. Apply Stashed Changes

```bash
# Apply the most recent stash (keeps it in stash list)
git stash apply

# Apply a specific stash
git stash apply stash@{1}

# Apply and remove from stash list in one command
git stash pop

# Pop a specific stash
git stash pop stash@{1}
```

### 5. Remove Stashes

```bash
# Remove the most recent stash
git stash drop

# Remove a specific stash
git stash drop stash@{1}

# Remove all stashes (use with caution!)
git stash clear
```

### 6. Create a Branch from Stash

```bash
# Create a new branch and apply the stash
git stash branch new-branch-name

# Create a branch from a specific stash
git stash branch new-branch-name stash@{1}
```

---

## Practical Examples

### Example 1: Emergency Context Switch

**Scenario**: You're working on a feature when a critical bug needs immediate attention.

```bash
# Currently on feature-branch with uncommitted changes
$ git status
On branch feature-login
Changes not staged for commit:
  modified:   src/auth/login.js
  modified:   src/auth/validation.js

# Stash your work
$ git stash push -m "WIP: login validation refactor"
Saved working directory and index state On feature-login: WIP: login validation refactor

# Switch to main branch to fix the bug
$ git checkout main

# Fix the bug, commit, and push
$ git add .
$ git commit -m "Fix critical authentication bug"
$ git push origin main

# Return to your feature branch
$ git checkout feature-login

# Restore your work
$ git stash pop
On branch feature-login
Changes not staged for commit:
  modified:   src/auth/login.js
  modified:   src/auth/validation.js

Dropped refs/stash@{0}
```

### Example 2: Testing Different Approaches

**Scenario**: You want to try a different approach without losing current work.

```bash
# Stash current implementation
$ git stash push -m "Approach 1: using callbacks"

# Try new approach
$ # ... make changes ...

# Not working? Discard and restore original
$ git reset --hard HEAD
$ git stash pop

# Or keep both approaches in separate stashes
$ git stash push -m "Approach 2: using promises"
$ git stash list
stash@{0}: On feature: Approach 2: using promises
stash@{1}: On feature: Approach 1: using callbacks
```

### Example 3: Pulling with Local Changes

**Scenario**: You need to pull remote changes but have local uncommitted work.

```bash
# You have local changes
$ git status
On branch main
Changes not staged for commit:
  modified:   README.md

# Try to pull
$ git pull
error: Your local changes would be overwritten by merge.
Please commit your changes or stash them before you merge.

# Stash your changes
$ git stash

# Pull successfully
$ git pull origin main

# Reapply your changes
$ git stash pop

# If there are conflicts, resolve them
# The stash remains in the list if pop fails due to conflicts
```

### Example 4: Stashing Specific Files

**Scenario**: You want to stash only certain files.

```bash
# Stash only specific files
$ git stash push -m "Only config changes" config/database.js config/api.js

# Or use pathspec
$ git stash push -m "Only test files" -- tests/

# Interactive stashing (choose what to stash)
$ git stash push -p
# Git will prompt for each change: stash this hunk [y,n,q,a,d,e,?]?
```

### Example 5: Recovering from Mistakes

**Scenario**: You accidentally dropped a stash or want to recover old stashes.

```bash
# List all stash refs including dropped ones
$ git fsck --unreachable | grep commit | cut -d ' ' -f3 | xargs git log --merges --no-walk

# Or use reflog
$ git reflog show stash

# Recover a dropped stash (use the commit hash)
$ git stash apply <commit-hash>

# Create a branch from a dropped stash
$ git branch recovered-branch <commit-hash>
```

### Example 6: Working Across Multiple Features

**Scenario**: Juggling multiple features simultaneously.

```bash
# Working on feature A
$ git stash push -m "Feature A: user profile updates"

# Switch to feature B
$ git checkout feature-b
# ... make changes ...
$ git stash push -m "Feature B: payment integration"

# Switch to feature C
$ git checkout feature-c
# ... make changes ...
$ git stash push -m "Feature C: email notifications"

# List all stashed work
$ git stash list
stash@{0}: On feature-c: Feature C: email notifications
stash@{1}: On feature-b: Feature B: payment integration
stash@{2}: On feature-a: Feature A: user profile updates

# Return to feature A
$ git checkout feature-a
$ git stash apply stash@{2}
```

---

## Advanced Usage

### Stash Untracked Files

By default, git stash only saves tracked files. To include untracked files:

```bash
# Include untracked files
$ git stash -u

# Include untracked AND ignored files
$ git stash -a
```

### Partial Stashing (Interactive Mode)

Choose exactly what to stash:

```bash
$ git stash push -p

# Interactive prompts:
# y - stash this hunk
# n - do not stash this hunk
# q - quit; do not stash this hunk or any remaining ones
# a - stash this hunk and all later hunks in the file
# d - do not stash this hunk or any later hunks in the file
# e - manually edit the current hunk
# ? - print help
```

### Stash with Keep Index

Keep staged changes in the index while stashing:

```bash
# Stash unstaged changes but keep staged changes
$ git stash --keep-index
```

---

## Best Practices

### 1. Use Descriptive Messages

```bash
# Bad
$ git stash

# Good
$ git stash push -m "WIP: refactoring authentication module - halfway through token validation"
```

### 2. Keep Stash List Clean

Don't let stashes accumulate. Regularly clean up:

```bash
# Review your stashes
$ git stash list

# Drop stashes you no longer need
$ git stash drop stash@{3}
```

### 3. Don't Use Stash as Long-term Storage

Stashes are temporary. For long-term storage:
- Create a proper commit
- Create a WIP (Work In Progress) branch
- Use feature branches

### 4. Apply vs Pop

- Use `git stash pop` when you're done with the stash
- Use `git stash apply` when you might need the stash again

### 5. Be Careful with Stash Clear

```bash
# This deletes ALL stashes permanently
$ git stash clear  # Use with extreme caution!
```

---

## Common Pitfalls and Solutions

### Pitfall 1: Forgetting About Stashed Work

**Problem**: Stashes pile up and you forget what's in them.

**Solution**: 
- Use descriptive messages
- Regularly review: `git stash list`
- Clean up old stashes

### Pitfall 2: Merge Conflicts When Popping

**Problem**: `git stash pop` causes conflicts.

**Solution**:
```bash
# Pop creates conflicts
$ git stash pop
# Resolve conflicts manually
$ git status
# After resolving, the stash remains in the list
$ git stash drop  # Remove it after successful resolution
```

### Pitfall 3: Stashing on Wrong Branch

**Problem**: Applied stash to wrong branch.

**Solution**:
```bash
# Undo the applied stash
$ git reset --hard HEAD

# Switch to correct branch
$ git checkout correct-branch

# Apply stash there
$ git stash apply
```

### Pitfall 4: Lost Stashes

**Problem**: Accidentally dropped a stash.

**Solution**: Use git reflog (see Example 5 above)

---

## Quick Reference Cheat Sheet

| Command | Description |
|---------|-------------|
| `git stash` | Stash tracked changes |
| `git stash -u` | Stash including untracked files |
| `git stash -a` | Stash including ignored files |
| `git stash push -m "msg"` | Stash with message |
| `git stash list` | List all stashes |
| `git stash show` | Show most recent stash |
| `git stash show -p` | Show diff of stash |
| `git stash apply` | Apply stash (keep in list) |
| `git stash pop` | Apply and remove stash |
| `git stash drop` | Remove most recent stash |
| `git stash drop stash@{n}` | Remove specific stash |
| `git stash clear` | Remove all stashes |
| `git stash branch <name>` | Create branch from stash |
| `git stash push -p` | Interactive stashing |

---

## Workflow Integration

### Git Stash in Daily Workflow

```bash
# Morning: Resume yesterday's work
$ git stash list
$ git stash pop

# During the day: Context switching
$ git stash push -m "WIP: current task"
$ git checkout urgent-fix-branch
# ... fix and commit ...
$ git checkout -
$ git stash pop

# End of day: Clean workspace
$ git stash push -m "EOD: $(date +%Y-%m-%d) - description of work"
```

### Integration with Git Hooks

You can create hooks to remind you about stashes:

```bash
# .git/hooks/post-checkout
#!/bin/bash
stash_count=$(git stash list | wc -l)
if [ $stash_count -gt 0 ]; then
    echo "⚠️  You have $stash_count stashed change(s)"
    git stash list
fi
```

---

## Conclusion

Git stash is an essential tool for managing your working directory and switching contexts efficiently. Key takeaways:

1. **Use stash for temporary storage** of uncommitted changes
2. **Always add descriptive messages** to your stashes
3. **Keep your stash list clean** - don't let it become a junk drawer
4. **Use branches for long-term storage**, not stashes
5. **Remember**: `apply` keeps the stash, `pop` removes it

Mastering git stash will significantly improve your Git workflow and productivity!

---

---

## Advanced Techniques

### 1. Stash and Apply to Multiple Branches

**Scenario**: You made changes that should be tested on multiple branches.

```bash
# On feature-branch-1 with uncommitted changes
$ git stash push -m "Database optimization changes"

# Apply to current branch
$ git stash apply

# Test and commit if good
$ git add .
$ git commit -m "Apply database optimizations"

# Switch to another branch and apply the same changes
$ git checkout feature-branch-2
$ git stash apply stash@{0}  # Stash is still available

# Test and commit
$ git add .
$ git commit -m "Apply database optimizations"

# Switch to main branch
$ git checkout main
$ git stash apply stash@{0}  # Apply same stash again

# When done, clean up the stash
$ git stash drop stash@{0}
```

### 2. Stashing with Pathspec Patterns

**Scenario**: Stash only files matching specific patterns.

```bash
# Stash all JavaScript files
$ git stash push -m "JS changes only" -- "*.js"

# Stash all files in a specific directory
$ git stash push -m "API changes" -- src/api/

# Stash multiple patterns
$ git stash push -m "Frontend assets" -- "*.css" "*.js" "*.html"

# Stash everything except specific files
$ git stash push -m "Everything except tests" -- . ":(exclude)tests/"

# Stash only modified files in specific subdirectories
$ git stash push -- src/controllers/ src/models/
```

### 3. Creating Commits from Stashes

**Scenario**: Convert a stash into a proper commit without applying it first.

```bash
# View stash as patches
$ git stash show -p stash@{0} > temp.patch

# Create a new branch from current HEAD
$ git checkout -b feature-from-stash

# Apply the stash as a commit
$ git stash apply stash@{0}
$ git add .
$ git commit -m "Convert stash to commit: database optimizations"

# Or use the stash branch command (creates branch + applies stash)
$ git stash branch feature-from-stash stash@{0}
```

### 4. Stash Inspection and Debugging

```bash
# Show detailed statistics of a stash
$ git stash show --stat stash@{1}

# Show only file names in stash
$ git stash show --name-only

# Show the parent commit of a stash
$ git rev-parse stash@{0}^

# Compare stash with current working directory
$ git diff stash@{0}

# Compare two stashes
$ git diff stash@{0} stash@{1}

# Search for specific content in all stashes
$ git stash list | while read stash; do
    echo "Checking $stash"
    git stash show -p "$stash" | grep -q "searchTerm" && echo "Found in $stash"
done
```

### 5. Conditional Stashing in Scripts

**Scenario**: Automate stashing in deployment or build scripts.

```bash
#!/bin/bash
# Smart stash for deployment scripts

# Check if there are changes to stash
if ! git diff-index --quiet HEAD --; then
    echo "Uncommitted changes detected. Stashing..."
    STASHED=true
    git stash push -u -m "Auto-stash before deployment $(date +%Y-%m-%d_%H:%M:%S)"
else
    echo "No changes to stash."
    STASHED=false
fi

# Perform deployment operations
git pull origin main
npm install
npm run build

# Restore stashed changes if any
if [ "$STASHED" = true ]; then
    echo "Restoring stashed changes..."
    if git stash pop; then
        echo "Successfully restored changes."
    else
        echo "Conflicts detected. Please resolve manually."
        exit 1
    fi
fi
```

### 6. Stash with Custom Metadata

**Scenario**: Add rich metadata to stashes for better tracking.

```bash
# Stash with timestamp and ticket number
$ TICKET="JIRA-1234"
$ git stash push -m "[$TICKET] $(date +%Y-%m-%d) - User authentication refactor"

# Stash with current branch name
$ BRANCH=$(git branch --show-current)
$ git stash push -m "[$BRANCH] WIP: implementing feature"

# Create a stash naming convention
$ git stash push -m "[$(git branch --show-current)] [$(date +%Y-%m-%d)] - Description"

# Example output:
# stash@{0}: On main: [feature-auth] [2025-11-01] - OAuth integration
```

---

## Enterprise & Team Workflows

### 1. Code Review Workflow with Stash

**Scenario**: You're reviewing a PR while working on your own feature.

```bash
# You're on feature-user-dashboard with uncommitted work
$ git status
On branch feature-user-dashboard
Changes not staged for commit:
  modified:   src/components/Dashboard.js
  modified:   src/services/userService.js

# Stash your work with detailed context
$ git stash push -m "[REVIEW-PAUSE] Dashboard improvements - halfway through chart integration"

# Checkout the PR branch for review
$ git fetch origin pull/123/head:pr-123
$ git checkout pr-123

# Review, test, and leave comments
$ npm test
$ npm run lint

# Done with review, return to your work
$ git checkout feature-user-dashboard

# Check what you had stashed
$ git stash show -p
# Verify it's the right stash
$ git stash list

# Restore your work
$ git stash pop

# Continue working
```

### 2. Hotfix Workflow in Production Environment

**Scenario**: Production issue needs immediate fix while you're mid-feature.

```bash
# Working on feature-payment-gateway
$ git stash push -u -m "[HOTFIX-INTERRUPT] Payment gateway integration - WIP"

# Create hotfix branch from production
$ git checkout main
$ git pull origin main
$ git checkout -b hotfix/critical-login-issue

# Fix the issue
$ vim src/auth/login.js
$ git add src/auth/login.js
$ git commit -m "hotfix: resolve login timeout issue"

# Push and create PR
$ git push origin hotfix/critical-login-issue

# After merge, cleanup and return to feature work
$ git checkout main
$ git pull origin main
$ git checkout feature-payment-gateway

# Check if your stashed changes conflict with the hotfix
$ git diff stash@{0}..HEAD

# Apply stash
$ git stash pop

# Rebase on updated main if needed
$ git fetch origin main
$ git rebase origin/main
```

### 3. Multi-Developer Stash Sharing (via Patches)

**Scenario**: Share uncommitted work with team member without committing.

```bash
# Developer A: Create patch from stash
$ git stash push -m "WIP: shared authentication logic"
$ git stash show -p stash@{0} > auth-wip.patch

# Share the patch file (email, Slack, shared drive)
$ cat auth-wip.patch

# Developer B: Apply the patch
$ git apply auth-wip.patch

# Or apply with 3-way merge for better conflict handling
$ git apply --3way auth-wip.patch

# Verify the changes
$ git diff
```

### 4. Stash in CI/CD Pipeline

**Scenario**: Preserve developer changes during automated testing.

```bash
#!/bin/bash
# .github/workflows/pre-commit-test.sh

echo "Running pre-commit tests..."

# Stash unstaged changes (keep staged for testing)
HAS_UNSTAGED=$(git diff --quiet || echo "yes")

if [ "$HAS_UNSTAGED" = "yes" ]; then
    echo "Stashing unstaged changes..."
    git stash push --keep-index -m "Pre-commit test stash"
fi

# Run tests on staged changes
npm run test:staged
npm run lint:staged

TEST_RESULT=$?

# Restore unstaged changes
if [ "$HAS_UNSTAGED" = "yes" ]; then
    echo "Restoring unstaged changes..."
    git stash pop
fi

# Exit with test result
exit $TEST_RESULT
```

### 5. Feature Flag Development Workflow

**Scenario**: Testing features with different flag combinations.

```bash
# Stash current feature flag configuration
$ git stash push -m "Feature flags: Auth v2 enabled" config/features.json

# Test with different configuration
$ vim config/features.json  # Enable different flags
$ npm test

# Stash this configuration
$ git stash push -m "Feature flags: Payment v3 enabled" config/features.json

# Test with another configuration
$ vim config/features.json  # Enable another set
$ npm test

# Compare results from different flag combinations
$ git stash list
stash@{0}: On main: Feature flags: Payment v3 enabled
stash@{1}: On main: Feature flags: Auth v2 enabled

# Apply optimal configuration
$ git stash apply stash@{1}
```

---

## Complex Real-World Scenarios

### Scenario 1: Emergency Rollback with Stash Recovery

**Context**: Deployed code has issues, need to rollback and recover in-progress work.

```bash
# Currently on feature-branch with uncommitted changes
$ git stash push -u -m "[EMERGENCY] Pre-rollback stash - $(date +%Y-%m-%d_%H:%M)"

# Checkout to last known good commit
$ git checkout main
$ git log --oneline -10  # Find last good commit
$ git checkout abc123def

# Create rollback branch
$ git checkout -b rollback/production-fix

# Test this version
$ npm test
$ npm run build

# If good, deploy this version
$ ./deploy.sh

# Return to feature work after incident resolution
$ git checkout feature-branch

# Recover your work
$ git stash list | grep "EMERGENCY"
$ git stash apply stash@{0}

# Rebase on latest main if needed
$ git fetch origin main
$ git rebase origin/main
```

### Scenario 2: Database Migration with Stash

**Context**: Testing database migrations with different code states.

```bash
# Current state: migration code written but not tested
$ git stash push -m "Migration v1: add user_preferences table"

# Rollback to test current production state
$ git checkout HEAD~1

# Run existing migrations
$ php bin/magento setup:upgrade

# Return to migration branch
$ git checkout -

# Apply stash and test migration
$ git stash pop

# Run new migration
$ php bin/magento setup:upgrade

# If migration fails, stash changes and debug
$ git stash push -m "Failed migration - debugging"

# Create test branch for alternative approach
$ git checkout -b migration-alternative

# Try different approach
$ vim Setup/UpgradeSchema.php

# Compare approaches
$ git stash show -p stash@{0}  # View original approach
```

### Scenario 3: Merging Stashed Changes from Multiple Branches

**Context**: Consolidating work from multiple experimental branches.

```bash
# On experiment-1 branch
$ git stash push -m "Experiment 1: Redis caching implementation"

# Switch to experiment-2
$ git checkout experiment-2
$ # ... make changes ...
$ git stash push -m "Experiment 2: Varnish caching implementation"

# Switch to experiment-3
$ git checkout experiment-3
$ # ... make changes ...
$ git stash push -m "Experiment 3: In-memory caching implementation"

# Create consolidation branch
$ git checkout main
$ git checkout -b feature-caching-consolidation

# Apply and merge all experiments
$ git stash apply stash@{2}  # Experiment 1
$ git add .
$ git commit -m "Add Redis caching base"

$ git stash apply stash@{1}  # Experiment 2
# Resolve conflicts
$ git add .
$ git commit -m "Integrate Varnish caching"

$ git stash apply stash@{0}  # Experiment 3
# Resolve conflicts
$ git add .
$ git commit -m "Add in-memory caching fallback"

# Clean up stashes
$ git stash drop stash@{2}
$ git stash drop stash@{1}
$ git stash drop stash@{0}
```

### Scenario 4: Bisecting with Stash

**Context**: Finding which commit introduced a bug while preserving current work.

```bash
# You're working on a feature when you discover a bug
$ git stash push -u -m "Pre-bisect stash: feature work"

# Start bisection
$ git bisect start
$ git bisect bad HEAD  # Current version is bad
$ git bisect good v2.4.0  # Known good version

# Git checks out a commit halfway between
# Test the commit
$ npm test

# Mark as good or bad
$ git bisect good  # or 'bad'

# Continue until bug is found
# Git will report: "abc123def is the first bad commit"

# End bisection
$ git bisect reset

# Return to your branch
$ git checkout feature-branch

# Restore your work
$ git stash pop
```

### Scenario 5: Stash-Based Backup Before Risky Operations

**Context**: Creating safety checkpoints during complex refactoring.

```bash
# Before starting major refactoring
$ git stash push -u -m "[CHECKPOINT-1] Before refactoring auth module"
$ git stash apply  # Keep changes in working directory

# Make some changes
$ # ... refactor authentication code ...

# Create another checkpoint
$ git stash push -u -m "[CHECKPOINT-2] After splitting auth service"
$ git stash apply

# Continue refactoring
$ # ... more changes ...

# Create final checkpoint
$ git stash push -u -m "[CHECKPOINT-3] After adding new auth providers"
$ git stash apply

# If something goes wrong, rollback to checkpoint
$ git reset --hard HEAD
$ git stash apply stash@{1}  # Apply CHECKPOINT-2

# View all checkpoints
$ git stash list | grep CHECKPOINT
```

---

## Performance and Optimization

### 1. Large Repository Stash Performance

```bash
# For large repositories, stashing can be slow
# Optimize by stashing only necessary paths

# Bad: Stash everything (slow in large repos)
$ git stash -u

# Good: Stash specific directories
$ git stash push -m "Only source code" -- src/ lib/

# Even better: Stash specific file patterns
$ git stash push -m "Only PHP files" -- "*.php"

# Check stash size
$ git cat-file -p stash@{0}^{tree} | head -20
```

### 2. Stash Cleanup and Maintenance

```bash
# List stashes with creation dates
$ git stash list --date=relative

# Find old stashes (older than 30 days)
$ git stash list --date=unix | awk -v cutoff=$(date -d '30 days ago' +%s) '$3 < cutoff'

# Create a cleanup script
#!/bin/bash
echo "Stashes older than 30 days:"
git stash list --date=relative | grep -E "(month|year)" 

read -p "Delete old stashes? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    # Archive old stashes before deletion
    mkdir -p .git/stash-archive
    git stash list --date=relative | grep -E "(month|year)" | while read line; do
        stash_ref=$(echo $line | cut -d: -f1)
        git stash show -p "$stash_ref" > ".git/stash-archive/$stash_ref-$(date +%Y%m%d).patch"
    done
    
    # Note: Git doesn't support selective stash deletion by age easily
    # Manual review and deletion recommended
    echo "Old stashes archived to .git/stash-archive/"
fi
```

### 3. Stash Compression for Storage

```bash
# Export stash as compressed patch
$ git stash show -p stash@{0} | gzip > stash-backup-$(date +%Y%m%d).patch.gz

# Restore from compressed patch
$ gunzip -c stash-backup-20251101.patch.gz | git apply

# Create stash archive system
#!/bin/bash
ARCHIVE_DIR=".git/stash-archives/$(date +%Y/%m)"
mkdir -p "$ARCHIVE_DIR"

for stash in $(git stash list | cut -d: -f1); do
    filename=$(echo $stash | tr '@{}' '___')
    git stash show -p "$stash" | gzip > "$ARCHIVE_DIR/$filename.patch.gz"
done

echo "Archived $(git stash list | wc -l) stashes to $ARCHIVE_DIR"
```

---

## Advanced Recovery Techniques

### 1. Recovering Deleted Stashes

```bash
# View all unreachable commits (including dropped stashes)
$ git fsck --unreachable | grep commit

# More focused: use reflog
$ git reflog show --all | grep stash

# Find dropped stash by date
$ git log --graph --oneline --all --reflog --grep="WIP on"

# Recover a specific dropped stash
$ git stash apply <commit-hash>

# Or create a branch from the dropped stash
$ git branch recovered-stash <commit-hash>
$ git checkout recovered-stash
```

### 2. Extracting Specific Files from Stash

```bash
# Checkout single file from stash without applying entire stash
$ git checkout stash@{0} -- src/config/database.php

# Checkout multiple files
$ git checkout stash@{0} -- src/models/User.php src/models/Role.php

# Extract file to different location
$ git show stash@{0}:src/config/app.php > config-backup.php

# Compare file in stash with current version
$ git diff stash@{0} -- src/services/auth.php
```

### 3. Merging Stashes

**Scenario**: Combine changes from multiple stashes.

```bash
# Apply first stash
$ git stash apply stash@{2}
$ git add .

# Apply second stash (may cause conflicts)
$ git stash apply stash@{1}

# Resolve conflicts manually
$ git status
$ vim conflicted-file.js

# Add resolved files
$ git add .

# Apply third stash
$ git stash apply stash@{0}

# Resolve any conflicts
$ git add .

# Commit the merged result
$ git commit -m "Merge changes from multiple stash entries"

# Clean up stashes
$ git stash clear  # Or drop individually
```

### 4. Stash Archaeology: Finding Lost Work

```bash
# Search for specific content in all stashes
function stash-search() {
    local search_term="$1"
    echo "Searching for '$search_term' in all stashes..."
    
    git stash list | while read stash; do
        local stash_ref=$(echo $stash | cut -d: -f1)
        if git stash show -p "$stash_ref" | grep -q "$search_term"; then
            echo "========================================="
            echo "Found in: $stash"
            echo "========================================="
            git stash show -p "$stash_ref" | grep -C 3 "$search_term"
        fi
    done
}

# Usage
$ stash-search "function login"

# Find stash by file path
function stash-by-file() {
    local file_path="$1"
    git stash list | while read stash; do
        local stash_ref=$(echo $stash | cut -d: -f1)
        if git stash show --name-only "$stash_ref" | grep -q "$file_path"; then
            echo "$stash"
        fi
    done
}

# Usage
$ stash-by-file "src/auth/login.js"
```

---

## Stash Automation and Tooling

### 1. Git Aliases for Stash Operations

Add these to your `~/.gitconfig`:

```ini
[alias]
    # Stash with auto-generated message
    stash-auto = "!f() { git stash push -m \"[$(git branch --show-current)] $(date +'%Y-%m-%d %H:%M') - ${1:-WIP}\"; }; f"
    
    # List stashes with better formatting
    stash-list = stash list --date=relative --format='%C(yellow)%gd%C(reset): %C(cyan)%<(70,trunc)%gs%C(reset) %C(green)(%cr)%C(reset)'
    
    # Show stash with stat
    stash-stat = "!f() { git stash show --stat ${1:-stash@{0}}; }; f"
    
    # Interactive stash selection
    stash-pick = "!git stash list | fzf --preview 'git stash show -p {1}' | cut -d: -f1 | xargs git stash apply"
    
    # Stash only unstaged changes
    stash-unstaged = "!git stash push --keep-index -m \"Unstaged changes - $(date +'%Y-%m-%d %H:%M')\""
    
    # Create branch from stash
    stash-branch = "!f() { git stash branch ${1} ${2:-stash@{0}}; }; f"
    
    # Save and create a backup
    stash-snapshot = "!f() { git stash push -u -m \"Snapshot: $(date +'%Y-%m-%d %H:%M:%S')\" && git stash apply; }; f"
```

Usage:

```bash
$ git stash-auto "working on auth feature"
$ git stash-list
$ git stash-pick  # Interactive selection with fzf
$ git stash-snapshot  # Create snapshot without removing changes
```

### 2. Pre-commit Hook with Auto-stash

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash

# Auto-stash unstaged changes before commit
# This ensures only staged changes are committed

if [ -n "$(git diff)" ]; then
    echo "Unstaged changes detected. Auto-stashing..."
    git stash push --keep-index -u -m "[PRE-COMMIT] Auto-stash $(date +'%Y-%m-%d %H:%M:%S')"
    STASHED=1
fi

# Run tests on staged changes
npm run test:staged
TESTS_PASSED=$?

# Restore unstaged changes
if [ "$STASHED" = "1" ]; then
    echo "Restoring unstaged changes..."
    git stash pop
fi

exit $TESTS_PASSED
```

### 3. Stash Dashboard Script

Create `bin/stash-dashboard.sh`:

```bash
#!/bin/bash

# Stash Dashboard - Visual overview of all stashes

echo "╔════════════════════════════════════════════════════════════════╗"
echo "║                    GIT STASH DASHBOARD                         ║"
echo "╚════════════════════════════════════════════════════════════════╝"
echo ""

STASH_COUNT=$(git stash list | wc -l | tr -d ' ')

if [ "$STASH_COUNT" -eq 0 ]; then
    echo "✓ No stashes found. Working directory is clean."
    exit 0
fi

echo "Total Stashes: $STASH_COUNT"
echo ""

git stash list --date=relative | while IFS=: read -r ref branch rest; do
    # Get stash details
    MODIFIED=$(git stash show --stat "$ref" 2>/dev/null | tail -1 | awk '{print $1, $4}')
    FILES=$(git stash show --name-only "$ref" 2>/dev/null | wc -l | tr -d ' ')
    DATE=$(git show -s --format=%ar "$ref" 2>/dev/null)
    
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📦 $ref"
    echo "   Branch: $branch"
    echo "   Message: $rest"
    echo "   Files: $FILES | Changes: $MODIFIED"
    echo "   Created: $DATE"
    echo ""
done

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Commands:"
echo "  git stash apply stash@{N}  - Apply stash N"
echo "  git stash drop stash@{N}   - Delete stash N"
echo "  git stash show -p stash@{N} - View stash N contents"
```

Make it executable and use:

```bash
$ chmod +x bin/stash-dashboard.sh
$ ./bin/stash-dashboard.sh
```

### 4. Stash Expiry Warning System

Create `.git/hooks/post-checkout`:

```bash
#!/bin/bash

# Warn about old stashes after checkout

DAYS_OLD=7
CUTOFF_DATE=$(date -d "$DAYS_OLD days ago" +%s 2>/dev/null || date -v-${DAYS_OLD}d +%s)

OLD_STASHES=$(git reflog show stash --date=unix 2>/dev/null | \
    awk -v cutoff="$CUTOFF_DATE" '$3 ~ /^[0-9]+$/ && $3 < cutoff' | wc -l)

if [ "$OLD_STASHES" -gt 0 ]; then
    echo ""
    echo "⚠️  WARNING: You have $OLD_STASHES stash(es) older than $DAYS_OLD days"
    echo "   Run 'git stash list --date=relative' to review them"
    echo "   Consider cleaning up with 'git stash drop stash@{N}'"
    echo ""
fi
```

---

## Troubleshooting Guide

### Issue 1: Stash Pop Fails with Conflicts

**Symptom**: `git stash pop` results in merge conflicts.

**Solution**:

```bash
# Pop failed - conflicts detected
$ git stash pop
CONFLICT (content): Merge conflict in src/app.js
The stash entry is kept in case you need it again.

# View conflict markers
$ git diff

# Resolve conflicts manually
$ vim src/app.js

# Mark as resolved
$ git add src/app.js

# Verify all conflicts are resolved
$ git status

# Manually drop the stash (pop didn't auto-drop due to conflicts)
$ git stash drop stash@{0}
```

**Alternative approach**:

```bash
# Use apply instead of pop (safer for conflicts)
$ git stash apply

# If conflicts, reset and try different strategy
$ git reset --hard HEAD
$ git stash apply --index  # Try to preserve staged/unstaged distinction
```

### Issue 2: Stash Contains Deleted Files

**Symptom**: Files in stash were deleted in current branch.

**Solution**:

```bash
# Check what's in the stash
$ git stash show --name-status stash@{0}

# Apply selectively
$ git show stash@{0}:path/to/deleted/file.js > restored-file.js

# Or apply the full stash and handle deleted files
$ git stash apply stash@{0}  # Will recreate deleted files

# If you don't want the deleted files back
$ git stash show --name-only stash@{0} | while read file; do
    if [ ! -f "$file" ]; then
        git rm "$file" 2>/dev/null
    fi
done
```

### Issue 3: Stash Applied to Wrong Branch

**Symptom**: Applied stash to incorrect branch by mistake.

**Solution**:

```bash
# Undo the stash application immediately
$ git reset --hard HEAD

# Or if you've made additional changes
$ git stash push -m "Additional work on wrong branch"
$ git reset --hard HEAD
$ git stash pop  # Get back just the additional work

# Switch to correct branch
$ git checkout correct-branch

# Apply the original stash
$ git stash apply stash@{1}  # Original stash
```

### Issue 4: Can't Find Important Stashed Work

**Symptom**: Lost track of which stash contains needed changes.

**Solution**:

```bash
# Search all stashes for specific content
$ for i in $(git stash list | cut -d: -f1); do 
    echo "=== $i ==="
    git stash show -p "$i" | grep -C 2 "searchterm"
done

# Interactive search with preview
$ git stash list | fzf --preview 'git stash show -p {1} | head -100'

# Export all stashes for external search
$ mkdir stash-exports
$ git stash list | while read stash; do
    ref=$(echo $stash | cut -d: -f1)
    filename=$(echo $ref | tr '@{}' '___')
    git stash show -p "$ref" > "stash-exports/$filename.patch"
done
$ grep -r "searchterm" stash-exports/
```

### Issue 5: Stash is Too Large

**Symptom**: Stash contains too many files, making operations slow.

**Solution**:

```bash
# Split stash into smaller stashes
$ git stash apply stash@{0}

# Stash by directory
$ git stash push -m "Part 1: Backend" -- backend/
$ git stash push -m "Part 2: Frontend" -- frontend/
$ git stash push -m "Part 3: Tests" -- tests/

# Drop the original large stash
$ git stash drop stash@{3}

# Or use interactive mode to split
$ git reset --hard HEAD
$ git stash apply stash@{0}
$ git add -p  # Stage selectively
$ git stash push --staged -m "Part 1"
$ git stash push -m "Part 2"  # Remaining changes
```

---

## Best Practices for Teams

### 1. Stash Naming Conventions

Establish team standards:

```bash
# Format: [TYPE] [TICKET] Description
$ git stash push -m "[WIP] [JIRA-1234] User authentication refactor"
$ git stash push -m "[EXPERIMENT] [JIRA-1234] Alternative caching approach"
$ git stash push -m "[BACKUP] [JIRA-1234] Before risky merge"
$ git stash push -m "[REVIEW-PAUSE] Code review interruption"
$ git stash push -m "[HOTFIX-PREP] Pre-emergency stash"

# Include branch context
$ git stash push -m "[$(git branch --show-current)] [JIRA-1234] Description"
```

### 2. Stash Hygiene Rules

```bash
# Daily stash cleanup ritual
$ git stash list --date=relative

# Drop stashes older than 3 days
$ git stash list --date=relative | grep -E "days? ago" | cut -d: -f1

# Weekly stash review
$ echo "Reviewing stashes..."
$ git stash list | while read stash; do
    echo "$stash"
    read -p "Keep this stash? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        ref=$(echo $stash | cut -d: -f1)
        git stash drop "$ref"
    fi
done
```

### 3. Documentation Integration

Document stashes in commit messages when appropriate:

```bash
# When committing after stash experiments
$ git commit -m "Implement caching layer

Based on experiments in stash@{2} (Redis) and stash@{1} (Varnish).
Chose Redis approach for better integration with existing architecture.

Related stashes archived in: .git/stash-archives/2025-11-01/"
```

### 4. Team Communication

```bash
# Before sharing branches with team
$ git stash list
# Ensure no stashes prevent proper branch testing

# Creating shareable patch from stash
$ git stash show -p stash@{0} > feature-wip.patch
# Share via: "Here's my WIP - git apply < feature-wip.patch"

# Before going on vacation
$ git stash list
# Either commit or document stashes for team
$ git stash show -p stash@{0} > vacation-backup-$(date +%Y%m%d).patch
```

---

## Platform-Specific Tips

### macOS

```bash
# Use macOS Notification Center for stash reminders
$ cat > ~/.git-templates/hooks/post-checkout << 'EOF'
#!/bin/bash
STASH_COUNT=$(git stash list | wc -l | tr -d ' ')
if [ "$STASH_COUNT" -gt 0 ]; then
    osascript -e "display notification \"$STASH_COUNT stashed changes\" with title \"Git Stash Reminder\""
fi
EOF

# Integrate with macOS Spotlight
$ git stash show -p stash@{0} > ~/Documents/git-stashes/$(date +%Y%m%d)-stash.txt
```

### Linux

```bash
# Use notify-send for stash notifications
$ notify-send "Git Stash" "You have $(git stash list | wc -l) stashed changes"

# Integrate with system journal
$ git stash list | systemd-cat -t git-stash -p info
```

### Windows

```powershell
# PowerShell stash helpers
function Show-GitStashes {
    $stashes = git stash list
    if ($stashes) {
        $stashes | ForEach-Object {
            Write-Host $_ -ForegroundColor Yellow
        }
    } else {
        Write-Host "No stashes found" -ForegroundColor Green
    }
}

# Windows Toast Notification
$stashCount = (git stash list).Count
if ($stashCount -gt 0) {
    New-BurntToastNotification -Text "Git Stash", "$stashCount stashed changes"
}
```

---

## Integration with Development Tools

### 1. VS Code Integration

Create `.vscode/tasks.json`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Git: Stash with Message",
      "type": "shell",
      "command": "git stash push -m \"${input:stashMessage}\"",
      "problemMatcher": []
    },
    {
      "label": "Git: Stash List",
      "type": "shell",
      "command": "git stash list --date=relative",
      "problemMatcher": []
    },
    {
      "label": "Git: Stash Pop",
      "type": "shell",
      "command": "git stash pop",
      "problemMatcher": []
    }
  ],
  "inputs": [
    {
      "id": "stashMessage",
      "type": "promptString",
      "description": "Stash message"
    }
  ]
}
```

### 2. IDE Integration Scripts

```bash
# JetBrains IDEs (PHPStorm, WebStorm, etc.)
# Add as External Tool in Settings > Tools > External Tools

# Name: Smart Stash
# Program: git
# Arguments: stash push -u -m "[$(git branch --show-current)] WIP - $(date +%Y-%m-%d)"
# Working directory: $ProjectFileDir$

# Name: Stash Dashboard
# Program: bash
# Arguments: -c "git stash list | cat"
# Working directory: $ProjectFileDir$
```

### 3. CI/CD Integration

```yaml
# GitHub Actions - Preserve developer changes during CI
name: Test with Stash Preservation
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Check for uncommitted changes
        id: check_changes
        run: |
          if ! git diff-index --quiet HEAD --; then
            echo "has_changes=true" >> $GITHUB_OUTPUT
            git stash push -u -m "CI run stash"
          fi
      
      - name: Run tests
        run: npm test
      
      - name: Restore changes
        if: steps.check_changes.outputs.has_changes == 'true'
        run: git stash pop
```

---

## Stash Metrics and Analytics

### Track Stash Usage

```bash
#!/bin/bash
# stash-metrics.sh - Analyze stash usage patterns

echo "=== Stash Metrics ==="
echo ""

# Total stashes
TOTAL=$(git stash list | wc -l)
echo "Total stashes: $TOTAL"

# Average stash age
if [ "$TOTAL" -gt 0 ]; then
    echo ""
    echo "Age distribution:"
    git stash list --date=relative | cut -d: -f3 | sort | uniq -c | sort -rn
    
    echo ""
    echo "Stashes by branch:"
    git stash list | cut -d: -f2 | sed 's/^ On //' | sed 's/^ WIP on //' | sort | uniq -c | sort -rn
    
    echo ""
    echo "Largest stashes (by file count):"
    git stash list | while read stash; do
        ref=$(echo $stash | cut -d: -f1)
        count=$(git stash show --name-only "$ref" 2>/dev/null | wc -l)
        echo "$count files - $ref"
    done | sort -rn | head -5
fi
```

---

## Additional Resources

- [Official Git Documentation - git-stash](https://git-scm.com/docs/git-stash)
- [Pro Git Book - Stashing](https://git-scm.com/book/en/v2/Git-Tools-Stashing-and-Cleaning)
- [Atlassian Git Stash Tutorial](https://www.atlassian.com/git/tutorials/saving-changes/git-stash)
- [Git Stash Internals](https://git-scm.com/docs/git-stash#_discussion)

