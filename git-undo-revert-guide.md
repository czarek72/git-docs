# Git Undo and Revert Guide: A Practical Tutorial

## Table of Contents
1. [Understanding Git's Three States](#understanding-gits-three-states)
2. [Undoing Changes in Working Directory](#undoing-changes-in-working-directory)
3. [Unstaging Changes](#unstaging-changes)
4. [Modifying Commits](#modifying-commits)
5. [Reverting Commits](#reverting-commits)
6. [Resetting to Previous States](#resetting-to-previous-states)
7. [Recovering Lost Work](#recovering-lost-work)
8. [Undoing Pushed Changes](#undoing-pushed-changes)
9. [Fixing Merge Mistakes](#fixing-merge-mistakes)
10. [Advanced Recovery Techniques](#advanced-recovery-techniques)
11. [Real-World Scenarios](#real-world-scenarios)
12. [Best Practices and Safety Guidelines](#best-practices-and-safety-guidelines)

---

## Understanding Git's Three States

Before learning how to undo changes, you must understand Git's three states:

```
Working Directory  →  Staging Area (Index)  →  Repository (.git)
   (modified)            (staged)                 (committed)
```

**Key Concepts:**

- **Working Directory**: Where you edit files
- **Staging Area**: Where you prepare changes for commit
- **Repository**: Where commits are permanently stored

**Commands affect different states:**
- `git checkout` - affects working directory
- `git reset` - affects staging area and/or commits
- `git revert` - creates new commits
- `git clean` - removes untracked files

---

## Undoing Changes in Working Directory

### Scenario 1: Discard Changes in a Single File

**Problem:** You modified a file but want to discard all changes and restore it to the last committed version.

```bash
# Check what changed
git diff src/main/java/Employee.java

# Discard changes in the file (restore from staging area/last commit)
git checkout -- src/main/java/Employee.java

# Or using the newer syntax (Git 2.23+)
git restore src/main/java/Employee.java

# Verify the file is restored
git status
```

**⚠️ WARNING:** This permanently deletes your changes. They cannot be recovered!

### Scenario 2: Discard All Changes in Working Directory

**Problem:** You made changes to multiple files and want to discard all of them.

```bash
# See what will be affected
git status

# Discard all tracked file changes
git checkout -- .

# Or using the newer syntax
git restore .

# For specific directory
git restore backend/src/
```

### Scenario 3: Discard Changes But Keep Untracked Files

**Problem:** You want to discard changes in tracked files but keep newly created files.

```bash
# Discard tracked file changes only
git checkout -- .

# Untracked files remain
git status  # Will still show untracked files
```

### Scenario 4: Remove Untracked Files

**Problem:** You created files that shouldn't exist (build artifacts, test files, etc.).

```bash
# Preview what will be deleted (DRY RUN)
git clean -n

# Delete untracked files
git clean -f

# Delete untracked files AND directories
git clean -fd

# Include ignored files too (use with caution!)
git clean -fdx

# Interactive mode (choose what to delete)
git clean -i
```

**Real-world example:**
```bash
# After accidentally running build without proper .gitignore
git status
# Shows: target/, dist/, node_modules/ as untracked

# Preview
git clean -dn
# Shows what will be removed

# Remove
git clean -fd
```

### Scenario 5: Discard Changes in Part of a File

**Problem:** You want to keep some changes but discard others in the same file.

```bash
# Interactive patch mode
git checkout -p src/main/java/Employee.java

# Git will show each change and ask:
# Stage this hunk [y,n,q,a,d,e,?]?
# y - discard this hunk
# n - keep this hunk
# e - manually edit the hunk
# q - quit
# ? - help
```

---

## Unstaging Changes

### Scenario 1: Unstage a Specific File

**Problem:** You staged a file but don't want to include it in the next commit.

```bash
# Stage a file
git add src/main/java/Employee.java
git status  # Shows file in "Changes to be committed"

# Unstage the file (changes remain in working directory)
git reset HEAD src/main/java/Employee.java

# Or using the newer syntax (Git 2.23+)
git restore --staged src/main/java/Employee.java

# Verify
git status  # File now in "Changes not staged for commit"
```

### Scenario 2: Unstage All Files

**Problem:** You ran `git add .` but want to unstage everything.

```bash
# Unstage all files
git reset HEAD

# Or using the newer syntax
git restore --staged .

# Files remain modified in working directory
```

### Scenario 3: Unstage and Discard Changes

**Problem:** You want to completely undo staging and modifications.

```bash
# Unstage and discard in one command
git reset --hard HEAD

# Or in two steps (safer)
git restore --staged .   # Unstage
git restore .            # Discard changes
```

---

## Modifying Commits

### Scenario 1: Fix the Last Commit Message

**Problem:** You just committed but the message has a typo or is unclear.

```bash
# Last commit
git commit -m "feat: Add employee validaton"  # Oops, typo!

# Fix the message
git commit --amend -m "feat: Add employee validation"

# Check the updated message
git log -1
```

**⚠️ WARNING:** Only do this if you haven't pushed yet!

### Scenario 2: Add Forgotten Files to Last Commit

**Problem:** You committed but forgot to include a file.

```bash
# Made a commit
git commit -m "feat: Add employee service"

# Realize you forgot the test file
git status  # Shows EmployeeServiceTest.java as modified

# Add the forgotten file
git add src/test/java/EmployeeServiceTest.java

# Amend the last commit (include the file)
git commit --amend --no-edit

# Check the commit
git show HEAD
```

### Scenario 3: Add Changes and Update Message

**Problem:** You want to add more changes AND update the commit message.

```bash
# Last commit
git commit -m "feat: Add employee service"

# Make additional changes
# ... edit files ...

# Stage new changes
git add .

# Amend with new message
git commit --amend -m "feat: Add employee service with validation"
```

### Scenario 4: Amend Without Changing Message

**Problem:** You want to add changes but keep the same message.

```bash
# Stage additional changes
git add src/main/java/Employee.java

# Amend without opening editor
git commit --amend --no-edit
```

**Real-world example:**
```bash
# Commit with incomplete code
git add Employee.java
git commit -m "feat: Add employee entity"

# Realize you forgot to add validation annotations
# Add @NotNull, @Email, etc.
git add Employee.java
git commit --amend --no-edit

# Same commit, updated content!
```

---

## Reverting Commits

### Scenario 1: Undo the Last Commit (Keep Changes)

**Problem:** You committed too early and want to continue working on the changes.

```bash
# Undo last commit, keep changes staged
git reset --soft HEAD~1

# Changes are now staged
git status

# Or keep changes unstaged
git reset HEAD~1

# Or completely discard the commit and changes
git reset --hard HEAD~1  # ⚠️ DANGEROUS!
```

**Real-world example:**
```bash
# Commit too early
git commit -m "feat: Add employee search"

# Realize more work needed
git reset --soft HEAD~1

# Continue working
# ... edit files ...
git add .
git commit -m "feat: Add employee search with filters"
```

### Scenario 2: Undo Multiple Commits (Keep Changes)

**Problem:** You made several commits but want to combine them into one.

```bash
# Undo last 3 commits, keep all changes staged
git reset --soft HEAD~3

# All changes from 3 commits are now staged
git status

# Create a single commit
git commit -m "feat: Complete employee module"
```

### Scenario 3: Revert a Specific Commit (Already Pushed)

**Problem:** You pushed a commit that introduced a bug. Others may have pulled it.

```bash
# Find the problematic commit
git log --oneline
# abc1234 feat: Add employee validation
# def5678 feat: Add employee search
# ghi9012 fix: Fix login bug

# Revert the specific commit (creates a NEW commit)
git revert abc1234

# Git opens an editor for the revert commit message
# Default: "Revert 'feat: Add employee validation'"

# Push the revert
git push origin develop
```

**This is the SAFE way to undo pushed commits!**

### Scenario 4: Revert a Range of Commits

**Problem:** Multiple recent commits need to be undone.

```bash
# Revert last 3 commits (creates 3 revert commits)
git revert HEAD~3..HEAD

# Or revert without creating individual commits
git revert -n HEAD~3..HEAD
git commit -m "Revert: Undo feature X (commits 1-3)"
```

### Scenario 5: Revert a Merge Commit

**Problem:** You merged a feature branch but it broke production.

```bash
# Find the merge commit
git log --oneline --graph
#   abc1234 (HEAD -> main) Merge branch 'feature/problem'
#   |\
#   | * def5678 feat: Problematic feature
#   | * ghi9012 feat: More problems
#   |/
#   * jkl3456 fix: Previous work

# Revert the merge (specify parent)
git revert -m 1 abc1234

# -m 1 means keep the main branch history
# -m 2 would keep the feature branch history

# Push
git push origin main
```

**Real-world example:**
```bash
# Emergency: merged feature broke production
git log --oneline
# abc1234 (HEAD -> main) Merge branch 'feature/new-dashboard'

# Revert the merge immediately
git revert -m 1 abc1234 --no-edit
git push origin main

# Production is now stable again
# Debug the feature branch separately
```

---

## Resetting to Previous States

### Understanding Reset Modes

```bash
git reset --soft  # Move HEAD, keep staging area and working directory
git reset --mixed # Move HEAD, reset staging area, keep working directory (DEFAULT)
git reset --hard  # Move HEAD, reset staging area and working directory (⚠️ DANGEROUS)
```

### Scenario 1: Undo Last Commit (Soft Reset)

**Problem:** You want to undo a commit but keep all changes staged.

```bash
# Before
git log --oneline
# abc1234 (HEAD -> feature) feat: New feature
# def5678 feat: Previous work

# Reset
git reset --soft HEAD~1

# After
git log --oneline
# def5678 (HEAD -> feature) feat: Previous work

git status
# Changes from abc1234 are staged
```

**Use case:** Re-write commit message or add more changes before committing again.

### Scenario 2: Undo Last Commit (Mixed Reset - Default)

**Problem:** You want to undo a commit and have changes unstaged for selective staging.

```bash
# Reset without mode (defaults to --mixed)
git reset HEAD~1

# Changes are in working directory but not staged
git status
# Shows files as "Changes not staged for commit"

# Selectively stage what you want
git add Employee.java
git commit -m "feat: Add employee entity"
```

### Scenario 3: Completely Remove Last Commit (Hard Reset)

**Problem:** You want to completely delete the last commit and all its changes.

```bash
# ⚠️ THIS DELETES CHANGES PERMANENTLY!
git reset --hard HEAD~1

# Verify
git log --oneline
git status  # Clean working directory
```

**⚠️ WARNING:** Only use on commits that haven't been pushed!

### Scenario 4: Reset to a Specific Commit

**Problem:** You want to go back to a specific point in history.

```bash
# View history
git log --oneline
# abc1234 (HEAD -> feature) Bad commit 3
# def5678 Bad commit 2
# ghi9012 Bad commit 1
# jkl3456 Good commit  <-- Want to go back here
# mno7890 Earlier work

# Reset to specific commit (soft - keep changes)
git reset --soft jkl3456

# Or reset to specific commit (hard - delete changes)
git reset --hard jkl3456

# Verify
git log --oneline
# jkl3456 (HEAD -> feature) Good commit
```

### Scenario 5: Reset After Accidentally Committing to Wrong Branch

**Problem:** You committed to `main` instead of a feature branch.

```bash
# You're on main with new commits
git log --oneline
# abc1234 (HEAD -> main) Oops, wrong branch
# def5678 Also wrong branch
# ghi9012 Correct commit

# Create a branch at current position
git branch feature/my-work

# Reset main to before your commits
git reset --hard ghi9012

# Switch to your feature branch
git checkout feature/my-work

# Your commits are now on the correct branch!
git log --oneline
# abc1234 (HEAD -> feature/my-work) Oops, wrong branch
# def5678 Also wrong branch
```

**Real-world example:**
```bash
# Morning: forgot to create feature branch
git status
# On branch develop
git add .
git commit -m "feat: Add new feature"  # Oops!

# Fix it
git branch feature/new-feature  # Create branch
git reset --hard HEAD~1         # Remove commit from develop
git checkout feature/new-feature # Switch to correct branch
# Continue working...
```

### Scenario 6: Reset to Remote State

**Problem:** Your local branch is messed up, and you want to match the remote exactly.

```bash
# Fetch latest from remote
git fetch origin

# Reset to remote branch (⚠️ DELETES LOCAL CHANGES)
git reset --hard origin/develop

# Now your local develop matches remote exactly
```

---

## Recovering Lost Work

### Scenario 1: Recover After Accidental Hard Reset

**Problem:** You did `git reset --hard` and lost commits.

```bash
# View reflog (history of HEAD movements)
git reflog

# Output:
# def5678 (HEAD -> feature) HEAD@{0}: reset: moving to HEAD~1
# abc1234 HEAD@{1}: commit: feat: Lost feature
# ghi9012 HEAD@{2}: commit: fix: Previous work

# Restore to before the reset
git reset --hard HEAD@{1}

# Or use the commit hash
git reset --hard abc1234

# Your "lost" commit is back!
```

### Scenario 2: Recover Deleted Branch

**Problem:** You deleted a branch before merging it.

```bash
# Delete branch accidentally
git branch -D feature/important-work
# warning: Deleting branch 'feature/important-work' that is not yet merged

# Find the last commit of the deleted branch
git reflog | grep important-work
# abc1234 HEAD@{5}: checkout: moving from feature/important-work to main

# Recreate the branch
git checkout -b feature/important-work abc1234

# Branch recovered!
```

### Scenario 3: Recover Specific Lost Commits

**Problem:** You lost specific commits but remember what they were about.

```bash
# Search reflog for keywords
git reflog | grep "employee"
# abc1234 HEAD@{10}: commit: feat: Add employee validation

# Cherry-pick the lost commit
git cherry-pick abc1234

# Or view the commit first
git show abc1234
```

### Scenario 4: Recover Lost Stash

**Problem:** You stashed changes and then lost the stash.

```bash
# Stash appears empty
git stash list
# (empty)

# Stashes are in reflog!
git fsck --no-reflog | grep commit | cut -d' ' -f3 | xargs git log --merges --no-walk --grep=WIP

# Or search reflog directly
git log --graph --oneline --decorate $(git fsck --no-reflog | awk '/dangling commit/ {print $3}')

# Once found, apply it
git stash apply <stash-hash>
```

### Scenario 5: Recover After Git Clean

**Problem:** You ran `git clean -fd` and deleted important untracked files.

```bash
# Unfortunately, git clean permanently deletes untracked files
# They are NOT in Git's history

# Prevention: ALWAYS use dry run first
git clean -n  # Preview

# Recovery options:
# 1. Check IDE/editor auto-save or local history (VS Code, IntelliJ)
# 2. Check system trash/recycle bin
# 3. Use file recovery tools (TestDisk, PhotoRec)
```

**Lesson:** Never run `git clean -fd` without `git clean -n` first!

---

## Undoing Pushed Changes

### Scenario 1: Fix Pushed Commit on Feature Branch

**Problem:** You pushed a commit to your feature branch and want to change it.

```bash
# Only do this if you're the only one working on this branch!

# Option 1: Amend and force push
git commit --amend -m "feat: Corrected feature"
git push --force-with-lease origin feature/my-branch

# Option 2: Reset and force push
git reset --hard HEAD~1
# ... make corrections ...
git add .
git commit -m "feat: Corrected feature"
git push --force-with-lease origin feature/my-branch
```

**⚠️ Use `--force-with-lease` instead of `--force`** - it's safer!

### Scenario 2: Revert Pushed Commit on Shared Branch

**Problem:** You pushed a bad commit to `develop` or `main`, and others may have pulled it.

```bash
# NEVER use reset/rebase on shared branches!
# Use revert instead

# Find the bad commit
git log --oneline
# abc1234 (HEAD -> develop) Bad commit
# def5678 Good commit

# Revert it (creates new commit)
git revert abc1234 --no-edit
git push origin develop

# The bad commit stays in history, but its changes are undone
```

**This is the safe way for shared branches!**

### Scenario 3: Undo Multiple Pushed Commits

**Problem:** Several recent pushed commits need to be undone.

```bash
# Revert multiple commits
git revert --no-commit HEAD~3..HEAD
git commit -m "Revert: Undo problematic feature (commits 1-3)"
git push origin develop
```

### Scenario 4: Fix Pushed Commit with Fixup

**Problem:** You want to cleanly fix a pushed commit.

```bash
# Original commit
git log --oneline
# abc1234 (HEAD -> feature) feat: Add employee validation

# Make corrections
# ... edit files ...
git add .
git commit --fixup abc1234

# Interactive rebase to squash fixup
git rebase -i --autosquash HEAD~2

# Push
git push --force-with-lease origin feature/my-branch
```

---

## Fixing Merge Mistakes

### Scenario 1: Abort Ongoing Merge

**Problem:** You started a merge and encountered conflicts you're not ready to resolve.

```bash
# During merge conflict
git status
# Unmerged paths:
#   both modified:   Employee.java

# Abort the merge (restore to before merge)
git merge --abort

# Everything back to normal
git status
# On branch feature/my-work
# nothing to commit, working tree clean
```

### Scenario 2: Undo Completed Merge (Not Pushed)

**Problem:** You merged but immediately regret it.

```bash
# Just completed merge
git log --oneline --graph
# * abc1234 (HEAD -> develop) Merge branch 'feature/bad-feature'

# Undo the merge (keep working directory clean)
git reset --hard HEAD~1

# Or keep changes from the merge
git reset --soft HEAD~1
```

### Scenario 3: Undo Pushed Merge

**Problem:** You merged and pushed, but the merge broke things.

```bash
# Find the merge commit
git log --oneline --graph
# * abc1234 (HEAD -> develop) Merge branch 'feature/problem'

# Revert the merge
git revert -m 1 abc1234
git push origin develop

# Production is safe again
```

### Scenario 4: Redo Merge with Different Strategy

**Problem:** Merge created a mess; you want to try a different approach.

```bash
# Abort current merge
git merge --abort

# Try merge with different strategy
git merge --strategy=recursive --strategy-option=theirs feature/my-branch

# Or keep our changes in conflicts
git merge --strategy=recursive --strategy-option=ours feature/my-branch
```

### Scenario 5: Fix Merge Conflicts After the Fact

**Problem:** You resolved conflicts incorrectly and need to redo them.

```bash
# If not yet pushed
git reset --hard HEAD~1
git merge feature/my-branch
# Resolve conflicts correctly this time

# If already pushed
git revert -m 1 HEAD
git push
git merge feature/my-branch
# Resolve conflicts correctly
git push
```

---

## Advanced Recovery Techniques

### Scenario 1: Interactive Rebase to Rewrite History

**Problem:** You have multiple messy commits that need to be cleaned up.

```bash
# View last 5 commits
git log --oneline
# abc1234 typo fix
# def5678 wip
# ghi9012 feat: Add feature
# jkl3456 wip more work
# mno7890 feat: Start feature

# Interactive rebase
git rebase -i HEAD~5

# Editor opens:
# pick mno7890 feat: Start feature
# pick jkl3456 wip more work
# pick ghi9012 feat: Add feature
# pick def5678 wip
# pick abc1234 typo fix

# Reorganize:
# pick mno7890 feat: Start feature
# squash jkl3456 wip more work
# squash ghi9012 feat: Add feature
# squash def5678 wip
# squash abc1234 typo fix

# Save and close - all commits squashed into one!

# Force push to your branch
git push --force-with-lease origin feature/my-branch
```

### Scenario 2: Extract Commits to New Branch

**Problem:** You made commits on `develop` that should be on a feature branch.

```bash
# On develop with extra commits
git log --oneline
# abc1234 (HEAD -> develop) Feature commit 2
# def5678 Feature commit 1
# ghi9012 Normal develop work

# Create new branch at current HEAD
git branch feature/extracted-work

# Reset develop to before feature commits
git reset --hard ghi9012

# Switch to feature branch
git checkout feature/extracted-work

# All feature commits are now isolated!
```

### Scenario 3: Reorder Commits

**Problem:** Commits are in the wrong order.

```bash
# Interactive rebase
git rebase -i HEAD~4

# Editor shows:
# pick abc1234 Commit A
# pick def5678 Commit B
# pick ghi9012 Commit C
# pick jkl3456 Commit D

# Reorder by moving lines:
# pick def5678 Commit B
# pick ghi9012 Commit C
# pick abc1234 Commit A
# pick jkl3456 Commit D

# Save and close
```

### Scenario 4: Remove Sensitive Data from History

**Problem:** You committed a password or API key.

```bash
# Remove file from all history
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch config/secrets.yml' \
  --prune-empty --tag-name-filter cat -- --all

# Or use BFG Repo Cleaner (faster, recommended)
# Download BFG from https://rtyley.github.io/bfg-repo-cleaner/
java -jar bfg.jar --delete-files secrets.yml

# Force push all branches
git push --force --all origin

# Force push tags
git push --force --tags origin
```

**⚠️ WARNING:** This rewrites history. Coordinate with team!

### Scenario 5: Split a Commit into Multiple Commits

**Problem:** One commit contains multiple unrelated changes.

```bash
# Interactive rebase
git rebase -i HEAD~1

# Mark commit as 'edit'
# edit abc1234 Big commit with multiple changes

# Reset the commit but keep changes
git reset HEAD~1

# Selectively stage and commit
git add Employee.java
git commit -m "feat: Add employee entity"

git add EmployeeService.java
git commit -m "feat: Add employee service"

git add EmployeeController.java
git commit -m "feat: Add employee controller"

# Continue rebase
git rebase --continue
```

---

## Real-World Scenarios

### Scenario 1: "Oh no, I committed to main!"

```bash
# You're on main (should be on feature branch)
git log --oneline
# abc1234 (HEAD -> main) My feature work
# def5678 (origin/main) Last good commit

# Fix:
git branch feature/my-work  # Save work
git reset --hard origin/main  # Reset main
git checkout feature/my-work  # Continue on feature branch
git push -u origin feature/my-work  # Push to remote
```

### Scenario 2: "I need to test without my changes"

```bash
# Stash changes
git stash save "Testing without my changes"

# Test
git status  # Clean working directory
# ... run tests, reproduce bug, etc. ...

# Restore changes
git stash pop
```

### Scenario 3: "I merged the wrong branch"

```bash
# Just merged wrong branch
git log --oneline
# abc1234 (HEAD -> develop) Merge branch 'feature/wrong'

# Undo merge
git reset --hard HEAD~1

# Merge correct branch
git merge feature/correct
```

### Scenario 4: "My branch is completely broken"

```bash
# Nuclear option: restart from remote
git fetch origin

# Save any important local changes
git stash save "Before resetting to remote"

# Reset to remote
git reset --hard origin/feature/my-branch

# If you want your local changes back
git stash pop
```

### Scenario 5: "I need to undo work from 3 days ago"

```bash
# Find the commit from 3 days ago
git log --since="3 days ago" --oneline
# abc1234 The commit I want to undo

# Revert it
git revert abc1234
git push origin develop
```

### Scenario 6: "I force pushed and broke everything!"

```bash
# If you force pushed to a shared branch (bad!)
# And others notify you their work is gone

# Find the previous state
git reflog origin/develop
# abc1234 origin/develop@{0}: forced push
# def5678 origin/develop@{1}: previous state

# Reset to previous state
git reset --hard def5678

# Force push back
git push --force origin develop

# Apologize to team and communicate what happened!
```

### Scenario 7: "I need to undo a commit in the middle of history"

```bash
# View history
git log --oneline
# abc1234 (HEAD -> develop) Latest work
# def5678 More work
# ghi9012 BAD COMMIT - need to remove
# jkl3456 Good work
# mno7890 Earlier work

# Option 1: Revert the specific commit
git revert ghi9012
git push origin develop

# Option 2: Interactive rebase (if not pushed)
git rebase -i jkl3456
# Delete the line with ghi9012
# Save and close

# Option 3: Revert and squash
git revert ghi9012
git rebase -i HEAD~2
# Squash revert with latest work if needed
```

### Scenario 8: "I committed a huge file by mistake"

```bash
# File not pushed yet
git reset HEAD~1
git rm --cached large-file.zip
echo "large-file.zip" >> .gitignore
git add .
git commit -m "Add feature (without large file)"

# File already pushed
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch large-file.zip' HEAD
git push --force-with-lease origin feature/my-branch
```

### Scenario 9: "I want to test a hotfix before committing"

```bash
# Make changes for hotfix
# ... edit files ...

# Don't commit yet - test first
# Run application and verify fix

# If fix works
git add .
git commit -m "hotfix: Critical bug fix"

# If fix doesn't work
git restore .  # Discard changes
# Try different approach
```

### Scenario 10: "I need to synchronize with a coworker's work"

```bash
# Coworker pushed to feature branch
git checkout feature/shared-work
git pull origin feature/shared-work

# Conflicts! You both changed the same file
# Resolve conflicts
# ... edit files ...

git add .
git commit -m "Merge coworker's changes"
git push origin feature/shared-work
```

---

## Best Practices and Safety Guidelines

### Golden Rules

1. **Never rewrite history on shared branches**
   - Don't use `reset`, `rebase`, or `amend` on main/develop
   - Use `revert` instead

2. **Always use `--force-with-lease` instead of `--force`**
   ```bash
   # Bad
   git push --force

   # Good
   git push --force-with-lease
   ```

3. **Preview before destroying**
   ```bash
   # Before: git clean -fd
   # Do: git clean -n  (dry run)

   # Before: git reset --hard
   # Do: git log  (check what you'll lose)
   ```

4. **Commit often, push frequently**
   - Can't lose committed work (it's in reflog)
   - Pushed work is backed up remotely

### Safe Undo Checklist

Before using destructive commands, ask:

- [ ] Have I committed my work?
- [ ] Have I pushed to remote?
- [ ] Are others working on this branch?
- [ ] Do I have a backup?
- [ ] Did I preview the operation (dry run)?

### Command Safety Levels

**🟢 SAFE** (can always undo):
- `git revert` - creates new commits
- `git stash` - saves work temporarily
- `git commit --amend` (unpushed commits)
- `git reset --soft` - keeps all changes

**🟡 CAUTION** (can undo with reflog):
- `git reset --mixed` - unstages changes
- `git reset --hard` (committed work)
- `git rebase` (local branches)
- `git commit --amend` (pushed commits)

**🔴 DANGEROUS** (may be permanent):
- `git reset --hard` (uncommitted changes)
- `git clean -fd` (untracked files)
- `git push --force` (shared branches)
- `git filter-branch` (history rewrite)

### Emergency Recovery Commands

Keep these handy:

```bash
# View reflog (history of HEAD movements)
git reflog

# Find dangling commits
git fsck --lost-found

# Restore to previous state
git reset --hard HEAD@{n}

# Recover deleted branch
git checkout -b recovered <commit-hash>

# Abort operation
git merge --abort
git rebase --abort
git cherry-pick --abort
```

### Backup Strategies

```bash
# 1. Create backup branch before dangerous operations
git branch backup-$(date +%Y%m%d-%H%M%S)

# 2. Tag important states
git tag -a backup-before-rebase -m "Backup before rebase"

# 3. Use worktrees for experiments
git worktree add ../experiment feature/risky-change

# 4. Regular backups of important work
git push origin HEAD:backup/my-work-$(date +%Y%m%d)
```

### Communication Protocol

When undoing pushed changes:

1. **Notify team immediately**
   ```
   "⚠️ Reverting commit abc1234 from develop due to [reason]"
   ```

2. **Explain what happened**
   ```
   "This will undo changes to Employee.java from 2 hours ago"
   ```

3. **Provide recovery instructions**
   ```
   "If you pulled the bad commit, run: git pull origin develop"
   ```

4. **Document in commit message**
   ```bash
   git revert abc1234 -m "Revert due to breaking production deployment. See incident #123"
   ```

### Prevention Tips

```bash
# 1. Set up commit template
git config --global commit.template ~/.gitmessage

# 2. Enable commit message editor
git config --global core.editor "code --wait"

# 3. Set up pre-commit hooks
# Create .git/hooks/pre-commit
#!/bin/bash
# Check for debugging statements
if git grep -E '(console\.log|debugger|TODO: FIX)'; then
  echo "⚠️  Found debugging statements. Remove before committing."
  exit 1
fi

# 4. Use aliases for safer commands
git config --global alias.undo-commit 'reset --soft HEAD~1'
git config --global alias.discard 'checkout --'
git config --global alias.unstage 'reset HEAD --'
git config --global alias.amend 'commit --amend --no-edit'

# 5. Require explicit confirmation for dangerous commands
git config --global alias.clean-all '!f() { echo "This will delete untracked files. Continue? (y/n)"; read answer; if [ "$answer" = "y" ]; then git clean -fd; fi; }; f'
```

---

## Quick Reference Guide

### Decision Tree: "I Need to Undo..."

```
Changes in working directory?
├─ Single file → git restore <file>
├─ All files → git restore .
└─ Untracked files → git clean -fd

Changes staged?
├─ Single file → git restore --staged <file>
└─ All files → git restore --staged .

Last commit?
├─ Keep changes staged → git reset --soft HEAD~1
├─ Keep changes unstaged → git reset HEAD~1
├─ Delete changes → git reset --hard HEAD~1
└─ Commit already pushed → git revert HEAD

Multiple commits?
├─ Not pushed → git reset --soft HEAD~n
├─ Already pushed → git revert HEAD~n..HEAD
└─ Specific commit → git revert <commit-hash>

Merged by mistake?
├─ Not pushed → git reset --hard HEAD~1
└─ Already pushed → git revert -m 1 <merge-commit>

Lost commits?
├─ Check reflog → git reflog
└─ Restore → git reset --hard HEAD@{n}
```

### Essential Commands Summary

```bash
# Undo working directory changes
git restore <file>              # Discard changes in file
git restore .                   # Discard all changes
git clean -fd                   # Remove untracked files

# Undo staging
git restore --staged <file>     # Unstage file
git restore --staged .          # Unstage all

# Undo commits (local)
git reset --soft HEAD~1         # Undo commit, keep changes staged
git reset HEAD~1                # Undo commit, keep changes unstaged
git reset --hard HEAD~1         # Undo commit, delete changes

# Undo commits (pushed)
git revert <commit>             # Create new commit that undoes
git revert HEAD                 # Undo last commit
git revert -m 1 <merge>         # Undo merge commit

# Modify last commit
git commit --amend              # Add to last commit
git commit --amend -m "msg"     # Change last commit message

# Recovery
git reflog                      # View history of HEAD
git reset --hard HEAD@{n}       # Restore to previous state
git checkout -b name <hash>     # Recover deleted branch

# Abort operations
git merge --abort               # Abort merge
git rebase --abort              # Abort rebase
git cherry-pick --abort         # Abort cherry-pick
```

---

## Summary

**Key Takeaways:**

1. **Understand the three states** - Working directory, staging area, repository
2. **Committed work is recoverable** - Use reflog to find "lost" commits
3. **Uncommitted changes can be lost** - Be careful with `--hard` and `clean`
4. **Never rewrite shared history** - Use `revert` on main/develop
5. **Preview destructive operations** - Use `-n` flags and `--dry-run`
6. **Communicate when undoing pushed work** - Team coordination is critical
7. **Keep backups before risky operations** - Create backup branches/tags
8. **Practice in a test repository** - Get comfortable with undo commands

**Remember:**
- Git is forgiving if you've committed your work
- Reflog is your safety net for committed changes
- Revert is safer than reset for shared branches
- When in doubt, create a backup branch first
- Always prefer `--force-with-lease` over `--force`

**Most Common Mistakes:**
1. Running `git reset --hard` on uncommitted work
2. Force pushing to shared branches without communication
3. Forgetting to check reflog when work seems lost
4. Using `git clean -fd` without `git clean -n` first
5. Amending pushed commits on shared branches

**Final Advice:**
> "The best way to undo a Git mistake is to not make it in the first place. But when you do make a mistake, Git has your back - you just need to know where to look (reflog) and what tools to use (reset, revert, restore)."

Happy Git-ing! 🎉
