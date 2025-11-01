# Git Stash Guide

## What is Git Stash?

`git stash` is a powerful Git command that temporarily saves (stashes) uncommitted changes in your working directory, allowing you to switch contexts without committing incomplete work. It's like putting your current work in a drawer so you can work on something else and retrieve it later.

When you stash changes, Git saves both:
- **Staged changes** (files you've added with `git add`)
- **Unstaged changes** (modified tracked files)

After stashing, your working directory becomes clean, matching the HEAD commit.

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

## Additional Resources

- [Official Git Documentation - git-stash](https://git-scm.com/docs/git-stash)
- [Pro Git Book - Stashing](https://git-scm.com/book/en/v2/Git-Tools-Stashing-and-Cleaning)
- [Atlassian Git Stash Tutorial](https://www.atlassian.com/git/tutorials/saving-changes/git-stash)

