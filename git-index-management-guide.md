# Git Index Management Guide: Mastering Staging Area Commands

## Table of Contents
1. [Understanding the Git Index (Staging Area)](#understanding-the-git-index-staging-area)
2. [Git Restore Command](#git-restore-command)
3. [Adding Files to the Index](#adding-files-to-the-index)
4. [Removing Files from the Index](#removing-files-from-the-index)
5. [Moving and Renaming Files](#moving-and-renaming-files)
6. [Viewing Index Status](#viewing-index-status)
7. [Advanced Index Operations](#advanced-index-operations)
8. [Practical Workflows](#practical-workflows)
9. [Best Practices](#best-practices)
10. [Quick Reference](#quick-reference)

---

## Understanding the Git Index (Staging Area)

The **index** (also called the **staging area**) is Git's intermediate storage between your working directory and the repository. It's one of Git's "three trees":

```
Working Directory  →  Index/Staging Area  →  Repository
   (modified)           (staged/added)         (committed)
```

**Key Concepts:**

- **Working Directory**: Where you edit files
- **Index/Staging Area**: Where you prepare changes for commit
- **Repository**: Where commits are permanently stored

**Why the Index Matters:**

The index allows you to:
- Stage only specific changes for commit (selective commits)
- Review changes before committing
- Build commits incrementally
- Separate "work in progress" from "ready to commit"

**File States:**

Files can be in these states:
1. **Untracked**: New files not yet added to Git
2. **Tracked & Unmodified**: Files in repository, no changes
3. **Tracked & Modified**: Files changed but not staged
4. **Tracked & Staged**: Files added to index, ready to commit

---

## Git Restore Command

`git restore` is the modern, intuitive command (introduced in Git 2.23) for managing changes. It replaces the confusing overloaded uses of `git checkout`.

### Basic Syntax

```bash
git restore [options] <file>...
```

### Restoring Working Directory Files

#### Scenario 1: Discard Changes in a File

**Problem:** You modified a file but want to discard all changes.

```bash
# View changes first
git diff src/Employee.java

# Discard changes (restore from index or HEAD)
git restore src/Employee.java

# Verify
git status
```

**What happens:**
- If file is staged: restores from index
- If file is not staged: restores from HEAD
- Changes are **permanently lost** ⚠️

#### Scenario 2: Discard All Changes

**Problem:** You want to discard all modifications in tracked files.

```bash
# Discard all changes in current directory
git restore .

# Discard all changes in specific directory
git restore backend/src/

# Discard changes in all tracked files (from repo root)
git restore :/
```

#### Scenario 3: Restore Specific Lines (Interactive)

**Problem:** You want to keep some changes but discard others in the same file.

```bash
# Interactive patch mode
git restore --patch src/Employee.java
# or short form
git restore -p src/Employee.java

# Git shows each change (hunk) and prompts:
# Discard this hunk from worktree [y,n,q,a,d,e,?]?
# y - yes, discard this hunk
# n - no, keep this hunk
# q - quit
# a - discard this and all remaining hunks
# d - do not discard this or any remaining hunks
# e - manually edit the hunk
# ? - print help
```

**Example workflow:**
```bash
# Made multiple changes to Employee.java
git restore -p src/Employee.java
# Keep validation logic changes (answer 'n')
# Discard debug logging (answer 'y')
# Keep bug fix (answer 'n')
```

### Restoring from the Index (Unstaging)

#### Scenario 1: Unstage a Specific File

**Problem:** You staged a file but don't want to commit it yet.

```bash
# Stage file
git add src/Employee.java
git status  # Shows "Changes to be committed"

# Unstage the file (remove from index)
git restore --staged src/Employee.java

# Verify
git status  # Now shows "Changes not staged for commit"
```

**Note:** The file remains modified in your working directory.

#### Scenario 2: Unstage All Files

**Problem:** You ran `git add .` but want to unstage everything.

```bash
# Unstage all files
git restore --staged .

# Files remain modified in working directory
git status
```

#### Scenario 3: Unstage Specific Lines

**Problem:** You staged too much and want to unstage specific changes.

```bash
# Interactive unstaging
git restore --staged --patch Employee.java

# Git shows each staged change and asks:
# Unstage this hunk [y,n,q,a,d,e,?]?
```

### Restoring from Specific Commits

#### Scenario 1: Restore File from HEAD

**Problem:** You want to restore a file to the last committed state.

```bash
# Explicitly restore from HEAD
git restore --source=HEAD src/Employee.java

# This is the default behavior of git restore
```

#### Scenario 2: Restore File from Specific Commit

**Problem:** You want to restore a file to its state in a previous commit.

```bash
# Restore from specific commit
git restore --source=abc1234 src/Employee.java

# Restore from commit 3 commits ago
git restore --source=HEAD~3 src/Employee.java

# Restore from specific branch
git restore --source=develop src/Employee.java

# Restore from specific tag
git restore --source=v1.0.0 src/config.yml
```

**What happens:**
- File in working directory is replaced
- Change is **not staged** by default
- To stage the change, add `--staged` or use `--worktree --staged`

#### Scenario 3: Restore and Stage

**Problem:** You want to restore a file from a commit and immediately stage it.

```bash
# Restore and stage in one command
git restore --source=HEAD~5 --staged --worktree src/Employee.java

# Now the file is both:
# 1. Updated in working directory
# 2. Staged for commit
```

### Advanced Restore Options

#### Restore to Worktree Only

```bash
# Default behavior (updates working directory)
git restore src/Employee.java
git restore --worktree src/Employee.java  # Explicit
```

#### Restore to Staging Area Only

```bash
# Update only index, not working directory
git restore --staged src/Employee.java
```

#### Restore Both Worktree and Staging Area

```bash
# Update both working directory and index
git restore --staged --worktree src/Employee.java

# Effectively resets file to HEAD state completely
```

#### Ignore Unmerged Files

```bash
# During merge conflicts, ignore unmerged entries
git restore --ours src/Employee.java     # Keep "our" version
git restore --theirs src/Employee.java   # Keep "their" version
```

### Real-World Examples

#### Example 1: Undo Experimental Changes

```bash
# Experimenting with new approach
# ... edit Employee.java ...
# Doesn't work, want to go back

# Discard changes
git restore src/Employee.java
```

#### Example 2: Stage Wrong File

```bash
# Accidentally stage configuration file
git add config/database.yml

# Unstage it
git restore --staged config/database.yml

# File still modified, just not staged
```

#### Example 3: Restore from Old Version

```bash
# Bug introduced recently, need old version of file
git log --oneline src/Employee.java
# abc1234 Recent changes
# def5678 Bug introduced here
# ghi9012 Last known good version

# Restore from last known good version
git restore --source=ghi9012 src/Employee.java

# Test the restored version
# If it works, stage and commit
git add src/Employee.java
git commit -m "fix: Restore working version of Employee.java"
```

#### Example 4: Selective Staging

```bash
# Made multiple changes to a file
# Want to commit only bug fixes, not new features

# Stage file
git add src/Employee.java

# Unstage feature code interactively
git restore --staged --patch src/Employee.java
# Answer 'y' to unstage feature code
# Answer 'n' to keep bug fixes staged

# Commit only bug fixes
git commit -m "fix: Correct validation logic"

# Later commit features
git add src/Employee.java
git commit -m "feat: Add new validation rules"
```

### Git Restore vs. Git Checkout

`git restore` was introduced to clarify operations previously done with `git checkout`.

**Old way (git checkout):**
```bash
git checkout -- file.txt          # Discard changes
git checkout HEAD file.txt        # Restore from HEAD
git checkout abc1234 file.txt     # Restore from commit
```

**New way (git restore):**
```bash
git restore file.txt              # Discard changes
git restore --source=HEAD file.txt        # Explicit
git restore --source=abc1234 file.txt     # Restore from commit
```

**Why git restore is better:**
- More explicit and clear purpose
- Separate from branch switching (`git switch`)
- Better option naming (`--staged`, `--worktree`, `--source`)
- Less confusion for beginners

---

## Adding Files to the Index

### Basic git add

#### Add Specific Files

```bash
# Add single file
git add src/Employee.java

# Add multiple files
git add src/Employee.java src/Department.java

# Add all files in directory
git add src/

# Add files matching pattern
git add *.java
git add src/**/*.java
```

#### Add All Changes

```bash
# Add all changes (new, modified, deleted) in current directory
git add .

# Add all changes in entire repository
git add --all
git add -A

# Add only modified and deleted files (not new files)
git add -u
git add --update
```

### Interactive Adding

#### Scenario 1: Choose Files Interactively

**Problem:** You have many changed files and want to select which to stage.

```bash
# Start interactive mode
git add -i

# Menu appears:
#            staged     unstaged path
#   1:    unchanged        +4/-0 src/Employee.java
#   2:    unchanged        +2/-1 src/Department.java
# 
# *** Commands ***
#   1: status       2: update       3: revert       4: add untracked
#   5: patch        6: diff         7: quit         8: help
# What now>

# Type '2' (update) to stage files
# Type '1,2' to stage files 1 and 2
# Type '5' (patch) for hunk-by-hunk staging
```

#### Scenario 2: Stage Hunks (Patch Mode)

**Problem:** You want to stage only specific changes within a file.

```bash
# Patch mode (most useful)
git add -p
git add --patch

# Git shows each change and asks:
# Stage this hunk [y,n,q,a,d,s,e,?]?
# y - yes, stage this hunk
# n - no, don't stage this hunk
# q - quit; do not stage this hunk or any remaining
# a - stage this hunk and all later hunks in the file
# d - do not stage this hunk or any later hunks in the file
# s - split the current hunk into smaller hunks
# e - manually edit the current hunk
# ? - print help
```

**Practical example:**
```bash
# Made bug fix and added new feature in same file
git add -p src/Employee.java

# Hunk 1: Bug fix
# Stage this hunk [y,n,q,a,d,s,e,?]? y

# Hunk 2: New feature
# Stage this hunk [y,n,q,a,d,s,e,?]? n

# Now commit just the bug fix
git commit -m "fix: Correct validation in Employee"

# Later stage and commit the feature
git add src/Employee.java
git commit -m "feat: Add new validation rule"
```

### Advanced git add Options

#### Intent to Add

**Problem:** You want Git to track a new file but not stage its content yet.

```bash
# Add new file to index (shows in git diff, but content not staged)
git add -N newfile.java
git add --intent-to-add newfile.java

# Now you can see it in git diff
git diff  # Shows newfile.java

# Use patch mode on the new file
git add -p newfile.java
```

#### Force Add Ignored Files

**Problem:** A file is in `.gitignore` but you need to add it.

```bash
# Override .gitignore
git add -f config/local-settings.yml
git add --force config/local-settings.yml
```

#### Refresh Index

**Problem:** Index is out of sync with working directory.

```bash
# Refresh index (doesn't add new content)
git add --refresh

# Useful after external tools modify files
```

#### Dry Run

**Problem:** You want to see what would be added.

```bash
# Preview what would be added
git add --dry-run .
git add -n .

# Shows files that would be added without actually adding them
```

### Real-World Workflows

#### Workflow 1: Incremental Commits

```bash
# Working on multiple features
# ... edit Employee.java, Department.java, Company.java ...

# Commit Feature 1
git add src/Employee.java
git commit -m "feat: Add employee validation"

# Commit Feature 2
git add src/Department.java
git commit -m "feat: Add department hierarchy"

# Commit Feature 3
git add src/Company.java
git commit -m "feat: Add company settings"
```

#### Workflow 2: Selective Line Staging

```bash
# Made changes to fix bug and refactor
# File has both bug fix and code cleanup

# Stage only bug fix
git add -p src/Employee.java
# Answer 'y' for bug fix hunks
# Answer 'n' for refactoring hunks

# Commit bug fix
git commit -m "fix: Correct null pointer in validation"

# Stage refactoring
git add src/Employee.java
git commit -m "refactor: Clean up validation method"
```

#### Workflow 3: Review Before Staging

```bash
# Made many changes
git status  # See what changed

# Review each file
git diff src/Employee.java
git diff src/Department.java

# Stage individually after review
git add src/Employee.java  # Looks good
git diff src/Department.java  # Found issue, fix it first

# Fix and then stage
git add src/Department.java
git commit -m "feat: Add new entities"
```

---

## Removing Files from the Index

### Git Reset (Unstaging)

`git reset` is used to unstage files (move them from index back to working directory).

#### Basic Unstaging

```bash
# Unstage specific file
git reset HEAD Employee.java
git reset Employee.java  # HEAD is default

# Unstage all files
git reset
git reset HEAD

# Using git restore (newer, recommended)
git restore --staged Employee.java
git restore --staged .
```

#### Reset Modes

```bash
# --mixed (default): Unstage, keep changes in working directory
git reset HEAD~1           # Undo last commit, keep changes unstaged
git reset --mixed HEAD~1   # Explicit

# --soft: Unstage, keep changes staged
git reset --soft HEAD~1    # Undo last commit, keep changes staged

# --hard: Unstage and discard changes ⚠️
git reset --hard HEAD~1    # Undo last commit, discard changes
```

### Git rm (Remove from Tracking)

#### Remove File from Git and Filesystem

```bash
# Remove file (delete from working directory and stage deletion)
git rm file.txt

# Verify
git status  # Shows deletion staged
ls         # File is gone

# Commit the removal
git commit -m "Remove obsolete file"
```

#### Remove File from Git Only (Keep Locally)

**Problem:** You accidentally committed a file that should be ignored.

```bash
# Remove from Git but keep in filesystem
git rm --cached file.txt

# File remains on disk, but Git stops tracking it
ls  # File exists
git status  # Shows deletion staged

# Add to .gitignore
echo "file.txt" >> .gitignore
git add .gitignore

# Commit
git commit -m "Stop tracking file.txt"
```

**Common use case: Accidentally committed node_modules/**
```bash
# Remove from tracking
git rm -r --cached node_modules/

# Add to .gitignore
echo "node_modules/" >> .gitignore

# Commit
git add .gitignore
git commit -m "Remove node_modules from tracking"
```

#### Remove Multiple Files

```bash
# Remove all .log files
git rm *.log

# Remove all files in directory
git rm -r logs/

# Remove files matching pattern
git rm src/temp*.java
```

#### Force Remove Modified Files

**Problem:** You want to remove a file that has unstaged changes.

```bash
# Git prevents removing modified files
git rm modified-file.txt
# error: the following file has local modifications

# Force removal (changes are lost!)
git rm -f modified-file.txt
git rm --force modified-file.txt
```

#### Remove from Index but Keep Staged Changes

**Problem:** You want to stop tracking a file without losing changes.

```bash
# Remove from index, keep modifications
git rm --cached -r config/

# Changes stay in working directory
# Git stops tracking these files
```

### Real-World Scenarios

#### Scenario 1: Remove Sensitive Data

```bash
# Accidentally committed password file
git rm --cached config/secrets.yml

# Add to .gitignore
echo "config/secrets.yml" >> .gitignore

# Commit
git add .gitignore
git commit -m "Remove secrets from tracking"

# File still exists locally with your passwords
```

#### Scenario 2: Clean Up Old Files

```bash
# Remove all temporary test files
git rm src/test/temp*.java

# Remove entire deprecated module
git rm -r src/deprecated/

# Commit cleanup
git commit -m "Remove deprecated code and temp files"
```

#### Scenario 3: Accidentally Staged Wrong Files

```bash
# Staged too many files
git add .

# Unstage specific files
git restore --staged config/local.yml
git restore --staged test/temp*.java

# Or unstage all and start over
git restore --staged .
```

---

## Moving and Renaming Files

### Git mv Command

Git doesn't explicitly track file renames. A rename is recorded as a deletion and addition, but Git is smart enough to detect it.

#### Basic Rename

```bash
# Rename file
git mv old-name.java new-name.java

# This is equivalent to:
# mv old-name.java new-name.java
# git rm old-name.java
# git add new-name.java

# Verify
git status
# On branch main
# Changes to be committed:
#   renamed:    old-name.java -> new-name.java

# Commit
git commit -m "Rename old-name to new-name"
```

#### Move File to Different Directory

```bash
# Move file to directory
git mv Employee.java src/models/

# Move multiple files
git mv Employee.java Department.java src/models/

# Move and rename
git mv old-name.java src/models/new-name.java
```

#### Rename Directory

```bash
# Rename directory
git mv old-dir/ new-dir/

# Git stages the rename of all files inside
git status  # Shows multiple renames

# Commit
git commit -m "Rename old-dir to new-dir"
```

### Detecting Renames

Git uses similarity index to detect renames.

```bash
# View rename detection in diff
git diff --name-status
# R100 old-name.java new-name.java
# (R100 = 100% similar, it's a rename)

# View with less rename detection
git diff --no-renames
# D old-name.java (deleted)
# A new-name.java (added)

# Configure rename detection threshold
git config diff.renames true
git config diff.renameLimit 1000
```

### Manual Rename (Without git mv)

If you rename files manually, Git can still detect it:

```bash
# Rename manually (outside Git)
mv old-name.java new-name.java

# Git sees it as deletion + addition
git status
# deleted:    old-name.java
# untracked:  new-name.java

# Stage both operations
git add old-name.java new-name.java
# or
git add -A

# Git detects rename
git status
# renamed:    old-name.java -> new-name.java
```

### Real-World Scenarios

#### Scenario 1: Refactor Package Structure

```bash
# Move all model classes to new package
git mv src/Employee.java src/models/
git mv src/Department.java src/models/
git mv src/Company.java src/models/

# Update imports in other files
# ... edit files ...

# Stage import changes
git add .

# Commit
git commit -m "refactor: Move model classes to models package"
```

#### Scenario 2: Rename for Naming Convention

```bash
# Rename to follow naming convention
git mv employeeService.java EmployeeService.java
git mv departmentDAO.java DepartmentDao.java

# Commit
git commit -m "refactor: Apply naming conventions"
```

#### Scenario 3: Reorganize Test Files

```bash
# Move tests to mirror source structure
git mv test/EmployeeTest.java test/models/EmployeeTest.java
git mv test/ServiceTest.java test/services/ServiceTest.java

# Commit
git commit -m "test: Reorganize test directory structure"
```

---

## Viewing Index Status

### Git status

The primary command for viewing index state.

#### Basic Status

```bash
# Standard status
git status

# Output shows:
# - Current branch
# - Branch tracking status
# - Changes to be committed (staged)
# - Changes not staged for commit
# - Untracked files
```

#### Short Status

```bash
# Compact format
git status -s
git status --short

# Output format:
# M  staged-modified.java      (modified and staged)
#  M unstaged-modified.java    (modified, not staged)
# MM both-modified.java        (staged and working dir modified)
# A  new-staged.java           (new file, staged)
# ?? untracked.java            (untracked)
# D  deleted-staged.java       (deleted and staged)
#  D deleted-unstaged.java     (deleted, not staged)
```

#### Branch Status

```bash
# Show branch tracking info
git status -b
git status --branch

# Shows:
# On branch feature/my-work
# Your branch is ahead of 'origin/feature/my-work' by 2 commits.
```

#### Ignored Files

```bash
# Show ignored files too
git status --ignored

# Shows files in .gitignore that are ignored
```

### Git diff

Compare different states of files.

#### View Unstaged Changes

```bash
# Show changes not yet staged
git diff

# Shows difference between working directory and index
# (what you would stage with 'git add')
```

#### View Staged Changes

```bash
# Show changes staged for commit
git diff --staged
git diff --cached  # Same as --staged

# Shows difference between index and HEAD
# (what would be committed with 'git commit')
```

#### View All Changes

```bash
# Show both staged and unstaged changes
git diff HEAD

# Shows difference between working directory and HEAD
# (all changes since last commit)
```

#### View Changes in Specific File

```bash
# Unstaged changes in file
git diff src/Employee.java

# Staged changes in file
git diff --staged src/Employee.java

# All changes in file
git diff HEAD src/Employee.java
```

#### Better Diff Display

```bash
# Word diff (show word-level changes)
git diff --word-diff

# Color words (inline color highlighting)
git diff --color-words

# Show function names
git diff --function-context

# Show statistics
git diff --stat
```

### Git diff with Rename Detection

```bash
# Show renames
git diff --find-renames
git diff -M  # Same

# Show renames with similarity threshold
git diff -M90%  # 90% similarity to detect rename
```

### Viewing Index Contents

#### List Files in Index

```bash
# List all files in index
git ls-files

# List only modified files
git ls-files -m
git ls-files --modified

# List only deleted files
git ls-files -d
git ls-files --deleted

# List only unmerged files (during conflicts)
git ls-files -u
git ls-files --unmerged

# List ignored files
git ls-files --ignored --exclude-standard

# List with staging info
git ls-files -s
git ls-files --stage
```

**Example output:**
```bash
git ls-files -s
# 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	src/Employee.java
# 100644 a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0 0	src/Department.java
# (mode) (hash)                                      (stage) (path)
```

### Real-World Workflows

#### Workflow 1: Review Before Committing

```bash
# See what's changed
git status

# Review unstaged changes
git diff

# Review staged changes
git diff --staged

# If looks good, commit
git commit -m "Your message"
```

#### Workflow 2: Check What's Staged

```bash
# Quick check
git status -s

# Detailed view
git diff --staged --stat

# View each staged file
git diff --staged src/Employee.java
git diff --staged src/Department.java
```

#### Workflow 3: Find Specific Changes

```bash
# Find changes containing specific text
git diff | grep -A 5 -B 5 "function_name"

# Find changes in specific file type
git diff -- '*.java'

# Show only files changed
git diff --name-only
git diff --name-status
```

---

## Advanced Index Operations

### Partial File Staging

#### Stage Specific Lines

You've seen `git add -p`, but you can also use:

```bash
# Stage specific lines interactively
git add --patch file.txt

# Edit hunks manually
# (choose 'e' when prompted)
# Opens editor where you can modify what to stage
```

**Manual edit example:**
```diff
# In the editor:
# Lines starting with '-' will be removed
# Lines starting with '+' will be added
# To NOT stage a change, remove that line

# Original hunk shown:
-old line 1
+new line 1
-old line 2
+new line 2

# To stage only first change, modify to:
-old line 1
+new line 1
```

### Working with Multiple Branches

#### Compare Index Across Branches

```bash
# Compare staged changes with another branch
git diff --staged origin/develop

# See what would change if you committed now
git diff develop...HEAD
```

#### Stage Changes from Another Branch

```bash
# Stage file from another branch
git restore --source=feature/other --staged --worktree file.txt

# Now file is staged and in working directory
```

### Index Manipulation

#### Reset Specific Paths

```bash
# Reset specific file to specific commit
git reset abc1234 -- src/Employee.java

# File content from abc1234 is staged
# Working directory unchanged
```

#### Update Index Without Working Directory

```bash
# Update index from HEAD without touching working dir
git reset --mixed HEAD

# Update specific file in index
git restore --staged file.txt
```

### Git Clean (Removing Untracked Files)

Not directly index-related, but important for working directory management.

#### Preview Untracked Files to Remove

```bash
# Dry run (ALWAYS do this first!)
git clean -n
git clean --dry-run

# Show directories too
git clean -nd
```

#### Remove Untracked Files

```bash
# Remove untracked files
git clean -f
git clean --force

# Remove untracked files and directories
git clean -fd

# Remove ignored files too
git clean -fdx

# Remove only ignored files
git clean -fdX
```

#### Interactive Clean

```bash
# Choose what to remove
git clean -i
git clean --interactive

# Options:
# 1: clean
# 2: filter by pattern
# 3: select by numbers
# 4: ask each
# 5: quit
# 6: help
```

### Stashing and Index

#### Stash with Index

```bash
# Stash both staged and unstaged changes
git stash

# Stash keeps index state!
git stash pop
# Restores both working directory and staged changes
```

#### Stash Working Directory Only

```bash
# Stash unstaged changes, keep staged
git stash --keep-index

# Your staged changes remain
git status  # Shows staged files
```

#### Stash Including Untracked

```bash
# Stash everything including untracked files
git stash --include-untracked
git stash -u
```

### Real-World Advanced Workflows

#### Workflow 1: Split Staged Changes

```bash
# Staged too much
git status
# Changes to be committed:
#   modified:   Employee.java
#   modified:   Department.java

# Want to commit only Employee.java
git restore --staged Department.java

# Commit
git commit -m "feat: Update Employee"

# Later commit Department
git add Department.java
git commit -m "feat: Update Department"
```

#### Workflow 2: Emergency Fix During Work

```bash
# Working on feature, many changes
git status  # Lots of modified files

# Emergency: need to fix bug
git stash --include-untracked

# Now working directory is clean
git status  # Clean

# Fix bug
git add bugfix.java
git commit -m "fix: Critical bug"
git push origin develop

# Resume feature work
git stash pop
```

#### Workflow 3: Review Staged Hunks

```bash
# Staged some changes
git add -p Employee.java

# Review what was staged
git diff --staged Employee.java

# Looks good, commit
git commit -m "feat: Add validation"
```

---

## Practical Workflows

### Workflow 1: Feature Development

```bash
# Start new feature
git checkout -b feature/employee-validation

# Implement feature
# ... edit Employee.java ...

# Stage specific changes
git add -p Employee.java
# Stage validation logic, skip debug code

# Commit feature
git commit -m "feat: Add employee email validation"

# Later clean up debug code
git restore Employee.java  # Or edit and commit separately
```

### Workflow 2: Bug Fix with Multiple Files

```bash
# Bug affects multiple files
# ... edit Employee.java, Department.java, Company.java ...

# Review changes
git status
git diff

# Stage all fixes
git add Employee.java Department.java Company.java

# Review staged changes
git diff --staged

# Commit
git commit -m "fix: Correct null pointer exceptions"
```

### Workflow 3: Incremental Refactoring

```bash
# Large refactoring task
# ... rename methods, extract classes, update tests ...

# Commit in small steps

# Step 1: Rename methods
git add -p Employee.java
# Stage only method renames
git commit -m "refactor: Rename validation methods"

# Step 2: Extract class
git add EmployeeValidator.java
git add -p Employee.java
# Stage changes related to extraction
git commit -m "refactor: Extract EmployeeValidator class"

# Step 3: Update tests
git add test/EmployeeValidatorTest.java
git commit -m "test: Add EmployeeValidator tests"
```

### Workflow 4: Review and Adjust Commits

```bash
# Made changes
git add .
git status

# Wait, that file shouldn't be included
git restore --staged config/local.yml

# This file needs more work
git restore --staged Employee.java

# Commit what's ready
git commit -m "feat: Add department hierarchy"

# Continue working on Employee.java
```

### Workflow 5: Working with Patches

#### Create Patch from Staged Changes

```bash
# Stage changes
git add Employee.java

# Create patch file
git diff --staged > feature.patch

# Unstage changes
git restore --staged Employee.java

# Apply patch later
git apply feature.patch
```

#### Apply Patch to Index

```bash
# Apply patch and stage
git apply --cached feature.patch

# Changes are staged
git status
```

### Workflow 6: Selective Commit from Large Changes

```bash
# Made many changes
git status
# Modified: Employee.java (3 different features)
# Modified: Department.java (2 bug fixes)
# Modified: Company.java (refactoring)

# Commit Feature 1 from Employee.java
git add -p Employee.java
# Select only Feature 1 hunks
git commit -m "feat: Add employee age validation"

# Commit Feature 2 from Employee.java
git add -p Employee.java
# Select only Feature 2 hunks
git commit -m "feat: Add employee phone validation"

# Commit Feature 3 from Employee.java
git add Employee.java
# Everything else in Employee.java
git commit -m "feat: Add employee address validation"

# Commit bug fixes from Department.java
git add Department.java
git commit -m "fix: Department hierarchy issues"

# Commit refactoring
git add Company.java
git commit -m "refactor: Clean up Company class"
```

---

## Best Practices

### 1. Commit Often, Commit Logically

```bash
# Good: Logical, atomic commits
git add Employee.java
git commit -m "feat: Add employee validation"

git add test/EmployeeTest.java
git commit -m "test: Add employee validation tests"

# Bad: One giant commit with everything
git add .
git commit -m "Changes"
```

### 2. Review Before Staging

```bash
# Always review changes first
git diff file.txt

# Then stage
git add file.txt
```

### 3. Use Patch Mode for Complex Changes

```bash
# When file has multiple unrelated changes
git add -p file.txt

# Stage related changes together
# Leave unrelated changes for separate commit
```

### 4. Keep Index Clean

```bash
# Don't leave files staged forever
git status  # Check regularly

# Unstage if not committing soon
git restore --staged .
```

### 5. Use Descriptive Commit Messages

```bash
# Good: Describes what and why
git commit -m "fix: Correct null pointer in employee validation

The validation method didn't handle null email addresses.
Added null check before regex validation."

# Bad: Vague message
git commit -m "fix stuff"
```

### 6. Stage Related Changes Together

```bash
# Good: Related changes in one commit
git add Employee.java EmployeeValidator.java
git commit -m "feat: Add employee email validation"

# Bad: Unrelated changes together
git add .
git commit -m "Various updates"
```

### 7. Use git restore Over git checkout

```bash
# Modern, clear syntax (Git 2.23+)
git restore file.txt              # Discard changes
git restore --staged file.txt     # Unstage

# Old syntax (still works, but confusing)
git checkout -- file.txt          # Discard changes
git reset HEAD file.txt           # Unstage
```

### 8. Preview Destructive Operations

```bash
# Before git clean -fd
git clean -n

# Before discarding changes
git diff file.txt

# Before unstaging
git diff --staged
```

### 9. Use Interactive Mode for Learning

```bash
# Interactive add helps understand staging
git add -i

# Interactive clean helps avoid mistakes
git clean -i
```

### 10. Understand What's in Your Index

```bash
# Before committing
git status           # See what's staged
git diff --staged    # See actual changes
git log --oneline    # Know where you are

# Then commit with confidence
git commit -m "feat: ..."
```

---

## Quick Reference

### Essential Commands Summary

```bash
# GIT RESTORE (Modern way)
git restore <file>                    # Discard changes in working directory
git restore .                         # Discard all changes
git restore --staged <file>           # Unstage file (keep changes)
git restore --staged .                # Unstage all files
git restore --source=HEAD~1 <file>    # Restore from previous commit
git restore -p <file>                 # Interactive restore (patch mode)

# GIT ADD (Staging)
git add <file>                        # Stage specific file
git add .                             # Stage all in current directory
git add -A                            # Stage all changes everywhere
git add -p                            # Stage interactively (patch mode)
git add -i                            # Interactive staging menu
git add -u                            # Stage only modified/deleted (no new files)

# GIT RESET (Unstaging)
git reset                             # Unstage all files
git reset <file>                      # Unstage specific file
git reset --soft HEAD~1               # Undo commit, keep changes staged
git reset HEAD~1                      # Undo commit, keep changes unstaged
git reset --hard HEAD~1               # Undo commit, discard changes ⚠️

# GIT RM (Remove from tracking)
git rm <file>                         # Remove file from Git and filesystem
git rm --cached <file>                # Remove from Git, keep in filesystem
git rm -r <directory>                 # Remove directory
git rm -f <file>                      # Force remove modified file

# GIT MV (Move/rename)
git mv <old> <new>                    # Rename file
git mv <file> <directory>/            # Move file to directory
git mv <old-dir> <new-dir>            # Rename directory

# GIT STATUS (View state)
git status                            # Full status
git status -s                         # Short status
git status -b                         # Include branch info
git status --ignored                  # Show ignored files

# GIT DIFF (View changes)
git diff                              # Unstaged changes
git diff --staged                     # Staged changes
git diff HEAD                         # All changes since last commit
git diff --stat                       # Change statistics
git diff --word-diff                  # Word-level diff

# GIT LS-FILES (List index contents)
git ls-files                          # All tracked files
git ls-files -m                       # Modified files
git ls-files -d                       # Deleted files
git ls-files -s                       # With staging info

# GIT CLEAN (Remove untracked)
git clean -n                          # Preview (DRY RUN - always use first!)
git clean -f                          # Remove untracked files
git clean -fd                         # Remove files and directories
git clean -i                          # Interactive mode
```

### Patch Mode Commands

When in patch mode (`git add -p`, `git restore -p`), these are your options:

```
y - yes, stage/restore this hunk
n - no, don't stage/restore this hunk
q - quit; do not stage/restore this hunk or any remaining
a - stage/restore this hunk and all later hunks in the file
d - do not stage/restore this hunk or any later hunks in the file
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
```

### Common Workflows Cheat Sheet

```bash
# Discard all changes
git restore .

# Unstage all files
git restore --staged .

# Discard and unstage everything
git restore --staged --worktree .

# Stage only bug fixes, not new features
git add -p file.txt

# Restore file from 3 commits ago
git restore --source=HEAD~3 file.txt

# Remove file from Git but keep locally
git rm --cached file.txt

# Rename file
git mv old-name.txt new-name.txt

# Review staged changes
git diff --staged

# Undo last commit but keep changes
git reset HEAD~1

# Clean untracked files (safely)
git clean -n  # Preview
git clean -fd # Actually remove
```

### Decision Tree: "What command do I need?"

```
I want to...

Discard changes in working directory?
  → Single file: git restore <file>
  → All files: git restore .
  → Interactive: git restore -p <file>

Unstage files?
  → Single file: git restore --staged <file>
  → All files: git restore --staged .
  → Interactive: git restore --staged -p <file>

Stage files?
  → Single file: git add <file>
  → All changes: git add .
  → Interactive: git add -p

Stop tracking a file?
  → Remove from Git only: git rm --cached <file>
  → Remove completely: git rm <file>

Rename/move file?
  → git mv <old> <new>

View what's changed?
  → Working directory: git diff
  → Staged: git diff --staged
  → All: git diff HEAD

See what's staged?
  → git status
  → git diff --staged

Undo last commit?
  → Keep staged: git reset --soft HEAD~1
  → Keep unstaged: git reset HEAD~1
  → Discard all: git reset --hard HEAD~1

Remove untracked files?
  → Preview: git clean -n
  → Remove: git clean -fd
```

---

## Summary

### Key Takeaways

1. **Use `git restore` for modern index management** - It's clearer than `git checkout`

2. **The index is your staging ground** - Use it to craft meaningful commits

3. **Review before committing** - Use `git status` and `git diff --staged`

4. **Patch mode is powerful** - Use `git add -p` for selective staging

5. **Commit logically** - Stage related changes together

6. **Preview destructive operations** - Use `-n` flags and dry runs

7. **Index management is safe** - Most operations can be undone

8. **Understand three states** - Working directory, index, repository

### Command Evolution

**Old → New:**
- `git checkout -- <file>` → `git restore <file>`
- `git reset HEAD <file>` → `git restore --staged <file>`
- `git checkout <branch>` → `git switch <branch>`

### Most Important Commands

```bash
# Restore working directory
git restore <file>

# Unstage
git restore --staged <file>

# Stage interactively
git add -p

# View staged changes
git diff --staged

# Clean untracked (safely)
git clean -n && git clean -fd
```

### Remember

- **Working Directory**: Where you make changes
- **Index/Staging Area**: Where you prepare commits
- **Repository**: Where commits are stored
- **git restore**: Manages working directory and index
- **git add**: Adds to index
- **git commit**: Saves index to repository

The index gives you control over **what** goes into each commit, enabling you to create clean, logical commit history. Master these commands, and you'll have precise control over your Git workflow! 🚀

---

**Happy Git-ing!**
