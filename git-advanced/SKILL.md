---
name: git-advanced
description: "Advanced Git operations and workflows including interactive rebasing, conflict resolution, history manipulation, bisecting for bugs, cherry-picking, reflog recovery, and branch management strategies. Use for: (1) Interactive rebasing and commit cleanup, (2) Complex merge conflict resolution, (3) Git bisect for bug hunting, (4) History rewriting and cleanup, (5) Branch strategy implementation (Git Flow, trunk-based), (6) Recovering lost commits with reflog"
---

# Advanced Git Operations

## Overview

Master advanced Git workflows for complex version control scenarios. This skill covers sophisticated operations beyond basic commits and merges, including interactive rebasing, advanced conflict resolution, history manipulation, and strategic branch management.

## Core Capabilities

### Interactive Rebasing
- Reorder commits for logical history
- Squash multiple commits into one
- Edit commit messages retroactively
- Split commits into smaller pieces
- Remove unwanted commits from history

### Conflict Resolution
- Strategic merge conflict handling
- Three-way merge understanding
- Ours vs theirs strategy selection
- Conflict marker interpretation
- Partial file staging during conflicts

### Git Bisect
- Binary search for bug introduction
- Automated bisect with scripts
- Good/bad commit identification
- Efficient debugging of regressions

### History Manipulation
- Amend commits safely
- Filter-branch operations
- BFG Repo-Cleaner for large files
- Rewrite history with caution
- Preserve attribution and dates

### Branch Strategies
- Git Flow workflow
- Trunk-based development
- Feature branch workflows
- Release branch management
- Hotfix procedures

## Quick Command Reference

### Interactive Rebase
```bash
# Rebase last 5 commits interactively
git rebase -i HEAD~5

# Rebase onto main branch
git rebase -i main

# Continue after resolving conflicts
git rebase --continue

# Abort and return to original state
git rebase --abort

# Skip problematic commit
git rebase --skip
```

### Conflict Resolution
```bash
# Accept ours (current branch)
git checkout --ours <file>

# Accept theirs (incoming branch)
git checkout --theirs <file>

# Show conflict in 3-way diff
git diff --ours
git diff --theirs

# List conflicted files
git diff --name-only --diff-filter=U

# Mark as resolved
git add <file>
```

### Git Bisect
```bash
# Start bisect session
git bisect start

# Mark current commit as bad
git bisect bad

# Mark known good commit
git bisect good <commit-hash>

# Automate with test script
git bisect run ./test-script.sh

# End bisect session
git bisect reset
```

### Cherry-Pick
```bash
# Apply specific commit
git cherry-pick <commit-hash>

# Cherry-pick without committing
git cherry-pick -n <commit-hash>

# Cherry-pick range of commits
git cherry-pick A^..B

# Abort cherry-pick
git cherry-pick --abort
```

### Reflog Recovery
```bash
# View reflog history
git reflog

# Recover lost commit
git checkout -b recovery <commit-hash>

# Reset to previous state
git reset --hard HEAD@{2}

# Find dropped stashes
git fsck --unreachable | grep commit
```

## Detailed Workflows

### 1. Interactive Rebase Workflow

**When to Use:**
- Clean up messy commit history before pushing
- Combine "WIP" or "fix typo" commits
- Reorder commits for logical progression
- Split large commits into focused changes

**Steps:**

```bash
# 1. Start interactive rebase for last N commits
git rebase -i HEAD~5

# Interactive editor opens with:
# pick abc1234 Add feature X
# pick def5678 Fix typo
# pick ghi9012 WIP commit
# pick jkl3456 Update documentation
# pick mno7890 Refactor feature X
```

**Rebase Commands:**
- `pick` (p): Keep commit as-is
- `reword` (r): Keep commit, edit message
- `edit` (e): Keep commit, stop to amend
- `squash` (s): Combine with previous commit, keep message
- `fixup` (f): Combine with previous commit, discard message
- `drop` (d): Remove commit entirely
- `exec` (x): Run shell command

**Example Cleanup:**
```bash
pick abc1234 Add feature X
fixup def5678 Fix typo
drop ghi9012 WIP commit
reword jkl3456 Update documentation
squash mno7890 Refactor feature X
```

**Result:**
- Three commits become two
- Typo fix merged into feature
- WIP commit removed
- Documentation message rewritten
- Refactor squashed with feature

**Safety Tips:**
- Never rebase commits already pushed to shared branches
- Create backup branch: `git branch backup-branch`
- Use `git reflog` if something goes wrong
- Force push carefully: `git push --force-with-lease`

### 2. Advanced Conflict Resolution

**Understanding Conflict Markers:**
```
<<<<<<< HEAD (Current Change)
Your code from current branch
=======
Their code from merging branch
>>>>>>> branch-name (Incoming Change)
```

**Resolution Strategies:**

**Strategy 1: Manual Resolution**
```bash
# 1. Identify conflicts
git status

# 2. Open file and manually resolve
# Edit between <<< === >>> markers

# 3. Stage resolved file
git add <file>

# 4. Continue merge/rebase
git merge --continue
# or
git rebase --continue
```

**Strategy 2: Accept One Side**
```bash
# Accept all changes from current branch
git checkout --ours <file>

# Accept all changes from merging branch
git checkout --theirs <file>

# Stage and continue
git add <file>
git merge --continue
```

**Strategy 3: Use Merge Tool**
```bash
# Configure merge tool (one-time setup)
git config --global merge.tool vimdiff
# or meld, kdiff3, p4merge

# Launch merge tool
git mergetool

# Review changes, save, and exit
# Tool creates .orig backup files
git clean -f *.orig  # Remove backups
```

**Complex Conflict Pattern:**
```bash
# For large conflicts, use 3-way diff
git diff --ours      # Your changes
git diff --theirs    # Their changes
git diff --base      # Common ancestor

# Cherry-pick specific hunks
git checkout --patch <branch> <file>
```

### 3. Git Bisect for Bug Hunting

**Scenario:** Bug exists in current version, find which commit introduced it.

**Manual Bisect:**
```bash
# 1. Start bisect
git bisect start

# 2. Mark current state as bad
git bisect bad

# 3. Mark known good commit (e.g., last release)
git bisect good v1.0.0

# Git checks out middle commit
# Bisecting: 50 revisions left to test

# 4. Test the code manually
# Run app, check if bug exists

# 5a. If bug exists:
git bisect bad

# 5b. If bug doesn't exist:
git bisect good

# Repeat until Git finds first bad commit
# Git will output:
# abc1234 is the first bad commit

# 6. End bisect
git bisect reset
```

**Automated Bisect:**
```bash
# Create test script (test.sh)
#!/bin/bash
# Exit 0 if good, 1 if bad

# Run your test
npm test
exit $?

# Make executable
chmod +x test.sh

# Run automated bisect
git bisect start
git bisect bad
git bisect good v1.0.0
git bisect run ./test.sh

# Git automatically finds bad commit
# Review result
git show <bad-commit>

# End bisect
git bisect reset
```

**Skip Commits:**
```bash
# Skip unbuildable commits
git bisect skip

# Skip range of commits
git bisect skip v1.0.0..v1.2.0
```

### 4. Branch Cleanup and Management

**Use Helper Script:**
```bash
# Run branch cleanup helper
bash scripts/git_helper.sh cleanup-branches
```

**Manual Cleanup:**
```bash
# List all branches
git branch -a

# List merged branches
git branch --merged main

# Delete merged local branches
git branch -d feature/completed

# Force delete unmerged branch
git branch -D feature/abandoned

# Delete remote branch
git push origin --delete feature/old

# Prune deleted remote branches
git fetch --prune

# Remove all merged branches (except main/develop)
git branch --merged | grep -v "\*\|main\|develop" | xargs -n 1 git branch -d
```

**Stale Branch Detection:**
```bash
# Show branches with last commit date
for branch in $(git branch -r | grep -v HEAD); do
    echo -e "$(git show --format="%ci %cr %an" $branch | head -n 1)\t$branch"
done | sort -r

# Archive old branches as tags
git tag archive/feature-x feature/feature-x
git branch -D feature/feature-x
git push origin --delete feature/feature-x
```

## Branch Strategy Workflows

### Git Flow

**Branch Types:**
- `main`: Production-ready code
- `develop`: Integration branch for features
- `feature/*`: New feature development
- `release/*`: Release preparation
- `hotfix/*`: Production bug fixes

**Feature Development:**
```bash
# Start feature from develop
git checkout develop
git pull origin develop
git checkout -b feature/user-auth

# Work on feature
git add .
git commit -m "Add authentication logic"

# Finish feature
git checkout develop
git merge --no-ff feature/user-auth
git push origin develop
git branch -d feature/user-auth
```

**Release Process:**
```bash
# Create release branch
git checkout develop
git checkout -b release/v1.2.0

# Prepare release (update versions, docs)
git commit -m "Prepare v1.2.0 release"

# Merge to main
git checkout main
git merge --no-ff release/v1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin main --tags

# Merge back to develop
git checkout develop
git merge --no-ff release/v1.2.0
git push origin develop

# Delete release branch
git branch -d release/v1.2.0
```

**Hotfix Process:**
```bash
# Create hotfix from main
git checkout main
git checkout -b hotfix/security-patch

# Fix issue
git commit -m "Fix security vulnerability"

# Merge to main
git checkout main
git merge --no-ff hotfix/security-patch
git tag -a v1.2.1 -m "Hotfix v1.2.1"
git push origin main --tags

# Merge to develop
git checkout develop
git merge --no-ff hotfix/security-patch
git push origin develop

# Delete hotfix branch
git branch -d hotfix/security-patch
```

### Trunk-Based Development

**Principles:**
- Single main branch (trunk)
- Short-lived feature branches (< 1 day)
- Frequent integration
- Feature flags for incomplete features

**Workflow:**
```bash
# Create short-lived feature branch
git checkout main
git pull origin main
git checkout -b feature/quick-fix

# Make changes and push quickly
git add .
git commit -m "Implement feature behind flag"
git push origin feature/quick-fix

# Create PR and merge same day
# After merge, delete branch
git checkout main
git pull origin main
git branch -d feature/quick-fix
```

## Recovery and History Rewriting

### Reflog Recovery

**Recover Deleted Branch:**
```bash
# View reflog
git reflog

# Find commit where branch was deleted
# Output shows: abc1234 HEAD@{5}: checkout: moving from feature-x to main

# Restore branch
git checkout -b feature-x abc1234
```

**Undo Bad Reset:**
```bash
# Accidentally ran: git reset --hard HEAD~5

# View reflog
git reflog

# Find state before reset
# Output: def5678 HEAD@{1}: reset: moving to HEAD~5

# Restore previous state
git reset --hard HEAD@{1}
```

**Recover Lost Commits:**
```bash
# Find dangling commits
git fsck --lost-found

# Or use reflog
git reflog show --all

# Cherry-pick recovered commit
git cherry-pick <lost-commit-hash>
```

### Safe History Rewriting

**Amend Last Commit:**
```bash
# Change last commit message
git commit --amend -m "New message"

# Add forgotten file to last commit
git add forgotten-file.txt
git commit --amend --no-edit

# Change author of last commit
git commit --amend --author="Name <email@example.com>"
```

**Filter Branch (Remove Sensitive Data):**
```bash
# Remove file from entire history
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD

# Better: Use BFG Repo-Cleaner (faster)
java -jar bfg.jar --delete-files passwords.txt
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

## Best Practices

### Before Rebase
- Create backup branch: `git branch backup`
- Ensure working directory is clean: `git status`
- Know your commit history: `git log --oneline`
- Never rebase public/shared branches

### During Conflicts
- Understand both sides of the conflict
- Test thoroughly after resolution
- Use meaningful merge commit messages
- Document complex resolutions in commit message

### Branch Management
- Delete merged branches promptly
- Use descriptive branch names: `feature/user-login`
- Keep branches focused and short-lived
- Sync with main branch regularly

### History Hygiene
- Squash "fixup" commits before pushing
- Write clear, descriptive commit messages
- Keep commits atomic (one logical change)
- Use conventional commit format when possible

## Troubleshooting

### Rebase Conflicts
```bash
# If conflicts are too complex
git rebase --abort

# Start over with different approach
git merge --no-ff <branch>
```

### Lost Commits
```bash
# Always check reflog first
git reflog

# Find and recover
git checkout -b recovery <commit-hash>
```

### Detached HEAD State
```bash
# Create branch from detached HEAD
git checkout -b new-branch-name

# Or go back to branch
git checkout main
```

### Failed Push After Rebase
```bash
# Only if you're certain and branch is not shared
git push --force-with-lease

# Safer: Create new branch
git checkout -b feature-v2
git push origin feature-v2
```

## Additional Resources

See detailed guides in `examples/`:
- `interactive_rebase.md`: Step-by-step rebase examples
- `conflict_resolution.md`: Common conflict patterns and solutions
- `branch_strategies.md`: Complete Git Flow and trunk-based workflows

Helper scripts in `scripts/`:
- `git_helper.sh`: Branch cleanup and conflict resolution utilities

## Quick Reference

### Must-Know Commands
```bash
# Interactive rebase
git rebase -i HEAD~N

# Abort operations
git rebase --abort
git merge --abort
git cherry-pick --abort

# View history
git log --oneline --graph --all
git reflog

# Conflict resolution
git checkout --ours <file>
git checkout --theirs <file>

# Recovery
git fsck --lost-found
git reflog

# Cleanup
git branch --merged | xargs git branch -d
git fetch --prune
git gc --aggressive
```

### Safety First
1. **Always backup before history rewrite**: `git branch backup`
2. **Never force push to shared branches**: Use `--force-with-lease`
3. **Test after conflicts**: Don't assume resolution is correct
4. **Document complex operations**: Leave comments in merge commits
5. **Use reflog**: It's your safety net for 30+ days

---

Master these advanced Git operations to maintain clean history, resolve conflicts efficiently, and manage complex development workflows with confidence.
